---
type: knowledge
title: Layout.vue 页面布局
created: 2026-07-28
updated: 2026-07-28
tags: [Vue3, 布局, Layout, 前端]
source: Tlias 管理系统前端
conclusion: Layout.vue 是后台管理系统的核心布局组件，包含侧边栏导航、顶部工具栏和内容区。通过嵌套路由实现"固定布局+动态内容"架构，所有管理页面共享此布局。
---

## 详细

### 概念
`src/views/layout/Layout.vue` 是管理系统的父路由组件，提供固定的侧边栏和顶部导航，中间的 `<router-view>` 渲染具体的子页面组件。

### 重点
- **嵌套路由**：父路由 Layout 包含子路由，子路由渲染在 Layout 内部的 `<router-view>` 中
- **侧边栏折叠**：通过 `collapsed` 变量控制侧边栏宽度（64px/200px），配合 `el-menu` 的 collapse 属性
- **动态菜单**：菜单项与路由一一对应，点击后通过 `router.push()` 跳转，选中状态自动高亮
- **登出逻辑**：确认对话框 → 清除 Store 和 localStorage → 跳转登录页，形成完整退出流程
