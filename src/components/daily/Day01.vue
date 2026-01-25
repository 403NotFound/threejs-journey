<template>
  <div class="threejs-container">
    <!-- 顶部标题栏 -->
    <header class="top-bar">
      <h1>Day 01 - 环境搭建</h1>
      <span class="date">📅 2025-12-11</span>
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
import { CSS2DRenderer, CSS2DObject } from 'three/examples/jsm/renderers/CSS2DRenderer'
// Canvas 容器引用
const canvasContainer = ref(null)

const initThreeJs = () => {
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

  // 渲染器
  const renderer = new THREE.WebGLRenderer({
    antialias: true, // 抗锯齿
    alpha: true, // 透明背景
  })
  renderer.setSize(window.innerWidth, window.innerHeight)

  // 将渲染器的 canvas 添加到容器中
  canvasContainer.value.appendChild(renderer.domElement)

  // 添加网格辅助线
  const gridHelper = new THREE.GridHelper(30, 30) // 参数：网格大小，网格分段数
  scene.add(gridHelper)

  // 添加坐标轴辅助线
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  // 创建立方体几何体和材质
  const geometry = new THREE.BoxGeometry(1, 1, 1)
  // const basic_material = THREE.MeshBasicMaterial({ color: 0x00ff00 }) 
  const material = new THREE.MeshStandardMaterial({ color: 0x00ff00 }) // 使用 Standard 材质才能接受光照
  const cube = new THREE.Mesh(geometry, material)
  cube.position.x = -1 // 将立方体放在左边
  scene.add(cube)

  // 创建一个基础材质立方体
  const basicGeometry = new THREE.BoxGeometry(1, 1, 1)
  const basicMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
  const basicCube = new THREE.Mesh(basicGeometry, basicMaterial)
  basicCube.position.x = -3 // 将立方体放在左边
  scene.add(basicCube)

  // 创建一个网格漫反射材质立方体
  const lambertMaterial = new THREE.MeshLambertMaterial({ color: 0x00ff00 })
  const lambertCube = new THREE.Mesh(geometry, lambertMaterial)
  lambertCube.position.x = 1 // 将立方体放在左边
  scene.add(lambertCube)

  // 创建一个网格高光材质的立方体
  const phongMaterial = new THREE.MeshPhongMaterial({ color: 0x00ff00 })
  const phongCube = new THREE.Mesh(geometry, phongMaterial)
  phongCube.position.x = 3 // 将立方体放在左边
  scene.add(phongCube)

  // 创建一个物理材质的立方体
  const physicalMaterial = new THREE.MeshPhysicalMaterial({ color: 0x00ff00 })
  const physicalCube = new THREE.Mesh(geometry, physicalMaterial)
  physicalCube.position.x = 5 // 将立方体放在左边
  scene.add(physicalCube)

  // 创建一个点材质立方体
  const pointMaterial = new THREE.PointsMaterial({ color: 0x00ff00 })
  const pointCube = new THREE.Points(geometry, pointMaterial)
  pointCube.position.x = 7 // 将立方体放在左边
  scene.add(pointCube)

  // 创建一个线材质立方体
  const lineMaterial = new THREE.LineBasicMaterial({ color: 0x00ff00 })
  const lineCube = new THREE.Line(geometry, lineMaterial)
  lineCube.position.x = 9 // 将立方体放在左边
  scene.add(lineCube)

  // 创建一个精灵材质立方体
  const spriteMaterial = new THREE.SpriteMaterial({ color: 0x00ff00 })
  const spriteCube = new THREE.Sprite(spriteMaterial)
  spriteCube.position.x = 11 // 将立方体放在左边
  scene.add(spriteCube)

  // 创建一个基础材质圆柱体
  // const cylinderGeometry = new THREE.CylinderGeometry(1, 1, 2)
  // const cylinderMaterial = new THREE.MeshStandardMaterial({ color: 0xff0000 }) // 使用 Standard 材质才能接受光照
  // const cylinder = new THREE.Mesh(cylinderGeometry, cylinderMaterial)
  // cylinder.position.x = 2 // 将圆柱体放在右边
  // scene.add(cylinder)


  // 创建环境光
  // const ambientLight = new THREE.AmbientLight(0xffffff, 1)
  // scene.add(ambientLight)

  // 模拟太阳光
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(2, 2, 2)
  scene.add(directionalLight)

  // 添加轨道控制器
  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true // 启用阻尼效果
  controls.dampingFactor = 0.05 // 设置阻尼系数

  // 创建 CSS2D 渲染器（用于显示文字标签）
  const labelRenderer = new CSS2DRenderer()
  labelRenderer.setSize(window.innerWidth, window.innerHeight)
  labelRenderer.domElement.style.position = 'absolute'
  labelRenderer.domElement.style.top = '0'
  labelRenderer.domElement.style.pointerEvents = 'none' // 标签不响应鼠标事件
  canvasContainer.value.appendChild(labelRenderer.domElement)

  // 创建标签的辅助函数
  function createLabel(text, position) {
    const div = document.createElement('div')
    div.className = 'label'
    div.textContent = text
    div.style.color = '#fff'
    div.style.fontSize = '14px'
    div.style.fontWeight = 'bold'
    div.style.padding = '4px 8px'
    div.style.background = 'rgba(0, 0, 0, 0.6)'
    div.style.borderRadius = '4px'
    div.style.textAlign = 'center'

    const label = new CSS2DObject(div)
    label.position.set(position.x, position.y, position.z)
    return label
  }

  // 为每个立方体添加标签
  const basicLabel = createLabel('Basic Material', { x: 0, y: 1, z: 0 })
  basicCube.add(basicLabel)

  const lambertLabel = createLabel('Lambert Material', { x: 0, y: 1, z: 0 })
  lambertCube.add(lambertLabel)

  const standardLabel = createLabel('Standard Material', { x: 0, y: 1, z: 0 })
  cube.add(standardLabel)

  const phongLabel = createLabel('Phong Material', { x: 0, y: 1, z: 0 })
  phongCube.add(phongLabel)

  const physicalLabel = createLabel('Physical Material', { x: 0, y: 1, z: 0 })
  physicalCube.add(physicalLabel)

  const pointLabel = createLabel('Points Material', { x: 0, y: 1, z: 0 })
  pointCube.add(pointLabel)

  const lineLabel = createLabel('Line Material', { x: 0, y: 1, z: 0 })
  lineCube.add(lineLabel)

  const spriteLabel = createLabel('Sprite Material', { x: 0, y: 1, z: 0 })
  spriteCube.add(spriteLabel)

  // 渲染循环
  const animate = () => {
    requestAnimationFrame(animate)
    // 旋转立方体
    cube.rotation.x += 0.01
    cube.rotation.y += 0.01
    // 旋转基础材质立方体
    basicCube.rotation.x += 0.01
    basicCube.rotation.y += 0.01
    // 旋转网格漫反射材质立方体
    lambertCube.rotation.x += 0.01
    lambertCube.rotation.y += 0.01
    // 旋转网格高光材质立方体
    phongCube.rotation.x += 0.01
    phongCube.rotation.y += 0.01
    // 旋转物理材质立方体
    physicalCube.rotation.x += 0.01
    physicalCube.rotation.y += 0.01
    // 旋转点材质立方体
    pointCube.rotation.x += 0.01
    pointCube.rotation.y += 0.01
    // 旋转线材质立方体
    lineCube.rotation.x += 0.01
    lineCube.rotation.y += 0.01
    // 旋转精灵材质立方体
    spriteCube.rotation.x += 0.01
    spriteCube.rotation.y += 0.01
    controls.update() // 更新控制器
    renderer.render(scene, camera)
    labelRenderer.render(scene, camera) // 渲染标签
  }
  animate()
}
// Day 01: 基础页面搭建
onMounted(() => {
  console.log('Day 01: Three.js 学习页面已加载')
  // TODO: 明天将在这里初始化 Three.js 场景
  initThreeJs()
})

onUnmounted(() => {
  console.log('Day 01: 页面已卸载')
  // TODO: 后续需要在这里清理 Three.js 资源
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
