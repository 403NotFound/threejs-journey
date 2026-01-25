<template>
  <div class="threejs-container">
    <!-- 顶部标题栏 -->
    <header class="top-bar">
      <h1>Day XX - [今天的主题]</h1>
      <span class="date">📅 YYYY-MM-DD</span>
    </header>

    <!-- Three.js 画布容器 -->
    <div ref="canvasContainer" class="canvas-wrapper"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
// import * as THREE from 'three'

// Canvas 容器引用
const canvasContainer = ref(null)

// Three.js 相关变量
// let scene, camera, renderer, mesh

// Day XX: [今天学习的内容描述]
onMounted(() => {
  console.log('Day XX: 开始学习')

  // TODO: 初始化 Three.js 场景
  // initThreeJS()
  // animate()
})

onUnmounted(() => {
  console.log('Day XX: 清理资源')

  // TODO: 清理 Three.js 资源
  // if (renderer) {
  //   renderer.dispose()
  // }
})
// 初始化 Three.js
// function initThreeJS() {
//   // 创建场景
//   scene = new THREE.Scene()
//   
//   // 创建相机
//   camera = new THREE.PerspectiveCamera(
//     75,
//     window.innerWidth / window.innerHeight,
//     0.1,
//     1000
//   )
//   camera.position.z = 5
//   
//   // 创建渲染器
//   renderer = new THREE.WebGLRenderer({ antialias: true })
//   renderer.setSize(window.innerWidth, window.innerHeight)
//   canvasContainer.value.appendChild(renderer.domElement)
//   
//   // TODO: 添加你的 3D 对象
// }

// 动画循环
// function animate() {
//   requestAnimationFrame(animate)
//   
//   // TODO: 更新动画
//   
//   renderer.render(scene, camera)
// }
</script>

<style scoped>
.threejs-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  background: #000;
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
