<template>
  <div class="threejs-container">
    <!-- 顶部标题栏 -->
    <header class="top-bar">
      <h1>Day 02 - 深入学习</h1>
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
import Stats from 'three/examples/jsm/libs/stats.module'
import GUI from 'lil-gui'
// Canvas 容器引用
const canvasContainer = ref(null)

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

  // 将渲染器的 canvas 添加到容器中
  canvasContainer.value.appendChild(renderer.domElement)

  // 添加网格辅助线
  const gridHelper = new THREE.GridHelper(30, 30) // 参数：网格大小，网格分段数
  scene.add(gridHelper)

  // 添加坐标轴辅助线
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  // 创建立方体
  const geometry = new THREE.BoxGeometry(1, 1, 1);

  // 使用 Standard 材质支持金属度
  const material = new THREE.MeshStandardMaterial({
    color: 0xff0000,
    metalness: 1,    // 金属度 (0-1)，0=非金属，1=完全金属
    roughness: 0.5,    // 粗糙度 (0-1)，0=光滑镜面，1=完全粗糙
  });
  for (let i = 0; i < 10; i++) {
    const cube = new THREE.Mesh(geometry, material);
    // 沿着 x 轴分布，间距 2 个单位，居中显示
    cube.position.set(i * 2 - 9, 0, 0);  // -9 让阵列居中
    scene.add(cube);
  }

  // 创建 GUI 调试面板
  const gui = new GUI()
  gui.title('材质调试面板')

  // 添加材质参数控制
  const materialFolder = gui.addFolder('材质参数')
  materialFolder.add(material, 'metalness', 0, 1, 0.01).name('金属度')
  materialFolder.add(material, 'roughness', 0, 1, 0.01).name('粗糙度')
  materialFolder.addColor({ color: 0xff0000 }, 'color').name('颜色').onChange((value) => {
    material.color.set(value)
  })
  materialFolder.open()  // 默认展开

  // ========== 纹理贴图示例 ==========

  // 创建纹理加载器
  const textureLoader = new THREE.TextureLoader()

  // 方法 1: 使用纯色生成的 Canvas 纹理（不需要图片文件）
  const canvas = document.createElement('canvas')
  canvas.width = 256
  canvas.height = 256
  const ctx = canvas.getContext('2d')

  // 绘制棋盘格纹理
  const tileSize = 32
  for (let i = 0; i < 8; i++) {
    for (let j = 0; j < 8; j++) {
      ctx.fillStyle = (i + j) % 2 === 0 ? '#ffffff' : '#000000'
      ctx.fillRect(i * tileSize, j * tileSize, tileSize, tileSize)
    }
  }

  const canvasTexture = new THREE.CanvasTexture(canvas)

  // 创建使用纹理的立方体
  const texturedGeometry = new THREE.BoxGeometry(2, 2, 2)
  const texturedMaterial = new THREE.MeshStandardMaterial({
    map: canvasTexture,  // 颜色贴图
    metalness: 0.3,
    roughness: 0.7,
  })

  const texturedCube = new THREE.Mesh(texturedGeometry, texturedMaterial)
  texturedCube.position.set(0, 3, 0)  // 放在上方
  scene.add(texturedCube)

  // 添加纹理立方体的旋转
  const texturedCubeRotation = { x: 0.005, y: 0.01 }

  // 添加纹理控制到 GUI
  const textureFolder = gui.addFolder('纹理立方体')
  textureFolder.add(texturedCube.position, 'x', -10, 10, 0.1).name('X 位置')
  textureFolder.add(texturedCube.position, 'y', -10, 10, 0.1).name('Y 位置')
  textureFolder.add(texturedCube.position, 'z', -10, 10, 0.1).name('Z 位置')
  textureFolder.add(texturedCubeRotation, 'x', 0, 0.1, 0.001).name('X 旋转速度')
  textureFolder.add(texturedCubeRotation, 'y', 0, 0.1, 0.001).name('Y 旋转速度')

  // 方法 2: 从 URL 加载纹理（示例代码，需要实际图片）
  // const texture = textureLoader.load('/textures/brick.jpg')
  // texture.wrapS = THREE.RepeatWrapping  // 水平重复
  // texture.wrapT = THREE.RepeatWrapping  // 垂直重复
  // texture.repeat.set(2, 2)  // 重复次数

  // const material2 = new THREE.MeshStandardMaterial({
  //   map: texture,              // 颜色贴图
  //   normalMap: normalTexture,  // 法线贴图（增加细节）
  //   roughnessMap: roughTexture,// 粗糙度贴图
  //   metalnessMap: metalTexture,// 金属度贴图
  // })



  // 添加光照
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(2, 2, 2)
  scene.add(directionalLight)

  const pointLight = new THREE.PointLight(0xff0000, 1, 10); // (颜色, 强度, 范围)
  pointLight.position.set(5, 5, 5);
  scene.add(pointLight);

  // 光源辅助观察
  const pointLightHelper = new THREE.PointLightHelper(pointLight, 0.5); // (光源, 辅助线长度)
  scene.add(pointLightHelper);

  // GUI 添加光源参数控制
  const lightFolder = gui.addFolder('光源参数')
  lightFolder.add(pointLight, 'intensity', 0, 10, 0.01).name('强度')
  lightFolder.add(pointLight.position, 'x', -10, 10, 0.01).name('X')
  lightFolder.add(pointLight.position, 'y', -10, 10, 0.01).name('Y')
  lightFolder.add(pointLight.position, 'z', -10, 10, 0.01).name('Z')


  // 添加轨道控制器
  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05

  // 渲染循环
  const animate = () => {
    requestAnimationFrame(animate)

    // 旋转立方体
    // cube.rotation.x += 0.01
    // cube.rotation.y += 0.01

    // 旋转纹理立方体
    texturedCube.rotation.x += texturedCubeRotation.x
    texturedCube.rotation.y += texturedCubeRotation.y

    stats.update()

    controls.update()
    renderer.render(scene, camera)
  }
  animate()
}

// Day 02: 深入学习
onMounted(() => {
  console.log('Day 02: Three.js 学习页面已加载')
  initThreeJs()
})

onUnmounted(() => {
  console.log('Day 02: 页面已卸载')
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
