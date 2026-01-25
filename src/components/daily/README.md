# 每日学习组件

这个文件夹包含每天的 Three.js 学习组件，每个组件都是独立的，保留了当天的学习代码。

## 📁 文件结构

```
daily/
├── README.md              # 本文件
├── DayXX-template.vue     # 每日组件模板
├── Day01.vue              # 第 1 天：环境搭建
├── Day02.vue              # 第 2 天：第一个 3D 场景
├── Day03.vue              # 第 3 天：几何体和材质
└── ...
```

## 🎯 使用方法

### 创建新的一天的组件

1. **复制模板文件**
   ```bash
   cp src/components/daily/DayXX-template.vue src/components/daily/Day02.vue
   ```

2. **修改组件内容**
   - 更新模板中的日期和主题
   - 复制上一天的代码作为起点（如果需要）
   - 添加今天的新功能

3. **更新 App.vue**
   ```vue
   <script setup>
   import Day02 from './components/daily/Day02.vue'
   </script>

   <template>
     <Day02 />
   </template>
   ```

4. **更新学习记录**
   - 在 `learning-notes/` 中创建对应的 `day-02.md`
   - 记录今天学到的知识点和代码

## 📋 组件列表

| 组件 | 日期 | 主题 | 状态 |
|------|------|------|------|
| Day01.vue | 2025-12-11 | 环境搭建 | ✅ 完成 |
| Day02.vue | - | 第一个 3D 场景 | ⏳ 待创建 |
| Day03.vue | - | 几何体和材质 | ⏳ 待创建 |
| Day04.vue | - | 光照系统 | ⏳ 待创建 |
| Day05.vue | - | 动画和交互 | ⏳ 待创建 |

## 💡 最佳实践

### 1. 继承上一天的代码
如果今天的学习是基于昨天的代码，可以这样做：
```bash
# 复制上一天的组件作为起点
cp src/components/daily/Day01.vue src/components/daily/Day02.vue
# 然后在 Day02.vue 基础上添加新功能
```

### 2. 保持组件独立
每个组件应该是完整的、可独立运行的，这样方便：
- 回顾某一天的学习成果
- 对比不同天的代码差异
- 快速切换到任意一天的状态

### 3. 添加注释
在关键代码处添加注释，说明：
- 这段代码的作用
- 为什么这样写
- 遇到的问题和解决方案

### 4. 使用调试面板
模板中包含了一个可选的调试面板，可以显示：
- 当前是第几天
- 今天的学习主题
- 实时的参数信息

## 🔄 切换不同天的组件

在 `App.vue` 中切换组件即可查看不同天的学习成果：

```vue
<script setup>
// 切换这里的导入来查看不同天的代码
import Day01 from './components/daily/Day01.vue'
// import Day02 from './components/daily/Day02.vue'
// import Day03 from './components/daily/Day03.vue'
</script>

<template>
  <Day01 />
</template>
```

## 📝 命名规范

- 组件文件名：`DayXX.vue`（两位数字，如 `Day01.vue`, `Day15.vue`）
- 组件内部名称：与文件名保持一致
- 对应的学习笔记：`learning-notes/day-XX.md`

## 🎨 样式建议

- 每天可以使用不同的背景色或主题，方便区分
- 保持基础样式的一致性，只在必要时修改
- 使用 scoped 样式避免污染全局

## 🚀 快速开始下一天

```bash
# 1. 复制模板或上一天的组件
cp src/components/daily/Day01.vue src/components/daily/Day02.vue

# 2. 复制学习笔记模板
cp learning-notes/template.md learning-notes/day-02.md

# 3. 开始编码和学习！
```
