<template>
  <div class="practice-container">
    <div class="header">
      <h2>Day 09 专项训练：Shader 着色器入门</h2>
      <div class="controls">
        <button @click="currentMode = 'A'" :class="{ active: currentMode === 'A' }">实验 J: 基础着色器结构</button>
      </div>
    </div>
    <div ref="canvasContainer" class="canvas-wrapper"></div>

    <div class="info-panel">
      <div v-if="currentMode === 'A'">
        <h3>实验 J：ShaderMaterial 基础</h3>
        <p>🔴 红色脉动效果</p>
        <p>关键：通过 Uniform 将 `uTime` 传给 GPU。</p>
        <p>语言：GLSL (OpenGL Shading Language)</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const canvasContainer = ref(null)
const currentMode = ref('A')

let scene, camera, renderer, controls, rafId, clock
let shaderMesh

// --- Shader 代码部分 ---

// 顶点着色器：决定位置
const vertexShader = `
  varying vec2 vUv;
  void main() {
    vUv = uv; // 将 UV 坐标传给片元着色器
    // 标准的模型视图投影变换
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
  }
`

const vertexShader2 = `
  varying vec2 vUv;
  varying float vElevation; // 新增：用来传递高度
  uniform float uTime;

  void main() {
    vUv = uv;
    vec3 newPosition = position;

    // 算出高度
    float elevation = sin(newPosition.x * 10.0 + uTime) * 0.1;
    newPosition.z += elevation;
    
    // 把高度传出去
    vElevation = elevation; 

    gl_Position = projectionMatrix * modelViewMatrix * vec4(newPosition, 1.0);
  }
`

const vertexShader3 = `
  varying vec2 vUv;
  uniform float uTime; // 别忘了在这里也要声明 uTime

  void main() {
    vUv = uv;

    // 1. 拷贝原始位置
    vec3 newPosition = position;

    // 2. 让 z 轴根据 x 坐标和时间产生正弦波动
    // sin(x * 10.0) 决定波浪的密集程度
    // + uTime 决定波浪移动的速度
    // * 0.1 决定波浪的高度
    newPosition.z += sin(newPosition.x * 10.0 + uTime) * 0.1;

    // 3. 使用新的位置进行投影
    gl_Position = projectionMatrix * modelViewMatrix * vec4(newPosition, 1.0);
  }
`

// 片元着色器：决定颜色
const fragmentShader2 = `
  uniform float uTime; // 从 JS 传进来的时间
  varying vec2 vUv;     // 从顶点着色器传过来的 UV

  void main() {
    // 使用 sin 函数随时间改变红色分量
    float strength = step(0.5, vUv.x);
    gl_FragColor = vec4(vUv.x, vUv.y, vUv.x, 1.0); // RGBA
  }
`

const fragmentShader3 = `
  varying vec2 vUv;
  varying float vElevation; // 接收来自顶点的能量值

  void main() {
    // 根据高度进行颜色偏移 (高度范围约 -0.1 到 0.1)
    // 我们把它映射到更明显的范围
    float colorStrength = vElevation * 2.0 + 0.5; 
    
    gl_FragColor = vec4(colorStrength, colorStrength * 0.5, 1.0, 1.0); 
  }
`

const fragmentShader = `
  varying vec2 vUv;
uniform float uTime;
    void main() {
    float dist = distance(vUv, vec2(0.5));

    // smoothstep(起始点, 结束点, 当前值)
    // 它在 0.3 到 0.6 之间产生平滑的 0 到 1 的过渡
    float radius = 0.3 + sin(uTime * 3.0) * 0.1; // 动态半径 
    // 用 sin 结合距离，创造出从中心扩散的涟漪效果
    float strength = abs(sin(dist * 20.0 - uTime * 5.0));
    
    // 给光圈一点对比度（让暗处更暗）
    strength = pow(0.1 / strength, 1.2); 
    float density = sin(dist * 20.0);
gl_FragColor = vec4(vec3(density), 1.0);

    //gl_FragColor = vec4(vec3(dist), 1.0);
  }
`

const initBase = () => {
  if (renderer) {
    renderer.dispose()
    canvasContainer.value.innerHTML = ''
  }
  if (rafId) cancelAnimationFrame(rafId)
  clock = new THREE.Clock()

  scene = new THREE.Scene()
  scene.background = new THREE.Color('#050505')

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100)
  camera.position.set(0, 0, 2)

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  canvasContainer.value.appendChild(renderer.domElement)

  controls = new OrbitControls(camera, renderer.domElement)
}

const startModeA = () => {
  initBase()

  // 1. 定义几何体
  const geometry = new THREE.PlaneGeometry(1, 1, 32, 32)

  // 2. 定义 Shader 材质
  const material = new THREE.ShaderMaterial({
    vertexShader: vertexShader,
    fragmentShader: fragmentShader,
    uniforms: {
      uTime: { value: 0 } // 初始时间
    }
  })

  shaderMesh = new THREE.Mesh(geometry, material)
  scene.add(shaderMesh)

  const animate = () => {
    rafId = requestAnimationFrame(animate)

    // 关键：每一帧更新 Uniform 中的时间
    const elapsedTime = clock.getElapsedTime()
    material.uniforms.uTime.value = elapsedTime

    controls.update()
    renderer.render(scene, camera)
  }
  animate()
}

onMounted(() => {
  startModeA()
})

onUnmounted(() => {
  if (rafId) cancelAnimationFrame(rafId)
  if (renderer) renderer.dispose()
})
</script>

<style scoped>
.practice-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  background: #000;
  overflow: hidden;
}

.header {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  padding: 20px;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  color: white;
  display: flex;
  justify-content: space-between;
  align-items: center;
  z-index: 10;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.controls button {
  margin-left: 10px;
  padding: 8px 20px;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #fff;
  cursor: pointer;
  border-radius: 6px;
}

.controls button.active {
  background: #6366f1;
  color: white;
  border-color: #818cf8;
}

.info-panel {
  position: absolute;
  top: 100px;
  left: 20px;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(10px);
  color: white;
  padding: 20px;
  border-radius: 12px;
  max-width: 320px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

h3 {
  margin-top: 0;
  color: #818cf8;
}

p {
  color: #ccc;
  font-size: 0.9rem;
}
</style>
