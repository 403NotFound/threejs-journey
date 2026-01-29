<template>
  <div class="geometry-container">
    <div class="header">
      <h2>Textures</h2>
      <p>参考 Three.js Journey 经典章节 - Textures</p>
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
import textureImage from '../../../public/textures/1.jpg'

// SUBTANCE Designer
// https://substance3d.com/

const canvasContainer = ref(null)
let scene, camera, renderer, controls, gui, clock
let galaxyGeometry, galaxyMaterial, galaxyPoints

const loadingManager = new THREE.LoadingManager()

// 监听事件
loadingManager.onStart = () => {
  console.log('loadingManager onStart')
}
loadingManager.onLoad = () => {
  console.log('loadingManager onLoad')
}
loadingManager.onProgress = () => {
  console.log('loadingManager onProgress')
}
loadingManager.onError = () => {
  console.log('loadingManager onError')
}


const generateGalaxy = () => {

  const geometry = new THREE.BoxGeometry(1, 1, 1)

  const material = new THREE.MeshBasicMaterial({
    // color: 'red',
    // wireframe: true
  })
  const texture = new THREE.TextureLoader(loadingManager).load(textureImage)

  material.map = texture
  const mesh = new THREE.Mesh(geometry, material)

  scene.add(mesh)

  // GUI
  gui = new GUI()
  gui.add(material, 'wireframe')
  gui.add(mesh.position, 'x').min(-10).max(10).step(0.01)
  gui.add(mesh.position, 'y').min(-10).max(10).step(0.01)
  gui.add(mesh.position, 'z').min(-10).max(10).step(0.01)

  gui.addColor(material, 'color')

  // gui.add(parameters, 'spin')
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
