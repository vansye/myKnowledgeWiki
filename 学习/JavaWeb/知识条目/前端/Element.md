---
type: knowledge
title: Element UI / Element Plus
created: 2026-07-25
updated: 2026-07-25
tags: [Vue, UI组件库, Element, 前端]
source: 无
conclusion: Element 是 Vue 生态中最主流的桌面端 UI 组件库，由饿了么前端团队开发。Element UI 基于 Vue 2，Element Plus 是其 Vue 3 官方升级版，两者根据 Vue 版本选择即可。
---

## 详细

### 概念
Element 是一套为开发者、设计师和产品经理准备的基于 Vue 的桌面端组件库[reference:0][reference:1]。提供丰富的 UI 组件（按钮、表单、表格、弹窗等），帮助快速搭建企业级中后台应用[reference:2][reference:3]。

- **Element UI**：基于 Vue 2.x[reference:4][reference:5]
- **Element Plus**：Element UI 官方升级版，基于 Vue 3.x[reference:6][reference:7]

### 重点
- **组件丰富**：Element UI 提供 60+ 组件，Element Plus 提供 70+ 组件，覆盖中后台所有常见场景[reference:8][reference:9]
- **Element Plus 核心升级**：
  - 基于 Vue 3 Composition API 重构，性能更优[reference:10][reference:11]
  - 原生 TypeScript 支持，类型定义完善[reference:12][reference:13]
  - 原生支持按需导入和 Tree Shaking，打包体积更小[reference:14]
  - 支持在线主题定制[reference:15]
- **选择建议**：Vue 2 项目用 Element UI，Vue 3 项目用 Element Plus[reference:16][reference:17]
- **安装方式**：`npm install element-ui` 或 `npm install element-plus`[reference:18][reference:19]
- 在 [[Vue工程化]] 项目中，配合 Vue Router 和 Pinia 搭建完整的中后台应用

本项目中使用 **Element Plus**，配合 `<script setup>` 语法和 Composition API，实现高效开发。完整 UI 组件用法见 [[Element Plus 使用指南]]（Web前端设计/Element-Plus.md）。