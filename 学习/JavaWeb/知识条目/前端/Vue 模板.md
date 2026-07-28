---
type: knowledge
title: Vue 模板语法
created: 2026-07-28
updated: 2026-07-28
tags: [Vue, 模板语法, 前端, V-bind, V-on]
source: Tlias 管理系统前端
conclusion: Vue 模板使用插值 `{{ }}` 和指令 `v-*` 实现数据绑定和事件处理，包括 v-model、v-if、v-for、@click 等核心指令，是连接数据与视图的桥梁。
---

## 详细

### 概念
Vue 模板是描述视图如何基于数据渲染的 HTML 扩展模板。它使用特殊的指令和插值语法，将响应式数据与 DOM 元素绑定，数据变化时视图自动更新。

### 重点
- **文本插值**：`{{ message }}` 在 DOM 中插入响应式数据，数据变化自动更新
- **指令**：以 `v-` 开头的特殊属性，如 `v-if`（条件渲染）、`v-for`（列表渲染）、`v-model`（双向绑定）
- **事件绑定**：`@click` 简写为 `v-on:click`，绑定事件处理函数
- **属性绑定**：`:href` 简写为 `v-bind:href`，动态设置 HTML 属性
- **表达式**：模板中支持有限的 JavaScript 表达式，支持三元运算符、逻辑运算等
