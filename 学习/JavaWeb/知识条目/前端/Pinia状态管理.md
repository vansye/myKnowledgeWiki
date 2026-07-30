---
type: knowledge
title: Pinia 状态管理
created: 2026-07-28
updated: 2026-08-30
tags: [Vue3, 状态管理, Pinia, 前端]
source: Tlias 管理系统前端
conclusion: Pinia 是 Vue3 官方推荐的轻量级状态管理库，用于在组件间共享状态。它基于 Vue 3 Composition API 设计，去除了 Mutations，API 更简洁，且对 TypeScript 支持完美。
---

## 详细

### 概念
Pinia 是 Vue.js 的状态管理库，用于在组件或页面间共享响应式状态。它使用 `defineStore()` 定义 store，每个 store 是一个包含 state（状态）、getters（计算属性）和 actions（方法）的独立模块。在 actions 中经常调用 [[API接口封装]] 中定义的后端接口方法来获取数据。

### 重点
- **核心组成**：State 定义全局数据源；Getters 类似计算属性，从 state 派生出新数据；Actions 封装同步/异步逻辑，如 API 调用
- **对比 Vuex**：Pinia API 更简洁，无需 Mutations；对 TypeScript 支持更友好；支持创建多个独立 Store，而非单一 Store
- **安装与使用**：通过 `npm install pinia` 安装；在 main.js 中注册 `app.use(createPinia())`；在组件中用 `useXXXStore()` 使用 Store
- **持久化**：localStorage 保存 token 和用户信息，刷新后恢复
