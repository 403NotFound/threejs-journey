<template>
  <div class="threejs-container">
    <div ref="canvasContainer" class="canvas-wrapper"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// Canvas 容器引用
const canvasContainer = ref(null)

// 在这里可以添加 Three.js 相关的代码
onMounted(() => {
  console.log('Three.js 学习页面已加载')
  // TODO: 初始化 Three.js 场景
})

onUnmounted(() => {
  console.log('Three.js 学习页面已卸载')
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

.info-panel {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(255, 255, 255, 0.95);
  padding: 3rem;
  border-radius: 20px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  text-align: center;
  backdrop-filter: blur(10px);
  z-index: 10;
}

.info-panel h1 {
  font-size: 2.5rem;
  margin: 0 0 1rem 0;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 700;
}

.info-panel p {
  font-size: 1.2rem;
  color: #555;
  margin: 0.5rem 0;
  line-height: 1.6;
}
</style>
