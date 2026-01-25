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
        <p>当前按键状态：</p>
        <p>W: {{ moveState.w ? '按下' : '抬起' }}</p>
        <p>A: {{ moveState.a ? '按下' : '抬起' }}</p>
        <p>S: {{ moveState.s ? '按下' : '抬起' }}</p>
        <p>D: {{ moveState.d ? '按下' : '抬起' }}</p>
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
import { ref, onMounted, onUnmounted, watch, reactive } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

let canvasContainer = ref(null)
let currentMode = ref('C')

let scene, camera, renderer, controls, rafId
let objects = [] // 存储当前场景的对象，方便清理

// 实验A变量
let ball, arrowHelper, plane
const moveState = reactive({
  w: false,
  s: false,
  a: false,
  d: false,
})
const raycaster = new THREE.Raycaster()
const mouse = new THREE.Vector2()

// 实验B变量
let pointerMesh, mousePlane  //  指针, 鼠标平面
let mouseLog = ref('')


// 实验C变量
let cubes = []

const initBase = () => {
  if (renderer) {
    renderer.dispose()
    canvasContainer.value.innerHTML = ''
  }
  if (rafId) {
    cancelAnimationFrame(rafId)
  }
  objects = []
  // 创建场景
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#ccc')

  // 网格辅助
  const gridHelper = new THREE.GridHelper(20, 20)
  scene.add(gridHelper)

  // 红色X轴 绿色Y轴 蓝色Z轴
  const axesHelper = new THREE.AxesHelper(2)
  scene.add(axesHelper)

  // 创建相机
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100)
  camera.position.set(0, 5, 5)

  // 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  canvasContainer.value.appendChild(renderer.domElement)

  // 创建控制器
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true


  // 创建灯光
  const light = new THREE.DirectionalLight(0xffffff, 1)
  light.position.set(5, 5, 5)
  scene.add(light)

  // 创建环境光
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5)
  scene.add(ambientLight)

  // 创建地板
  const planeGeometry = new THREE.PlaneGeometry(20, 20)
  const planeMaterial = new THREE.MeshBasicMaterial({ color: 0x999999 })
  plane = new THREE.Mesh(planeGeometry, planeMaterial)
  plane.rotation.x = -Math.PI / 2
  scene.add(plane)
}



const startModeA = () => {
  initBase()
  // 创建红色球体
  const ballGermetry = new THREE.SphereGeometry(0.5)
  const ballMaterial = new THREE.MeshPhongMaterial({ color: 0xff0000 })
  ball = new THREE.Mesh(ballGermetry, ballMaterial)

  scene.add(ball)
  objects.push(ball)

  // 创建一个可视化箭头，用来展示当前的“速度向量”
  // 参数：dir:方向向量, origin:起点, length:长度, color:颜色 
  arrowHelper = new THREE.ArrowHelper(new THREE.Vector3(1, 0, 0), ball.position, 2, 0x00ff00)
  scene.add(arrowHelper)
  objects.push(arrowHelper)
  // 动画
  const animateA = () => {
    rafId = requestAnimationFrame(animateA)
    // 向量合成
    const direction = new THREE.Vector3(0, 0, 0)
    if (moveState.w) direction.z -= 1
    if (moveState.s) direction.z += 1
    if (moveState.a) direction.x -= 1
    if (moveState.d) direction.x += 1

    //有输入才处理
    if (direction.lengthSq() > 0) {
      //归一化, 确保斜着走不会更快
      direction.normalize()
      // 更新可视化箭头
      arrowHelper.setDirection(direction)
      // 更新箭头起点, 与球体位置保持一致
      arrowHelper.position.copy(ball.position)

      // 小球移动
      const speed = 0.1
      ball.position.addScaledVector(direction, speed)
    }

    // 鼠标点击移动逻辑
    if (isMoving) {
      // 1. 计算方向向量：目标点 - 当前点
      const direction = new THREE.Vector3().subVectors(targetPos, ball.position)
      console.log(direction);

      // 2. 检查距离，如果很近了就停止
      if (direction.length() < 0.1) {
        isMoving = false
        ball.position.copy(targetPos) // 强制吸附到终点，消除微小误差
      } else {
        // 3. 归一化方向，确保匀速移动
        direction.normalize()

        // 4. 移动一步
        const speed = 0.1
        ball.position.addScaledVector(direction, speed)

        // 5. 更新箭头，让它指向我们要去的方向
        arrowHelper.setDirection(direction)
        arrowHelper.position.copy(ball.position)
      }
    }
    renderer.render(scene, camera)
  }
  animateA()
}

const startModeB = () => {
  initBase()

  // 1. 创建一个长方体长条
  const geometry = new THREE.BoxGeometry(0.2, 0.2, 2)

  geometry.translate(0, 0, 1)

  const material = new THREE.MeshPhongMaterial({ color: 0x0088ff })
  pointerMesh = new THREE.Mesh(geometry, material)
  scene.add(pointerMesh)

  // 2. 创建一个隐形地板，用于接收鼠标
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(20, 20),
    new THREE.MeshBasicMaterial({ color: 0x000000, side: THREE.DoubleSide })
  )
  plane.rotation.x = -Math.PI / 2
  mousePlane = plane // 保存鼠标平面
  scene.add(plane)

  const animateB = () => {
    rafId = requestAnimationFrame(animateB)
    renderer.render(scene, camera)
  }
  animateB()
  window.addEventListener('mousemove', onMouseMoveB)
}

const onMouseMoveB = (e) => {
  if (currentMode.value !== 'B') return // 只在模式B下才处理
  mouse.x = (e.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(e.clientY / window.innerHeight) * 2 + 1

  raycaster.setFromCamera(mouse, camera)

  const intersects = raycaster.intersectObject(mousePlane) // 与鼠标平面相交
  if (intersects.length > 0) {
    console.log(intersects[0].point);
    const targetPoint = intersects[0].point
    mouseLog.value = `x:${targetPoint.x.toFixed(2)}, z:${targetPoint.z.toFixed(2)}`
    pointerMesh.lookAt(targetPoint)
  }
}
const onKeyChange = (e, isPressed) => {
  if (currentMode.value !== 'A') return // 只在模式A下才处理
  const key = e.key.toLowerCase()
  if (moveState.hasOwnProperty(key)) {
    moveState[key] = isPressed
  }

}


const startModeC = () => {
  initBase()

  // 创建9个立方体
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

const onClickC = (e) => {
  if (currentMode.value !== 'C') return // 只在模式C下才处理

  mouse.x = (e.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(e.clientY / window.innerHeight) * 2 + 1


  console.log(`屏幕:(${e.clientX}, ${e.clientY}) -> NDC:(${mouse.x.toFixed(2)}, ${mouse.y.toFixed(2)})`)
  raycaster.setFromCamera(mouse, camera)

  const intersects = raycaster.intersectObjects(cubes)
  console.log(intersects.length);
  if (intersects.length > 0) {
    intersects[0].object.material.color.setHex(Math.random() * 0xffffff)
    console.log(intersects[0].object);
  }
}


let targetPos = new THREE.Vector3() // 目标位置
let isMoving = false // 是否正在移动
const onClick = (e) => {
  // 点击场景移动小球到指定位置
  if (currentMode.value !== 'A') return // 只在模式A下才处理
  // 获取鼠标位置
  mouse.x = (e.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(e.clientY / window.innerHeight) * 2 + 1
  // 发出射线
  raycaster.setFromCamera(mouse, camera)
  const intersects = raycaster.intersectObject(plane)
  if (intersects.length > 0) {
    targetPos.copy(intersects[0].point)
    isMoving = true
  }
}


onMounted(() => {
  window.addEventListener('keydown', (e) => onKeyChange(e, true)) // e: event, isPressed: boolean
  window.addEventListener('keyup', (e) => onKeyChange(e, false))
  window.addEventListener('click', (e) => onClick(e))
  startModeC()
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
