---
type: project
title: GSD Skill
created: 2026-07-26T10:00:00
updated: 2026-07-26T10:00:00
tags:
  - Skill
  - 开发流程
status: 进行中
stack: Claude Code Skills
link: github.com/vansye/gsd.skill
goal: Claude Code 技能 — 审查变更中的复用、简化、效率和清理，然后应用修复
---

## 背景

代码审查后发现的复用、简化、效率问题和风格不一致需要系统化修复。

## 功能

审查和修复以下问题：
- **复用**（Reuse）：是否有重复代码可以提取为公共函数/模块
- **简化**（Simplification）：是否可以减少嵌套层级、合并逻辑
- **效率**（Efficiency）：是否有冗余计算或低效算法
- **Altitude cleanups**：格式、注释一致性等细节

## 技术关联

与 [[LoopEngineer]] 互补——GSD 修复已有变更中的问题，LoopEngineer 控制变更本身的质量。两者都是 Claude Code Skill。

<!-- KB:ANNOTATIONS -->
