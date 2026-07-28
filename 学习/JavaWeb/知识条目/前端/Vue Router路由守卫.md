---
type: knowledge
title: Vue Router 路由守卫
created: 2026-07-28
updated: 2026-07-28
tags: [Vue Router, 路由守卫, 权限校验, 前端]
source: Tlias 管理系统前端
conclusion: Vue Router 提供全局和路由级守卫，beforeEach 在每次跳转前触发，常用于权限校验（检查登录态）、路由拦截、数据预加载等场景。本项目用 localStorage.token 做简单权限控制。
---

## 详细

### 概念
路由守卫是 Vue Router 在导航不同阶段触发的钩子函数，`beforeEach` 是最常用的全局前置守卫，在每次路由跳转前触发，可用于权限检查、数据预加载等。

### 重点
- **beforeEach**：全局前置守卫，接收 `(to, from, next)` 三个参数，`next()` 继续导航，`next('/path')` 重定向
- **权限校验**：检查 `localStorage.token`，未登录且非登录页时跳转登录页
- **多层防护**：与 API 响应拦截器配合，形成路由层 + API 层的双重防护
- **其他守卫**：`beforeEnter`（路由级）、`beforeResolve`（解析后）、`afterEach`（后置）、组件级守卫（beforeRouteEnter/Update/Leave）
