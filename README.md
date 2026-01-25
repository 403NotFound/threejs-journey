# Three.js Journey - 进阶开发实践

🚀 **这是一个基于 [Three.js Journey](https://threejs-journey.com/) 课程体系，结合个人核心逻辑理解与工程化实践的 3D 学习项目。**

本项目旨在通过系统性的实验（Daily Practice），深入探索 WebGL 底层数学、Shader 编程、高性能粒子系统以及 3D 场景渲染技术。

## 🛠️ 技术栈
- **核心**: Three.js (WebGL)
- **前端架构**: Vue 3 + Vite
- **UI 调试**: lil-gui (实时参数调优)
- **数学库**: 原生 JavaScript Math / Three.js Math Classes

## 🌌 已实现的阶段性成果 (Daily Practice)

### [Day 10] - 星系生成器 (Galaxy Generator) 🧪 *Current Focus*
- **核心逻辑**: 基于极坐标系的程序化几何生成。
- **技术点**: 
  - 数学控制: 通过半径、自旋 (Spin)、旋臂分支 (Branches) 公式构建星系骨架。
  - 颜色插值: 实现从星系中心 (Center Color) 到边缘 (Outside Color) 的径向颜色平滑过渡。
  - 散布算法: 利用 `Math.pow` 指数分布模拟自然星系“中间稠密、边缘稀疏”的质感。

### [Day 09] - Shader 着色器硬核入门
- **Vertex Shader**: 实现正弦波形位移，模拟动态海浪。
- **Fragment Shader**: 
  - 深入 `vUv` 坐标系与 `distance` 公式。
  - 数学画圆: 使用 `step` 与 `smoothstep` 控制形体边缘。
  - 科幻效果: 结合 `sin` 与 `uTime` 实现动态扩散的能量环特效。

### [Day 08] - 粒子系统 (Particle Systems)
- **底层架构**: 使用 `BufferGeometry` 手动管理数万个顶点坐标。
- **应用场景**: 场景星空背景、幽灵火点阵、模型粒子化变换。
- **性能优化**: 理解 `needsUpdate` 机制与 GPU 顶点缓存同步。

### [Day 07] - 3D 核心交互与数学控制
- **Raycaster**: 实现精准的鼠标射线检测、物体拾取与地面移动控制。
- **Vector Math**: 熟练掌握三维向量运算、物体平滑移动与 `lookAt` 方向控制。
- **层级架构**: 深入理解 World Space 与 Local Space 的转换关系。

## 🎨 视觉美学
项目中强调“视觉优先”的开发理念：
- **混合模式**: 大量使用 `AdditiveBlending` 实现发光效果。
- **材质系统**: 深度实践 `ShaderMaterial`、`PointsMaterial` 与标准物理材质。
- **实时调试**: 每一个核心参数（颜色、密度、旋转速度）均接入 GUI 实时面板，方便在开发过程中寻找最佳视觉效果。

## 🔨 开发预览
```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 编译打包
npm run build
```

---
*持续学习中... 目标是完成电影级的网页 3D 交互表现。*
