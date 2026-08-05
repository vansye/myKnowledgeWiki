---
type: knowledge
title: Docker 挂载（Mount）存储机制
created: 2026-08-05
updated: 2026-08-05
tags: [Docker, 挂载, 存储, Volume, Bind Mount]
source: 无
conclusion: Docker 挂载是将外部存储（宿主机目录、数据卷或内存）接入容器内部文件系统的机制，是容器数据持久化和动态配置的核心手段。
---

## 详细

### 概念
Docker 挂载（Mount）是指将宿主机文件系统中的目录或数据卷挂载到容器内部指定路径，使容器可以读写外部存储。通过挂载，容器可以获得持久化能力、共享数据能力以及与宿主机实时同步的能力。

### 重点
- **三种挂载类型**：

  | 类型 | 存储位置 | 适用场景 |
  | :--- | :--- | :--- |
  | **Bind Mount** | 宿主机任意路径 | 开发热加载、配置文件注入 |
  | **Volume** | Docker 管理（`/var/lib/docker/volumes/`） | 生产数据持久化、容器间共享 |
  | **tmpfs** | 宿主机内存 | 临时敏感数据（不落盘） |
- **挂载语法**：
  - `-v /宿主机路径:/容器路径`（旧式，支持 Bind Mount 和 Volume）
  - `--mount type=bind,source=/宿主机路径,target=/容器路径`（推荐，更明确）
  - `--mount type=volume,source=myvol,target=/容器路径`
  - `--mount type=tmpfs,target=/容器路径`
- **关键特性**：
  - 容器内挂载点会被外部内容覆盖
  - 对挂载目录的修改实时双向同步（tmpfs 除外）
  - 容器删除后，Bind Mount 和 tmpfs 数据不保留（除非主动清理），Volume 数据保留
- **选择建议**：
  - 生产数据 → **Volume**
  - 开发调试 → **Bind Mount**
  - 敏感临时数据 → **tmpfs**