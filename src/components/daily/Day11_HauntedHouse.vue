<template>
  <div class="haunted-house">
    <div class="header">
      <h2>Day 11: Haunted House (氛围造景)</h2>
      <p>学习阴影、雾效与场景层级架构</p>
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
let house, bushs, graves // 场景组

const initHountedHouse = () => {
  // 1. 初始化基础场景
  scene = new THREE.Scene()
  clock = new THREE.Clock()

  // 2. 这里的重点：雾 (Fog)
  // 参数：颜色, 开始距离, 结束距离
  // 雾的颜色最好和背景色保持一致，这样场景边缘会自然消失
  const fog = new THREE.Fog('#262837', 1, 15)
  scene.fog = fog
  scene.background = new THREE.Color('#262837')

  // 3. 房子组 (House Group)
  house = new THREE.Group()
  scene.add(house)

  // 墙壁
  const walls = new THREE.Mesh(
    new THREE.BoxGeometry(4, 2.5, 4),
    new THREE.MeshStandardMaterial({ color: '#ac8e82' })
  )
  walls.position.y = 1.25
  house.add(walls)

  // 屋顶 (使用锥形)
  const roof = new THREE.Mesh(
    new THREE.ConeGeometry(3.5, 1, 4),
    new THREE.MeshStandardMaterial({ color: '#b35f3f' })
  )
  roof.rotation.y = Math.PI * 0.25
  roof.position.y = 2.5 + 0.5
  house.add(roof)

  // 门
  const door = new THREE.Mesh(
    new THREE.PlaneGeometry(2, 2.2),
    new THREE.MeshStandardMaterial({ color: '#aa7b7b' })
  )
  door.position.y = 1
  door.position.z = 2 + 0.01 // 稍微偏移防止闪烁 (Z-fighting)
  house.add(door)

  // 4. 地面
  const floor = new THREE.Mesh(
    new THREE.PlaneGeometry(20, 20),
    new THREE.MeshStandardMaterial({ color: '#a9c388' })
  )
  floor.rotation.x = -Math.PI * 0.5
  floor.position.y = 0
  scene.add(floor)

  // 5. 光照
  // 环境光：淡淡的蓝色，模拟月光
  const ambientLight = new THREE.AmbientLight('#b9d5ff', 0.12)
  scene.add(ambientLight)

  // 月光 (方向光)
  const moonLight = new THREE.DirectionalLight('#b9d5ff', 0.12)
  moonLight.position.set(4, 5, -2)
  scene.add(moonLight)

  // 门灯 (点光源)
  const doorLight = new THREE.PointLight('#ff7d46', 1, 7)
  doorLight.position.set(0, 2.2, 2.7)
  house.add(doorLight)

  // 6. 相机与渲染器
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100)
  camera.position.set(4, 2, 5)

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  canvasContainer.value.appendChild(renderer.domElement)

  // 开启阴影渲染
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap

  // 开启物体阴影
  walls.castShadow = true
  floor.receiveShadow = true
  doorLight.castShadow = true

  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true

  const animate = () => {
    const elapsedTime = clock.getElapsedTime()
    controls.update()
    renderer.render(scene, camera)
    requestAnimationFrame(animate)
  }
  animate()
}

onMounted(() => {
  initHountedHouse()
})

onUnmounted(() => {
  if (renderer) renderer.dispose()
})
</script>

<style scoped>
.haunted-house {
  width: 100vw;
  height: 100vh;
  position: relative;
  background: #000;
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
  color: #b35f3f;
}

p {
  opacity: 0.6;
}

.canvas-wrapper {
  width: 100%;
  height: 100%;
}
</style>
