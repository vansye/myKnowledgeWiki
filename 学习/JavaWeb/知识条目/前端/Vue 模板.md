---
type: knowledge
title: Vue 模板语法
created: 2026-07-24
updated: 2026-07-24
tags: [Vue, 模板, 前端]
source: 无
conclusion: Vue 模板语法是一种基于 HTML 的声明式绑定方式，核心是插值（{{ }}）和指令（v-*），用于将数据渲染到 DOM。
---

## 详细

### 概念
Vue 模板语法是一种基于 HTML 的声明式绑定语法，用于将数据渲染到 DOM[reference:0][reference:1]。所有 Vue 模板都是合法的 HTML，可被浏览器正常解析[reference:2][reference:3]。

### 重点
- **插值（{{ }}）**：用于标签文本内容，支持 JavaScript 表达式[reference:4][reference:5]
- **指令（v-*）**：带 `v-` 前缀的特殊属性，用于属性绑定、事件监听、条件/列表渲染等[reference:6][reference:7]
- **v-bind（:）**：动态绑定 HTML 属性，插值不能用于属性[reference:8][reference:9]
- **v-on（@）**：绑定 DOM 事件[reference:10][reference:11]
- **v-if / v-show**：条件渲染[reference:12]
- **v-for**：列表渲染，需加 `:key`[reference:13]
- **v-model**：表单双向绑定[reference:14]
- **表达式限制**：模板中仅支持单个 JavaScript 表达式，不支持语句[reference:15][reference:16]
- **动态参数**：`v-bind:[attrName]` 可动态绑定属性名[reference:17]