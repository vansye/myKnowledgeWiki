---
type: project
title: Loop Engineer
created: 2026-07-26T10:00:00
updated: 2026-07-26T10:00:00
tags: [开发流程, Claude Code, Skill, AI编程]
status: 进行中
stack: Claude Code Skills, Shell, Git
link: github.com/vansye/loop-engineer
goal: Claude Code 的规划型 skill，把"让 AI 改代码"变成可验证的闭环节水线
---

## 背景

AI 改代码频繁翻车：一次改太多找不到 bug、改了"看起来没问题"但一跑就崩、失败了只能 git checkout . 全丢。

## 功能

- **目标分层**：每轮只改一个最小可验证目标（10-30 分钟），不改完不进下一轮
- **Harness 验证**：每个改动绑定一条 shell 命令，验证通过才算完成
- **分层回滚**：根据场景自动选择 git_stash / git_revert / manual_prompt，不丢未提交工作
- **持久化状态**：`_loop-state.yaml` 记录进度，重启自动恢复
- **Gate Check 硬拒绝**：目标树不达标直接拒绝，防止跑错方向

## 核心工作流

```
接收目标树 → Gate Check 验收 → 设计 Harness → 生成 Loop 方案
```

## 技术关联

与 GSD_Skill（代码审查修复）和 PromptOptimizer（popt / KV Cache 优化）同属 Claude Code 生态，一个管流程闭环，一个管 token 节省。

<!-- KB:ANNOTATIONS -->
