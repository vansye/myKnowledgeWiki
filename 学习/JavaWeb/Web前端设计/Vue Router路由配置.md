---
type: 学习笔记
title: Vue Router路由配置
created: 2026-07-28
updated: 2026-07-28
tags:
  - Vue Router
  - 路由配置
  - 懒加载
  - 嵌套路由
  - 路由守卫
subject: JavaWeb
description: 详细讲解 Vue Router 的配置方法，包括懒加载、嵌套路由、路由守卫等核心特性的实现原理和使用技巧。
---

> 本笔记以 Tlias 管理系统前端为例，系统讲解 Vue Router 4 的配置方式和工作原理。从路由定义到懒加载、嵌套路由、权限守卫，每个细节都深入剖析。

## 目录

- [1. Vue Router 基础概念](#1-vue-router基础概念)
  - [1.1 SPA 路由](#11-spa-路由)
  - [1.2 路由模式](#12-路由模式)
- [2. 路由配置](#2-路由配置)
  - [2.1 route 对象结构](#21-route-对象结构)
  - [2.2 路由定义示例](#22-路由定义示例)
- [3. 懒加载（代码分割）](#3-懒加载代码分割)
  - [3.1 实现原理](#31-实现原理)
  - [3.2 性能优势](#32-性能优势)
- [4. 嵌套路由（布局架构）](#4-嵌套路由布局架构)
  - [4.1 父路由与子路由](#41-父路由与子路由)
  - [4.2 嵌套路由的工作原理](#42-嵌套路由的工作原理)
  - [4.3 Layout 布局组件](#43-layout-布局组件)
- [5. 路由守卫](#5-路由守卫)
  - [5.1 beforeEach 守卫](#51-beforeeach-守卫)
  - [5.2 权限验证实现](#52-权限验证实现)
- [6. 路由元信息 (meta)](#6-路由元信息-meta)
- [7. 编程式导航](#7-编程式导航)
- [8. 完整路由配置](#8-完整路由配置)
- [小结](小结)

---

## 1. Vue Router 基础概念

### 1.1 SPA 路由

SPA（Single Page Application，单页面应用）的核心特性是：**页面切换不刷新整个页面，只替换部分内容**。Vue Router 是实现 SPA 路由的核心库。

**传统多页应用**：
```
点击"用户管理" → 浏览器请求 /user.html → 服务器返回完整 HTML → 页面刷新
```

**SPA 模式**：
```
点击"用户管理" → Vue Router 更新 URL → 渲染对应的组件 → 不刷新整个页面
```

**优势**：
- ✅ 用户体验流畅（无白屏、无闪烁）
- ✅ 前端缓存数据（无需重新请求）
- ✅ 前后端分离（后端只返回数据，不返回 HTML）

### 1.2 路由模式

Vue Router 提供两种路由模式：

| 模式 | 函数 | URL 特点 | 适用场景 |
|------|------|----------|----------|
| **Hash 模式** | `createWebHashHistory()` | `domain.com/#/login` | ✅ 兼容老浏览器 ❌ URL 带 `#` |
| **History 模式** | `createWebHistory()` | `domain.com/login` | ✅ 干净 URL ❌ 需后端配置 |

**本项目采用 Hash 模式**，因为：
- 部署到 Spring Boot 静态资源目录时，不需要后端配置路由转发
- 兼容所有浏览器（包括 IE11）
- URL 中 `#` 后面的部分不会发送到服务器，服务器只需返回 `index.html` 即可

---

## 2. 路由配置

`src/router/index.js` 是路由配置的核心文件，定义路由映射关系和路由器实例。

### 2.1 route 对象结构

每个路由对象包含以下关键属性：

| 属性 | 类型 | 说明 |
|------|------|------|
| `path` | string | URL 路径（如 `/login`） |
| `name` | string | 路由名称（用于编程式跳转） |
| `component` | function | 组件（必须用 `() => import()` 懒加载） |
| `children` | array | 子路由（嵌套路由） |
| `meta` | object | 元信息（如页面标题、权限标记） |
| `redirect` | string | 重定向目标 |

### 2.2 路由定义示例

```javascript
// src/router/index.js (简化版)
import { createRouter, createWebHashHistory } from 'vue-router'

# 懒加载组件定义
const Login = () => import('../views/login/Login.vue')
const Layout = () => import('../views/layout/Layout.vue')
const DepartmentManagement = () => import('../views/department/DepartmentManagement.vue')
# ...其他组件

const routes = [
  # 独立路由（不使用 Layout 布局）
  {
    path: '/login',
    name: 'Login',
    component: Login
  },
  
  # 嵌套路由（使用 Layout 布局）
  {
    path: '/',              # 父路径
    name: 'Layout',
    component: Layout,      # 父组件
    children: [            # 子路由（在 Layout 内部的 <router-view> 中渲染）
      { 
        path: 'dashboard',  # 完整路径：/dashboard
        name: 'Dashboard',
        component: () => import('../views/layout/Dashboard.vue'),
        meta: { title: '工作台' }
      },
      { 
        path: 'department', # 完整路径：/department
        name: 'DepartmentManagement',
        component: DepartmentManagement,
        meta: { title: '部门管理' }
      },
      { 
        path: 'employee',   # 完整路径：/employee
        name: 'EmployeeManagement',
        component: EmployeeManagement,
        meta: { title: '员工管理' }
      },
      # ...其他子路由
      { 
        path: '',           # 空路径，重定向到 dashboard
        redirect: '/dashboard'
      }
    ]
  }
]

const router = createRouter({
  history: createWebHashHistory(),   # 使用 Hash 模式
  routes                           # 路由定义
})

export default router
```

**路径匹配规则**：
- 绝对路径 `/login` 匹配 `/login`
- 相对路径 `dashboard`（在 children 中）匹配 `/dashboard`（因为父路径是 `/`）
- 空路径 `''` 匹配父路径本身（`/`），这里做了重定向

---

## 3. 懒加载（代码分割）

### 3.1 实现原理

懒加载使用动态导入语法 `() => import('...')` 实现：

```javascript
const DepartmentManagement = () => import('../views/department/DepartmentManagement.vue')
```

**工作原理**：
1. Vite 分析依赖关系，为每个懒加载的组件生成独立的代码块
2. 初始加载只打包必要的代码（如登录页、Layout 等）
3. 当用户访问 `/department` 时，Vite 动态请求 `department-xxx.js` 文件
4. 组件加载完成后，Vue Router 渲染该组件

**代码块命名**：Vite 自动根据文件路径生成代码块名，如 `department-xxx.js`。

### 3.2 性能优势

| 指标 | 不懒加载 | 懒加载 |
|------|----------|--------|
| 初始包体积 | 所有页面 JS 一起打包 | 只打包首屏所需 |
| 首屏加载时间 | 较长（下载所有 JS） | 较短（只下载必要 JS） |
| 按需加载 | ❌ 所有 JS 一次性下载 | ✅ 访问时再下载 |
| 缓存效果 | 修改一个页面影响整体缓存 | 各页面独立缓存 |

**典型场景**：
- 登录页、首页（Dashboard）必须立即加载
- 部门管理、员工管理等不常用页面懒加载
- 用户首次访问时，点击菜单会触发懒加载（有短暂等待）

### 3.3 预加载优化

对于可能频繁访问的页面，可以预加载：

```javascript
# 在组件加载前预加载
import(/* webpackPrefetch: true */ '../views/department/DepartmentManagement.vue')
```

或者使用 `webpackPrefetch` 注释（Vite 支持）：

```javascript
const DepartmentManagement = () => import(/* webpackPrefetch: true */ '../views/department/DepartmentManagement.vue')
```

这样在首页加载时，会并行下载部门管理组件的代码块。

---

## 4. 嵌套路由（布局架构）

### 4.1 父路由与子路由

嵌套路由的核心思想是：**一个路由组件内部再包含一个 `<router-view>`，用于渲染子路由组件**。

```javascript
routes: [
  {
    path: '/',
    component: Layout,     # 父组件，包含侧边栏和顶部导航
    children: [
      { path: 'dashboard', component: Dashboard }, # 子组件，在 Layout 内部渲染
      { path: 'department', component: DepartmentManagement },
      # ...
    ]
  }
]
```

**URL 组合规则**：
- 父路径 `/` + 子路径 `department` → 完整路径 `/department`
- 子路径是相对于父路径的，不包含 `/` 前缀

### 4.2 嵌套路由的工作原理

```
用户访问 /department
  ↓
router 匹配到路由：{ path: '/', component: Layout, children: [...] }
  ↓
渲染 Layout 组件（此时 Layout 内部的 <router-view> 为空）
  ↓
router 继续匹配子路由：{ path: 'department', component: DepartmentManagement }
  ↓
将 DepartmentManagement 渲染到 Layout 内部的 <router-view>
  ↓
最终页面：Layout（侧边栏+顶部导航） + DepartmentManagement（内容区）
```

**嵌套层数**：Vue Router 支持任意层级的嵌套路由，但一般建议不超过 2-3 层，避免结构复杂。

### 4.3 Layout 布局组件

`Layout.vue` 是嵌套路由的核心组件，它包含固定布局和动态内容：

```vue
<!-- src/views/layout/Layout.vue -->
<template>
  <el-container class="app-container">
    <!-- 侧边栏（固定） -->
    <el-aside class="sidebar">
      <el-menu>...</el-menu>
    </el-aside>

    <!-- 主内容区 -->
    <el-container class="main-container">
      <!-- 顶部导航（固定） -->
      <el-header class="top-header">...</el-header>
      
      <!-- 动态内容（子路由渲染处） -->
      <el-scrollbar class="content-scroll">
        <router-view />   # 子路由组件在这里渲染
      </el-scrollbar>
    </el-container>
  </el-container>
</template>
```

**设计要点**：
- 侧边栏和顶部导航是固定的，不随子路由变化
- `<router-view>` 是动态区域，子路由组件替换这里的内容
- 这种架构实现了"固定布局 + 动态内容"的经典中后台模式

---

## 5. 路由守卫

路由守卫是路由导航过程中的钩子函数，可以在不同阶段执行逻辑。

### 5.1 beforeEach 守卫

`beforeEach` 是最常用的全局前置守卫，在**每次路由跳转前**触发：

```javascript
router.beforeEach((to, from, next) => {
  const to = { path: '/dashboard', name: 'Dashboard' }   # 目标路由
  const from = { path: '/login' }                       # 源路由
  
  # 获取登录态
  const isAuthenticated = localStorage.getItem('token')
  
  # 如果不是登录页且未授权，跳转登录
  if (to.path !== '/login' && !isAuthenticated) {
    next('/login')   # 阻止跳转，重定向到登录
  } else {
    next()           # 允许继续跳转
  }
})
```

**守卫参数**：
- `to`: 即将进入的路由对象（包含 path、name、meta、fullPath 等）
- `from`: 当前离开的路由对象
- `next()`: 必须调用，继续导航
- `next('/path')`: 取消当前跳转，重定向到新路径
- `next(false)`: 中断导航（保持在当前页面）

### 5.2 权限验证实现

**登录态验证**：从 `localStorage.getItem('token')` 检查是否有 Token。

**页面权限验证**（扩展）：可以结合用户角色做更细粒度的控制：

```javascript
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')
  if (!token) {
    next('/login')
    return
  }

  # 从 Store 获取用户角色
  const userStore = useUserStore()
  const userRole = userStore.userInfo?.role

  # 检查页面是否有角色限制
  if (to.meta.requiresRole && !userRole) {
    next('/login')
    return
  }

  next()
})
```

在路由配置中标记需要角色的页面：

```javascript
{
  path: 'admin',
  component: AdminPage,
  meta: { requiresRole: 'admin' }   # 需要 admin 角色
}
```

### 5.3 其他守卫类型

| 守卫 | 触发时机 | 典型用途 |
|------|----------|----------|
| `beforeEach` | 每次全局跳转前 | 权限校验、登录检查 |
| `beforeEnter` | 单个路由的跳转前 | 路由级权限校验 |
| `beforeResolve` | 所有组件解析后，确认前 | 最终确认守卫 |
| `afterEach` | 导航完成（无 next）| analytics、页面标题更新 |

**afterExample（页面标题更新）**：

```javascript
router.afterEach((to, from) => {
  # 更新页面标题（不需要 next）
  document.title = to.meta.title || '系统管理'
})
```

---

## 6. 路由元信息 (meta)

`meta` 是路由对象的附加属性，用于存储路由的元数据，如页面标题、权限标记等。

```javascript
{
  path: 'department',
  name: 'DepartmentManagement',
  component: DepartmentManagement,
  meta: { 
    title: '部门管理',      # 页面标题
    requiresAuth: true,     # 需要权限校验
    roles: ['admin', 'dept'] # 允许的角色
  }
}
```

**使用场景**：
- **页面标题**：在 `Layout.vue` 中通过 `route.meta.title` 获取并显示在顶部
- **权限校验**：在守卫中检查 `meta.requiresAuth` 或 `meta.roles`
- **面包屑导航**：根据 `meta.title` 生成面包屑路径
- **页面标识**：用 `meta.keepAlive` 标记需要缓存的页面

**在 Layout.vue 中使用**：

```javascript
import { useRoute } from 'vue-router'
const route = useRoute()

function getRouteName() {
  return route.meta.title || '首页'   # 从 meta 获取标题，没有则默认
}
```

---

## 7. 编程式导航

除了使用 `<router-link>` 声明式导航，还可以通过编程方式跳转页面。

### 7.1 使用 useRouter

```vue
<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

# 简单跳转
router.push('/dashboard')

# 带参数跳转（命名路由）
router.push({ name: 'Employee', params: { id: 1 }})

# 带查询参数
router.push({ path: '/search', query: { keyword: '张三' }})

# 后退一步
router.go(-1)

# 前进一步
router.go(1)

# 替换当前记录（不留下历史记录）
router.replace('/login')
</script>
```

### 7.2 路由参数

**动态路径参数**：在路由定义中使用 `:` 前缀：

```javascript
# 路由定义
{ path: '/user/:id', name: 'User', component: UserPage }

# 匹配 /user/1 → route.params.id = '1'
# 匹配 /user/2 → route.params.id = '2'
```

**获取参数**：

```vue
<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()
const userId = route.params.id   # 获取动态参数
const keyword = route.query.keyword   # 获取查询参数
</script>
```

---

## 8. 完整路由配置

下面是 Tlias 管理系统的完整路由配置（包含所有功能页面）：

```javascript
// src/router/index.js
import { createRouter, createWebHashHistory } from 'vue-router'

# 懒加载所有页面组件
const Login = () => import('../views/login/Login.vue')
const Layout = () => import('../views/layout/Layout.vue')
const Dashboard = () => import('../views/layout/Dashboard.vue')
const DepartmentManagement = () => import('../views/department/DepartmentManagement.vue')
const EmployeeManagement = () => import('../views/employee/EmployeeManagement.vue')
const ClassManagement = () => import('../views/classroom/ClassManagement.vue')
const StudentManagement = () => import('../views/student/StudentManagement.vue')
const Statistics = () => import('../views/statistics/Statistics.vue')
const LogManagement = () => import('../views/log/LogManagement.vue')

const routes = [
  # 登录页（独立路由，不使用 Layout）
  {
    path: '/login',
    name: 'Login',
    component: Login
  },

  # 主布局（嵌套路由）
  {
    path: '/',
    name: 'Layout',
    component: Layout,
    children: [
      { 
        path: 'dashboard', 
        name: 'Dashboard',
        component: Dashboard,
        meta: { title: '工作台' }
      },
      { 
        path: 'department', 
        name: 'DepartmentManagement',
        component: DepartmentManagement,
        meta: { title: '部门管理' }
      },
      { 
        path: 'employee', 
        name: 'EmployeeManagement',
        component: EmployeeManagement,
        meta: { title: '员工管理' }
      },
      { 
        path: 'classroom', 
        name: 'ClassManagement',
        component: ClassManagement,
        meta: { title: '班级管理' }
      },
      { 
        path: 'student', 
        name: 'StudentManagement',
        component: StudentManagement,
        meta: { title: '学员管理' }
      },
      { 
        path: 'statistics', 
        name: 'Statistics',
        component: Statistics,
        meta: { title: '数据统计' }
      },
      { 
        path: 'log', 
        name: 'LogManagement',
        component: LogManagement,
        meta: { title: '日志管理' }
      },
      { 
        path: '',        # 空路径重定向
        redirect: '/dashboard'
      }
    ]
  }
]

const router = createRouter({
  history: createWebHashHistory(),
  routes
})

# 路由守卫：权限验证
router.beforeEach((to, from, next) => {
  const isAuthenticated = localStorage.getItem('token')
  if (to.path !== '/login' && !isAuthenticated) {
    next('/login')
  } else {
    next()
  }
})

# 更新页面标题
router.afterEach((to) => {
  document.title = to.meta.title || 'Tlias 管理系统'
})

export default router
```

**路由配置要点总结**：

1. **登录页独立**：`/login` 不使用 Layout 布局，直接显示登录页面
2. **嵌套布局**：`/` 路径使用 Layout 作为父组件，子路由在 Layout 内部渲染
3. **懒加载**：所有页面组件都用 `() => import()` 懒加载
4. **元信息**：每个子路由都定义 `meta.title`，用于页面标题和菜单显示
5. **默认重定向**：空路径 `/` 默认重定向到 `/dashboard`
6. **路由守卫**：`beforeEach` 检查登录态，未登录跳登录页
7. **标题更新**：`afterEach` 更新页面标题，体验更佳

---

## 小结

Vue Router 是 Vue3 应用的核心路由管理库，关键特性包括：

1. **路由定义**：通过 `routes` 数组定义路径到组件的映射
2. **懒加载**：使用 `() => import()` 实现代码分割，提升首屏性能
3. **嵌套路由**：父组件内部再放 `<router-view>`，实现布局架构
4. **路由守卫**：`beforeEach` 用于权限校验，`afterEach` 用于标题更新
5. **元信息**：`meta` 属性存储路由的附加数据（标题、权限等）
6. **编程导航**：通过 `router.push()` 编程式跳转
7. **路由模式**：Hash 模式兼容性好，History 模式 URL 更干净

掌握这些核心技术，就可以构建功能完善的前端路由系统，实现 SPA 应用的页面管理和导航功能。

关联知识点：
- [[学习/JavaWeb/知识条目/前端/API接口封装]] 前后端请求配合
- [[学习/JavaWeb/知识条目/前端/Pinia状态管理]] 状态与路由配合
- [[学习/JavaWeb/知识条目/前端/Layout.vue页面布局]] 布局组件设计
- [[学习/JavaWeb/知识条目/前端/script setup语法]] 路由组件写法