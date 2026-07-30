---
type: knowledge
title: App.vue 根组件
created: 2026-07-28
updated: 2026-08-30
tags: [Vue3, 根组件, App.vue, 前端]
source: Tlias 管理系统前端
conclusion: App.vue 是 Vue 应用的根容器，设计原则为最小化——仅承载 `<router-view>`，不添加业务逻辑。所有路由匹配的组件都会在这里被渲染替换。
---

## 详细

### 概念
`src/App.vue` 是整个 Vue 应用的根容器，是最外层组件。它只包含一个 `div#app` 和一个 `<router-view>`，由 `main.js` 的 `app.mount('#app')` 挂载到页面 DOM 上。`[[Layout.vue页面布局]]` 作为嵌套路由的父组件渲染在 `<router-view>` 内部。

### 重点
- **单一职责**：仅提供挂载点和路由出口，无业务逻辑
- **路由出口**：`<router-view>` 根据当前匹配的路径渲染对应组件
- **全局样式**：可在其中设置跨页面统一的字体、边距等基础样式
- **最外层位置**：位于应用层次的最顶层，[[Layout.vue页面布局]] 组件作为其子路由渲染在内部
