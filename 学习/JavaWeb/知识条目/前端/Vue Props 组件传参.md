---
type: knowledge
title: Vue Props 组件传参
created: 2026-07-28
updated: 2026-07-28
tags: [Vue, 组件通信, Props, 前端]
source: 无
conclusion: Props 是 Vue 中父组件向子组件传递数据的核心机制，遵循单向数据流原则，子组件不能直接修改父组件传入的 props。
---

## 详细

### 概念
Props（Properties）是 Vue 组件用于接收父组件传递数据的自定义属性。父组件通过模板中的属性绑定将数据传给子组件，子组件通过 `defineProps`（Vue 3）或 `props` 选项（Vue 2）声明接收。

### 重点
- **传递方向**：父 → 子，单向数据流
- **子组件不可修改**：props 是只读的，修改会触发警告
- **类型验证**：支持 String、Number、Boolean、Array、Object、Function 等
- **必填与默认值**：可设置 `required: true` 或 `default` 默认值
- **Vue 3 写法**：`<script setup>` 中使用 `defineProps<{ msg: string }>()` 或 `defineProps(['msg'])`
- **Vue 2 写法**：`props: { msg: { type: String, required: true } }`
- **驼峰 vs 短横线**：Props 名在 JavaScript 中用驼峰，模板中用短横线（如 `userName` → `user-name`）