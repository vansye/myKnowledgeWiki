---
type: 学习笔记
title: Vue3应用入口
created: 2026-07-28
updated: 2026-07-28
tags:
  - Vue3
  - Vite
  - 应用入口
  - main.js
  - 插件注册
subject: JavaWeb
description: 详细讲解 Vue3 应用的创建过程、插件注册机制、挂载原理以及完整的工作流程。
---

> 本笔记以 Tlias 管理系统前端为例，系统讲解 Vue3 应用入口 `main.js` 的完整实现细节。从应用创建到插件注册再到 DOM 挂载，每个环节的原理和实践都深入剖析。

## 目录

- [1. Vue3 应用创建过程](#1-vue3-应用创建过程)
  - [1.1 `createApp()` 原理](#11-createapp原理)
  - [1.2 根组件 App.vue](#12-根组件-appvue)
- [2. 全局插件注册](#2-全局插件注册)
  - [2.1 Element Plus UI 库](#21-element-plus-ui-库)
  - [2.2 Pinia 状态管理](#22-pinia-状态管理)
  - [2.3 Vue Router 路由](#23-vue-router-路由)
- [3. 全局图标注册](#3-全局图标注册)
- [4. 应用挂载](#4-应用挂载)
- [5. 完整工作流](#5-完整工作流)
- [6. 常见问题](#6-常见问题)
- [7. 扩展：自定义插件](#7-扩展-自定义插件)
- [小结](小结)

---

## 1. Vue3 应用创建过程

### 1.1 `createApp()` 原理

`createApp()` 是 Vue3 创建应用实例的核心函数，它返回一个 **应用实例（Application Instance）**，这个实例是 Vue3 应用的根容器，所有的插件注册、全局属性配置、组件注册等都要通过这个实例完成。

```javascript
// Vue3 源码简化版
function createApp(rootComponent) {
  const app = {}  // 应用实例对象
  
  app.component = (name, component) => { ... }  # 注册全局组件
  app.use = (plugin, ...options) => { ... }     # 注册插件
  app.provide = (key, value) => { ... }         # 全局提供
  app.mount = (container) => { ... }            # 挂载应用
  app.unmount = () => { ... }                   # 卸载应用
  
  return app
}
```

**关键点**：
- `createApp()` 创建的应用实例是**单例**的，一个页面只能有一个应用实例
- 应用实例保存了整个 Vue3 应用的上下文，包括组件注册表、全局属性、插件信息等
- 根组件 `App.vue` 是这个应用实例的根节点，所有的子组件都从这里展开

### 1.2 根组件 App.vue

根组件是应用入口的最外层组件，`main.js` 中通过 `createApp(App)` 将根组件包装进应用实例。

```vue
<!-- src/App.vue -->
<template>
  <div id="app">
    <router-view />   <!-- 路由匹配组件在此渲染 -->
  </div>
</template>

<style>
body { margin: 0; font-family: -apple-system, sans-serif; background: #f0f2f5; }
#app { min-height: 100vh; }
</style>
```

**根组件的设计原则**：
- **最小化**：只做必要的事，不包含业务逻辑
- **全局样式**：可在此设置跨页面一致的样式
- **路由出口**：通过 `<router-view>` 显示动态内容

## 2. 全局插件注册

Vue3 的插件系统通过 `app.use(plugin)` 注册，插件可以是对象、函数或安装函数。插件注册后，该插件的功能对应用内的所有组件可用。

### 2.1 Element Plus UI 库

Element Plus 是一个基于 Vue3 的 UI 组件库，注册后可以在任何 `.vue` 文件中使用 `<el-button>`、`<el-table>` 等组件，无需单独导入。

```javascript
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'  # 样式必须引入

app.use(ElementPlus)   # 全局注册
```

**注册后效果**：
- 所有 Element Plus 组件（Button、Table、Dialog 等）自动可用
- 全局样式注入（全局的 CSS 样式）
- 配置项：可通过 `app.config` 设置全局属性（如 size、theme 等）

### 2.2 Pinia 状态管理

Pinia 是 Vue3 官方推荐的状态管理库，比 Vuex 更轻量、TypeScript 支持更好。

```javascript
import { createPinia } from 'pinia'
const pinia = createPinia()   # 创建 Pinia 实例
app.use(pinia)                # 注册到应用
```

**工作原理**：
1. `createPinia()` 创建 Pinia 实例，内部创建 Vuex store 的容器
2. `app.use(pinia)` 将 Pinia 注入到 Vue 应用，使所有组件可通过 `inject('$pinia')` 访问
3. 在组件中通过 `useUserStore()` 获取具体的 store 实例

```vue
<script setup>
import { useUserStore } from '@/stores/user'
const userStore = useUserStore()  # 直接使用，无需 new
</script>
```

### 2.3 Vue Router 路由

Vue Router 是 Vue 官方路由库，实现单页面应用的路由管理。

```javascript
import router from './router'  # 从 router/index.js 导入
app.use(router)               # 注册路由
```

**注入效果**：
- 所有组件可用 `$router` 对象（编程式导航）
- 所有组件可用 `$route` 对象（当前路由信息）
- `<router-link>` 和 `<router-view>` 组件自动可用

## 3. 全局图标注册

Element Plus 提供了大量图标组件，项目通过自动注册所有图标，让在任何组件中直接使用 `<el-icon><Odometer /></el-icon>`，无需单独导入每个图标。

```javascript
import * as ElementPlusIconsVue from '@element-plus/icons-vue'

# 遍历所有图标组件，批量注册
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
  app.component(key, component)   # 注册为全局组件
}
```

**注册原理**：
- `Object.entries(ElementPlusIconsVue)` 获取所有图标组件（如 Odometer、OfficeBuilding、User 等）
- `app.component(key, component)` 将每个图标注册为全局组件
- 注册后，任何 `.vue` 文件中直接 `<el-icon><Odometer /></el-icon>` 即可使用

## 4. 应用挂载

`app.mount('#app')` 是将 Vue 应用挂载到 DOM 上的最后一步，也是应用启动的关键时刻。

```javascript
app.mount('#app')   # 挂载到 id="app" 的 DOM 元素
```

**挂载过程**：
1. Vue 创建根组件的虚拟 DOM 树
2. 将虚拟 DOM 渲染到真实的 DOM 节点（#app）
3. 启动响应式系统，开始监听数据变化
4. 应用启动完成，用户可见

**挂载要求**：
- `#app` 元素在 HTML 中必须存在（通常在 index.html 中）
- 挂载前必须完成所有插件注册（否则部分功能不可用）
- 同一个应用实例只能 mount 一次

## 5. 完整工作流

```
index.html
  ↓ (Vite 解析 script 标签)
main.js (入口文件)
  ├── createApp(App)           # 创建应用实例，根组件为 App.vue
  ├── app.use(ElementPlus)     # 注册 UI 库
  ├── app.use(createPinia())   # 注册状态管理
  ├── app.use(router)          # 注册路由
  ├── 注册全局图标组件         # 批量注册图标
  └── app.mount('#app)         # 挂载到 DOM，应用启动
           ↓
  #app 元素内渲染 <router-view>
           ↓
  用户访问 /login → 路由匹配 Login 组件 → 渲染到 <router-view>
           ↓
  用户访问 /dashboard → 路由匹配 Layout → Layout 内部再匹配 Dashboard
```

**详细流程**：
1. Vite 启动开发服务器，加载 index.html
2. Vite 解析 `<script type="module" src="/src/main.js">`，加载 main.js
3. main.js 执行，创建 Vue 应用实例并挂载到 #app
4. 应用启动后，Vue Router 监听 URL 变化
5. 用户访问 /login，router 匹配到 Login 组件，渲染到 App.vue 的 <router-view>
6. 用户点击菜单切换到 /dashboard，router 匹配到 Layout 组件（作为父路由）
7. Layout 组件内部又有 <router-view>，匹配到 Dashboard 子组件，渲染在 Layout 内部

## 6. 常见问题

### Q1: 页面空白，控制台无错误？
**可能原因**：忘记调用 `app.mount('#app')` 或 mount 的 ID 不存在。检查 index.html 中是否有 `<div id="app"></div>`。

### Q2: 组件无法使用 Element Plus 组件？
**可能原因**：忘记在 main.js 中 `app.use(ElementPlus)` 或未引入 CSS 文件。

### Q3: Store 无法访问？
**可能原因**：忘记注册 Pinia 或导入错误的 store 路径。确保 `app.use(createPinia())` 已执行。

### Q4: 路由跳转无效？
**可能原因**：忘记 `app.use(router)` 或使用 `router.push()` 前未导入 router 实例。

## 7. 扩展：自定义插件

除了官方插件，你也可以创建自己的 Vue 插件。插件必须是一个 install 函数，或者包含 install 方法的对象。

```javascript
# 自定义插件：设置全局属性
const MyPlugin = {
  install(app, options) {
    # 注册全局方法
    app.config.globalProperties.$myMethod = function() { ... }
    
    # 注册全局组件
    app.component('MyComponent', MyComponent)
    
    # 注入 provide/app.config
    app.provide('myKey', options.value)
  }
}

# 使用
app.use(MyPlugin, { value: 123 })
```

在全局属性中定义的 `$myMethod` 可在所有组件中通过 `this.$myMethod()` 访问。

---

## 小结

`main.js` 是 Vue3 应用的启动入口，核心流程为：

1. `createApp(App)` 创建应用实例，以 App.vue 为根组件
2. `app.use(ElementPlus)` 注册 UI 组件库，使 `<el-xxx>` 可用
3. `app.use(createPinia())` 注册状态管理，提供全局 store
4. `app.use(router)` 注册路由，提供导航和视图切换
5. 批量注册图标组件，全局可用 `<el-icon>`
6. `app.mount('#app')` 挂载应用，启动响应式系统

理解每个环节的原理，有助于排查前端应用启动时的问题，也能更好地设计自定义插件和全局功能。

关联知识点：
- [[学习/JavaWeb/知识条目/前端/App.vue根组件]] 根组件设计
- [[学习/JavaWeb/知识条目/前端/Vue Router路由配置]] 路由原理
- [[学习/JavaWeb/知识条目/前端/Pinia状态管理]] 状态管理实现
- [[学习/JavaWeb/知识条目/前端/API接口封装]] 前后端交互
- [[学习/JavaWeb/知识条目/前端/script setup语法]] 组件语法