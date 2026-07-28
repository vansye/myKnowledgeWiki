---
type: knowledge
title: Vue Router 路由配置
created: 2026-07-28
updated: 2026-07-28
tags: [Vue Router, 路由配置, 前端, SPA]
source: Tlias 管理系统前端
conclusion: Vue Router 4 实现 SPA 路由管理：使用 createWebHashHistory 模式，组件懒加载（import()），嵌套路由实现布局架构，beforeEach 守卫做权限校验。
---

## 详细

### 概念
`src/router/index.js` 配置前端路由，通过定义路径到组件的映射关系，实现单页面应用的路由管理。使用 `createRouter` 和 `createWebHashHistory` 创建路由器实例。

### 重点
- **懒加载**：使用 `() => import('...')` 实现代码分割，首屏只加载必要资源
- **嵌套路由**：父路由组件（如 Layout）内部再放 `<router-view>`，子路由渲染在其内
- **路由守卫**：`beforeEach` 在每次跳转前触发，用于权限校验；`afterEach` 用于标题更新
- **Hash 模式**：URL 带 `#`，兼容性好，无需后端配置；适合嵌入 Spring Boot 静态资源
