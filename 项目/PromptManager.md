---
type: project
title: PromptManager 浏览器插件
created: 2026-07-26T10:00:00
updated: 2026-07-26T10:00:00
tags: [前端, TypeScript, React, 浏览器扩展]
status: 已完成
stack: React 18, Vite 5, TypeScript 5, Dexie + IndexedDB, lucide-react
link: github.com/vansye/PromotHero.AI
goal: 提示词管理浏览器扩展，支持分类、检索、编辑、复制，数据存本地 IndexedDB
---

## 背景

日常使用 AI 时提示词分散在笔记、聊天历史各处，需要快速查找和复用，但缺乏轻量级管理工具。

## 功能

- **提示词 CRUD**：新增、编辑、删除、复制
- **变量识别**：自动识别 `{{变量名}}` 格式
- **分类管理**：分类创建、筛选、计数
- **多端兼容**：侧边栏 + 弹出窗口两种打开方式（暗色、高密度 UI）
- **本地存储**：数据保存在 IndexedDB，不依赖服务器
- **已上架**：Microsoft Edge Addons 商店

## 技术栈

React 18 + Vite 5 + TypeScript 5 + Dexie (IndexedDB ORM)

## 技术关联

使用的技术栈与 Vue工程化（Vite + 组件化）和 Qwen_RAG（Streamlit Web UI）形成对照——前端框架各有侧重，React 生态 vs Vue 生态。

<!-- KB:ANNOTATIONS -->
