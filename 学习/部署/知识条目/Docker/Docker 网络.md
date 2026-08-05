---
type: knowledge
title: Docker 网络
created: 2026-08-05
updated: 2026-08-05
tags: [Docker, 网络, 容器, 运维]
source: 无
conclusion: Docker 网络是容器间通信以及与外部通信的基础设施，提供多种网络驱动以满足不同场景，其中 bridge 是单机默认模式，overlay 用于跨主机通信。
---

## 详细

### 概念
Docker 网络是连接容器、宿主机和外部网络的基础设施层。它通过虚拟网桥、网卡和路由规则，为容器提供网络隔离和通信能力。Docker 在安装时会自动创建三个默认网络：`bridge`、`host` 和 `none`。

### 重点
- **网络驱动模式**：
  - **bridge（桥接）**：默认模式。容器间通过 docker0 网桥通信，容器拥有独立网络命名空间。
  - **host（主机）**：容器共享宿主机网络命名空间，无性能损耗，但端口冲突风险高。
  - **none（无网络）**：容器无网络接口，完全隔离，适合不需要网络的场景。
  - **container（容器共享）**：新容器与指定容器共享网络。
  - **overlay（覆盖网络）**：跨多台 Docker 主机的容器通信（需 Swarm 或第三方键值存储），用于集群部署。
- **常用命令**：
  - `docker network ls` 列出网络
  - `docker network create --driver bridge mynet` 创建自定义桥接网络
  - `docker run --network mynet nginx` 指定网络启动容器
  - `docker network connect mynet containerA` 将容器接入网络
  - `docker network inspect bridge` 查看网络详情
- **端口映射**：`docker run -p 宿主机端口:容器端口` 将容器端口暴露到宿主机
- **容器间通信方式**：
  - 通过 IP 直接访问（需先获取容器 IP）
  - 通过**容器名**访问（自定义 bridge 网络内置 DNS 解析）
  - 通过 `--link` 连接（传统方式，不推荐）
- **最佳实践**：
  - 优先使用**自定义 bridge 网络**而非默认 bridge，支持容器名 DNS 解析
  - 按应用分组创建独立网络，实现网络隔离（如 `frontend`、`backend`）
  - 生产环境使用 `overlay` 网络实现跨主机通信