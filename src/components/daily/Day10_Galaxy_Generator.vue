<template>
  <div class="galaxy-container">
    <div class="header">
      <h2>Day 10: Galaxy Generator (星系生成器)</h2>
      <p>参考 Three.js Journey 经典章节 - 数学与艺术的结合</p>
    </div>
    <div ref="canvasContainer" class="canvas-wrapper"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import GUI from 'lil-gui'

const canvasContainer = ref(null)
let scene, camera, renderer, controls, gui, clock
let galaxyGeometry, galaxyMaterial, galaxyPoints

// 星系参数 (待会儿会在 GUI 中调节)
const parameters = {
  count: 10000,          // 粒子数量
  size: 0.02,           // 粒子大小
  radius: 5,            // 星系半径
  branches: 3,          // 星系分支(旋臂)数量
  spin: 1,              // 旋转弯曲度
  randomness: 0.2,      // 随机散布程度
  randomnessPower: 3,   // 散布的平滑度
  insideColor: '#ff6030', // 中心颜色
  outsideColor: '#1b3984' // 边缘颜色
}

const generateGalaxy = () => {
  // 1. 如果之前有星系，先销毁掉 (防止内存泄露)
  if (galaxyPoints !== undefined) {
    galaxyGeometry.dispose()
    galaxyMaterial.dispose()
    scene.remove(galaxyPoints)
  }

  // 2. 准备底层数据
  galaxyGeometry = new THREE.BufferGeometry()
  const positions = new Float32Array(parameters.count * 3)
  const colors = new Float32Array(parameters.count * 3)

  const colorInside = new THREE.Color(parameters.insideColor)
  const colorOutside = new THREE.Color(parameters.outsideColor)

  for (let i = 0; i < parameters.count; i++) {
    const i3 = i * 3

    // --- 核心数学：星系分布逻辑 ---
    // 基础半径
    const radius = Math.random() * parameters.radius

    // 算出该粒子属于第几个分支 (0, 1, 2...)
    const branchAngle = (i % parameters.branches) / parameters.branches * Math.PI * 2

    // 旋转偏移：离中心越远，旋转角度越大
    const spinAngle = radius * parameters.spin

    // 计算 X, Y, Z (我们先做最基础的圆周运动分布)
    // 注意：这里我们加入一点简易的随机散布
    const randomX = Math.pow(Math.random(), parameters.randomnessPower) * (Math.random() < 0.5 ? 1 : -1)
    const randomY = Math.pow(Math.random(), parameters.randomnessPower) * (Math.random() < 0.5 ? 1 : -1)
    const randomZ = Math.pow(Math.random(), parameters.randomnessPower) * (Math.random() < 0.5 ? 1 : -1)

    positions[i3 + 0] = Math.cos(branchAngle + spinAngle) * radius + randomX
    positions[i3 + 1] = randomY // 可以在 y 轴也加点起伏
    positions[i3 + 2] = Math.sin(branchAngle + spinAngle) * radius + randomZ

    // --- 颜色混合：由中心向边缘过渡 ---
    const mixedColor = colorInside.clone()
    mixedColor.lerp(colorOutside, radius / parameters.radius)

    colors[i3 + 0] = mixedColor.r
    colors[i3 + 1] = mixedColor.g
    colors[i3 + 2] = mixedColor.b
  }

  galaxyGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  galaxyGeometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))

  // 3. 材质
  galaxyMaterial = new THREE.PointsMaterial({
    size: parameters.size,
    sizeAttenuation: true, // 开启远小近大
    depthWrite: false,      // 粒子叠加时更自然
    blending: THREE.AdditiveBlending, // 混合模式：亮点叠加
    vertexColors: true      // 启用上面定义的顶点颜色
  })

  // 4. 创建物体并加入场景
  galaxyPoints = new THREE.Points(galaxyGeometry, galaxyMaterial)
  scene.add(galaxyPoints)
}

const initThree = () => {
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#030303')
  clock = new THREE.Clock()

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100)
  camera.position.set(3, 3, 3)

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  canvasContainer.value.appendChild(renderer.domElement)

  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true

  // 初始化星系
  generateGalaxy()

  // 初始化 GUI 控制面板
  gui = new GUI()
  gui.add(parameters, 'count').min(100).max(100000).step(100).onFinishChange(generateGalaxy)
  gui.add(parameters, 'size').min(0.001).max(0.1).step(0.001).onFinishChange(generateGalaxy)
  gui.add(parameters, 'radius').min(0.01).max(20).step(0.01).onFinishChange(generateGalaxy)
  gui.add(parameters, 'branches').min(2).max(20).step(1).onFinishChange(generateGalaxy)
  gui.add(parameters, 'spin').min(-5).max(5).step(0.001).onFinishChange(generateGalaxy)
  gui.add(parameters, 'randomness').min(0).max(2).step(0.001).onFinishChange(generateGalaxy)
  gui.addColor(parameters, 'insideColor').onFinishChange(generateGalaxy)
  gui.addColor(parameters, 'outsideColor').onFinishChange(generateGalaxy)

  const animate = () => {
    rafId = requestAnimationFrame(animate)
    const elapsedTime = clock.getElapsedTime()

    // 让星系缓缓转动
    galaxyPoints.rotation.y = elapsedTime * 0.1

    controls.update()
    renderer.render(scene, camera)
  }
  let rafId = requestAnimationFrame(animate)

  // 响应式
  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  })
}

onMounted(() => {
  initThree()
})

onUnmounted(() => {
  if (gui) gui.destroy()
  if (renderer) renderer.dispose()
})
</script>

<style scoped>
.galaxy-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  overflow: hidden;
}

.header {
  position: absolute;
  top: 20px;
  left: 20px;
  color: white;
  z-index: 10;
  pointer-events: none;
}

h2 {
  margin: 0;
  font-size: 1.5rem;
  color: #ff6030;
}

p {
  margin: 5px 0 0;
  font-size: 0.9rem;
  opacity: 0.6;
}

.canvas-wrapper {
  width: 100%;
  height: 100%;
}
</style>
