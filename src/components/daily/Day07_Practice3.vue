<template>
  <div class="practice-container">
    <div class="header">
      <h2>Day 07.5 专项训练：层级架构与核心数据结构</h2>
      <div class="controls">
        <button @click="currentMode = 'A'" :class="{ active: currentMode === 'A' }">实验 A: 父子层级</button>
        <button @click="currentMode = 'B'" :class="{ active: currentMode === 'B' }">实验 B: 世界坐标</button>
        <button @click="currentMode = 'C'" :class="{ active: currentMode === 'C' }">实验 C: 变换动画</button>
        <button @click="currentMode = 'D'" :class="{ active: currentMode === 'D' }">实验 D: 手动造物体</button>
        <button @click="currentMode = 'E'" :class="{ active: currentMode === 'E' }">实验 E: 基础点云</button>
      </div>
    </div>
    <div ref="canvasContainer" class="canvas-wrapper"></div>

    <div class="info-panel">
      <div v-if="currentMode === 'A'">
        <h3>实验 A：父子层级 (Hierarchy)</h3>
        <p>🟡 太阳 | 🔵 地球</p>
        <p>关键：`sun.add(earth)`。地球继承了太阳的旋转空间。</p>
      </div>
      <div v-if="currentMode === 'B'">
        <h3>实验 B：世界坐标 (World Space)</h3>
        <p>🔴 旋转的子方块。</p>
        <p>真实坐标：{{ worldPosLog }}</p>
        <p>API：`getWorldPosition()`。</p>
      </div>
      <div v-if="currentMode === 'C'">
        <h3>实验 C：周期变换 (Math.sin)</h3>
        <p>🟢 心跳感。利用 `elapsedTime` 作为动力。</p>
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
const currentMode = ref('A')
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
  controls.enableDamping = true

  const light = new THREE.PointLight(0xffffff, 50)
  light.position.set(5, 5, 5)
  scene.add(light)
  scene.add(new THREE.AmbientLight(0xffffff, 0.4))
}

const startModeA = () => {
  initBase()
  sun = new THREE.Mesh(new THREE.SphereGeometry(1, 32, 32), new THREE.MeshStandardMaterial({ color: 0xffcc00, emissive: 0xffaa00 }))
  scene.add(sun)
  earth = new THREE.Mesh(new THREE.SphereGeometry(0.3, 24, 24), new THREE.MeshStandardMaterial({ color: 0x2288ff }))
  earth.position.x = 3
  sun.add(earth)
  const animateA = () => {
    rafId = requestAnimationFrame(animateA)
    sun.rotation.y += 0.01
    earth.rotation.y += 0.05
    controls.update()
    renderer.render(scene, camera)
  }
  animateA()
}

const startModeB = () => {
  initBase()
  parentGroup = new THREE.Group()
  scene.add(parentGroup)
  childCube = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.5, 0.5), new THREE.MeshStandardMaterial({ color: 0xff4444 }))
  childCube.position.x = 3
  parentGroup.add(childCube)
  const tempV3 = new THREE.Vector3()
  const animateB = () => {
    rafId = requestAnimationFrame(animateB)
    parentGroup.rotation.y += 0.02
    childCube.getWorldPosition(tempV3)
    worldPosLog.value = `X: ${tempV3.x.toFixed(2)}, Z: ${tempV3.z.toFixed(2)}`
    controls.update()
    renderer.render(scene, camera)
  }
  animateB()
}

const startModeC = () => {
  initBase()
  heartMesh = new THREE.Mesh(new THREE.SphereGeometry(1, 32, 32), new THREE.MeshStandardMaterial({ color: 0x44ff88 }))
  scene.add(heartMesh)
  const animateC = () => {
    rafId = requestAnimationFrame(animateC)
    const time = clock.getElapsedTime()
    const scale = 1 + Math.sin(time * 4) * 0.3
    heartMesh.scale.set(scale, scale, scale)
    controls.update()
    renderer.render(scene, camera)
  }
  animateC()
}

// === 实验 D: 手动造物体 (BufferGeometry) ===
const startModeD = () => {
  initBase()

  // 3个顶点，每个顶点3个分量 (x,y,z)
  const vertices = new Float32Array([
    0, 0, 0, // 点 1
    2, 0, 0, // 点 2
    1, 2, 0  // 点 3
  ])

  const geometry = new THREE.BufferGeometry()
  // 必须把数据存入名为 'position' 的属性中
  geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3))

  const material = new THREE.MeshBasicMaterial({ color: 0xffff00, side: THREE.DoubleSide })
  customMesh = new THREE.Mesh(geometry, material)
  scene.add(customMesh)

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
  const positions = new Float32Array(count * 3)
  for (let i = 0; i < count * 3; i++) {
    positions[i] = (Math.random() - 0.5) * 10
  }

  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))

  const material = new THREE.PointsMaterial({
    size: 0.1,
    color: 0x00ffff,
    sizeAttenuation: true
  })

  pointCloud = new THREE.Points(geometry, material)
  scene.add(pointCloud)

  const animateE = () => {
    rafId = requestAnimationFrame(animateE)
    pointCloud.rotation.y += 0.005
    controls.update()
    renderer.render(scene, camera)
  }
  animateE()
}

onMounted(() => {
  startModeA()
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
}

.controls button.active {
  background: #6366f1;
  color: white;
  border-color: #818cf8;
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
}

h3 {
  margin-top: 0;
  color: #818cf8;
}

p {
  color: #ccc;
  font-size: 0.9rem;
}
</style>
