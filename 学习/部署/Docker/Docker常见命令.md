---
type: 学习笔记
title: Docker 常见命令
created: 2026-08-04
updated: 2026-08-04
tags: [Docker, 容器, 命令, 部署, DevOps]
subject: 部署/Docker
---
![[Pasted image 20260804211448.png]]
> Docker 命令围绕三个核心对象展开：**镜像（Image）**、**容器（Container）**、**仓库（Registry）**。掌握这三类命令，即可覆盖日常 90% 的使用场景。

## 目录

- [一、镜像操作命令](#一镜像操作命令)
- [二、容器操作命令](#二容器操作命令)
- [三、数据卷（Volume）命令](#三数据卷volume命令)
- [四、网络操作命令](#四网络操作命令)
- [五、常用组合与实用技巧](#五常用组合与实用技巧)
- [小结](#小结)

---

## 一、镜像操作命令

镜像是容器的模板，所有容器都从镜像启动。

### 1. 查看镜像

```bash
docker images                    # 查看所有本地镜像
docker images -a                 # 查看所有镜像（含中间层）
docker images --digests          # 显示镜像摘要
docker images nginx              # 只看 nginx 相关镜像
docker images --format 'table {{.Repository}}\t{{.Tag}}\t{{.Size}}'  # 格式化输出
```

### 2. 拉取镜像

```bash
docker pull nginx                         # 拉取最新稳定版
docker pull nginx:latest                  # 明确拉取最新版
docker pull nginx:1.24                    # 拉取指定版本
docker pull docker.io/library/nginx:1.24  # 完整路径格式
docker pull mysql:8.0                     # 拉取 MySQL 8.0
```

> `docker pull` 不指定版本时默认拉取 `:latest`，生产环境建议明确指定版本号，避免不可预期的更新。

### 3. 搜索镜像

```bash
docker search nginx          # 搜索镜像，按 stars 排序
docker search mysql --stars=10   # 只搜 star 数 ≥ 10 的
docker search nginx --limit=5  # 限制返回数量
```

### 4. 删除镜像

```bash
docker rmi nginx             # 删除指定镜像
docker rmi nginx:1.24        # 删除指定版本
docker rmi <IMAGE_ID>        # 按 ID 删除
docker rmi $(docker images -q)  # 删除所有镜像（⚠️ 谨慎）
```

> 如果镜像正在被容器使用，需要先 `docker rm` 删除容器再删镜像，或用 `docker rmi -f` 强制删除。

### 5. 查看镜像详情

```bash
docker inspect nginx                    # 查看镜像的完整配置信息
docker inspect nginx --format='{{.RepoTags}}'   # 只看标签
docker history nginx                    # 查看镜像的分层构建历史
```

### 6. 构建镜像（从 Dockerfile）

```bash
docker build -t myapp:1.0 .            # 在当前目录构建，标签为 myapp:1.0
docker build -t myapp:1.0 -f Dockerfile.prod .  # 指定 Dockerfile 路径
docker build --no-cache -t myapp:1.0 .  # 不使用缓存重新构建
```

---

## 二、容器操作命令

容器是镜像的运行实例，日常操作最多的一类。

### 1. 启动容器

```bash
# 最简启动（前台运行，Ctrl+C 退出）
docker run nginx

# 后台运行 + 端口映射（最常用）
docker run -d -p 80:80 nginx

# 端口映射 + 挂载数据卷 + 设置环境变量
docker run -d \
  -p 3306:3306 \
  -v /data/mysql:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  --name mysql8 \
  mysql:8.0

# 常用参数说明
# -d        后台运行（detach）
# -p 宿主机端口:容器端口   端口映射（可多个）
# -v 宿主机路径:容器路径   挂载数据卷（可多个）
# -e KEY=VALUE   设置环境变量
# --name 名称     给容器起名字（方便后续操作）
# --restart=always  容器崩溃或宿主机重启时自动重启
# --network bridge  指定网络（默认 bridge）
```

**实际例子**：

```bash
# 启动 Nginx，映射 8080 → 80
docker run -d --name web \
  -p 8080:80 \
  -v /opt/html:/usr/share/nginx/html \
  --restart=always \
  nginx:latest

# 启动 MySQL，挂载数据卷保证数据持久化
docker run -d --name mysql \
  -p 3306:3306 \
  -v /data/mysql:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=Root@123456 \
  --restart=always \
  mysql:8.0

# 启动 Redis，持久化数据
docker run -d --name redis \
  -p 6379:6379 \
  -v /data/redis:/data \
  --restart=always \
  redis:7-alpine \
  redis-server --appendonly yes
```

### 2. 查看容器

```bash
docker ps                    # 查看运行中的容器
docker ps -a                 # 查看所有容器（含已停止的）
docker ps -q                 # 只显示容器 ID
docker ps -n 3               # 只显示最近的 3 个
docker ps --size             # 显示容器文件系统大小
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'  # 自定义格式
```

### 3. 停止/启动/重启容器

```bash
docker stop <容器名或ID>      # 停止容器（发送 SIGTERM，等 10 秒后强制杀）
docker start <容器名或ID>     # 启动已停止的容器
docker restart <容器名或ID>   # 重启容器
docker kill <容器名或ID>      # 强制立即终止（发送 SIGKILL）
docker pause <容器名或ID>     # 暂停容器（冻结所有进程）
docker unpause <容器名或ID>   # 恢复被暂停的容器
```

### 4. 进入容器

```bash
docker exec -it <容器名> bash      # 进入容器内部（推荐，不依赖容器是否主进程）
docker exec -it <容器名> sh        # 部分精简镜像只有 sh
docker attach <容器名>             # 附着到容器主进程（退出容器会导致容器停止，慎用）
```

**实际例子**：

```bash
# 进入 MySQL 容器执行 SQL
docker exec -it mysql mysql -uroot -pRoot@123456

# 进入 Nginx 容器查看配置
docker exec -it web cat /etc/nginx/nginx.conf

# 在容器内执行单条命令（不进入交互模式）
docker exec mysql mysql -uroot -pRoot@123456 -e "SHOW DATABASES;"
```

### 5. 查看日志

```bash
docker logs <容器名>                 # 查看容器日志
docker logs -f <容器名>              # 实时追踪日志（类似 tail -f）
docker logs --tail 100 <容器名>      # 只看最后 100 行
docker logs --since 30m <容器名>     # 看最近 30 分钟的日志
docker logs <容器名> 2>&1            # 同时查看 stdout 和 stderr
```

### 6. 查看容器信息

```bash
docker inspect <容器名>          # 查看容器的完整配置（IP、卷、环境变量等）
docker inspect <容器名> --format='{{.NetworkSettings.IPAddress}}'  # 只看 IP
docker stats                     # 实时查看容器资源占用（CPU、内存、网络）
docker stats --no-stream         # 只看一次输出
```

### 7. 删除容器

```bash
docker rm <容器名>               # 删除已停止的容器
docker rm -f <容器名>            # 强制删除（运行中的容器也会被停掉）
docker rm $(docker ps -aq)       # 删除所有已停止的容器
docker container prune           # 清理所有已停止的容器（交互确认）
```

### 8. 复制文件到/从容器

```bash
# 从宿主机复制文件到容器
docker cp config.yml mycontainer:/app/config.yml

# 从容器复制文件到宿主机
docker cp mycontainer:/app/config.yml ./config.yml

# 从容器复制目录
docker cp mycontainer:/var/log/mysql ./mysql-logs
```

---

## 三、数据卷（Volume）命令

数据卷是持久化容器数据的核心机制，容器删除后数据仍然存在。

### 1. 管理数据卷

```bash
docker volume ls                       # 查看所有数据卷
docker volume inspect myvolume         # 查看数据卷详情
docker volume create myvolume          # 创建数据卷
docker volume rm myvolume              # 删除数据卷
docker volume prune                    # 清理所有未被使用的数据卷
```

### 2. 数据卷的两种挂载方式

**方式一：匿名卷（Docker 自动管理路径）**

```bash
docker run -d -v /data nginx    # 容器内 /data 目录挂载到自动分配的卷
```

**方式二：具名卷（推荐，命名清晰）**

```bash
docker run -d -v mydata:/data nginx    # 具名卷 mydata 挂载到容器 /data
```

**方式三：绑定挂载（宿主机路径直接映射，最常用）**

```bash
docker run -d -v /opt/html:/usr/share/nginx/html nginx
# 格式：-v 宿主机绝对路径:容器内路径
```

### 3. 只读挂载

```bash
docker run -d -v /opt/config:/etc/app:ro nginx
# :ro = read-only，容器内只能读不能写
```

---

## 四、网络操作命令

Docker 默认提供三种网络：`bridge`（桥接）、`host`（共享宿主机网络）、`none`（无网络）。

### 1. 查看网络

```bash
docker network ls                          # 查看所有网络
docker network inspect bridge              # 查看网络详情（IP 分配、连接的容器）
```

### 2. 创建自定义网络

```bash
docker network create mynet                # 创建自定义 bridge 网络
docker network create --driver bridge mynet
```

**为什么需要自定义网络？** 默认 bridge 网络不支持容器间通过名字互相访问，自定义网络内置 DNS 解析，同网络的容器可以直接用**容器名**互相通信：

```bash
# 创建网络
docker network create app-net

# 两个容器连到同一网络，可以互访
docker run -d --name web --network app-net nginx
docker run -d --name app --network app-net myapp
# web 容器可以直接访问 http://app:8080
```

### 3. 连接/断开网络

```bash
docker network connect mynet <容器名>     # 将容器连到网络
docker network disconnect mynet <容器名>  # 将容器从网络断开
```

---

## 五、常用组合与实用技巧

### 1. 一键删除所有停止的容器 + 悬空镜像

```bash
docker container prune        # 清理已停止的容器
docker image prune -a         # 清理所有悬空镜像（需确认）
docker system prune -a --volumes  # 清理容器、镜像、网络、数据卷（⚠️ 会删数据）
```

### 2. 查看容器占用资源排名

```bash
docker ps --format 'table {{.Names}}\t{{.CPUPerc}}\t{{.MemUsage}}' \
  | sort -k 3 -h -r
```

### 3. 容器内没有 bash 怎么办？

部分精简镜像（alpine）只有 `sh`：

```bash
docker exec -it <容器名> sh    # 用 sh 代替 bash
docker exec -it <容器名> /bin/sh
```

### 4. 在容器中执行命令后自动退出

```bash
docker run --rm nginx ls /usr/share/nginx/html
# --rm 容器退出后自动删除，适合一次性操作
```

### 5. 导出/导入镜像（离线转移）

```bash
# 导出镜像为 tar 包
docker save -o nginx-backup.tar nginx:latest

# 导入镜像
docker load -i nginx-backup.tar

# 导出/导入容器（含数据卷）
docker export mycontainer > container.tar
cat container.tar | docker import - myimage:latest
```

### 6. 端口映射多端口

```bash
# 映射多个端口
docker run -d \
  -p 80:80 \
  -p 443:443 \
  -p 8080:8080 \
  nginx
```

### 7. 常用快捷别名（写入 ~/.bashrc）

```bash
alias dps='docker ps'
alias dpa='docker ps -a'
alias di='docker images'
alias dl='docker logs -f'
alias de='docker exec -it'
alias dr='docker run -d --restart=always'
```

---

## 小结

| 操作 | 核心命令 |
|------|---------|
| **拉取镜像** | `docker pull <镜像名>` |
| **查看镜像** | `docker images` |
| **启动容器** | `docker run -d -p 宿主机端口:容器端口 --name 名称 镜像名` |
| **查看容器** | `docker ps` / `docker ps -a` |
| **进入容器** | `docker exec -it <容器名> bash` |
| **查看日志** | `docker logs -f <容器名>` |
| **停止/启动** | `docker stop` / `docker start` |
| **删除容器** | `docker rm -f <容器名>` |
| **删除镜像** | `docker rmi <镜像名>` |
| **数据持久化** | `-v 宿主机路径:容器路径` |
| **容器互访** | `docker network create 自定义网络` → `--network 网络名` |

> 日常最常用的命令组合就这一句：`docker run -d -p 80:80 --name myapp --restart=always 镜像名`，其他命令都是围绕这个命令的变体。
