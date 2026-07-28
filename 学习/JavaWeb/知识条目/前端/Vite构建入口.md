---
type: knowledge
title: Vite 构建入口
created: 2026-07-28
updated: 2026-07-28
tags: [Vite, 构建工具, index.html, 前端]
source: Tlias 管理系统前端
conclusion: Vite 以 index.html 为构建入口，必须手写 `<script type="module" src="/src/main.js">` 标签，缺失会导致 JS 完全不加载、页面白屏且控制台无任何报错。
---

## 详细

### 概念
Vite 与传统打包工具（Webpack / Vue CLI）最大的结构差异在于**入口的定义方式**。在 Vite 项目中，`index.html` 本身就是入口，Vite 扫描其中的 `<script type="module">` 标签找到 JS 入口文件，再递归分析依赖树。因此在 Vite 项目里，这个标签是**必须手写**的。

### 重点
- **缺失后果**：Vite 找不到任何 JS 入口 → `main.js` 从不执行 → `createApp().mount('#app')` 从不调用 → **页面白屏，且控制台无任何报错**（代码没加载，自然不会报错）
- **典型三特征**：标题正常显示 + 页面空白 + 控制台干净，三者同时出现基本可锁定此问题
- **路径写法**：`src` 用**绝对路径** `/src/main.js`（相对于项目根目录），不要写 `./src/main.js`
- **`type="module"` 不可省略**：Vite 开发服务器基于原生 ES Module，去掉它浏览器会把文件当普通脚本处理，`import` 语句直接报错
