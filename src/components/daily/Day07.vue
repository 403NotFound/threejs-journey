<template>
  <div class="threejs-container">
    <header class="top-bar">
      <h1>Day 07 - 交互与射线检测 (Raycaster)</h1>
      <span class="date">📅 2025-12-12</span>
    </header>
    <div ref="canvasContainer" class="canvas-wrapper"></div>
    <div v-if="loading" class="loading-overlay">
      <div class="loading-text">加载资源中...</div>
    </div>
    <!-- 提示信息 -->
    <div class="instruction-tip">
      👉 点击地面控制移动 | 点击僵尸播放攻击
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, reactive } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import GUI from 'lil-gui'

const canvasContainer = ref(null)
const loading = ref(true)
let gui = null

// 核心变量
let scene, camera, renderer, controls
let mixer = null
let clock = new THREE.Clock()
let zombieModel = null // 主角引用
let actions = {} // 存储所有动画动作
let activeAction = null // 当前播放的动作

// 射线检测相关
const raycaster = new THREE.Raycaster()
const mouse = new THREE.Vector2()
let groundPlane = null // 专门用于检测点击的地面
const targetPosition = new THREE.Vector3() // 移动目标点
let isMoving = false // 是否正在移动
const moveSpeed = 2 // 移动速度

// 点击指示器 (光标)
let cursorIndicator = null

const initThreeJs = () => {
  // 1. 场景
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#111111')
  // 加上雾效，增强氛围，同时遮挡远处边界
  scene.fog = new THREE.Fog('#111111', 5, 20)

  // 2. 相机
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(0, 4, 6)

  // 3. 渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true }) // antialias: 抗锯齿
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true // 开启阴影
  renderer.shadowMap.type = THREE.PCFSoftShadowMap // 设置阴影类型
  renderer.physicallyCorrectLights = true // 物理光照
  renderer.toneMapping = THREE.ACESFilmicToneMapping // 色彩映射
  renderer.toneMappingExposure = 1 // 明暗度
  canvasContainer.value.appendChild(renderer.domElement)

  // 4. 环境与光照
  new RGBELoader()
    .setPath('./textures/hdr/') // 改为相对路径
    .load('afrikaans_church_exterior_4k.hdr', (texture) => {
      texture.mapping = THREE.EquirectangularReflectionMapping
      scene.environment = texture
    })

  const dirLight = new THREE.DirectionalLight(0xffffff, 1)
  dirLight.position.set(5, 10, 5) // 光源位置
  dirLight.castShadow = true // 开启阴影
  dirLight.shadow.mapSize.set(2048, 2048) // 设置阴影分辨率
  scene.add(dirLight)

  // 5. 地面 (用于接收射线检测的隐形平面)
  const planeGeometry = new THREE.PlaneGeometry(100, 100)
  // 使用透明材质，只用于物理检测
  const planeMaterial = new THREE.MeshBasicMaterial({ visible: false })
  groundPlane = new THREE.Mesh(planeGeometry, planeMaterial)
  groundPlane.rotation.x = -Math.PI / 2
  scene.add(groundPlane)

  // 可见地面 (网格与阴影)
  const visualPlaneMat = new THREE.ShadowMaterial({ opacity: 0.3 })
  const visualPlane = new THREE.Mesh(planeGeometry, visualPlaneMat)
  visualPlane.rotation.x = -Math.PI / 2
  visualPlane.receiveShadow = true
  scene.add(visualPlane)

  const gridHelper = new THREE.GridHelper(50, 50, 0x444444, 0x222222)
  scene.add(gridHelper)

  // 点击光标
  const cursorGeometry = new THREE.RingGeometry(0.2, 0.25, 32)
  const cursorMaterial = new THREE.MeshBasicMaterial({
    color: 0xffff00,
    transparent: true,
    opacity: 0.8,
    side: THREE.DoubleSide
  })
  cursorIndicator = new THREE.Mesh(cursorGeometry, cursorMaterial)
  cursorIndicator.rotation.x = -Math.PI / 2
  cursorIndicator.visible = false
  scene.add(cursorIndicator)

  // 6. 加载资源
  const gltfLoader = new GLTFLoader()

  const loadModel = (path, pos, scale, rot, isMainChar = false) => {
    gltfLoader.load(path, (gltf) => {
      const model = gltf.scene;
      // 开启阴影
      model.traverse(child => {
        if (child.isMesh) {
          child.castShadow = true
          child.receiveShadow = true
        }
      })
      model.position.set(pos.x, pos.y, pos.z)
      model.scale.set(scale, scale, scale)
      model.rotation.y = rot
      scene.add(model)

      if (isMainChar) {
        zombieModel = model
        loading.value = false

        // 动画初始化
        mixer = new THREE.AnimationMixer(model)
        const gltfAnimations = gltf.animations

        // 存储动作
        gltfAnimations.forEach(clip => {
          actions[clip.name] = mixer.clipAction(clip)
        })

        // 默认播放 Idle
        playAction('idle')
      }
    })
  }

  // 加载场景 (改为相对路径)
  loadModel('./models/GLB/character-zombie.glb', { x: 0, y: 0, z: 0 }, 1, 0, true)
  loadModel('./models/GLB/crypt.glb', { x: -3, y: 0, z: -3 }, 1.5, Math.PI / 4)
  loadModel('./models/GLB/pine.glb', { x: 3.5, y: 0, z: -2 }, 1.2, 0)
  loadModel('./models/GLB/road.glb', { x: 0, y: 0.01, z: 0 }, 1, 0)
  loadModel('./models/GLB/gravestone-cross.glb', { x: 1.5, y: 0, z: -1.5 }, 0.8, -0.2)

  // 7. 控制器
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.maxPolarAngle = Math.PI / 2 - 0.1 // 禁止钻入地下 maxPolarAngle: 最大仰角
  controls.minDistance = 2 // 最小距离
  controls.maxDistance = 15 // 最大距离

  // 8. 事件监听
  window.addEventListener('resize', onResize)
  window.addEventListener('click', onClick)
  window.addEventListener('keydown', onKeyDown)
  window.addEventListener('keyup', onKeyUp)

  // 9. 动画循环
  animate()
}

// 切换动画函数
const playAction = (name) => {
  const newAction = actions[name]
  if (!newAction || activeAction === newAction) return

  if (activeAction) {
    activeAction.fadeOut(0.2)
  }

  newAction.reset().fadeIn(0.2).play()
  activeAction = newAction
}

// 鼠标点击事件 (核心逻辑)
const onClick = (event) => {
  if (!zombieModel) return

  // 1. 计算鼠标的归一化设备坐标 (NDC)
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1
  console.log(event.clientX, event.clientY, mouse.x, mouse.y);

  // 2. 发射射线
  raycaster.setFromCamera(mouse, camera)

  // 3. 检测与模型的交互 (点击僵尸)
  const intersectsZombie = raycaster.intersectObject(zombieModel, true)
  if (intersectsZombie.length > 0) {
    // 点中僵尸 -> 播放攻击动画
    playAction('attack-kick-left')
    // 动画播放完后恢复 Idle (简单处理)
    setTimeout(() => {
      if (!isMoving) playAction('idle')
    }, 1000)
    return
  }

  // 4. 检测与地面的交互 (移动)
  // intersectObject: 检测射线与模型的交点
  const intersectsGround = raycaster.intersectObject(groundPlane)
  if (intersectsGround.length > 0) {
    console.log(intersectsGround, '89');

    const point = intersectsGround[0].point // point: 交点

    // 设置目标点
    targetPosition.copy(point)
    isMoving = true

    // 显示点击光标
    cursorIndicator.position.copy(point)
    cursorIndicator.position.y = 0.01// 
    cursorIndicator.visible = true

    // 播放走路动画
    playAction('walk')

    // 让僵尸朝向目标
    zombieModel.lookAt(targetPosition.x, zombieModel.position.y, targetPosition.z)
  }
}

// 按键状态 (Reactive)
const keysPressed = reactive({
  w: false,
  a: false,
  s: false,
  d: false
})

// 统一处理按键逻辑
const handleKey = (event, isDown) => {
  const code = event.code

  switch (code) {
    case 'KeyW': keysPressed.w = isDown; break;
    case 'KeyA': keysPressed.a = isDown; break;
    case 'KeyS': keysPressed.s = isDown; break;
    case 'KeyD': keysPressed.d = isDown; break;
  }
}

const onKeyDown = (event) => handleKey(event, true)
const onKeyUp = (event) => handleKey(event, false)

// 更新逻辑 (核心帧循环)
const update = (delta) => {
  if (mixer) mixer.update(delta)

  if (!zombieModel) return

  // 1. 获取键盘输入方向
  let moveX = 0
  let moveZ = 0
  // 恢复标准方向
  if (keysPressed.w) moveZ -= 1
  if (keysPressed.s) moveZ += 1
  if (keysPressed.a) moveX -= 1
  if (keysPressed.d) moveX += 1

  const isKeyPressed = moveX !== 0 || moveZ !== 0

  // 防卡死检查：如果有键按下，但合力为0 (比如W+S同时按)，判定为卡死风险
  // 或者用户想用鼠标移动，我们在这里也优先重置一次以确保干净
  if (isMoving && isKeyPressed) {
    isMoving = false // 键盘介入，取消鼠标移动
  }

  // === 状态 A: 键盘控制中 ===
  if (isKeyPressed) {
    // 强制打断鼠标点击的移动
    isMoving = false
    cursorIndicator.visible = false

    // 归一化速度向量
    const moveVector = new THREE.Vector3(moveX, 0, moveZ).normalize()

    // 1. 移动位置
    // 根据键盘输入的方向向量 (moveVector) 更新僵尸模型的位置。
    // `moveSpeed * delta` 用于确保移动速度与帧率无关，实现平滑的动画效果。
    // `moveSpeed` 是每秒移动的距离，`delta` 是自上一帧以来经过的时间，它们的乘积代表了当前帧的实际移动量。
    zombieModel.position.addScaledVector(moveVector, moveSpeed * delta)

    // 2. 旋转朝向
    if (moveVector.lengthSq() > 0.001) {
      zombieModel.rotation.y = Math.atan2(moveVector.x, moveVector.z)
    }
    playAction('walk')
  }
  // === 状态 B: 鼠标点击移动中 ===
  else if (isMoving) {
    const startPos = zombieModel.position
    const endPos = targetPosition

    const dx = endPos.x - startPos.x
    const dz = endPos.z - startPos.z
    const distanceSq = dx * dx + dz * dz

    if (distanceSq > 0.01) {
      const moveDir = new THREE.Vector3(dx, 0, dz).normalize()
      zombieModel.position.addScaledVector(moveDir, moveSpeed * delta)

      // 使用 lookAt 替代手动计算角度，确保稳健
      // 注意：lookAt 会让 Z-轴(面部) 对准目标
      zombieModel.lookAt(targetPosition.x, zombieModel.position.y, targetPosition.z)

      playAction('walk')
    } else {
      isMoving = false
      cursorIndicator.visible = false
      playAction('idle')
    }
  }
  // === 状态 C: 站立待机 ===
  else {
    const isAttacking = activeAction && activeAction.getClip && activeAction.getClip().name === 'attack-kick-left'
    if (!isAttacking) {
      playAction('idle')
    }
  }
}

const animate = () => {
  requestAnimationFrame(animate)

  try {
    const delta = clock.getDelta()
    update(delta)
    controls.update()
    renderer.render(scene, camera)
  } catch (e) {
    console.error("Animation Loop Crash:", e)
  }
}

const onResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

onMounted(() => {
  initThreeJs()
  // 改用 document 监听，更稳健
  document.addEventListener('keydown', onKeyDown)
  document.addEventListener('keyup', onKeyUp)
})

onUnmounted(() => {
  window.removeEventListener('resize', onResize)
  window.removeEventListener('click', onClick)

  document.removeEventListener('keydown', onKeyDown)
  document.removeEventListener('keyup', onKeyUp)

  if (mixer) mixer.stopAllAction()
  renderer && renderer.dispose()
})
</script>

<style scoped>
.threejs-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  overflow: hidden;
  background: #000;
  user-select: none;
  /* 防止点击时选中文本 */
}

.canvas-wrapper {
  width: 100%;
  height: 100%;
}

.top-bar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 60px;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 2rem;
  z-index: 100;
  color: white;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  pointer-events: none;
  /* 让鼠标事件透过标题栏 */
}

.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 200;
  color: white;
  font-size: 1.5rem;
}

.instruction-tip {
  /* ... existing styles ... */
  position: absolute;
  bottom: 30px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0, 0, 0, 0.6);
  padding: 10px 20px;
  border-radius: 20px;
  color: #fbbf24;
  font-weight: bold;
  pointer-events: none;
  z-index: 10;
  border: 1px solid rgba(255, 255, 0, 0.3);
}
</style>
