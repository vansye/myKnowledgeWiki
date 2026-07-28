---
type: knowledge
title: Element Plus UI 组件库
created: 2026-07-28
updated: 2026-07-28
tags: [Element Plus, UI 组件库, Vue3, 前端]
source: Tlias 管理系统前端
conclusion: Element Plus 是饿了么团队推出的 Vue3 专属 UI 组件库，提供按钮、表格、表单、弹窗等 70+ 预制组件，中后台开发开箱即用。
---

## 详细

### 概念
Element Plus 是饿了么前端团队为 Vue3 开发的 UI 组件库，提供了一套完整的桌面端组件解决方案。它包含按钮、输入框、表格、对话框、导航等常见 UI 元素的预制组件，开发者可以直接使用 `<el-button>`、`<el-table>` 等标签构建界面。

### 重点
- **组件丰富**：提供 70+ 组件，覆盖中后台所有常见场景（按钮、表格、表单、日期选择器、通知等）
- **Vue3 专属**：基于 Vue3 Composition API 和 `<script setup>` 语法重构，性能更优
- **主题定制**：支持在线自定义主题色，一键下载修改后的 CSS
- **安装方式**：`npm install element-plus`，在 main.js 中 `app.use(ElementPlus)` 注册
- **引入样式**：必须引入 `element-plus/dist/index.css`，否则组件没有样式
