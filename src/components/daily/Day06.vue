<template>
  <div class="threejs-container">
    <header class="top-bar">
      <h1>Day 06 - 动画系统 (Animation)</h1>
      <span class="date">📅 2025-12-12</span>
    </header>
    <div ref="canvasContainer" class="canvas-wrapper"></div>
    <div v-if="loading" class="loading-overlay">
      <div class="loading-text">加载资源中...</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import GUI from 'lil-gui'

const canvasContainer = ref(null)
const loading = ref(true)
let gui = null
let mixer = null // 动画混合器
let clock = new THREE.Clock() // 时钟，用于更新动画

const initThreeJs = () => {
  // 1. 场景
  const scene = new THREE.Scene()
  scene.background = new THREE.Color('#cccccc')

  // 2. 相机
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(2, 2, 5)

  // 3. 渲染器
  const renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true
  // 三种阴影类型：
  // THREE.BasicShadowMap 基础阴影
  // THREE.PCFShadowMap 平滑阴影
  // THREE.PCFSoftShadowMap 软阴影
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  renderer.physicallyCorrectLights = true
  renderer.toneMapping = THREE.ACESFilmicToneMapping // 色调映射
  renderer.toneMappingExposure = 1 // 色调曝光
  canvasContainer.value.appendChild(renderer.domElement)

  // 4. 环境光照 (HDR)
  new RGBELoader()
    .setPath('/textures/hdr/')
    .load('afrikaans_church_exterior_4k.hdr', (texture) => {
      texture.mapping = THREE.EquirectangularReflectionMapping
      scene.environment = texture
    })

  const dirLight = new THREE.DirectionalLight(0xffffff, 1)
  dirLight.position.set(5, 5, 5)
  dirLight.castShadow = true // 投影阴影
  dirLight.shadow.mapSize.set(2048, 2048)
  scene.add(dirLight)

  // 5. 地面与阴影
  const planeGeometry = new THREE.PlaneGeometry(100, 100)
  const planeMaterial = new THREE.ShadowMaterial({ opacity: 0.3 })
  const plane = new THREE.Mesh(planeGeometry, planeMaterial)
  plane.rotation.x = -Math.PI / 2 // 旋转90度
  plane.receiveShadow = true
  scene.add(plane)

  // 6. 加载模型 & 动画
  const gltfLoader = new GLTFLoader()

  // 辅助加载函数 (path: 模型路径, pos: 位置, scale: 缩放, rot: 旋转, isMainChar: 是否为主要角色)
  const loadModel = (path, pos, scale, rot, isMainChar = false) => {
    gltfLoader.load(path, (gltf) => {
      const model = gltf.scene
      console.log(model);

      // 遍历模型的每个子对象
      model.traverse(child => {
        if (child.isMesh) { // 如果是网格
          child.castShadow = true // 投影阴影
          child.receiveShadow = true // 接收阴影
        }
      })
      model.position.set(pos.x, pos.y, pos.z) // 设置模型位置
      model.scale.set(scale, scale, scale) // 设置模型缩放
      model.rotation.y = rot // 设置模型旋转
      scene.add(model)

      if (isMainChar) {
        loading.value = false
        // === 动画系统初始化 ===
        mixer = new THREE.AnimationMixer(model) // 创建动画混合器
        const animations = gltf.animations // 获取所有动画片段

        console.log('Available Animations:', animations.map(a => a.name))

        initAnimationGUI(mixer, animations)

        // 默认播放第一个动画 (通常是 Idle)
        if (animations.length > 0) {
          const action = mixer.clipAction(animations[0])
          action.play()
        }
      }
    })
  }

  // 搭建场景
  loadModel('/models/GLB/character-zombie.glb', { x: 0, y: 0, z: 0 }, 1, 0, true)
  loadModel('/models/GLB/crypt.glb', { x: -3, y: 0, z: -3 }, 1.5, Math.PI / 4) // Math.PI / 4 = 45度 ?
  loadModel('/models/GLB/pine.glb', { x: 3.5, y: 0, z: -2 }, 1.2, 0)
  loadModel('/models/GLB/road.glb', { x: 0, y: 0.01, z: 0 }, 1, 0)
  loadModel('/models/GLB/fence.glb', { x: 0, y: 0, z: -4 }, 1, 0)
  loadModel('/models/GLB/gravestone-cross.glb', { x: 1.5, y: 0, z: -1.5 }, 0.8, -0.2)


  // 7. 控制器
  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.target.set(0, 1, 0)

  // 8. 动画循环
  const animate = () => {
    requestAnimationFrame(animate)

    // === 更新混合器 ===
    const delta = clock.getDelta() // 获取上一帧到现在的间隔时间
    if (mixer) {
      mixer.update(delta)
    }

    controls.update()
    renderer.render(scene, camera)
  }
  animate()

  // Resize handler
  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  })
}

// 动画 GUI
const initAnimationGUI = (mixer, animations) => {
  gui = new GUI()
  gui.title('Day 06 - 动画控制')

  const animFolder = gui.addFolder('动作列表') // 添加动作列表文件夹
  console.log(animFolder);

  let currentAction = null // 当前正在播放的动作

  const playAction = (clip) => {
    const newAction = mixer.clipAction(clip)

    // 如果有旧动作，进行平滑过渡 (Cross Fade)
    if (currentAction && currentAction !== newAction) {
      currentAction.fadeOut(0.5) // 旧动作 0.5秒淡出
      newAction.reset().fadeIn(0.5).play() // 新动作 0.5秒淡入
    } else {
      newAction.reset().play() // 重置并播放新动作
    }

    currentAction = newAction
  }

  // 为每个动画片段创建一个按钮
  animations.forEach((clip) => {
    const params = {
      [clip.name]: () => playAction(clip)
    }
    animFolder.add(params, clip.name)
  })

  // 默认设置第一个为当前动作
  if (animations.length > 0) {
    currentAction = mixer.clipAction(animations[0])
  }
}

onMounted(() => {
  initThreeJs()
})

onUnmounted(() => {
  if (gui) gui.destroy()
  if (mixer) mixer.stopAllAction()
})
</script>

<style scoped>
.threejs-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  overflow: hidden;
  background: #000;
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
</style>
