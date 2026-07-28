---
type: knowledge
title: Vue 组件状态
created: 2026-07-28
updated: 2026-07-28
tags:
  - Vue
  - 状态
  - data
  - ref
  - reactive
source: 无
conclusion: State 是 Vue 组件内部管理的响应式数据，用于驱动视图渲染。通过 data（Vue 2）或 ref/reactive（Vue 3）定义，变化时视图自动更新。
---

## 详细

### 概念
State（状态）是组件内部拥有的数据，用于描述组件的当前状况。Vue 中状态是响应式的——状态变化时，依赖该状态的视图部分会自动重新渲染。

### 重点
- **与 Props 的区别**：Props 由父组件传入（只读），State 由组件自身定义（可读写）
- **Vue 2 写法**：`data()` 函数返回对象，Vue 通过 `Object.defineProperty` 将其转为响应式
- **Vue 3 写法**：使用 `ref()` 或 `reactive()` 创建响应式状态
- **状态驱动视图**：模板中直接使用状态变量，无需手动操作 DOM
- **响应式条件**：只有响应式状态变化才会触发更新，普通变量不会
- **组件私有**：State 默认只在当前组件内有效，子组件无法直接访问（需要通过 Props 传递）