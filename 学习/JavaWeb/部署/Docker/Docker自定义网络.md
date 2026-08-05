---
type: 学习笔记
title: Docker 自定义网络
created: 2026-08-04
updated: 2026-08-04
tags: [Docker, 网络, 自定义网络, 部署, DevOps]
subject: 部署/Docker
---

> Docker 网络让容器之间、容器与宿主机之间、容器与外部网络之间建立通信。理解网络类型、IP 分配、DNS 解析和实际配置方式，是构建可靠多容器应用的基础。

## 目录

- [一、Docker 网络基础概念](#一docker-网络基础概念)
- [二、四种网络驱动详解](#二四种网络驱动详解)
- [三、自定义网络实操](#三自定义网络实操)
- [四、多容器项目网络配置实战](#四多容器项目网络配置实战)
- [五、容器网络常见问题排查](#五容器网络常见问题排查)
- [小结](#小结)

---

## 一、Docker 网络基础概念

### 容器网络工作原理

每个容器启动时，Docker 会为其创建一个虚拟网络接口（veth pair），一端在容器内（eth0），另一端连接到 Docker 创建的网桥上。通过 NAT（网络地址转换）实现容器与外部通信。

```
┌─────────────────────────────────────────────────────┐
│                   宿主机                             │
│                                                     │
│  ┌──────────┐    eth0    ┌──────────┐              │
│  │  容器A   │◄──────────►│  vethA   │              │
│  │ 172.17.0.2 │          │          │              │
│  └──────────┘            │          │              │
│       ▲                  │  docker0 │              │
│       │                  │  (网桥)  │              │
│  ┌──────────┐            │          │              │
│  │  容器B   │◄──────────►│  vethB   │              │
│  │ 172.17.0.3 │          │          │              │
│  └──────────┘            └──────────┘              │
│                        ↕ NAT                       │
│                  宿主机 eth0 (真实网卡)              │
└─────────────────────────────────────────────────────┘
                        ↕
                    外部网络
```

### 三种默认网络

```bash
docker network ls
# 输出示例：
# NETWORK ID   NAME      DRIVER    SCOPE
# abc123       bridge    bridge    local
# def456       host      host      local
# ghi789       none      null      local
```

| 网络类型 | 说明 | 容器间通信 | 访问外部 |
|---------|------|----------|---------|
| **bridge**（默认） | 独立网桥，每个容器有独立网络命名空间 | ❌ 需手动连接 | ✅ |
| **host** | 容器共享宿主机网络栈 | ✅ 直接访问 | ✅ |
| **none** | 无网络 | ❌ | ❌ |

> **关键区别**：默认 bridge 网络中的容器**不能通过容器名互相访问**，必须连到同一个自定义网络才能用 DNS 解析。

---

## 二、四种网络驱动详解

Docker 支持多种网络驱动，最常用的是 `bridge`，其他场景用 `host`、`overlay`、`macvlan`。

### 1. bridge（桥接网络）— 单机多容器

**适用场景**：单机上运行多个容器，需要容器间通信。

```bash
# 创建自定义 bridge 网络
docker network create --driver bridge my-net

# 创建容器时指定网络
docker run -d --name web --network my-net nginx
docker run -d --name app --network my-net myapp

# 也可以运行后连接
docker network connect my-net another-container
```

**特性**：
- 每个容器有独立的 IP（如 172.18.0.2、172.18.0.3）
- 容器间可通过**容器名**互相访问（内置 DNS）
- 容器可通过 `宿主机IP:映射端口` 访问
- 支持端口映射（`-p`）

### 2. host（主机网络）— 性能优先

**适用场景**：对网络性能要求高、不需要隔离的网络命名空间。

```bash
docker run -d --name myapp --network host nginx
```

**特性**：
- 容器直接共享宿主机网络栈，无 NAT 开销
- 容器进程直接监听宿主机端口，无需 `-p` 映射
- 容器间可以直接通过 `127.0.0.1` 通信
- **缺点**：端口冲突风险（两个容器都监听 80 会冲突）

```bash
# 验证 host 网络
docker run -d --name test-host --network host nginx
curl http://127.0.0.1:80    # 直接访问，无需端口映射
```

### 3. overlay（覆盖网络）— 多主机容器通信

**适用场景**：Docker Swarm 或多主机容器集群，容器分布在不同的物理机上。

```bash
# 需要 Docker Swarm 模式
docker network create -d overlay my-overlay-net

# 在 Swarm 中运行的服务自动加入 overlay 网络
docker service create --name web --network my-overlay-net nginx
docker service create --name app --network my-overlay-net myapp
```

**特性**：
- 跨主机的容器可以互相通信
- 基于 VXLAN 隧道封装
- 需要 Docker Swarm 或 Kubernetes 支持

### 4. macvlan（MAC 地址虚拟）— 容器拥有独立 IP

**适用场景**：容器需要像物理设备一样接入现有局域网，有独立 IP。

```bash
# 创建 macvlan 网络
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  macnet

# 启动容器，分配固定 IP
docker run -d --name web --network macnet --ip 192.168.1.100 nginx
```

**特性**：
- 容器获得局域网中的独立 IP
- 容器像是网络中的物理设备
- **缺点**：配置复杂，与宿主机网络深度耦合

---

## 三、自定义网络实操

### 1. 创建网络（带参数）

```bash
# 基础创建
docker network create my-net

# 指定网段（避免与默认 bridge 冲突）
docker network create \
  --subnet=172.20.0.0/16 \
  --gateway=172.20.0.1 \
  my-net

# 指定驱动 + 子网
docker network create \
  --driver bridge \
  --subnet=10.0.0.0/24 \
  --gateway=10.0.0.1 \
  my-net

# 禁用容器间默认通信（安全隔离）
docker network create --internal secure-net
```

### 2. 网络参数说明

| 参数 | 作用 |
|------|------|
| `--subnet` | 指定 IP 子网范围 |
| `--gateway` | 指定网关 IP |
| `--driver` | 指定网络驱动（bridge/host/overlay/macvlan） |
| `--internal` | 隔离网络，容器无法访问外部 |
| `--dns` | 指定 DNS 服务器 |
| `--dns-search` | 指定 DNS 搜索域 |
| `-o parent` | macvlan 模式下指定宿主机网卡 |

### 3. 连接/断开容器

```bash
# 启动时指定网络
docker run -d --name web --network my-net nginx

# 运行后连接到已有网络
docker network connect my-net web

# 连接到多个网络（容器可以有多个网络接口）
docker network connect my-net app
docker network connect external-net app

# 断开网络连接
docker network disconnect my-net web

# 强制断开（容器运行时也可断开）
docker network disconnect -f my-net web
```

> **多网络容器**：一个容器可以连接到多个网络，每个网络获得一个 IP，适合网关/代理场景。

### 4. 查看网络详情

```bash
# 查看所有网络
docker network ls

# 查看网络详细信息（IP 分配、连接的容器）
docker network inspect my-net

# 只看某个容器的网络信息
docker network inspect my-net --format '{{range .Containers}}{{.Name}}: {{.IPv4Address}}{{end}}'

# 查看容器的网络配置
docker inspect web --format '{{json .NetworkSettings.Networks}}' | jq
```

### 5. 删除网络

```bash
docker network rm my-net              # 删除空网络（无容器连接）
docker network prune                  # 删除所有未使用的网络
docker network rm $(docker network ls -q)  # 删除所有网络（⚠️ 谨慎）
```

> 如果网络中还有容器连接，需要先断开或删除容器才能删除网络。

---

## 四、多容器项目网络配置实战

### 场景一：前后端分离项目（单机）

```bash
# 1. 创建专属网络
docker network create myapp-net

# 2. 启动 MySQL（不暴露端口给外部，只供后端访问）
docker run -d \
  --name mysql \
  --network myapp-net \
  -e MYSQL_ROOT_PASSWORD=Root@123456 \
  -e MYSQL_DATABASE=myapp \
  -v /data/mysql:/var/lib/mysql \
  mysql:8.0

# 3. 启动后端（连接 myapp-net，通过容器名访问 MySQL）
docker run -d \
  --name backend \
  --network myapp-net \
  -p 8080:8080 \
  -e DB_HOST=mysql \
  -e DB_PORT=3306 \
  myapp-backend:1.0.0

# 4. 启动前端 Nginx（连接 myapp-net，通过容器名访问后端）
docker run -d \
  --name frontend \
  --network myapp-net \
  -p 80:80 \
  myapp-frontend:1.0.0
```

**关键点**：
- 后端配置 `DB_HOST=mysql`（容器名），而非 `127.0.0.1` 或 IP
- MySQL 不映射端口（`-p`），只通过内部网络访问，更安全
- 前端 Nginx 配置反向代理 `proxy_pass http://backend:8080/`

### 场景二：带 Redis 缓存的全栈项目

```bash
# 创建网络
docker network create fullstack-net

# MySQL
docker run -d --name mysql \
  --network fullstack-net \
  -e MYSQL_ROOT_PASSWORD=Root@123456 \
  mysql:8.0

# Redis
docker run -d --name redis \
  --network fullstack-net \
  redis:7-alpine

# 后端（访问 mysql 和 redis）
docker run -d --name backend \
  --network fullstack-net \
  -p 8080:8080 \
  -e REDIS_HOST=redis \
  myapp-backend:1.0.0

# 前端（代理到 backend）
docker run -d --name frontend \
  --network fullstack-net \
  -p 80:80 \
  myapp-frontend:1.0.0
```

### 场景三：隔离外部访问的内部服务

```bash
# 创建内部网络（无外部访问）
docker network create --internal db-net

# 数据库只在内部网络，外部无法直接访问
docker run -d --name mysql \
  --network db-net \
  -e MYSQL_ROOT_PASSWORD=Root@123456 \
  mysql:8.0

# 后端连接内部网络访问数据库
docker run -d --name backend \
  --network db-net \
  --network fullstack-net \    # 同时连接两个网络
  -p 8080:8080                 # 只暴露给外部网络
  myapp-backend:1.0.0
```

> 这种配置下，MySQL 只被后端容器访问，外部无法直接连接数据库，安全性更高。

---

## 五、容器网络常见问题排查

### 问题 1：容器间无法通过名称互相访问

```bash
# 检查是否在同一个网络
docker network inspect my-net --format '{{range .Containers}}{{.Name}}{{"\n"}}{{end}}'

# 检查容器连接的网络
docker inspect web --format '{{range $net, $config := .NetworkSettings.Networks }}{{$net}}{{"\n"}}{{end}}'

# 解决：确保容器连到同一个网络
docker network connect my-net web
```

### 问题 2：容器无法访问外网

```bash
# 检查 DNS 配置
docker inspect web --format '{{.Config.DNS}}'

# 检查容器的 DNS 解析
docker exec web cat /etc/resolv.conf
docker exec web nslookup baidu.com

# 临时指定 DNS
docker run -d --name web --dns 8.8.8.8 nginx
```

### 问题 3：端口冲突

```bash
# 查看端口占用情况
sudo lsof -i :8080
netstat -tlnp | grep 8080

# 解决：修改映射端口或停止占用进程
docker run -d -p 8081:8080 --name web nginx
```

### 问题 4：容器内 curl 不通宿主机

```bash
# 在容器中，宿主机地址不是 127.0.0.1
# bridge 网络下，宿主机内网 IP 是 172.17.0.1
curl http://172.17.0.1:8080

# 或者使用宿主机实际网卡 IP
docker inspect bridge --format '{{.IPAM.Config[0].Gateway}}'
```

### 问题 5：查看容器实际 IP

```bash
# 方法一：inspect
docker inspect web --format '{{.NetworkSettings.IPAddress}}'

# 方法二：进入容器查看
docker exec web ip addr

# 方法三：在容器内
docker exec web hostname -I
```

### 问题 6：默认 bridge 网络 vs 自定义网络的区别

| 对比项 | 默认 bridge | 自定义 bridge |
|--------|------------|--------------|
| 创建方式 | Docker 自动创建 | 手动 `docker network create` |
| 容器名 DNS 解析 | ❌ 不支持 | ✅ 支持 |
| 手动连接 | 需要 `docker network connect` | 启动时 `--network` 指定 |
| 隔离性 | 所有默认网络容器在同一网桥 | 可按项目隔离 |
| 适用场景 | 简单单容器 | 多容器项目 |

> **建议**：多容器项目一律使用自定义网络，不要依赖默认 bridge。

---

## 小结

| 要点 | 说明 |
|------|------|
| **bridge** | 最常用，单机多容器，支持 DNS 解析 |
| **host** | 性能最好，无网络隔离，端口不能重复 |
| **overlay** | 多主机容器集群，需 Swarm/K8s |
| **macvlan** | 容器获得局域网独立 IP，配置复杂 |
| **自定义网络 DNS** | 容器间用**容器名**互相访问，无需记 IP |
| **多网络容器** | 一个容器可连多个网络，适合网关/代理 |
| **安全隔离** | `--internal` 网络阻断外部访问，数据库放里面 |

> 网络配置的核心原则：**同项目容器放同一自定义网络，敏感服务（数据库）不暴露端口给外部**。
