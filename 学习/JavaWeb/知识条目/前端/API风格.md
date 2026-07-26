---
type: knowledge
title: Vue API 风格
created: 2026-07-24
updated: 2026-07-24
tags:
  - Vue
  - 前端
source: 无
conclusion: Vue 提供选项式 API（Options）和组合式 API（Composition）两种写法，组合式 API 是 Vue 3 的推荐方式，逻辑组织更清晰、TS 支持更好。
---

## 详细

### 概念
Vue 组件逻辑的两种组织方式：选项式 API（Options API）和组合式 API（Composition API）。两者底层实现相同，仅代码组织风格不同。

### 重点
- **Options API（选项式）**：Vue 2 主流方式，代码按 `data`、`methods`、`computed`、`watch` 等选项划分。适合新手，但同一功能逻辑分散在各选项中
- **Composition API（组合式）**：Vue 3 引入，通过 `setup()` 或 `<script setup>` 使用。代码按功能聚合（如全部搜索逻辑写在一起），逻辑复用更优雅
- 推荐使用 **Composition API + `<script setup lang="ts">`**，TypeScript 类型推导更好
- 两者可混用，但同一组件内不建议混用
- 在 [[Vue工程化]] 项目中，推荐统一使用 Composition API + `<script setup>`