---
type: knowledge
title: JavaScript 模块导入导出
created: 2026-07-28
updated: 2026-07-28
tags: [JavaScript, 模块, ESM, CommonJS]
source: 无
conclusion: 模块导入导出用于拆分和复用代码。主流分 ESM（官方标准）和 CommonJS（Node.js 传统），ESM 是现代化项目的推荐方案。
---

## 详细

### 概念
模块导入导出是将代码拆分到独立文件，通过 `export` 暴露接口、`import` 引入功能的机制。它解决了全局污染、依赖管理和代码复用问题。

### 重点

#### 一、ES Module（官方标准）

| 导出方式 | 语法 | 说明 |
| :--- | :--- | :--- |
| 默认导出 | `export default value` | 每模块一个，导入可自定义名称 |
| 命名导出 | `export const foo = ...` | 多个，导入需同名 |
| 重命名导出 | `export { a as b }` | 导出时改名 |

| 导入方式 | 语法 | 说明 |
| :--- | :--- | :--- |
| 默认导入 | `import X from './mod.js'` | 匹配默认导出 |
| 命名导入 | `import { foo } from './mod.js'` | 匹配命名导出 |
| 重命名导入 | `import { foo as f } from './mod.js'` | 导入时改名 |
| 全部导入 | `import * as obj from './mod.js'` | 挂载到对象上 |
| 仅执行 | `import './mod.js'` | 不取值，只执行模块 |

#### 二、CommonJS（Node.js 传统）

| 导出方式 | 语法 | 说明 |
| :--- | :--- | :--- |
| 导出单个 | `module.exports = value` | 整体替换导出 |
| 导出多个 | `exports.foo = value` | 挂在 exports 对象上 |

| 导入方式 | 语法 | 说明 |
| :--- | :--- | :--- |
| 标准导入 | `const mod = require('./mod.js')` | 获得整个模块 |

#### 三、示例

```javascript
// ----- user.js (ESM) -----
export default function getUser() { return 'Tom' }
export const version = '1.0'

// ----- app.js -----
import getUser, { version } from './user.js'

// ----- user.cjs (CommonJS) -----
module.exports = { getUser: () => 'Tom', version: '1.0' }

// ----- app.cjs -----
const { getUser, version } = require('./user.cjs')