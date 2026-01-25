<template>
  <div class="practice-container">
    <div class="header">
      <h2>Day 07.5 专项训练：层级架构与高级变换</h2>
      <div class="controls">
        <button @click="currentMode = 'A'" :class="{ active: currentMode === 'A' }">实验 A: 父子层级</button>
        <button @click="currentMode = 'B'" :class="{ active: currentMode === 'B' }">实验 B: 世界坐标抓取</button>
        <button @click="currentMode = 'C'" :class="{ active: currentMode === 'C' }">实验 C: 变换与动画</button>
        <button @click="currentMode = 'D'" :class="{ active: currentMode === 'D' }">实验 D: 手动造物体</button>
        <button @click="currentMode = 'E'" :class="{ active: currentMode === 'E' }">实验 E: 基础点云</button>
      </div>
    </div>
    <div ref="canvasContainer" class="canvas-wrapper"></div>

    <div class="info-panel">
      <div v-if="currentMode === 'A'">
        <h3>实验 A：父子层级 (Hierarchy)</h3>
        <p>🟡 中心: 太阳 | 🔵 旋转: 地球</p>
        <p>代码逻辑：`sun.add(earth)`。</p>
        <p>理解：太阳在自转，地球没动，但地球由于是子物体，会跟着太阳一起旋转。</p>
      </div>
      <div v-if="currentMode === 'B'">
        <h3>实验 B：世界坐标 (World Space)</h3>
        <p>🔴 它是旋转容器里的子物体。</p>
        <p>局部坐标 (Relative): x: 2, y: 0, z: 0 (恒定)</p>
        <p style="color: #fbbf24;">世界坐标 (World): {{ worldPosLog }}</p>
        <p>理解：这是物体在整个 3D 宇宙里的“真实”坐标。</p>
      </div>
      <div v-if="currentMode === 'C'">
        <h3>实验 C：周期变换 (Math.sin)</h3>
        <p>🟢 利用正弦曲线控制 `scale` 缩放。</p>
        <p>效果：心跳呼吸感。</p>
        <p>公式：1 + sin(time * 3) * 0.5</p>
      </div>
      <div v-if="currentMode === 'D'">
        <h3>实验 D：BufferGeometry (底层数据)</h3>
        <p>📐 手动定义 3 个顶点的坐标数据。</p>
        <p>重点：理解 `Float32Array` 和 `setAttribute`。</p>
      </div>
      <div v-if="currentMode === 'E'">
        <h3>实验 E：基础点云 (Points)</h3>
        <p>✨ 随机生成 500 个零散的点。</p>
        <p>重点：`THREE.Points` 是粒子系统的基石。</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const canvasContainer = ref(null)
const currentMode = ref('E')
const worldPosLog = ref('')

let scene, camera, renderer, controls, rafId, clock
let sun, earth, parentGroup, childCube, heartMesh, customMesh, pointCloud

const initBase = () => {
  if (renderer) {
    renderer.dispose()
    canvasContainer.value.innerHTML = ''
  }

  if (rafId) cancelAnimationFrame(rafId)
  clock = new THREE.Clock()

  scene = new THREE.Scene()

  scene.background = new THREE.Color('#0a0a0a')
  scene.add(new THREE.GridHelper(10, 10, 0x333333, 0x222222))

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100)
  camera.position.set(5, 5, 5)

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)

  canvasContainer.value.appendChild(renderer.domElement)

  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true // 启用惯性

  const light = new THREE.PointLight(0xffffff, 50)
  light.position.set(5, 5, 5)
  scene.add(light)

  scene.add(new THREE.AmbientLight(0xffffff, 0.4))

}

// === 实验 A: 父子层级 (模拟系) ===
const startModeA = () => {
  initBase()

  // 1. 太阳
  sun = new THREE.Mesh(
    new THREE.SphereGeometry(1, 32, 32),
    new THREE.MeshStandardMaterial({ color: 0xffcc00, emissive: 0xffaa00 })
  )
  scene.add(sun)

  // 2. 地球
  earth = new THREE.Mesh(
    new THREE.SphereGeometry(0.3, 24, 24),
    new THREE.MeshStandardMaterial({ color: 0x2288ff })
  )
  earth.position.x = 3
  sun.add(earth)

  const animateA = () => {
    rafId = requestAnimationFrame(animateA)
    const delta = clock.getDelta()

    // 太阳自转
    sun.rotation.y += 2 * delta
    earth.rotation.y += 0.05

    controls.update()
    renderer.render(scene, camera)
  }
  animateA()
}

// === 实验 B: 世界坐标抓取 ===
const startModeB = () => {
  initBase()
  // 创建一个旋转的父容器
  parentGroup = new THREE.Group()
  scene.add(parentGroup)

  // 将子物体放入容器并偏移
  childCube = new THREE.Mesh(
    new THREE.BoxGeometry(1, 1, 1),
    new THREE.MeshStandardMaterial({ color: 0xff4444 })
  )

  childCube.position.x = 3
  parentGroup.add(childCube)
  // 用于存放世界坐标的临时向量对象（避免在每一帧 new 对象）
  const tempV3 = new THREE.Vector3()

  const animateB = () => {
    rafId = requestAnimationFrame(animateB)

    // 父级旋转
    parentGroup.rotation.y += 0.02
    // 关键：抓取子物体的世界坐标
    // 即使 childCube.position.x 永远是 3，它的真实坐标在不断变化

    childCube.getWorldPosition(tempV3)
    worldPosLog.value = `x: ${tempV3.x.toFixed(2)}, y: ${tempV3.y.toFixed(2)}, z: ${tempV3.z.toFixed(2)}`

    controls.update()
    renderer.render(scene, camera)
  }

  animateB()
}

// === 实验 C: 变换与动画 (Math.sin) ===
const startModeC = () => {
  initBase()

  heartMesh = new THREE.Mesh(
    new THREE.SphereGeometry(1, 32, 32),
    new THREE.MeshStandardMaterial({ color: 0x44ff88, roughness: 0.2 }) // 
  )
  scene.add(heartMesh)



  const animateC = () => {
    rafId = requestAnimationFrame(animateC)

    // 使用三角函数做周期变化
    const time = clock.getElapsedTime() // 总时间
    const scaleFactor = 1 + Math.sin(time) * 0.3
    // const scaleFactor = 1 + Math.abs(Math.sin(time * 10)) * 0.5
    heartMesh.scale.set(scaleFactor, scaleFactor, scaleFactor)

    controls.update()
    renderer.render(scene, camera)
  }

  animateC()
}

// === 实验 D: 手动造物体 (BufferGeometry) ===
const startModeD = () => {
  initBase()

  const vertices = new Float32Array([
    0, 0, 0,
    2, 0, 0,
    2, 2, 0
  ])

  const geometry = new THREE.BufferGeometry() // 创建一个空的几何体
  geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3))


  const material = new THREE.MeshBasicMaterial({ color: 0xffff00, side: THREE.DoubleSide })
  const mesh = new THREE.Mesh(geometry, material)
  scene.add(mesh)


  const animateD = () => {
    rafId = requestAnimationFrame(animateD)
    controls.update()
    renderer.render(scene, camera)
  }
  animateD()
}

// === 实验 E: 基础点云 (Points) ===
const startModeE = () => {
  initBase()

  const count = 1000

  const positions = new Float32Array(count * 3) // 1000 个点，每个点 3 个坐标 (x,y,z)
  for (let i = 0; i < count * 3; i++) {
    positions[i] = (Math.random() - 0.5) * 10 // 随机填充坐标，范围在 -5 到 5 之间
  }

  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  const material = new THREE.PointsMaterial({
    size: 0.1,
    color: 0x00ffff,
    sizeAttenuation: true// 开启远小近大的效果
  })

  pointCloud = new THREE.Points(geometry, material)
  scene.add(pointCloud)



  const animateE = () => {
    rafId = requestAnimationFrame(animateE)
    // 让点云整体转动一下，增加视觉感
    const time = clock.getElapsedTime()

    const s = 1 + Math.sin(time * 2) * 0.2
    pointCloud.scale.set(s, s, s)

    pointCloud.material.size = 0.1 + Math.sin(time * 3) * 0.05
    pointCloud.rotation.y += 0.002
    controls.update()
    renderer.render(scene, camera)
  }
  animateE()
}

onMounted(() => {
  startModeE()
})

onUnmounted(() => {
  if (rafId) cancelAnimationFrame(rafId)
  if (renderer) renderer.dispose()
})

watch(currentMode, (val) => {
  if (val === 'A') startModeA()
  if (val === 'B') startModeB()
  if (val === 'C') startModeC()
  if (val === 'D') startModeD()
  if (val === 'E') startModeE()
})
</script>

<style scoped>
.practice-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  background: #000;
  overflow: hidden;
}

.header {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  padding: 20px;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  color: white;
  display: flex;
  justify-content: space-between;
  align-items: center;
  z-index: 10;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.controls button {
  margin-left: 10px;
  padding: 8px 20px;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #fff;
  cursor: pointer;
  border-radius: 6px;
  transition: all 0.3s;
}

.controls button:hover {
  background: rgba(255, 255, 255, 0.2);
}

.controls button.active {
  background: #6366f1;
  color: white;
  border-color: #818cf8;
  box-shadow: 0 0 15px rgba(99, 102, 241, 0.4);
}

.info-panel {
  position: absolute;
  top: 100px;
  left: 20px;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(10px);
  color: white;
  padding: 20px;
  border-radius: 12px;
  max-width: 320px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
}

h3 {
  margin-top: 0;
  color: #818cf8;
  font-size: 1.2rem;
}

p {
  color: #ccc;
  line-height: 1.6;
  font-size: 0.9rem;
}
</style>
