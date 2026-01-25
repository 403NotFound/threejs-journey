<template>
  <div class="threejs-container">
    <header class="top-bar">
      <h1>Day 05 - 加载外部模型 (GLTF)</h1>
      <span class="date">📅 2025-12-12</span>
    </header>
    <div ref="canvasContainer" class="canvas-wrapper"></div>
    <div v-if="loading" class="loading-overlay">
      <div class="loading-text">加载模型中... {{ loadingProgress }}%</div>
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
const loadingProgress = ref(0)
let gui = null

const initThreeJs = () => {
  // 1. 场景与相机
  const scene = new THREE.Scene()

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(0, 2, 5)

  // 2. 渲染器
  const renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  renderer.physicallyCorrectLights = true
  renderer.toneMapping = THREE.ACESFilmicToneMapping
  renderer.toneMappingExposure = 1
  canvasContainer.value.appendChild(renderer.domElement)

  // 3. 环境与光照
  // 加载 HDR
  new RGBELoader()
    .setPath('/textures/hdr/')
    .load('afrikaans_church_exterior_4k.hdr', (texture) => {
      // texture.mapping = THREE.EquirectangularReflectionMapping
      // scene.background = texture
      scene.environment = texture
      scene.backgroundBlurriness = 0 // 模糊背景


    })

  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5)
  scene.add(ambientLight)

  const dirLight = new THREE.DirectionalLight(0xffffff, 1)
  dirLight.position.set(5, 5, 5)
  dirLight.castShadow = true
  dirLight.shadow.mapSize.set(2048, 2048)
  scene.add(dirLight)

  // 4. 地面
  const planeGeometry = new THREE.PlaneGeometry(100, 100)
  const planeMaterial = new THREE.ShadowMaterial({ opacity: 0.3 }) // 仅显示阴影
  const plane = new THREE.Mesh(planeGeometry, planeMaterial)
  plane.rotation.x = -Math.PI / 2
  plane.position.y = 0
  plane.receiveShadow = true
  scene.add(plane)

  // 5. 加载模型 (场景搭建)
  const gltfLoader = new GLTFLoader()

  // 封装一个通用的模型加载函数
  const loadModel = (path, position = { x: 0, y: 0, z: 0 }, scale = 1, rotationY = 0) => {
    gltfLoader.load(
      path,
      (gltf) => {
        const model = gltf.scene
        model.traverse((child) => {
          if (child.isMesh) {
            child.castShadow = true
            child.receiveShadow = true
          }
        })

        model.position.set(position.x, position.y, position.z)
        model.scale.set(scale, scale, scale)
        model.rotation.y = rotationY

        scene.add(model)

        // 如果是主角僵尸，绑定到 GUI
        if (path.includes('zombie')) {
          initGUI(model, dirLight)
          loading.value = false // 只要主角加载完就隐藏 loading
        }
      },
      undefined, // We are not tracking progress for individual models in this refactor
      (error) => {
        console.error(`Error loading ${path}:`, error)
      }
    )
  }

  // === 开始搭建场景 ===

  // 1. 主角
  loadModel('/models/GLB/character-zombie.glb', { x: 0, y: 0, z: 0 }, 1, 0)

  // 2. 背景建筑
  loadModel('/models/GLB/crypt.glb', { x: -3, y: 0, z: -3 }, 1.5, Math.PI / 4)

  // 3. 植物 (树木)
  loadModel('/models/GLB/pine.glb', { x: 3.5, y: 0, z: -2 }, 1.2, 0)
  loadModel('/models/GLB/pine-crooked.glb', { x: -4, y: 0, z: 2 }, 1, Math.PI)

  // 4. 墓碑 & 装饰
  loadModel('/models/GLB/gravestone-cross.glb', { x: 1.5, y: 0, z: -1.5 }, 0.8, -0.2)
  loadModel('/models/GLB/gravestone-bevel.glb', { x: 2.5, y: 0, z: 1.5 }, 0.7, 0.5)
  loadModel('/models/GLB/pumpkin.glb', { x: -1, y: 0, z: 0.8 }, 0.6, Math.PI / 2)
  loadModel('/models/GLB/fence.glb', { x: 0, y: 0, z: -4 }, 1, 0)

  // 5. 地面 (换成稍微大一点的铺路石或直接用网格+ShadowMaterial)
  // 这里我们继续使用 ShadowMaterial，因为这些 Low Poly 模型本身带有一定的底座感，或者我们可以加载 loose rocks
  loadModel('/models/GLB/road.glb', { x: 0, y: 0.01, z: 0 }, 1, 0) // 把路铺在脚下

  // 添加网格辅助 (能让你看清"真实的地面"在哪里)
  const gridHelper = new THREE.GridHelper(50, 50)
  gridHelper.material.opacity = 0.4
  gridHelper.material.transparent = true
  scene.add(gridHelper)

  // 6. 控制器
  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.target.set(0, 1, 0)
  controls.maxPolarAngle = Math.PI / 2 - 0.1 // 限制相机不能钻到地底下

  // 7. GUI
  const initGUI = (model, light) => {
    gui = new GUI()
    gui.title('Day 05 - 模型控制')

    const modelFolder = gui.addFolder('模型位置')
    modelFolder.add(model.rotation, 'y', 0, Math.PI * 2).name('旋转 (Rotation)')
    modelFolder.add(model.scale, 'x', 0.1, 5).name('缩放 (Scale)').onChange(v => model.scale.set(v, v, v))
    modelFolder.add(model.position, 'y', -2, 2, 0.01).name('高度微调 (Y轴)') // 手动调整高度

    const lightFolder = gui.addFolder('光照')
    lightFolder.add(light, 'intensity', 0, 5).name('直射光强度')
    lightFolder.add(light.position, 'x', -10, 10).name('光照 X')
    lightFolder.add(light.position, 'y', 0, 20).name('光照 Y')
    lightFolder.add(light.position, 'z', -10, 10).name('光照 Z')
  }

  // 8. 动画循环
  const animate = () => {
    requestAnimationFrame(animate)
    controls.update()
    renderer.render(scene, camera)
    // if (model) {
    //   // model.rotation.y += 0.005 // 自动旋转
    // }
  }
  animate()

  // 窗口大小调整
  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  })
}

onMounted(() => {
  initThreeJs()
})

onUnmounted(() => {
  if (gui) gui.destroy()
  // 清理逻辑...
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
