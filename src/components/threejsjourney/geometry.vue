<template>
  <div class="geometry-container">
    <div class="header">
      <h2>Geometry</h2>
      <p>参考 Three.js Journey 经典章节 - Geometry</p>
    </div>
    <div ref="canvasContainer" class="canvas-wrapper"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import GUI from 'lil-gui'
import { gsap } from 'gsap'

const canvasContainer = ref(null)
let scene, camera, renderer, controls, gui, clock
let galaxyGeometry, galaxyMaterial, galaxyPoints


const generateGalaxy = () => {

  // 1. 准备底层数据
  const geometry = new THREE.BufferGeometry()

  // 2. 设置粒子数量
  const count = 5000

  // 3. 设置粒子位置
  const positionArray = new Float32Array(count * 3 * 3)

  for (let i = 0; i < count * 3 * 3; i++) {
    positionArray[i] = (Math.random() - 0.5) * 3
  }

  // 4. 设置粒子颜色
  const material = new THREE.MeshBasicMaterial({
    color: 'red',
    wireframe: true
  })

  geometry.setAttribute('position', new THREE.BufferAttribute(positionArray, 3))

  // 5. 创建物体并加入场景
  const mesh = new THREE.Mesh(geometry, material)

  const parameters = {
    spin() {
      gsap.to(mesh.rotation, {
        y: mesh.rotation.y + Math.PI,
        duration: 1,
        ease: 'power2.out'
      })
    }
  }

  scene.add(mesh)

  // GUI
  gui = new GUI()
  gui.add(material, 'wireframe')
  gui.add(mesh.position, 'x').min(-10).max(10).step(0.01)
  gui.add(mesh.position, 'y').min(-10).max(10).step(0.01)
  gui.add(mesh.position, 'z').min(-10).max(10).step(0.01)

  gui.addColor(material, 'color')

  gui.add(parameters, 'spin')
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



  const animate = () => {
    rafId = requestAnimationFrame(animate)
    const elapsedTime = clock.getElapsedTime()

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
.geometry-container {
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
