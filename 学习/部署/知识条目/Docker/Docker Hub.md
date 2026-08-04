---
type: knowledge
title: Docker Hub 容器镜像仓库
created: 2026-08-04
updated: 2026-08-04
tags: [Docker, 容器, 镜像仓库, DevOps]
source: 无
conclusion: Docker Hub 是 Docker 官方提供的全球最大容器镜像仓库，用于存储、管理和共享镜像，是容器化开发的核心组件。
---

## 详细

### 概念
Docker Hub 是一个基于云的容器镜像仓库服务。它是 Docker 默认的公共镜像仓库，开发者可以从中拉取预构建的镜像，也可以推送自己构建的镜像进行分享。

### 重点
- **核心功能**：海量公共镜像（官方+社区）；私有仓库（付费）；自动化构建；Webhook；可信内容认证。
- **关键概念**：Registry（注册中心）、Repository（仓库）、Tag（标签）。
- **基本操作**：`docker login`、`docker search`、`docker pull`、`docker tag`、`docker push`。
- **安全最佳实践**：优先使用官方镜像；生产环境用私有仓库或镜像加速器；使用 Access Token；注意匿名拉取限流。
- **国内访问**：建议配置镜像加速器（如阿里云、中科大），企业可部署私有 Harbor + 官方仓库。