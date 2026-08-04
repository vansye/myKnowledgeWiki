---
type: knowledge
title: Docker 容器
created: 2026-08-04
updated: 2026-08-04
tags: [Docker, 容器, 虚拟化, DevOps]
source: 无
conclusion: Docker 容器是镜像的运行实例，提供了轻量级的隔离运行环境，是 Docker 技术体系中真正执行业务逻辑的运行单元。
---

## 详细

### 概念
Docker 容器是 Docker 镜像的一个可运行实例。它通过 Namespace 实现资源隔离（如 PID、Network、Mount），通过 Cgroups 限制资源使用（如 CPU、内存），在共享宿主机内核的前提下提供独立的运行环境。

### 重点
- **与镜像的关系**：镜像是静态的只读模板（相当于类），容器是动态的可运行实例（相当于对象）
- **核心操作命令**：
  - `docker run -d --name nginx -p 80:80 nginx` 创建并启动容器
  - `docker ps -a` 列出所有容器（含停止的）
  - `docker start/stop/restart <容器名>` 控制容器状态
  - `docker rm <容器名>` 删除容器（需先停止）
  - `docker exec -it <容器名> bash` 进入容器内部执行命令
  - `docker logs <容器名>` 查看容器日志
- **主要状态转换**：已创建 → 运行中 → 已暂停 → 已停止 → 已删除
- **网络模式**：
  - `bridge`（默认）：容器使用虚拟网桥与宿主机通信
  - `host`：容器直接使用宿主机网络栈
  - `none`：无网络配置
  - `container:<容器名>`：与指定容器共享网络
- **数据持久化**：容器被删除后数据会丢失，需通过 Volume（卷）或 Bind Mount（绑定挂载）将数据持久化到宿主机
- **资源限制**：`--memory="512m"`、`--cpus="1.5"` 限制容器资源使用
- **常用命令**：`docker run -d` 后台运行，`docker run -it` 交互式运行，`docker container prune` 清理停止的容器
- **注意事项**：容器内的进程应以前台模式运行（不推荐后台守护进程），避免容器启动后立即退出；避免在容器内存储持久化数据