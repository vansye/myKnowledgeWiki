---
type: knowledge
title: script setup 语法
created: 2026-07-28
updated: 2026-07-28
tags: [Vue3, Composition API, script setup, 前端]
source: Tlias 管理系统前端
conclusion: <script setup> 是 Vue3 的语法糖，无需 export default，模板中直接使用声明的变量和方法。所有顶层变量自动暴露到模板，无需手动 return，代码更简洁。
---

## 详细

### 概念
`<script setup>` 是 Vue3 单文件组件的编译时语法糖，所有顶层声明自动在模板中可用，无需 `export default` 和手动 `return`，代码更简洁。

### 重点
- **自动暴露**：顶层变量和函数直接在模板中使用，无需 return
- **无需 export**：传统 Options API 需 export default，<script setup> 直接写
- **导入组件自动可用**：直接导入的 Vue 组件（如 ElButton）在模板中可用
- **组合式 API 天然支持**：完美配合 ref、reactive、生命周期钩子等 Composition API
