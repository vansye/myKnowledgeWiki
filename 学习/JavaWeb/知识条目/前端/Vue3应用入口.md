---
type: knowledge
title: Vue3 应用入口 (main.js)
created: 2026-07-28
updated: 2026-07-28
tags: [Vue3, 应用入口, main.js, 前端]
source: Tlias 管理系统前端
conclusion: main.js 是 Vue3 应用的启动入口，负责创建应用实例、全局注册插件（Element Plus/Pinia/Router）并挂载到 DOM。核心步骤：createApp → use 插件 → mount。
---

## 详细

### 概念
`src/main.js` 是应用的启动点，负责创建 Vue 实例、注册全局插件（Element Plus、Pinia、Router）并挂载应用。核心流程：createApp → use 插件 → mount。

### 重点
- **创建实例**：`createApp(App)` 以根组件 App.vue 为模板创建应用实例
- **全局注册**：使用 `app.use()` 注册 Element Plus、Pinia、Router，所有组件自动可用
- **图标批量**：遍历 ElementPlusIconsVue 对象，自动注册所有图标组件
- **挂载时机**：所有插件注册完成后调用 mount，否则功能可能不可用
