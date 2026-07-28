---
type: 学习笔记
title: Layout.vue页面布局
created: 2026-07-28
updated: 2026-07-28
tags:
  - Layout
  - 侧边栏
  - 嵌套路由
  - 布局架构
  - Vue3
subject: JavaWeb
description: 详细讲解 Layout.vue 页面布局的实现，包括侧边栏折叠、动态菜单、登出功能以及与路由的配合原理。
---

> 本笔记以 Tlias 管理系统前端为例，系统讲解 Layout.vue 页面布局的实现细节。从嵌套路由架构到侧边栏折叠，从动态菜单到登出功能，每个功能都深入剖析。

## 目录

- [1. Layout 组件的角色](#1-layout组件的角色)
- [2. 嵌套路由架构](#2-嵌套路由架构)
  - [2.1 父路由与子路由](#21-父路由与子路由)
  - [2.2 渲染流程](#22-渲染流程)
- [3. Layout.vue 结构](#3-layoutvue结构)
  - [3.1 el-container 布局](#31-el-container-布局)
  - [3.2 侧边栏](#32-侧边栏)
  - [3.3 主内容区](#33-主内容区)
- [4. 侧边栏折叠功能](#4-侧边栏折叠功能)
  - [4.1 折叠实现](#41-折叠实现)
  - [4.2 切换效果](#42-切换效果)
- [5. 动态菜单](#5-动态菜单)
  - [5.1 菜单与路由对应](#51-菜单与路由对应)
  - [5.2 选中状态](#52-选中状态)
- [6. 顶部导航栏](#6-顶部导航栏)
  - [6.1 折叠按钮](#61-折叠按钮)
  - [6.2 页面标题](#62-页面标题)
  - [6.3 用户信息](#63-用户信息)
  - [6.4 登出按钮](#64-登出按钮)
- [7. 登出逻辑](#7-登出逻辑)
- [8. 与 Store 的配合](#8-与-store的配合)
- [9. 完整实现代码](#9-完整实现代码)
- [10. 设计模式](#10-设计模式)
- [小结](小结)

---

## 1. Layout 组件的角色

`Layout.vue` 是整个管理系统的**主布局组件**，负责：

1. **提供固定布局**：侧边栏导航、顶部工具栏
2. **容纳动态内容**：通过 `<router-view>` 渲染子路由组件
3. **统一管理**：所有管理页面共享同一个 Layout，避免重复代码

**在路由中的位置**：
- `/` 路径（父路由）→ Layout.vue（布局组件）
- `/dashboard`、`/department` 等（子路由）→ 具体的页面组件

Layout 本身不显示具体业务内容，它只是一个**容器**，真正的业务内容由子路由的组件负责。

## 2. 嵌套路由架构

### 2.1 父路由与子路由

嵌套路由的核心是：**父路由组件内部包含一个 `<router-view>`，子路由组件渲染在这里**。

```javascript
# router/index.js
{
  path: '/',                  # 父路径
  component: Layout,          # 父组件
  children: [                # 子路由
    { path: 'dashboard', component: Dashboard },
    { path: 'department', component: DepartmentManagement },
    # ...
  ]
}
```

**路径组合规则**：
- 父路径 `/` + 子路径 `dashboard` → 完整路径 `/dashboard`
- 子路径是相对于父路径的，**不包含 `/` 前缀**
- 如果子路径是 `''`（空字符串），则匹配父路径本身

### 2.2 渲染流程

```
用户访问 /department
  ↓
router 匹配到父路由：path='/'，component=Layout
  ↓
渲染 Layout 组件（此时 Layout 内部的 <router-view> 为空）
  ↓
router 匹配子路由：path='department'，component=DepartmentManagement
  ↓
将 DepartmentManagement 渲染到 Layout 内部的 <router-view>
  ↓
最终页面：Layout（侧边栏+顶部导航） + DepartmentManagement（内容区）
```

这种架构的特点是：
- **不变部分**：侧边栏、顶部导航（在 Layout 中，不变化）
- **变化部分**：中间的页面内容（子路由组件，随 URL 变化）

## 3. Layout.vue 结构

### 3.1 el-container 布局

Layout 使用 Element Plus 的布局组件构建：

```vue
<el-container class="app-container">
  <!-- 侧边栏 -->
  <el-aside class="sidebar">...</el-aside>

  <!-- 主内容区 -->
  <el-container class="main-container">
    <!-- 顶部导航 -->
    <el-header class="top-header">...</el-header>
    
    <!-- 内容区（子页面渲染） -->
    <el-scrollbar><router-view /></el-scrollbar>
  </el-container>
</el-container>
```

**布局结构**：

```
+-------------------+-----------------------+
|   侧边栏 (sidebar)|      主内容区           |
|                   |  +------------------+ |
|                   |  | 顶部导航 (header)| |
|                   |  +------------------+ |
|                   |  |                  | |
|                   |  |  子页面内容      | |
|                   |  |  (<router-view>)| |
|                   |  |                  | |
|                   |  +------------------+ |
+-------------------+-----------------------+
```

### 3.2 侧边栏

侧边栏使用 `el-aside` 和 `el-menu` 实现：

```vue
<el-aside 
  :width="collapsed ? '64px' : '200px'" 
  class="sidebar"
>
  <el-container class="sidebar-header">
    <div v-if="collapsed" class="sidebar-logo T"></div>
    <div v-else class="sidebar-logo">Tlias 管理系统</div>
  </el-container>
  
  <el-menu 
    :default-active="route.path" 
    :collapse="collapsed" 
    background-color="#35424d"
    text-color="#bfcbd9"
    active-text-color="#ffd04b"
    unique-opened
  >
    # 菜单项
    <el-menu-item index="/dashboard" @click="router.push('/dashboard')">
      <el-icon><Odometer /></el-icon>
      <span>工作台</span>
    </el-menu-item>
    # ...其他菜单项
  </el-menu>
</el-aside>
```

**侧边栏特性**：
- **折叠/展开**：通过 `collapsed` 变量控制宽度
- **菜单项**：`el-menu-item` 对应每个页面
- **图标**：折叠时只显示图标，展开时显示图标+文字
- **选中状态**：`default-active` 高亮当前页面

### 3.3 主内容区

主内容区包含顶部导航和滚动内容区：

```vue
<el-container class="main-container">
  <!-- 顶部导航 -->
  <el-header class="top-header">
    <div class="header-left">
      <el-button @click="collapsed = !collapsed" plain>
        <el-icon><Fold /></el-icon>
      </el-button>
      <span class="header-title">{{ getRouteName() }}</span>
    </div>
    <div class="header-right">
      <span class="header-user">{{ userStore.userInfo?.name || '用户' }}</span>
      <el-avatar :src="userStore.userInfo?.image || ''" style="width: 32px; height: 32px;"></el-avatar>
      <el-button @click="logout" plain>
        <el-icon><SwitchButton /></el-icon>
      </el-button>
    </div>
  </el-header>

  <!-- 内容区（带滚动） -->
  <el-scrollbar class="content-scroll">
    <router-view />   # 子路由组件渲染在这里
  </el-scrollbar>
</el-container>
```

**特性**：
- **el-scrollbar**：自动添加滚动条，内容超出时出现滚动
- **<router-view>**：子路由组件渲染的位置
- **内容高度**：flex: 1，占据剩余空间

## 4. 侧边栏折叠功能

### 4.1 折叠实现

侧边栏折叠通过 `collapsed` 响应式变量控制：

```javascript
import { ref } from 'vue'

const collapsed = ref(false)   # false=展开，true=折叠

# 切换折叠状态
function toggleCollapse() {
  collapsed.value = !collapsed.value
}
```

**绑定到 el-aside**：

```vue
<el-aside :width="collapsed ? '64px' : '200px'" class="sidebar">
  # width 绑定到 collapsed，折叠时 64px，展开时 200px
</el-aside>
```

**绑定到 el-menu**：

```vue
<el-menu :collapse="collapsed">...</el-menu>
```

### 4.2 切换效果

Element Plus 的 `el-aside` 和 `el-menu` 都支持折叠动画：

- **CSS 过渡**：宽度变化有 0.3s 的过渡效果（`.sidebar { transition: width 0.3s; }`）
- **菜单自动折叠**：`el-menu` 的 `collapse` 属性控制菜单项的显示
- **文字隐藏**：CSS 类 `.collapse span { display: none; }` 在折叠时隐藏文字

**折叠状态变化**：

| 状态 | 侧边栏宽度 | 菜单显示 | 适用场景 |
|------|-----------|----------|----------|
| **展开** | 200px | 图标+文字 | 正常操作 |
| **折叠** | 64px | 仅图标 | 空间紧张时 |

## 5. 动态菜单

### 5.1 菜单与路由对应

菜单项与路由一一对应，点击后跳转到对应页面：

```vue
<el-menu-item index="/dashboard" @click="router.push('/dashboard')">
  <el-icon><Odometer /></el-icon>
  <span>工作台</span>
</el-menu-item>

<el-menu-item index="/department" @click="router.push('/department')">
  <el-icon><OfficeBuilding /></el-icon>
  <span>部门管理</span>
</el-menu-item>
```

**注意**：
- `index` 属性用于 `default-active` 高亮（选中状态）
- `@click` 事件通过 `router.push()` 编程式跳转
- 菜单顺序与路由顺序一致

### 5.2 选中状态

通过 `default-active` 属性高亮当前选中的菜单项：

```vue
<el-menu :default-active="route.path" :collapse="collapsed">
```

`route.path` 是当前路由的完整路径（如 `/department`），与菜单的 `index` 匹配时自动高亮。

**选中效果**：Element Plus 的菜单会自动为匹配的项添加选中样式（高亮背景色）。

## 6. 顶部导航栏

### 6.1 折叠按钮

顶部导航左侧有一个折叠按钮，点击可切换侧边栏折叠状态：

```vue
<el-button @click="collapsed = !collapsed" plain>
  <el-icon><Fold /></el-icon>
</el-button>
```

### 6.2 页面标题

页面标题通过路由的 `meta.title` 动态获取：

```javascript
import { useRoute } from 'vue-router'

const route = useRoute()

function getRouteName() {
  return route.meta.title || '首页'   # 从 meta 取 title，没有则默认
}
```

在模板中显示：

```vue
<span class="header-title">{{ getRouteName() }}</span>
```

**路由元信息**（在 `router/index.js` 中定义）：

```javascript
{
  path: 'department',
  name: 'DepartmentManagement',
  component: DepartmentManagement,
  meta: { title: '部门管理' }   # 页面标题
}
```

这样每个页面的标题可以单独设置，而不需要在 Layout 中硬编码。

### 6.3 用户信息

顶部右侧显示当前用户信息：

```vue
<span class="header-user">{{ userStore.userInfo?.name || '用户' }}</span>
<el-avatar :src="userStore.userInfo?.image || ''" style="width: 32px; height: 32px;"></el-avatar>
```

从 Pinia Store（`userStore`）获取用户信息：
- `userStore.userInfo.name`：用户姓名
- `userStore.userInfo.image`：用户头像（可选）

### 6.4 登出按钮

右侧有一个登出按钮，点击后弹出确认对话框，登出后跳转登录页：

```vue
<el-button @click="logout" plain>
  <el-icon><SwitchButton /></el-icon>
</el-button>
```

## 7. 登出逻辑

登出是一个完整的安全流程：

```javascript
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { ElMessageBox } from 'element-plus'

const router = useRouter()
const userStore = useUserStore()

async function logout() {
  # 1. 弹出确认对话框
  await ElMessageBox.confirm('确定要退出登录吗？', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  })
  
  # 2. 调用 Store 的 logout 方法
  await userStore.logout()
  
  # 3. 跳转到登录页
  router.push('/login')
}
```

**登出流程分解**：

1. **确认对话框**：防止误操作，使用 Element Plus 的 ElMessageBox
2. **清除 Store 状态**：`userStore.logout()` 设置 `token` 和 `userInfo` 为 null
3. **清除 localStorage**：删除保存的 token 和 userInfo
4. **跳转登录页**：`router.push('/login')` 跳转到登录页面

**为什么需要两步清除？**
- Store 清除：确保后续组件访问时取不到旧数据
- localStorage 清除：页面刷新后也不会恢复旧数据

## 8. 与 Store 的配合

Layout 与 Pinia Store 紧密配合：

### 8.1 初始化数据

页面加载时从 Store 获取数据：

```javascript
import { onMounted } from 'vue'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

onMounted(async () => {
  try {
    # 初始化加载部门列表和员工数据
    await userStore.loadDepartments()
    await userStore.loadAllEmployees()
  } catch (error) {
    console.warn('初始化数据失败', error)
  }
})
```

### 8.2 共享状态

Store 中的数据在多个组件间共享：

- Layout 中的 `userStore.userInfo` 显示用户信息
- 部门选择器使用 `userStore.departments`
- 员工选择器使用 `userStore.allEmployees`

这种共享机制避免了在多个页面中重复获取相同数据。

### 8.3 登出同步

登出时，Layout 调用 `userStore.logout()`，Store 会同时：
- 清除内存中的 state（`token`、`userInfo`）
- 清除 localStorage 中的持久化数据
- 后续组件访问时获取不到旧数据

---

## 9. 完整实现代码

以下是 Tlias 管理系统 Layout 组件的完整实现：

```vue
<!-- src/views/layout/Layout.vue -->
<script setup>
import { ref, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { ElMessage, ElMessageBox } from 'element-plus'

# Store 实例
const userStore = useUserStore()

# 路由实例
const router = useRouter()
const route = useRoute()

# 侧边栏折叠状态
const collapsed = ref(false)

# 页面加载时初始化数据
onMounted(async () => {
  try {
    await userStore.loadDepartments()
    await userStore.loadAllEmployees()
  } catch (error) {
    console.warn('Failed to load initial data', error)
  }
})

# 登出功能
async function logout() {
  await ElMessageBox.confirm('确定要退出登录吗？', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  })
  await userStore.logout()
  router.push('/login')
}

# 获取页面标题（来自 route.meta）
function getRouteName() {
  return route.meta.title || '首页'
}
</script>

<template>
  <el-container class="app-container">
    <!-- 侧边栏 -->
    <el-aside :width="collapsed ? '64px' : '200px'" class="sidebar">
      <el-container class="sidebar-header">
        <div v-if="collapsed" class="sidebar-logo T"></div>
        <div v-else class="sidebar-logo">Tlias 管理系统</div>
      </el-container>
      
      <el-menu 
        :default-active="route.path" 
        :collapse="collapsed" 
        background-color="#35424d" 
        text-color="#bfcbd9" 
        active-text-color="#ffd04b" 
        unique-opened
      >
        <el-menu-item index="/dashboard" @click="router.push('/dashboard')">
          <el-icon><Odometer /></el-icon>
          <span>工作台</span>
        </el-menu-item>
        <el-menu-item index="/department" @click="router.push('/department')">
          <el-icon><OfficeBuilding /></el-icon>
          <span>部门管理</span>
        </el-menu-item>
        <el-menu-item index="/employee" @click="router.push('/employee')">
          <el-icon><User /></el-icon>
          <span>员工管理</span>
        </el-menu-item>
        <el-menu-item index="/classroom" @click="router.push('/classroom')">
          <el-icon><Reading /></el-icon>
          <span>班级管理</span>
        </el-menu-item>
        <el-menu-item index="/student" @click="router.push('/student')">
          <el-icon><Avatar /></el-icon>
          <span>学员管理</span>
        </el-menu-item>
        <el-menu-item index="/statistics" @click="router.push('/statistics')">
          <el-icon><Histogram /></el-icon>
          <span>数据统计</span>
        </el-menu-item>
        <el-menu-item index="/log" @click="router.push('/log')">
          <el-icon><Document /></el-icon>
          <span>日志管理</span>
        </el-menu-item>
      </el-menu>
    </el-aside>

    <!-- 主内容区 -->
    <el-container class="main-container">
      <!-- 顶部导航 -->
      <el-header class="top-header">
        <div class="header-left">
          <el-button @click="collapsed = !collapsed" plain>
            <el-icon><Fold /></el-icon>
          </el-button>
          <span class="header-title">{{ getRouteName() }}</span>
        </div>
        <div class="header-right">
          <span class="header-user">{{ userStore.userInfo?.name || '用户' }}</span>
          <el-avatar :src="userStore.userInfo?.image || ''" style="width: 32px; height: 32px;"></el-avatar>
          <el-button @click="logout" plain>
            <el-icon><SwitchButton /></el-icon>
          </el-button>
        </div>
      </el-header>

      <!-- 内容区（带滚动） -->
      <el-scrollbar class="content-scroll">
        <router-view />   # 子路由组件渲染在这里
      </el-scrollbar>
    </el-container>
  </el-container>
</template>

<style scoped>
.app-container {
  height: 100vh;
  width: 100%;
}

.sidebar {
  background-color: #35424d;
  transition: width 0.3s;   # 折叠动画过渡
  overflow-x: hidden;       # 隐藏横向滚动
}

.sidebar-header {
  height: 60px;
  line-height: 60px;
  background-color: #2d3a46;
  text-align: center;
  color: #fff;
  font-size: 18px;
  font-weight: bold;
}

.sidebar-logo {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.main-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.top-header {
  height: 60px;
  background-color: #fff;
  border-bottom: 1px solid #e0e0e0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 15px;
}

.header-title {
  font-size: 16px;
  color: #333;
  font-weight: 500;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 15px;
}

.header-user {
  font-size: 14px;
  color: #333;
}

.content-scroll {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
}

/* 折叠时隐藏菜单文字 */
.collapse span {
  display: none;
}
</style>
```

---

## 10. 设计模式

### 10.1 模板方法模式

Layout 定义了页面布局的模板（侧边栏+顶部导航+内容区），具体的业务内容由子路由填充：

```
Layout 模板：
  <sidebar>...</sidebar>
  <header>...</header>
  <content><router-view /></content>   # 子类填充这里
```

### 10.2 组合模式

Layout 通过组合多个 Element Plus 组件（el-container、el-aside、el-menu 等）构建复杂布局，每个组合单元都可以独立复用。

### 10.3 代理模式

Layout 作为 Store 和路由的代理，组件不需要直接访问 Store 或 router，通过 Layout 提供的统一接口（如 `logout()`、`getRouteName()`）来操作。

---

## 小结

Layout.vue 是中后台管理系统的核心布局组件，通过嵌套路由实现"固定布局+动态内容"的架构：

1. **嵌套路由**：父路由（Layout）+ 子路由（具体页面），Layout 内部的 `<router-view>` 渲染子页面
2. **侧边栏折叠**：通过 `collapsed` 变量控制宽度，展开时显示文字，折叠时仅显示图标
3. **动态菜单**：菜单项与路由对应，点击跳转，选中状态自动高亮
4. **顶部导航**：显示页面标题（来自 route.meta.title）、用户信息和登出按钮
5. **登出流程**：确认对话框 → 清除 Store 和 localStorage → 跳转登录页
6. **与 Store 配合**：共享用户信息、部门列表等数据，初始化时加载数据

掌握 Layout 的嵌套路由架构和侧边栏折叠功能，就可以搭建标准的中后台管理页面布局。

关联知识点：
- [[Vue Router路由配置]] 嵌套路由原理
- [[Pinia状态管理]] Store 数据共享
- [[API接口封装]] 数据加载
- [[学习/JavaWeb/知识条目/前端/script setup语法]] 组件实现