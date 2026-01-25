<template>
  <div class="practice-container">
    <div class="header">
      <h2>Day 07+ 专项训练：数学与基础 API</h2>
      <div class="controls">
        <button @click="currentMode = 'A'" :class="{ active: currentMode === 'A' }">实验 A: 向量移动</button>
        <button @click="currentMode = 'B'" :class="{ active: currentMode === 'B' }">实验 B: LookAt 注视</button>
        <button @click="currentMode = 'C'" :class="{ active: currentMode === 'C' }">实验 C: 射线点击</button>
      </div>
    </div>
    <div ref="canvasContainer" class="canvas-wrapper"></div>

    <!-- 说明面板 -->
    <div class="info-panel">
      <div v-if="currentMode === 'A'">
        <h3>实验 A：向量 (Vector3)</h3>
        <p>🔴 红色小球代表 `position` (位置)</p>
        <p>🟢 绿色箭头代表 `direction` (方向向量)</p>
        <p>按 <b>W A S D</b> 移动。观察速度向量如何合成。</p>
      </div>
      <div v-if="currentMode === 'B'">
        <h3>实验 B：注视 (LookAt)</h3>
        <p>🟦 蓝色长条会死死盯着你的鼠标。</p>
        <p>鼠标位置：{{ mouseLog }}</p>
        <p>理解 LookAt 如何自动计算四元数旋转。</p>
      </div>
      <div v-if="currentMode === 'C'">
        <h3>实验 C：射线 (Raycaster)</h3>
        <p>点击任何方块使其变色。</p>
        <p>屏幕坐标 -> NDC 坐标的转换公式是关键。</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const canvasContainer = ref(null)
const currentMode = ref('A')
const mouseLog = ref('')

let scene, camera, renderer, controls, rafId // scene: 场景 camera: 相机 renderer: 渲染器 controls: 控制器 rafId: 动画ID
let objects = [] // 存储当前场景的对象，方便清理

// 实验 A 变量
let ball, arrowHelper
const moveState = { w: false, a: false, s: false, d: false }

// 实验 B 变量
let pointerMesh, mousePlane
const raycaster = new THREE.Raycaster()
const mouse = new THREE.Vector2()

// 实验 C 变量
let cubes = []

// 初始化基础场景 (灯光、相机、渲染器)
const initBase = () => {
  // 清理旧场景
  if (renderer) {
    renderer.dispose() // 释放渲染器
    canvasContainer.value.innerHTML = '' // 清空画布
  }
  if (rafId) cancelAnimationFrame(rafId) // 清理动画
  objects = [] // 清空对象

  scene = new THREE.Scene()
  scene.background = new THREE.Color('#222')

  // 网格辅助
  scene.add(new THREE.GridHelper(20, 20))
  scene.add(new THREE.AxesHelper(2)) // 红X 绿Y 蓝Z

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100)
  camera.position.set(0, 5, 5)

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  canvasContainer.value.appendChild(renderer.domElement)

  controls = new OrbitControls(camera, renderer.domElement)

  const light = new THREE.DirectionalLight(0xffffff, 1)
  light.position.set(5, 5, 5)
  scene.add(light)
  scene.add(new THREE.AmbientLight(0xffffff, 0.5))
}

// === 实验 A: 向量移动 ===
const startModeA = () => {
  initBase()

  // 1. 创建主角：球
  const geometry = new THREE.SphereGeometry(0.5)
  const material = new THREE.MeshPhongMaterial({ color: 0xff0000 })
  ball = new THREE.Mesh(geometry, material)
  scene.add(ball)
  objects.push(ball)

  // 2. 创建一个可视化箭头，用来展示当前的“速度向量”
  // 参数: dir, origin, length, color
  arrowHelper = new THREE.ArrowHelper(new THREE.Vector3(1, 0, 0), ball.position, 2, 0x00ff00)
  scene.add(arrowHelper)
  objects.push(arrowHelper)

  // 3. 动画循环
  const animateA = () => {
    rafId = requestAnimationFrame(animateA)

    // --> 核心知识点：向量合成 <--
    // 不要写 x += 0.1
    // 要创建一个“意图”向量
    const direction = new THREE.Vector3(0, 0, 0)

    if (moveState.w) direction.z -= 1
    if (moveState.s) direction.z += 1
    if (moveState.a) direction.x -= 1
    if (moveState.d) direction.x += 1

    // 如果有输入，才处理
    if (direction.lengthSq() > 0) {
      // 归一化：确保斜着走速度不会变快 (1.414 -> 1)
      direction.normalize()

      // 更新可视化箭头，哪怕你看不到数学，也能看到这个绿箭头指向哪里
      arrowHelper.setDirection(direction)
      arrowHelper.position.copy(ball.position) // 箭头跟着球走

      // 移动：当前位置 = 当前位置 + (方向 * 速度)
      const speed = 0.1
      ball.position.addScaledVector(direction, speed)
    }

    renderer.render(scene, camera)
  }
  animateA()
}

// === 实验 B: LookAt ===
const startModeB = () => {
  initBase()

  // 1. 创建一个长条 (针)
  const geometry = new THREE.BoxGeometry(0.2, 0.2, 2) // Z轴方向长的方块
  geometry.translate(0, 0, 1) // 技巧：把几何体中心往后挪，让它的“屁股”在原点，像炮台一样转
  const material = new THREE.MeshPhongMaterial({ color: 0x0088ff })
  pointerMesh = new THREE.Mesh(geometry, material)
  scene.add(pointerMesh)

  // 2. 创建一个隐形地板用来接收鼠标
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(20, 20),
    new THREE.MeshBasicMaterial({ visible: false })
  )
  plane.rotation.x = -Math.PI / 2
  scene.add(plane)
  mousePlane = plane

  const animateB = () => {
    rafId = requestAnimationFrame(animateB)
    renderer.render(scene, camera)
  }
  animateB()

  // 监听鼠标移动
  window.addEventListener('mousemove', onMouseMoveB)
}

const onMouseMoveB = (event) => {
  if (currentMode.value !== 'B') return

  // 转换鼠标坐标
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1

  raycaster.setFromCamera(mouse, camera)
  const intersects = raycaster.intersectObject(mousePlane)

  if (intersects.length > 0) {
    const targetPoint = intersects[0].point
    mouseLog.value = `x:${targetPoint.x.toFixed(2)}, z:${targetPoint.z.toFixed(2)}`
    // --> 核心知识点：LookAt <--
    // "请看着那个点" - Three.js 会自动计算旋转四元数
    pointerMesh.lookAt(targetPoint)
  }
}

// === 实验 C: 射线点击 ===
const startModeC = () => {
  initBase()

  // 放 9 个方块
  cubes = []
  const geometry = new THREE.BoxGeometry(1, 1, 1)

  for (let x = -2; x <= 2; x += 2) {
    for (let z = -2; z <= 2; z += 2) {
      const material = new THREE.MeshPhongMaterial({ color: 0xaaaaaa })
      const cube = new THREE.Mesh(geometry, material)
      cube.position.set(x, 0, z)
      scene.add(cube)
      cubes.push(cube)
    }
  }

  const animateC = () => {
    rafId = requestAnimationFrame(animateC)
    renderer.render(scene, camera)
  }
  animateC()

  window.addEventListener('click', onClickC)
}

const onClickC = (event) => {
  if (currentMode.value !== 'C') return

  // --> 核心知识点：手动计算 NDC <--
  const x = event.clientX
  const y = event.clientY
  const w = window.innerWidth
  const h = window.innerHeight

  // 公式：(像素 / 总宽) * 2 - 1
  mouse.x = (x / w) * 2 - 1
  mouse.y = -(y / h) * 2 + 1

  console.log(`屏幕:(${x}, ${y}) -> NDC:(${mouse.x.toFixed(2)}, ${mouse.y.toFixed(2)})`)

  raycaster.setFromCamera(mouse, camera)

  // 检测所有方块
  const intersects = raycaster.intersectObjects(cubes)
  if (intersects.length > 0) {
    // 变个随机色
    intersects[0].object.material.color.setHex(Math.random() * 0xffffff)
  }
}

// 键盘事件监听 (通用)
const onKeyChange = (e, isDown) => {
  if (currentMode.value !== 'A') return
  const key = e.key.toLowerCase() // w,a,s,d
  if (moveState.hasOwnProperty(key)) moveState[key] = isDown
}

onMounted(() => {
  window.addEventListener('keydown', (e) => onKeyChange(e, true))
  window.addEventListener('keyup', (e) => onKeyChange(e, false))

  // 默认启动模式 A
  startModeA()
})

onUnmounted(() => {
  window.removeEventListener('mousemove', onMouseMoveB)
  window.removeEventListener('click', onClickC) // 清理
  if (rafId) cancelAnimationFrame(rafId)
  if (renderer) renderer.dispose()
})

// 监听模式切换
watch(currentMode, (newMode) => {
  if (newMode === 'A') startModeA()
  if (newMode === 'B') startModeB()
  if (newMode === 'C') startModeC()
})

</script>

<style scoped>
.practice-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  background: #222;
}

.header {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  padding: 10px 20px;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  display: flex;
  justify-content: space-between;
  align-items: center;
  z-index: 10;
}

.controls button {
  margin-left: 10px;
  padding: 5px 15px;
  background: #444;
  border: 1px solid #666;
  color: #ccc;
  cursor: pointer;
}

.controls button.active {
  background: #0088ff;
  color: white;
  border-color: #0066cc;
}

.info-panel {
  position: absolute;
  top: 70px;
  left: 20px;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 15px;
  border-radius: 8px;
  max-width: 300px;
  pointer-events: none;
}

h3 {
  margin-top: 0;
  color: #fbbf24;
}
</style>
