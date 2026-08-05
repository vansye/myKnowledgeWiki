---
type: knowledge
title: Docker Compose 容器编排工具
created: 2026-08-05
updated: 2026-08-05
tags: [Docker, Compose, 容器编排, DevOps]
source: 无
conclusion: Docker Compose 是一个用于定义和运行多容器 Docker 应用程序的工具，通过一个 YAML 文件配置应用服务、网络和卷，实现一键启动整个应用栈。
---

## 详细

### 概念
Docker Compose 是 Docker 官方提供的容器编排工具，用于在单台 Docker 主机上定义和管理多个容器的应用。通过一个 `docker-compose.yml` 文件，可以声明多个服务（容器）、它们之间的依赖关系、网络配置和数据卷挂载，然后使用一条命令启动或停止整个应用栈。

### 重点
- **核心价值**：批量管理多容器应用；环境一致性（开发/测试/生产）；简化启动命令（无需逐个 `docker run`）
- **核心配置文件**：`docker-compose.yml`（YAML 格式）
- **关键配置项**：
  - `services`：定义各个容器（如 web、redis、mysql）
  - `image`：指定镜像
  - `build`：指定 Dockerfile 路径（构建镜像）
  - `ports`：端口映射（宿主机:容器）
  - `environment` / `env_file`：环境变量
  - `volumes`：数据卷挂载
  - `depends_on`：服务依赖关系（启动顺序）
  - `networks`：自定义网络
- **常用命令**：
  - `docker-compose up -d` 启动所有服务（后台运行）
  - `docker-compose down` 停止并移除容器、网络
  - `docker-compose logs -f` 查看日志
  - `docker-compose ps` 查看服务状态
  - `docker-compose exec <服务名> bash` 进入容器
  - `docker-compose restart` 重启服务
- **版本演进**：Compose V1（已弃用，使用 `docker-compose` 命令）→ Compose V2（集成到 Docker CLI，使用 `docker compose` 命令）
- **最佳实践**：使用 `.env` 文件管理敏感环境变量；生产环境使用 `docker-compose.prod.yml` 覆盖配置；通过 `depends_on` 控制启动顺序，但需结合健康检查确保服务就绪；避免将构建上下文设为根目录（`/`）导致构建性能问题