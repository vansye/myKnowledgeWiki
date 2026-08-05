---
type: 学习笔记
title: Docker Compose
created: 2026-08-04
updated: 2026-08-04
tags: [Docker, Compose, 多容器, 编排, 部署, DevOps]
subject: 部署/Docker
---

> Docker Compose 是 Docker 官方的多容器编排工具，通过一个 YAML 文件定义多容器应用的启动配置，一条命令即可同时启动/停止/管理整个应用栈（前端+Nginx+后端+MySQL+Redis）。

## 目录

- [一、Docker Compose 是什么](#一docker-compose-是什么)
- [二、安装 Docker Compose](#二安装-docker-compose)
- [三、docker-compose.yml 核心语法](#三docker-composeyml-核心语法)
- [四、完整示例：前后端分离项目](#四完整示例前后端分离项目)
- [五、常用命令与操作](#五常用命令与操作)
- [六、环境变量与配置注入](#六环境变量与配置注入)
- [七、进阶特性](#七进阶特性)
- [小结](#小结)

---

## 一、Docker Compose 是什么

### 没有 Compose 时的痛点

管理多个容器时，需要逐个启动，手动指定网络、卷、环境变量，非常繁琐：

```bash
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=xxx -v /data/mysql:/var/lib/mysql mysql:8.0
docker run -d --name redis -v /data/redis:/data redis:7-alpine
docker run -d --name backend -p 8080:8080 -e DB_HOST=mysql myapp-backend:1.0.0
docker run -d --name frontend -p 80:80 myapp-frontend:1.0.0
```

**问题**：
- 命令太长，容易忘参数
- 容器启动顺序不确定（MySQL 可能还没就绪后端就启动了）
- 换台服务器要重新敲一遍

### Compose 的解决方案

用一个 `docker-compose.yml` 文件描述所有容器及其关系，一条命令搞定：

```bash
docker compose up -d
```

```
一个 YAML 文件 = 定义整个应用栈
一条命令 = 启动所有容器 + 创建网络 + 挂载数据卷
```

---

## 二、安装 Docker Compose

### 方式一：Docker Desktop（最简单）

安装 Docker Desktop（Windows/Mac）后，Compose 已内置，无需额外安装。

```bash
docker compose version
# Docker Compose version v2.x.x
```

### 方式二：Linux 手动安装

```bash
# 下载二进制文件
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose

# 添加执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 验证
docker-compose --version
```

> **注意**：新版 Docker 推荐使用 `docker compose`（无连字符），旧版用 `docker-compose`（有连字符）。两者功能相同，以下统一用 `docker compose`。

---

## 三、docker-compose.yml 核心语法

### 基本结构

```yaml
version: "3.8"              # Compose 文件版本（可选，新语法可省略）

services:                   # 定义所有服务（容器）
  web:
    image: nginx:1.24-alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - frontend-net

  backend:
    build: ./backend        # 从 Dockerfile 构建
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=mysql
    networks:
      - frontend-net
      - backend-net

  mysql:
    image: mysql:8.0
    volumes:
      - db-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: Root@123456
    networks:
      - backend-net

networks:                   # 定义网络
  frontend-net:
  backend-net:

volumes:                    # 定义数据卷
  db-data:
```

### 核心指令详解

#### 1. services — 服务定义

每个 service 对应一个容器，核心字段：

| 字段 | 说明 | 示例 |
|------|------|------|
| `image` | 使用已有镜像 | `image: nginx:latest` |
| `build` | 从 Dockerfile 构建 | `build: ./backend` |
| `build.context` | Dockerfile 所在目录 | `build: { context: ./backend, dockerfile: Dockerfile.prod }` |
| `ports` | 端口映射 | `ports: ["8080:8080"]` |
| `volumes` | 数据卷挂载 | `volumes: ["./data:/app/data"]` |
| `environment` | 环境变量 | `environment: { DB_HOST: mysql }` |
| `env_file` | 从文件加载环境变量 | `env_file: .env` |
| `networks` | 连接的网络 | `networks: [web-net]` |
| `depends_on` | 依赖服务（启动顺序） | `depends_on: [mysql, redis]` |
| `restart` | 重启策略 | `restart: always` |
| `command` | 覆盖默认启动命令 | `command: ["node", "server.js"]` |
| `working_dir` | 容器工作目录 | `working_dir: /app` |
| `stdin_open` | 保持 stdin 打开 | `stdin_open: true`（配合 `docker compose exec -it`） |
| `tty` | 分配伪终端 | `tty: true` |
| `healthcheck` | 健康检查 | 见下文 |

**端口映射写法**：

```yaml
ports:
  - "80:80"              # 最简写法
  - "8080:80"            # 指定宿主机端口
  - "127.0.0.1:8080:80"  # 只绑定 localhost（不暴露给外部）
  - target: 80           # 完整写法
    published: 8080
    protocol: tcp
```

**volumes 写法**：

```yaml
volumes:
  - ./html:/usr/share/nginx/html    # 绑定挂载（宿主机路径）
  - db-data:/var/lib/mysql          # 命名卷（由 Compose 管理）
  - /host/path:/container/path:ro   # 只读挂载
  - app-data:/data                  # 匿名卷
```

**environment 写法**：

```yaml
# 写法一：列表（key=value）
environment:
  - DB_HOST=mysql
  - DB_PORT=3306
  - REDIS_URL=redis://redis:6379

# 写法二：字典（推荐，更清晰）
environment:
  DB_HOST: mysql
  DB_PORT: "3306"       # 数字建议加引号，避免被识别为整数
  REDIS_URL: redis://redis:6379

# 写法三：从文件加载
env_file:
  - .env
  - .env.production     # 多个文件，后者覆盖前者
```

#### 2. networks — 网络定义

```yaml
networks:
  frontend-net:            # 默认 bridge 驱动
  backend-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
  external-net:
    external: true         # 使用已存在的外部网络
```

#### 3. volumes — 数据卷定义

```yaml
volumes:
  db-data:                 # 命名卷，由 Docker 管理
  redis-data:
    driver: local          # 可选：指定驱动
    driver_opts:
      type: none
      o: bind
      device: /data/redis  # 绑定到宿主机路径
```

---

## 四、完整示例：前后端分离项目

### 项目结构

```
myapp/
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── dist/          # 前端构建产物
├── backend/
│   ├── Dockerfile
│   └── target/
│       └── myapp.jar
└── .env               # 环境变量（敏感信息放这里）
```

### .env 文件（敏感信息）

```bash
# .env
MYSQL_ROOT_PASSWORD=Root@123456
MYSQL_DATABASE=myapp
MYSQL_USER=myapp
MYSQL_PASSWORD=MyApp@2026
REDIS_PASSWORD=Redis@2026
APP_PORT=8080
```

### docker-compose.yml

```yaml
services:
  # ========== MySQL ==========
  mysql:
    image: mysql:8.0
    container_name: myapp-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - mysql-data:/var/lib/mysql
      - ./sql/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - backend-net
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u root", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ========== Redis ==========
  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD} --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - backend-net
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  # ========== 后端 Spring Boot ==========
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: myapp-backend
    restart: always
    ports:
      - "${APP_PORT}:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/${MYSQL_DATABASE}?useSSL=false&serverTimezone=Asia/Shanghai
      SPRING_DATASOURCE_USERNAME: ${MYSQL_USER}
      SPRING_DATASOURCE_PASSWORD: ${MYSQL_PASSWORD}
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PASSWORD: ${REDIS_PASSWORD}
    volumes:
      - app-logs:/var/log/myapp
    networks:
      - frontend-net
      - backend-net
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy

  # ========== 前端 Nginx ==========
  frontend:
    image: nginx:1.24-alpine
    container_name: myapp-frontend
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./frontend/dist:/usr/share/nginx/html:ro
      - ./frontend/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    networks:
      - frontend-net
    depends_on:
      - backend

networks:
  frontend-net:
    driver: bridge
  backend-net:
    driver: bridge

volumes:
  mysql-data:
  redis-data:
  app-logs:
```

### backend/Dockerfile

```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/myapp.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java"]
CMD ["-jar", "app.jar", "--spring.profiles.active=prod"]
```

### frontend/nginx.conf

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://backend:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 启动与验证

```bash
# 构建并启动所有服务
docker compose up -d

# 查看所有容器状态
docker compose ps

# 查看实时日志
docker compose logs -f

# 查看某个服务的日志
docker compose logs -f backend

# 停止所有服务
docker compose stop

# 启动所有服务
docker compose start

# 重启某个服务
docker compose restart backend

# 销毁所有容器（保留镜像和卷）
docker compose down

# 销毁所有容器 + 删除网络
docker compose down --remove-orphans

# 销毁所有容器 + 网络 + 数据卷（⚠️ 数据会丢失）
docker compose down -v
```

---

## 五、常用命令与操作

### 基本命令

```bash
# 构建并启动
docker compose up [-d]

# 后台运行
docker compose up -d

# 查看运行中的容器
docker compose ps

# 查看日志
docker compose logs [-f] [service]    # -f 实时追踪
docker compose logs --tail 100 backend

# 进入容器
docker compose exec backend bash
docker compose exec -it mysql mysql -uroot -p

# 重启服务
docker compose restart [service]
docker compose restart

# 停止服务
docker compose stop [service]
docker compose stop

# 启动已停止的服务
docker compose start [service]

# 删除容器
docker compose rm [service]
docker compose rm -f    # 强制删除运行中的容器

# 销毁（停止+删除容器+网络）
docker compose down
docker compose down -v          # 同时删除数据卷
docker compose down --rmi all   # 同时删除镜像

# 构建/重建
docker compose build
docker compose build --no-cache backend    # 强制重新构建某个服务

# 更新服务（先 stop 再 start，使用新镜像）
docker compose up -d --pull always backend

# 查看服务状态和端口映射
docker compose ps --format table "{{.Name}}\t{{.State}}\t{{.Ports}}"

# 扩展/缩容（仅对 Swarm 模式的服务有效，普通 compose 不支持）
docker compose up -d --scale backend=3
```

### 服务间通信

Compose 自动为每个服务创建 DNS 记录，服务名即主机名：

```yaml
# backend 可以直接访问 mysql、redis
SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/myapp_db
SPRING_REDIS_HOST: redis
```

```bash
# 进入后端容器验证 DNS 解析
docker compose exec backend ping mysql
docker compose exec backend ping redis
```

### 多环境配置

#### 方式一：多个 compose 文件（推荐）

```yaml
# docker-compose.yml（基础配置）
services:
  backend:
    image: myapp-backend:1.0.0
    ports:
      - "8080:8080"
    env_file: .env
    networks:
      - backend-net

  mysql:
    image: mysql:8.0
    env_file: .env
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - backend-net

networks:
  backend-net:

volumes:
  mysql-data:
```

```yaml
# docker-compose.prod.yml（生产环境覆盖）
services:
  backend:
    restart: always
    ports:
      - "8080:8080"

  mysql:
    healthcheck:
      test: ["CMD", "mysqladmin", "ping"]
      interval: 10s
```

```bash
# 组合使用
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# 或简化写法（Compose 会自动读取 docker-compose.prod.yml）
COMPOSE_FILE=docker-compose.yml:docker-compose.prod.yml docker compose up -d
```

#### 方式二：docker-compose.override.yml（开发覆盖）

```yaml
# docker-compose.override.yml（自动被 Compose 加载，用于开发环境）
services:
  backend:
    build: ./backend
    volumes:
      - ./backend:/app
      - app-logs:/var/log/myapp
    ports:
      - "8080:8080"
      - "5005:5005"    # 调试端口

  mysql:
    ports:
      - "3306:3306"    # 开发环境暴露端口方便本地连接
```

> **优先级**：`docker-compose.yml` → `docker-compose.override.yml`（后者覆盖前者），无需额外参数自动生效。

---

## 六、环境变量与配置注入

### 1. 环境变量优先级

```
命令行的 -e 参数 > docker-compose.yml 中的 environment > env_file 文件 > .env 文件
```

### 2. .env 文件规范

在项目根目录创建 `.env` 文件，Compose 会自动读取：

```bash
# .env
MYSQL_ROOT_PASSWORD=Root@123456
MYSQL_DATABASE=myapp
APP_VERSION=1.0.0
LOG_LEVEL=INFO
```

在 compose 文件中引用：

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
```

### 3. 条件变量（设置默认值）

```yaml
environment:
  LOG_LEVEL: ${LOG_LEVEL:-INFO}       # 未设置时默认 INFO
  APP_PORT: ${APP_PORT:-8080}         # 未设置时默认 8080
  DB_HOST: ${DB_HOST:?必填参数}       # 未设置时报错退出
```

### 4. 从外部传入

```bash
# 命令行覆盖
MYSQL_ROOT_PASSWORD=NewPass123 docker compose up -d

# 指定 .env 文件
docker compose --env-file .env.production up -d
```

---

## 七、进阶特性

### 1. depends_on + healthcheck（依赖服务就绪）

默认 `depends_on` 只等待服务启动，不等待服务就绪。配合健康检查可实现真正的依赖：

```yaml
services:
  backend:
    depends_on:
      mysql:
        condition: service_healthy    # 等 MySQL 健康检查通过后才启动
      redis:
        condition: service_healthy

  mysql:
    image: mysql:8.0
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "-u root", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s    # 给 MySQL 30 秒初始化时间

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
```

### 2. 资源限制

```yaml
services:
  backend:
    image: myapp-backend:1.0.0
    deploy:
      resources:
        limits:
          cpus: "1.0"        # 最多使用 1 个 CPU 核
          memory: 512M       # 最多使用 512MB 内存
        reservations:
          cpus: "0.5"        # 最少分配 0.5 个 CPU 核
          memory: 256M       # 最少分配 256MB 内存
    # 旧版写法（Swarm 模式外也可用）
    mem_limit: 512m
    cpus: 1.0
```

### 3. 日志管理

```yaml
services:
  backend:
    image: myapp-backend:1.0.0
    logging:
      driver: json-file      # 日志驱动
      options:
        max-size: "10m"      # 单个日志文件最大 10MB
        max-file: "3"        # 最多保留 3 个日志文件
```

### 4. 共享配置（configs）

用于注入配置文件而非环境变量：

```yaml
services:
  nginx:
    image: nginx:1.24-alpine
    configs:
      - source: my-nginx-conf
        target: /etc/nginx/conf.d/default.conf

configs:
  my-nginx-conf:
    file: ./nginx/nginx.conf
```

### 5. Secrets（敏感信息）

```yaml
services:
  backend:
    image: myapp-backend:1.0.0
    secrets:
      - db_password
      - api_key

secrets:
  db_password:
    file: ./secrets/db_password.txt    # 文件路径
  api_key:
    external: true                     # 使用外部管理的 secret
```

> 敏感信息存入文件而非 compose 文件，容器内通过 `/run/secrets/xxx` 读取。

---

## 小结

| 要点                           | 说明                                           |
| ---------------------------- | -------------------------------------------- |
| **一个 YAML = 整个应用栈**          | 所有容器、网络、卷在一个文件中定义                            |
| **服务名即 DNS 名**               | 容器间通过服务名互相访问，无需记 IP                          |
| **depends_on + healthcheck** | 保证依赖服务完全就绪后再启动                               |
| **.env 文件管理敏感信息**            | 密码、密钥不入 compose 文件，不入版本控制                    |
| **override 文件区分环境**          | `docker-compose.override.yml` 自动加载，适合开发/生产分离 |
| **日志轮转要配置**                  | `max-size` + `max-file` 防止日志撑爆磁盘             |
| **资源限制不可少**                  | `mem_limit` / `cpus` 防止单个容器拖垮宿主机             |

> **核心思想**：把"启动哪些容器、怎么连、传什么参数"全部写进 YAML，团队协作和服务器迁移时只需复制这一个文件。
