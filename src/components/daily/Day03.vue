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
import GUI from 'lil-gui'
// Canvas 容器引用
const canvasContainer = ref(null)

/**
 * 初始化 GUI 调试面板
 * @param {Object} params - GUI 控制的对象
 */
const initGUI = (params) => {
  const { box, sphere, torus, cone, planeTexture, ambientLight, directionalLight } = params

  const gui = new GUI()
  gui.title('Day 03 - 几何体与阴影')

  // 几何体控制
  const geometryFolder = gui.addFolder('几何体控制')
  geometryFolder.add(box.position, 'y', 0, 5, 0.1).name('立方体高度')
  geometryFolder.add(sphere.position, 'y', 0, 5, 0.1).name('球体高度')
  geometryFolder.add(torus.rotation, 'x', 0, Math.PI * 2, 0.01).name('圆环旋转X')
  geometryFolder.add(cone.position, 'y', 0, 5, 0.1).name('圆锥高度')
  geometryFolder.open()

  // 纹理控制
  const textureFolder = gui.addFolder('纹理控制')
  const textureSettings = {
    repeatX: 1,
    repeatY: 1
  }
  textureFolder.add(textureSettings, 'repeatX', 1, 10, 1)
    .name('地面纹理 X 重复')
    .onChange((value) => {
      planeTexture.repeat.x = value
    })
  textureFolder.add(textureSettings, 'repeatY', 1, 10, 1)
    .name('地面纹理 Y 重复')
    .onChange((value) => {
      planeTexture.repeat.y = value
    })

  // 光照控制
  const lightFolder = gui.addFolder('光照控制')
  lightFolder.add(ambientLight, 'intensity', 0, 2, 0.01).name('环境光强度')
  lightFolder.add(directionalLight, 'intensity', 0, 2, 0.01).name('平行光强度')
  lightFolder.add(directionalLight.position, 'x', -10, 10, 0.1).name('光源 X')
  lightFolder.add(directionalLight.position, 'y', 0, 10, 0.1).name('光源 Y')
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

  // 相机
  const camera = new THREE.PerspectiveCamera(
    75, // 视野角度
    window.innerWidth / window.innerHeight, // 宽高比
    0.1, // 近裁剪面
    1000 // 远裁剪面
  )
  camera.position.z = 5 // 设置相机位置，让它能看到立方体
  // camera.lookAt(0, 2, 0) // 让相机看向原点  
  // 渲染器
  const renderer = new THREE.WebGLRenderer({
    antialias: true, // 抗锯齿
    alpha: true, // 透明背景
  })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true  // 启用阴影渲染
  renderer.shadowMap.type = THREE.PCFSoftShadowMap  // 柔和阴影

  // 将渲染器的 canvas 添加到容器中
  canvasContainer.value.appendChild(renderer.domElement)

  // 添加网格辅助线
  const gridHelper = new THREE.GridHelper(30, 30) // 参数：网格大小，网格分段数
  scene.add(gridHelper)

  // 添加坐标轴辅助线
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  // ========== 创建多种几何体 ==========

  // 创建纹理加载器
  const textureLoader = new THREE.TextureLoader()
  // 1. 立方体（6个面不同纹理）
  const boxGeometry = new THREE.BoxGeometry(1, 1, 1)



  // 加载6个面的纹理，创建材质数组
  // 顺序通常是: right(x+), left(x-), top(y+), bottom(y-), front(z+), back(z-)
  const boxMaterials = [
    new THREE.MeshStandardMaterial({ map: textureLoader.load('/textures/1.jpg') }),
    new THREE.MeshStandardMaterial({ map: textureLoader.load('/textures/2.jpg') }),
    new THREE.MeshStandardMaterial({ map: textureLoader.load('/textures/3.jpg') }),
    new THREE.MeshStandardMaterial({ map: textureLoader.load('/textures/4.jpg') }),
    new THREE.MeshStandardMaterial({ map: textureLoader.load('/textures/5.jpg') }),
    new THREE.MeshStandardMaterial({ map: textureLoader.load('/textures/1.jpg') })
  ]

  const box = new THREE.Mesh(boxGeometry, boxMaterials)
  box.position.set(-3, 1, 0)
  box.castShadow = true  // 投射阴影
  scene.add(box)


  // 2. 球体（带纹理贴图）
  const sphereGeometry = new THREE.SphereGeometry(0.7, 32, 32)
  const sphereTexture = textureLoader.load('/textures/2.jpg')
  const sphereMaterial = new THREE.MeshStandardMaterial({
    map: sphereTexture
  })
  const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial)
  sphere.position.set(-1, 1, 0)
  sphere.castShadow = true
  scene.add(sphere)

  // 3. 圆环（带纹理贴图）
  const torusGeometry = new THREE.TorusGeometry(0.5, 0.2, 16, 100)
  const torusTexture = textureLoader.load('/textures/3.jpg')
  const torusMaterial = new THREE.MeshStandardMaterial({
    map: torusTexture
  })
  const torus = new THREE.Mesh(torusGeometry, torusMaterial)
  torus.position.set(1, 1, 0)
  torus.castShadow = true
  scene.add(torus)

  // 4. 圆锥（带纹理贴图）
  const coneGeometry = new THREE.ConeGeometry(0.5, 1, 32)
  const coneTexture = textureLoader.load('/textures/1.jpg')
  const coneMaterial = new THREE.MeshStandardMaterial({
    map: coneTexture
  })
  const cone = new THREE.Mesh(coneGeometry, coneMaterial)
  cone.position.set(3, 1, 0)
  cone.castShadow = true
  scene.add(cone)

  // 5. 地面（接收阴影，带纹理）
  const planeGeometry = new THREE.PlaneGeometry(20, 20)
  const planeTexture = textureLoader.load('/textures/5.jpg')

  // 设置纹理重复
  planeTexture.wrapS = THREE.RepeatWrapping
  planeTexture.wrapT = THREE.RepeatWrapping
  planeTexture.repeat.set(1, 1)  // 重复 4x4 次

  const planeMaterial = new THREE.MeshStandardMaterial({
    map: planeTexture,
    side: THREE.DoubleSide // 两面可见
  })
  const plane = new THREE.Mesh(planeGeometry, planeMaterial)
  plane.rotation.x = -Math.PI / 2  // 旋转90度使其水平
  plane.position.y = 0
  plane.receiveShadow = true  // 接收阴影
  scene.add(plane)

  // ========== 光照系统 ==========

  // 环境光（提供基础照明）
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.4)
  scene.add(ambientLight)

  // 平行光（模拟太阳光，投射阴影）
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8)
  directionalLight.position.set(5, 5, 5)
  directionalLight.castShadow = true  // 启用阴影

  // 配置阴影属性
  directionalLight.shadow.mapSize.width = 2048
  directionalLight.shadow.mapSize.height = 2048
  directionalLight.shadow.camera.near = 0.5
  directionalLight.shadow.camera.far = 50
  directionalLight.shadow.camera.left = -10
  directionalLight.shadow.camera.right = 10
  directionalLight.shadow.camera.top = 10
  directionalLight.shadow.camera.bottom = -10

  scene.add(directionalLight)

  // 添加光源辅助器
  const directionalLightHelper = new THREE.DirectionalLightHelper(directionalLight, 1)
  scene.add(directionalLightHelper)

  // ========== 初始化 GUI 调试面板 ==========
  initGUI({
    box,
    sphere,
    torus,
    cone,
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

    // 为不同几何体添加动画
    box.rotation.x += 0.01
    box.rotation.y += 0.01

    sphere.rotation.y += 0.005

    torus.rotation.x += 0.01
    torus.rotation.y += 0.02

    cone.rotation.y += 0.015

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
