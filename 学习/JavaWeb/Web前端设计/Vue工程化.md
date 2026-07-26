---
type: 学习笔记
title: Vue 工程化
created: 2026-07-25
updated: 2026-07-25
tags: [Vue, 工程化, Vite, 前端]
subject: JavaWeb
---

> Vue 3 生态的标准工程化方案：Vite + Vue Router + Pinia + 组件化，涵盖项目结构、路由、状态管理、前后端交互、构建部署等一整套开发流程。
>
> 前置知识：[[Vue]]（框架基础）、[[Vue 模板]]（模板语法）、[[API风格]]（Options vs Composition）、[[响应式对象]]（reactive / ref）

## 目录

- [1. 脚手架：Vite](#1-脚手架vite)
- [2. 组件化开发](#2-组件化开发)
- [3. 路由：Vue Router](#3-路由vue-router)
- [4. 状态管理：Pinia](#4-状态管理pinia)
- [5. 前后端交互](#5-前后端交互)
- [6. 环境变量](#6-环境变量)
- [7. 构建与部署](#7-构建与部署)
- [小结](#小结)

---

## 1. 脚手架：Vite

Vite 是 Vue 官方推荐的构建工具，替代了 Vue CLI。它基于原生 ES Module，开发时秒级启动，生产构建用 Rollup 打包。

**创建项目：**

```bash
npm create vue@latest          # 官方脚手架，交互式选择
# 或
npm create vite@latest my-app -- --template vue
```

**项目目录结构（标准 Vue 3 项目）：**

```
my-vue-app/
├── index.html                 # 入口 HTML
├── package.json               # 项目依赖
├── vite.config.js             # Vite 配置
├── public/                    # 不打包的静态资源
├── src/
│   ├── main.js                # 应用入口，创建 Vue 实例
│   ├── App.vue                # 根组件
│   ├── assets/                # 静态资源（图片/CSS 等，会被打包）
│   ├── components/            # 可复用的通用组件
│   ├── views/                 # 页面级组件（每个路由对应一个页面）
│   ├── router/                # 路由配置
│   │   └── index.js
│   ├── stores/                # Pinia 状态管理
│   │   └── counter.js
│   └── api/                   # 封装后端接口请求
│       └── user.js
```

## 2. 组件化开发

**单文件组件（.vue）：**

每个 `.vue` 文件包含三部分：

```vue
<script setup>
// 逻辑：数据、方法、生命周期
import { ref, onMounted } from 'vue'
const count = ref(0)
</script>

<template>
  <!-- 模板：HTML 结构 -->
  <button @click="count++">{{ count }}</button>
</template>

<style scoped>
/* 样式：scoped 表示只作用于当前组件 */
button { color: blue; }
</style>
```

**组件通信：**

| 方式 | 方向 | 场景 |
|------|------|------|
| `props` | 父 → 子 | 父组件传数据给子组件 |
| `emits` | 子 → 父 | 子组件通知父组件发生了某事 |
| `provide / inject` | 祖 → 孙 | 跨多层传递数据 |
| Pinia | 全局 | 任意组件间共享状态 |
| 路由参数 | 页面间 | 页面跳转时传递数据 |

## 3. 路由：Vue Router

Vue Router 是 Vue 官方路由库，实现单页面应用（SPA）的页面切换。URL 变化时不刷新整个页面，只替换对应的组件。

**基本配置：**

```javascript
// src/router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  { path: '/',          component: () => import('@/views/Home.vue') },
  { path: '/about',     component: () => import('@/views/About.vue') },
  // 动态路由：路径参数
  { path: '/user/:id',  component: () => import('@/views/User.vue') },
]

const router = createRouter({
  history: createWebHistory(),  // HTML5 History 模式
  routes,
})

export default router
```

**路由导航：**

```vue
<!-- 声明式 -->
<router-link to="/about">关于</router-link>

<!-- 编程式 -->
<script setup>
import { useRouter } from 'vue-router'
const router = useRouter()
router.push('/about')
router.push({ name: 'user', params: { id: 1 } })
</script>

<!-- 路由出口：匹配的组件渲染在这里 -->
<router-view />
```

## 4. 状态管理：Pinia

Pinia 是 Vue 官方状态管理库（替代 Vuex），用于管理跨组件共享的数据。比 Vuex 更简洁、TypeScript 支持更好。

**基本使用：**

```javascript
// src/stores/counter.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  // state
  const count = ref(0)

  // getter
  const doubleCount = computed(() => count.value * 2)

  // action
  function increment() {
    count.value++
  }

  return { count, doubleCount, increment }
})
```

```vue
<!-- 在组件中使用 -->
<script setup>
import { useCounterStore } from '@/stores/counter'
const store = useCounterStore()
// store.count      — 直接读取
// store.increment() — 调用方法（修改状态）
</script>
```

## 5. 前后端交互

```javascript
// src/api/request.js — 封装 axios 实例
import axios from 'axios'

const request = axios.create({
  baseURL: 'http://localhost:8080/api',
  timeout: 5000,
})

// 请求拦截器：统一添加 token
request.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

// 响应拦截器：统一处理错误
request.interceptors.response.use(
  res => res.data,
  err => {
    console.error('请求失败:', err)
    return Promise.reject(err)
  }
)

export default request
```

后端对应的接口规范见 [[Controller请求与响应]]，RESTful 设计见 [[Restful规范]]。

## 6. 环境变量

```
.env                # 所有环境都加载
.env.development    # 开发环境（npm run dev）
.env.production     # 生产环境（npm run build）
```

```bash
# .env.development
VITE_API_BASE_URL=http://localhost:8080/api
```

```javascript
// 代码中使用
const baseURL = import.meta.env.VITE_API_BASE_URL
```

## 7. 构建与部署

```bash
npm run dev       # 启动开发服务器（热更新）
npm run build     # 构建生产包 → dist/
npm run preview   # 本地预览生产构建
```

`dist/` 目录是最终产物，部署到 Nginx 或其他静态服务器即可。

---

## 小结

Vue 工程化的核心是**脚手架（Vite）+ 路由（Vue Router）+ 状态管理（Pinia）+ 组件化**四件套。掌握了项目结构、组件通信、路由配置和接口封装，就能搭建标准化的 Vue 3 前端项目，与后端 API 对接形成完整的前后端分离应用。
