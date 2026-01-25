<template>
  <div class="threejs-container">
    <!-- 顶部标题栏 -->
    <header class="top-bar">
      <h1>Day 03 - 几何体、纹理与阴影</h1>
      <span class="date">📅 2025-12-12</span>
    </header>

    <!-- Three.js 画布容器 -->
    <div ref="canvasContainer" class="canvas-wrapper">

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import Stats from 'three/examples/jsm/libs/stats.module'
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader'
import GUI from 'lil-gui'
// Canvas 容器引用
const canvasContainer = ref(null)

/**
 * 初始化 GUI 调试面板
 * @param {Object} params - GUI 控制的对象
 */
const initGUI = (params) => {
  const { box, sphere, sphereMaterial, planeTexture, ambientLight, directionalLight } = params

  const gui = new GUI()
  gui.title('Day 04 - PBR 材质与环境贴图')

  // 1. PBR 材质控制 (重点)
  const pbrFolder = gui.addFolder('PBR 材质控制 (球体)')

  const pbrParams = {
    color: sphereMaterial.color.getHex()
  }

  pbrFolder.addColor(pbrParams, 'color')
    .name('材质颜色')
    .onChange((value) => {
      sphereMaterial.color.set(value)
    })

  pbrFolder.add(sphereMaterial, 'metalness', 0, 1, 0.01)
    .name('金属度 (Metalness)')

  pbrFolder.add(sphereMaterial, 'roughness', 0, 1, 0.01)
    .name('粗糙度 (Roughness)')

  pbrFolder.add(sphereMaterial.normalScale, 'x', 0, 3, 0.1).name('法线强度 X')
  pbrFolder.add(sphereMaterial.normalScale, 'y', 0, 3, 0.1).name('法线强度 Y')

  pbrFolder.add(sphereMaterial, 'wireframe')
    .name('线框模式')

  pbrFolder.open()

  // 几何体控制
  const geometryFolder = gui.addFolder('几何体位置')
  geometryFolder.add(box.position, 'y', 0, 5, 0.1).name('立方体高度')
  geometryFolder.add(sphere.position, 'y', 0, 5, 0.1).name('球体高度')

  // 纹理控制
  const textureFolder = gui.addFolder('地面纹理')
  const textureSettings = {
    repeatX: 4,
    repeatY: 4
  }
  textureFolder.add(textureSettings, 'repeatX', 1, 20, 1)
    .name('纹理 X 重复')
    .onChange((value) => {
      planeTexture.repeat.x = value
    })
  textureFolder.add(textureSettings, 'repeatY', 1, 20, 1)
    .name('纹理 Y 重复')
    .onChange((value) => {
      planeTexture.repeat.y = value
    })

  // 光照控制
  const lightFolder = gui.addFolder('光照系统')
  lightFolder.add(ambientLight, 'intensity', 0, 2, 0.01).name('环境光强度')
  lightFolder.add(directionalLight, 'intensity', 0, 5, 0.01).name('平形光强度') // 物理光照下数值范围可能大一点
  lightFolder.add(directionalLight.position, 'x', -10, 10, 0.1).name('光源 X')
  lightFolder.add(directionalLight.position, 'y', 0, 20, 0.1).name('光源 Y')
  lightFolder.add(directionalLight.position, 'z', -10, 10, 0.1).name('光源 Z')

  return gui
}

const initThreeJs = () => {
  // 创建 Stats 实例
  const stats = new Stats()
  stats.dom.style.position = 'fixed'
  stats.dom.style.top = '60px'  // 标题栏高度是 60px
  stats.dom.style.left = '0'
  stats.dom.style.zIndex = '999'
  document.body.appendChild(stats.dom)

  // 场景
  const scene = new THREE.Scene()

  // 加载环境纹理 (HDR)
  new RGBELoader()
    .setPath('/textures/hdr/')
    .load('afrikaans_church_exterior_4k.hdr', (texture) => {
      texture.mapping = THREE.EquirectangularReflectionMapping // 环境贴图映射
      scene.background = texture
      scene.environment = texture
    })

  // 相机
  const camera = new THREE.PerspectiveCamera(
    75, // 视野角度
    window.innerWidth / window.innerHeight, // 宽高比
    0.1, // 近裁剪面
    1000 // 远裁剪面
  )
  camera.position.z = 8 // 设置相机位置
  camera.position.y = 2

  // 渲染器
  const renderer = new THREE.WebGLRenderer({
    antialias: true, // 抗锯齿
    // alpha: true, // 使用环境图时，alpha设为false或者不设置背景色
  })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true  // 启用阴影渲染
  renderer.shadowMap.type = THREE.PCFSoftShadowMap  // 柔和阴影
  renderer.physicallyCorrectLights = true // 物理光照模式
  renderer.toneMapping = THREE.ACESFilmicToneMapping // 电影级色调映射
  renderer.toneMappingExposure = 1

  // 将渲染器的 canvas 添加到容器中
  canvasContainer.value.appendChild(renderer.domElement)

  // 添加网格辅助线
  const gridHelper = new THREE.GridHelper(30, 30)
  scene.add(gridHelper)


  // 添加坐标轴辅助线
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  const textureLoader = new THREE.TextureLoader()

  // 1. 立方体（保留作为参照）
  const boxGeometry = new THREE.BoxGeometry(1.5, 1.5, 1.5)
  const boxMaterial = new THREE.MeshStandardMaterial({
    map: textureLoader.load('/textures/4.jpg'),
    roughness: 0.2,
    metalness: 0.8
  })
  const box = new THREE.Mesh(boxGeometry, boxMaterial)
  box.position.set(-2, 0.75, 0)
  box.castShadow = true
  scene.add(box)


  // 2. 球体（主要用于 PBR 演示）
  const sphereGeometry = new THREE.SphereGeometry(1, 64, 64) // 增加细分以更好显示细节
  const sphereMaterial = new THREE.MeshStandardMaterial({
    color: 0xffffff,
    metalness: 0.7,
    roughness: 0.2,
    // normalMap: textureLoader.load('/textures/water_normal.jpg'), // 加载法线贴图
    normalScale: new THREE.Vector2(1, 1) // 法线强度
  })
  const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial)
  sphere.position.set(2, 1, 0)
  sphere.castShadow = true
  scene.add(sphere)


  // 3. 地面
  const planeGeometry = new THREE.PlaneGeometry(20, 20) // 增加地面大小
  const planeTexture = textureLoader.load('/textures/5.jpg')
  planeTexture.wrapS = THREE.RepeatWrapping
  planeTexture.wrapT = THREE.RepeatWrapping
  planeTexture.repeat.set(1, 1)

  const planeMaterial = new THREE.MeshStandardMaterial({
    map: planeTexture,
    side: THREE.DoubleSide
  })
  const plane = new THREE.Mesh(planeGeometry, planeMaterial)
  plane.rotation.x = -Math.PI / 2 // -Math.PI / 2 旋转地面
  plane.position.y = 0
  plane.receiveShadow = true
  // scene.add(plane)

  // ========== 光照系统 ==========

  // 环境光
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.2) // 环境图已经提供了光照，环境光可以暗一点
  scene.add(ambientLight)

  // 平行光
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(5, 5, 5)
  directionalLight.castShadow = true
  directionalLight.shadow.mapSize.width = 2048
  directionalLight.shadow.mapSize.height = 2048

  scene.add(directionalLight)
  scene.add(new THREE.DirectionalLightHelper(directionalLight, 1))

  // ========== 初始化 GUI 调试面板 ==========
  initGUI({
    box,
    sphere,
    sphereMaterial, // 传入球体材质
    planeTexture,
    ambientLight,
    directionalLight
  })

  // 添加轨道控制器
  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05

  // 渲染循环
  const animate = () => {
    requestAnimationFrame(animate)

    // 旋转动画
    box.rotation.y += 0.005
    // sphere.rotation.y += 0.005 

    stats.update()
    controls.update()
    renderer.render(scene, camera)
  }
  animate()
}

// Day 03: 继续探索
onMounted(() => {
  console.log('Day 03: Three.js 学习页面已加载')
  initThreeJs()
})

onUnmounted(() => {
  console.log('Day 03: 页面已卸载')
  // TODO: 清理 Three.js 资源
})
</script>

<style scoped>
.threejs-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  overflow: hidden;
}

.canvas-wrapper {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
}

/* 顶部标题栏 */
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
  z-index: 1000;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.top-bar h1 {
  font-size: 1.2rem;
  margin: 0;
  color: #fff;
  font-weight: 600;
  letter-spacing: 0.5px;
}

.top-bar .date {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.7);
  font-weight: 400;
}
</style>
