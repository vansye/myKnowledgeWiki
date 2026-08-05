---
type: knowledge
title: Dockerfile 镜像构建文件
created: 2026-08-05
updated: 2026-08-05
tags: [Docker, Dockerfile, 镜像构建, DevOps]
source: 无
conclusion: Dockerfile 是一个文本文件，包含构建 Docker 镜像所需的指令序列，通过 `docker build` 命令可自动逐层执行这些指令生成镜像，是实现应用容器化的核心配置文件。
---

## 详细

### 概念
Dockerfile 是一个纯文本文件，其中包含了一系列构建指令和参数，用于自动化地构建一个 Docker 镜像。每条指令对应镜像的一层，`docker build` 命令按顺序解析并执行这些指令，生成最终的可运行镜像。

### 重点
- **核心指令**：
  - `FROM`：指定基础镜像（必须为第一条指令）
  - `RUN`：在构建时执行命令（如安装软件包、下载文件）
  - `COPY`：从宿主机复制文件/目录到镜像内
  - `ADD`：类似 COPY，额外支持自动解压 tar 和远程 URL
  - `WORKDIR`：设置工作目录（后续命令基于此路径）
  - `ENV`：设置环境变量
  - `EXPOSE`：声明容器运行时监听的端口（仅文档说明，不实际映射）
  - `CMD`：容器启动时执行的默认命令（可被 `docker run` 参数覆盖）
  - `ENTRYPOINT`：容器启动时执行的固定命令（不易被覆盖）
  - `VOLUME`：声明匿名卷（用于持久化）
  - `ARG`：定义构建时变量（`--build-arg` 传入）

- **基本示例**：
  ```dockerfile
  FROM node:18-alpine
  WORKDIR /app
  COPY package*.json ./
  RUN npm install
  COPY . .
  EXPOSE 3000
  CMD ["node", "server.js"]