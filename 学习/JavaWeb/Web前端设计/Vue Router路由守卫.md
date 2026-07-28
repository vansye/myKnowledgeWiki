---
type: 学习笔记
title: Vue Router路由守卫
created: 2026-07-28
updated: 2026-07-28
tags:
  - Vue Router
  - 路由守卫
  - beforeEach
  - 权限校验
  - 导航守卫
subject: JavaWeb
description: 详细讲解 Vue Router 路由守卫的原理、类型、使用场景以及权限验证的完整实现方案。
---

> 本笔记以 Tlias 管理系统前端为例，系统讲解 Vue Router 路由守卫的工作原理和使用方法。从全局前置守卫到路由级守卫，从权限校验到数据预加载，每个使用场景都深入剖析。

## 目录

- [1. 路由守卫概述](#1-路由守卫概述)
- [2. 全局前置守卫 beforeEach](#2-全局前置守卫-beforeeach)
  - [2.1 守卫参数](#21-守卫参数)
  - [2.2 next 函数的用法](#22-next-函数的用法)
  - [2.3 权限验证实现](#23-权限验证实现)
- [3. 路由级守卫 beforeEnter](#3-路由级守卫 beforeenter)
- [4. 解析后守卫 beforeResolve](#4-解析后守卫 beforeresolve)
- [5. 后置守卫 afterEach](#5-后置守卫 aftereach)
- [6. 组件级守卫](#6-组件级守卫)
  - [6.1 beforeRouteEnter](#61-boroutenter)
  - [6.2 beforeRouteUpdate](#62-borouteupdate)
  - [6.3 beforeRouteLeave](#63-borouteleave)
- [7. 权限验证完整方案](#7-权限验证完整方案)
  - [7.1 多层防护体系](#71-多层防护体系)
  - [7.2 与 API 拦截器的配合](#72-与-api-拦截器的配合)
- [8. 数据预加载](#8-数据预加载)
- [9. 页面标题更新](#9-页面标题更新)
- [10. 常见用法总结](#10-常见用法总结)
- [小结](小结)

---

## 1. 路由守卫概述

路由守卫是 Vue Router 提供的钩子函数，在导航的不同阶段触发，可以执行前置逻辑、权限检查、数据预处理等操作。

**核心作用**：
- 权限验证（检查是否登录、是否有角色）
- 数据预加载（进入页面前先获取数据）
- 页面标题更新
- 访问统计（埋点日志）
- 导航拦截（阻止不合法的跳转）

**守卫调用时机**：

```
路由变化
  ↓
beforeEach (全局前置守卫)
  ↓
beforeEnter (路由级守卫，如果有)
  ↓
beforeResolve (解析后守卫)
  ↓
组件激活/渲染
  ↓
afterEach (后置守卫，无 next)
```

---

## 2. 全局前置守卫 beforeEach

`beforeEach` 是最常用的守卫，在**每次路由跳转前**触发，适用于全局权限校验。

### 2.1 守卫参数

```javascript
router.beforeEach((to, from, next) => {
  # to: 即将进入的目标路由对象
  # from: 当前离开的路由对象  
  # next: 必须调用的函数，控制导航继续
})
```

**to 对象属性**：
- `path`：目标路径（如 `/dashboard`）
- `name`：路由名称（如 `Dashboard`）
- `meta`：路由元信息（如 `{ title: '工作台' }`）
- `fullPath`：完整路径（包含 query）
- `matched`：匹配的路由层级数组

**from 对象属性**：同 to，表示当前离开的路由。

### 2.2 next 函数的用法

`next()` 是守卫中**必须调用**的函数，控制导航继续或取消：

| 调用方式 | 效果 |
|----------|------|
| `next()` | 继续导航，按照原定的目标 |
| `next('/path')` | 取消当前导航，重定向到新路径 |
| `next({ name: 'Login' })` | 取消，跳转到命名路由 |
| `next(false)` | 中断导航，保持在当前页面 |
| `next(执行的函数)` | 等待函数执行后再继续（用于异步） |

**使用示例**：

```javascript
# 简单放行
next()

# 重定向登录
next('/login')

# 中断导航
next(false)

# 异步守卫（next 为函数）
router.beforeEach((to, from, next) => {
  checkAuth().then(() => next())   # 异步检查后继续
})
```

### 2.3 权限验证实现

最简单的权限验证：检查是否登录。

```javascript
// src/router/index.js
import { createRouter, createWebHashHistory } from 'vue-router'

const router = createRouter({
  history: createWebHashHistory(),
  routes
})

# 全局前置守卫：权限校验
router.beforeEach((to, from, next) => {
  # 从 localStorage 获取 Token
  const isAuthenticated = localStorage.getItem('token')
  
  # 如果是登录页，允许访问
  if (to.path === '/login') {
    next()
    return
  }
  
  # 如果不是登录页且未授权，跳转到登录页
  if (!isAuthenticated) {
    next('/login')
    return
  }
  
  # 其他情况：放行
  next()
})
```

**优化版（更简洁）**：

```javascript
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')
  # 非登录页且无 token → 跳登录
  if (to.path !== '/login' && !token) {
    next('/login')
  } else {
    next()
  }
})
```

**工作流程**：
1. 用户点击菜单或输入 URL
2. `beforeEach` 触发，检查 token
3. 有 token → `next()` 继续
4. 无 token 且不是登录页 → `next('/login')` 跳转
5. 是登录页 → `next()` 允许访问

---

## 3. 路由级守卫 beforeEnter

`beforeEnter` 是在单个路由定义的守卫，**只对该路由生效**，不共享给其他路由。

```javascript
const routes = [
  {
    path: '/secret',
    component: SecretPage,
    beforeEnter: (to, from, next) => {
      # 只有访问 /secret 时才触发
      if (userStore.isAdmin) {
        next()
      } else {
        next('/login')
      }
    }
  }
]
```

**与 beforeEach 的区别**：

| 特性 | beforeEach | beforeEnter |
|------|------------|-------------|
| **作用范围** | 全局（所有路由） | 单个路由 |
| **定义位置** | router 实例上 | 单个 route 对象 |
| **使用场景** | 全局权限 | 特定页面的权限 |
| **性能** | 每次跳转都检查 | 仅匹配该路由时检查 |

**典型使用场景**：
- 特定页面需要更复杂的权限校验
- 不同路由需要不同的前置条件
- 不想让全局守卫处理特定逻辑

---

## 4. 解析后守卫 beforeResolve

`beforeResolve` 在**所有组件解析后、导航确认前**触发。此时路由的组件已经确定，但还没有开始渲染。

```javascript
router.beforeResolve((to, from, next) => {
  # 此时组件已经确定，可以做一些最后的检查
  # 例如：检查用户是否完成了必要的基础设置
  if (userStore.hasCompletedOnboarding) {
    next()
  } else {
    next('/onboarding')
  }
})
```

**使用场景**：
- 需要在组件解析后进行最终检查
- 确保所有前置条件都满足后才允许导航
- 比 beforeEach 更晚，组件已经确定

---

## 5. 后置守卫 afterEach

`afterEach` 在**导航完成后触发**，不需要调用 `next`（因为没有决策要做）。

```javascript
router.afterEach((to, from) => {
  # 更新页面标题
  document.title = to.meta.title || '系统管理'
  
  # 页面访问统计（埋点）
  if (to.matched.some(record => record.meta.track)) {
    analytics.page(to.path)
  }
  
  # 滚动到顶部
  window.scrollTo(0, 0)
})
```

**特点**：
- 不需要 `next()`
- 可以用于副作用操作（标题更新、统计、日志）
- 无法阻止导航（因为导航已经完成了）

---

## 6. 组件级守卫

组件级守卫定义在 Vue 组件内部，只对该组件生效。

### 6.1 beforeRouteEnter

在组件实例创建**之前**触发，此时还没有 `this` 上下文。

```vue
<script setup>
import { onBeforeRouteEnter } from 'vue-router'

onBeforeRouteEnter((to, from, next) => {
  # 此时还没有组件实例，不能访问 this
  # 但可以访问 to/from 路由对象
  
  # 如果需要访问组件实例，需要通过 next 回调
  next(vm => {
    # vm 是组件实例，可以访问 this
    vm.loadData()
  })
})
</script>
```

### 6.2 beforeRouteUpdate

当路由变化但组件**复用时**触发（如从 `/user/1` 到 `/user/2`，组件相同但参数不同）。

```vue
<script setup>
import { onBeforeRouteUpdate } from 'vue-router'

onBeforeRouteUpdate((to, from, next) => {
  # 获取新的参数
  const userId = to.params.id
  # 重新加载数据
  loadData(userId)
  next()
})
</script>
```

### 6.3 beforeRouteLeave

在**离开当前组件前**触发，可以用于确认操作（如防止未保存的更改离开）。

```vue
<script setup>
import { onBeforeRouteLeave } from 'vue-router'
import { ElMessageBox } from 'element-plus'

const hasUnsavedChanges = ref(false)

onBeforeRouteLeave((to, from, next) => {
  if (hasUnsavedChanges.value) {
    ElMessageBox.confirm('有未保存的更改，确定要离开吗？', '提示', {
      confirmButtonText: '离开',
      cancelButtonText: '继续编辑',
      type: 'warning'
    }).then(() => {
      next()   # 允许离开
    }).catch(() => {
      next(false)   # 阻止离开
    })
  } else {
    next()   # 没有更改，直接允许离开
  }
})
</script>
```

---

## 7. 权限验证完整方案

### 7.1 多层防护体系

最佳实践是**多层防护**，不要只依赖单一守卫：

| 层级 | 位置 | 作用 |
|------|------|------|
| **路由层** | `router/beforeEach` | 控制能否进入页面 |
| **API层** | `http.js/响应拦截器` | 控制接口调用（如 401 跳登录） |
| **组件层** | 组件内部守卫 | 页面内的细粒度权限控制 |

这种多层防护确保即使绕过路由守卫直接请求 API，也能被拦截器捕获并处理。

### 7.2 与 API 拦截器的配合

本项目中，路由守卫和 API 拦截器共同构成完整的前端防护体系：

**路由守卫**（`router/beforeEach`）：
- 页面跳转前检查登录态
- 未登录者无法进入管理页面
- 保护页面入口

**API 拦截器**（`http.js` 响应拦截器）：
- 请求发出后检查响应
- 401 状态码时自动跳登录
- 即使绕过路由守卫直接发请求也能拦截

**典型场景分析**：

1. **正常流程**：
   - 用户访问 `/dashboard` → 路由守卫检查 token → 通过 → Layout 渲染 → API 请求正常

2. **Token 过期场景**：
   - 用户已登录，但 Token 已过期
   - 路由守卫通过（localStorage 仍有 token）
   - API 请求时后端返回 401
   - API 拦截器捕获 401 → 提示"请先登录" → 跳转登录页

3. **直接访问 API**：
   - 用户直接在浏览器发起 API 请求（绕过路由）
   - API 拦截器检查响应 → 401 → 跳转登录页

**注意**：路由守卫中使用 `localStorage`，而 API 拦截器中检查 `error.response.status`，两者的判断依据可能不一致，需要保持同步。

---

## 8. 数据预加载

在路由跳转前预加载数据，提升用户体验：

```javascript
# 在 beforeEnter 中预加载数据
{
  path: 'department',
  name: 'DepartmentManagement',
  component: DepartmentManagement,
  beforeEnter: async (to, from, next) => {
    # 获取 departmentStore
    const deptStore = useDeptStore()
    
    # 如果数据已存在，直接跳过
    if (deptStore.departments.length > 0) {
      next()
      return
    }
    
    # 否则预加载数据
    await deptStore.loadDepartments()
    next()
  }
}
```

**在组件中使用 beforeRouteEnter**：

```vue
<script setup>
import { onBeforeRouteEnter } from 'vue-router'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

onBeforeRouteEnter((to, from, next) => {
  next(vm => {
    # 组件实例创建后加载数据
    vm.loadInitialData()
  })
})

# 组件内部方法
async function loadInitialData() {
  await userStore.loadDepartments()
  await userStore.loadAllEmployees()
}
</script>
```

---

## 9. 页面标题更新

使用 `afterEach` 守卫自动更新页面标题：

```javascript
router.afterEach((to, from) => {
  # 从路由的 meta 中获取 title，没有则默认
  const title = to.meta.title || '系统管理'
  document.title = title
  
  # 滚动到顶部
  window.scrollTo(0, 0)
})
```

在路由配置中定义标题：

```javascript
{
  path: 'department',
  name: 'DepartmentManagement',
  component: DepartmentManagement,
  meta: { title: '部门管理' }   # 页面标题
}
```

这样每个页面的标题自动更新，无需在每个组件中单独设置。

---

## 10. 常见用法总结

### 10.1 基础权限检查

```javascript
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')
  if (!token && to.path !== '/login') {
    next('/login')
  } else {
    next()
  }
})
```

### 10.2 带角色的权限检查

```javascript
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')
  if (!token) {
    next('/login')
    return
  }

  # 从 store 获取角色
  const userStore = useUserStore()
  const userRole = userStore.userInfo?.role

  # 检查页面是否需要特定角色
  if (to.meta.requiresRole && !userRole) {
    next('/login')
    return
  }

  next()
})
```

### 10.3 带加载状态的权限检查

```javascript
router.beforeEach(async (to, from, next) => {
  const token = localStorage.getItem('token')
  
  if (!token) {
    next('/login')
    return
  }

  # 异步验证 token
  try {
    const userInfo = await validateToken(token)   # 调用后端验证
    # 保存用户信息到 store
    useUserStore().userInfo = userInfo
    next()
  } catch (error) {
    # token 验证失败
    localStorage.removeItem('token')
    next('/login')
  }
})
```

### 10.4 免登录页面

某些页面（如登录页、注册页、忘记密码）不需要登录就可以访问：

```javascript
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')
  
  # 如果目标是登录页且有 token，说明已登录，跳首页
  if (to.path === '/login' && token) {
    next('/dashboard')
    return
  }

  # 如果不是登录页且无 token，跳登录
  if (to.path !== '/login' && !token) {
    next('/login')
    return
  }

  next()
})
```

---

## 小结

路由守卫是 Vue Router 实现导航控制和权限管理的核心机制，主要类型：

1. **beforeEach**：全局前置守卫，每次跳转前触发，常用于权限验证
2. **beforeEnter**：路由级守卫，只对该路由生效
3. **beforeResolve**：解析后守卫，组件确定后触发
4. **afterEach**：后置守卫，导航完成后触发（无 next）
5. **组件级守卫**：`beforeRouteEnter`、`beforeRouteUpdate`、`beforeRouteLeave`

**最佳实践**：
- 使用 `beforeEach` 做全局权限检查
- 结合 API 拦截器形成多层防护
- 用 `afterEach` 更新页面标题和统计
- 用组件级守卫处理页面特定的逻辑
- 保持前后端权限验证策略一致

掌握路由守卫的使用，就可以实现完善的页面权限控制和导航管理功能。

关联知识点：
- [[Vue Router路由配置]] 路由定义
- [[API接口封装]] 响应拦截器配合
- [[Pinia状态管理]] Store 中保存用户信息
- [[学习/JavaWeb/知识条目/前端/script setup语法]] 组件内守卫实现