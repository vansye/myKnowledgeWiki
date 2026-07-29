---
type: project
title: popt (Prompt Cache Optimizer)
created: 2026-07-26T10:00:00
updated: 2026-07-26T10:00:00
tags: [Python, HTTP代理, AI, PromptCaching]
status: 已完成
stack: Python, pip, HTTP
link: github.com/vansye/prompt-cache-optimizer
goal: 轻量级 HTTP 透明代理，自动优化消息结构以最大化 KV Cache 命中率，减少 token 消耗
---

## 背景

AI 框架（Claude Code、OpenAI SDK、Anthropic SDK）和模型 API 之间请求结构和上下文重复率高，导致 KV Cache 命中率低、token 浪费严重。popt 是一个轻量级 HTTP 透明代理，插在中间自动优化。

## 功能

- **HTTP 透明代理**：插入 AI 框架与模型 API 之间，无需修改任何代码
- **KV Cache 优化**：自动重组消息结构，最大化 KV Cache 命中
- **一键启动**：`pip install poptimize` → `popt run -- claude`
- **多模型支持**：DeepSeek、GPT-4o、Groq、xAI 等通用格式适配

## 技术关联

与 Qwen_RAG 同属 AI 工具链——popt 优化 AI API 的 token 消耗，Qwen RAG 则是把私人文档喂给 AI。两者配合使用效果最佳。

<!-- KB:ANNOTATIONS -->
