---
type: knowledge
title: localStorage 本地存储
created: 2026-07-29
updated: 2026-07-29
tags: [JavaScript, 浏览器, 存储, WebAPI]
source: 无
conclusion: localStorage 是浏览器提供的持久化本地存储 API，以键值对形式存储字符串数据，容量约 5-10MB，数据在浏览器关闭后仍然保留。
---

## 详细

### 概念
localStorage 是 Web Storage API 的一部分，用于在浏览器端持久化存储数据。数据以字符串键值对形式存储，无过期时间，除非主动清除否则永久保留。

### 重点
- **存储位置**：浏览器本地，按域名隔离（同源策略）
- **存储容量**：通常 5~10MB（不同浏览器略有差异）
- **数据类型**：只能存储字符串，对象/数组需用 `JSON.stringify()` 转换，读取时用 `JSON.parse()` 还原
- **生命周期**：永久存储，除非手动清除或用户清理浏览器数据
- **同步操作**：API 是同步的，大数据量操作可能阻塞主线程
- **常用 API**：
  - `setItem(key, value)`：存入数据
  - `getItem(key)`：读取数据
  - `removeItem(key)`：删除指定键
  - `clear()`：清空所有数据
  - `key(index)`：根据索引获取键名
- **与 sessionStorage 对比**：sessionStorage 仅在当前会话（标签页）有效，关闭即清除；localStorage 永久保留
- **常见场景**：记住用户偏好设置、主题切换、登录状态保持、本地缓存接口数据
- **注意事项**：不适合存储敏感信息（如密码、Token）；与 Vue 响应式结合时，需手动将变化同步到 localStorage（如使用 `watch` 监听）