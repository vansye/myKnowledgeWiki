---
type: 学习笔记
title: 自定义 Docker 镜像
created: 2026-08-04
updated: 2026-08-04
tags: [Docker, Dockerfile, 镜像构建, 部署, DevOps]
subject: 部署/Docker
---

> 自定义 Docker 镜像的核心是 **Dockerfile** —— 一个文本文件，描述"用哪些基础镜像、装什么依赖、拷贝什么文件、启动什么命令"。写好 Dockerfile 后，一条 `docker build` 即可生成专属镜像。

## 目录

- [一、Dockerfile 基础语法](#一dockerfile-基础语法)
- [二、常用指令详解](#二常用指令详解)
- [三、完整示例：打包 Spring Boot 应用](#三完整示例打包-spring-boot-应用)
- [四、完整示例：打包 Vue 前端（Nginx 托管）](#四完整示例打包-vue-前端nginx-托管)
- [五、镜像分层原理与优化技巧](#五镜像分层原理与优化技巧)
- [六、构建与推送流程](#六构建与推送流程)
- [小结](#小结)

---

## 一、Dockerfile 基础语法

### 什么是 Dockerfile

Dockerfile 是一个纯文本文件，每一行是一个指令，Docker 按顺序执行这些指令来构建镜像。

**基本结构**：

```dockerfile
# 注释行（以 # 开头）
FROM node:20-alpine          # 基础镜像
WORKDIR /app                 # 工作目录
COPY package.json ./         # 拷贝文件
RUN npm install              # 执行命令
EXPOSE 3000                  # 暴露端口
CMD ["node", "index.js"]     # 启动命令
```

### 编写规范

- 文件名固定为 `Dockerfile`（无后缀），也可用 `-f` 指定其他名字
- 存放在项目根目录，与 `package.json` / `pom.xml` 同级
- 每行一个指令，指令名大写，参数小写
- 注释用 `#`

---

## 二、常用指令详解

### 1. FROM — 指定基础镜像

```dockerfile
FROM nginx:latest              # 最简写法，默认 latest
FROM nginx:1.24                # 指定版本（生产推荐）
FROM openjdk:17-slim           # Debian 系精简版
FROM node:20-alpine            # Alpine 版，镜像更小
FROM java:17                   # Oracle JDK
```

> **Alpine 镜像 vs 标准镜像**：Alpine 基于 musl libc，体积更小（约 5MB vs 200MB），但部分软件兼容性需注意。生产环境推荐优先用 Alpine 或 slim 变体。

### 2. WORKDIR — 设置工作目录

```dockerfile
WORKDIR /app                   # 设置工作目录为 /app
WORKDIR /opt                   # 再设一个
RUN pwd                        # 输出 /opt（继承上一次的 WORKDIR）
```

> `WORKDIR` 等价于 `RUN mkdir -p && cd`，但会创建目录并缓存，推荐优先使用。路径不存在时会自动创建。

### 3. COPY vs ADD — 拷贝文件

```dockerfile
# COPY：简单拷贝，推荐优先用这个
COPY package.json /app/                    # 拷贝单个文件
COPY dist/ /usr/share/nginx/html/          # 拷贝整个目录
COPY *.jar /app/app.jar                    # 通配符匹配

# ADD：功能更多，但有隐患，慎用
ADD app.tar.gz /app/                       # 自动解压 tar 文件（COPY 不会）
ADD https://example.com/file.tar.gz /app/  # 自动下载远程文件（不推荐）
```

> **规则**：能用 `COPY` 就用 `COPY`，`ADD` 仅在需要自动解压或远程下载时使用。

### 4. ENV — 设置环境变量

```dockerfile
ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk
ENV APP_PORT=8080
ENV NODE_ENV=production

# 在 RUN 指令中直接使用
RUN echo $JAVA_HOME    # 输出 /usr/lib/jvm/java-17-openjdk

# 也可以在 docker run 时覆盖
docker run -e APP_PORT=9090 myapp
```

### 5. RUN — 执行命令

```dockerfile
# 单行 RUN（推荐，每个 RUN 创建一层）
RUN apt-get update && apt-get install -y curl wget

# 多行 RUN（使用反斜杠换行，保持可读性）
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    unzip \
    && rm -rf /var/lib/apt/lists/*   # 清理缓存，减少镜像体积

# 多条命令用 && 连接（避免创建过多层）
RUN mkdir -p /app/logs \
    && chown -R www-data:www-data /app/logs
```

> 每个 `RUN` 指令都会创建一层新的镜像层，指令越多镜像越大。尽量合并相关操作，用 `&&` 连接。

### 6. EXPOSE — 声明端口

```dockerfile
EXPOSE 80              # 声明容器监听 80 端口
EXPOSE 8080 8443       # 声明多个端口
```

> `EXPOSE` **不会**自动映射端口到宿主机，只是文档说明。实际映射需要在 `docker run` 时用 `-p` 指定，或 Dockerfile 中用 `EXPOSE` 配合说明。

### 7. CMD vs ENTRYPOINT — 启动命令

```dockerfile
# CMD：容器启动时默认执行的命令，可被 docker run 覆盖
CMD ["nginx", "-g", "daemon off;"]
CMD ["node", "index.js"]
CMD ["java", "-jar", "/app/myapp.jar"]

# ENTRYPOINT：容器启动时必执行的命令，参数可被追加
ENTRYPOINT ["java", "-jar"]
CMD ["/app/myapp.jar"]    # 可被 docker run 追加参数覆盖

# 两种写法对比：
# Shell 写法（有子进程，信号传递有问题，不推荐）
CMD echo "hello"

# Exec 写法（推荐，数组格式，进程直接运行）
CMD ["echo", "hello"]
```

> **区别**：`CMD` 可以被 `docker run` 的参数覆盖；`ENTRYPOINT` 不会被覆盖，`docker run` 的参数会追加到 ENTRYPOINT 后面。

### 8. VOLUME — 声明数据卷

```dockerfile
VOLUME /var/lib/mysql          # 声明 MySQL 数据目录为卷
VOLUME ["/data", "/logs"]      # 声明多个卷
```

> `VOLUME` 告诉 Docker 这些目录应该被挂载为数据卷，容器删除时数据不会丢失。

### 9. ARG — 构建时变量

```dockerfile
ARG JAVA_VERSION=17
ARG APP_VERSION=1.0.0

FROM openjdk:${JAVA_VERSION}-slim

RUN echo "Building version $APP_VERSION"

# 构建时传入参数
docker build --build-arg JAVA_VERSION=21 -t myapp:latest .
```

> `ARG` 只在构建时生效，不会写入镜像；`ENV` 会写入镜像并在容器运行时生效。

### 10. HEALTHCHECK — 健康检查

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1
```

- `--interval`：检查间隔（默认 30s）
- `--timeout`：超时时间（默认 3s）
- `--retries`：失败重试次数（默认 3）
- 返回 0 表示健康，非 0 表示不健康

---

## 三、完整示例：打包 Spring Boot 应用

### 多阶段构建（推荐）

多阶段构建可以大幅减小最终镜像体积：第一阶段编译（用完整版 JDK），第二阶段运行（用精简 JRE）。

```dockerfile
# ---- 第一阶段：编译 ----
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /build
# 先拷贝 pom.xml，利用 Docker 层缓存（依赖不变时不重新下载）
COPY pom.xml .
RUN mvn dependency:go-offline -B
# 再拷贝源码并编译打包
COPY src ./src
RUN mvn package -DskipTests -B

# ---- 第二阶段：运行 ----
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
# 从第一阶段拷贝编译好的 jar
COPY --from=builder /build/target/*.jar app.jar
# 暴露端口
EXPOSE 8080
# 启动命令
ENTRYPOINT ["java"]
CMD ["-jar", "app.jar", "--spring.profiles.active=prod"]
```

### 简单单阶段构建（开发测试用）

```dockerfile
FROM openjdk:17-slim
WORKDIR /app
COPY target/myapp.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 构建与运行

```bash
# 构建镜像（-t 指定标签，. 表示当前目录）
docker build -t myapp:1.0.0 .

# 运行容器
docker run -d \
  -p 8080:8080 \
  -v /data/myapp/logs:/app/logs \
  -e SPRING_PROFILES_ACTIVE=prod \
  --name myapp \
  --restart=always \
  myapp:1.0.0

# 查看日志
docker logs -f myapp
```

---

## 四、完整示例：打包 Vue 前端（Nginx 托管）

### Dockerfile

```dockerfile
# ---- 第一阶段：构建前端 ----
FROM node:20-alpine AS builder
WORKDIR /app
# 先拷贝依赖文件，利用层缓存
COPY package.json ./
RUN npm install --production=false
# 拷贝源码并构建
COPY . .
RUN npm run build

# ---- 第二阶段：用 Nginx 托管构建产物 ----
FROM nginx:1.24-alpine
# 拷贝构建产物到 Nginx 默认目录
COPY --from=builder /app/dist /usr/share/nginx/html
# 拷贝自定义 Nginx 配置（解决 Vue Router History 模式刷新 404）
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
# Nginx 前台运行（Docker 要求主进程在前台）
CMD ["nginx", "-g", "daemon off;"]
```

### 配套的 nginx.conf

```nginx
server {
    listen       80;
    server_name  localhost;
    root   /usr/share/nginx/html;
    index  index.html;

    # Vue Router History 模式：刷新不 404
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API 反向代理（开发时使用，生产可用独立 Nginx）
    location /api/ {
        proxy_pass http://backend:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }
}
```

### 构建与运行

```bash
# 构建镜像
docker build -t my-frontend:1.0.0 .

# 运行
docker run -d \
  -p 80:80 \
  --name frontend \
  --restart=always \
  my-frontend:1.0.0
```

---

## 五、镜像分层原理与优化技巧

### 镜像分层原理

```
最终镜像 = 所有 RUN/COPY/ADD 指令产生的层叠加
          + 基础镜像的层
```

每执行一个指令，Docker 就会创建一层新的只读层。层从上到下依次叠加，上层可以修改下层的内容。

```
┌─────────────────────┐  ← 最新层（RUN npm build）
│   dist/ 编译产物      │
├─────────────────────┤
│   src/ 源码          │
├─────────────────────┤
│   node_modules/     │
├─────────────────────┤
│   package.json      │
├─────────────────────┤
│   node:20-alpine    │  ← 基础镜像层
└─────────────────────┘
```

**关键原则**：**变化越频繁的文件，越靠后拷贝**。这样可以复用前面的缓存层，加快构建速度。

### 优化技巧

#### 1. 善用缓存：先拷依赖文件，再拷源码

```dockerfile
# ✅ 正确：package.json 变化才重新 install，源码变化不触发重装依赖
COPY package.json ./
RUN npm install
COPY . .

# ❌ 错误：源码一变动，npm install 全部重来
COPY . .
RUN npm install
```

#### 2. 合并 RUN 指令，减少层数

```dockerfile
# ✅ 合并为一步，清理缓存一起完成
RUN apt-get update && apt-get install -y \
    curl wget unzip \
    && rm -rf /var/lib/apt/lists/*

# ❌ 分三步，多了两层无用层
RUN apt-get update
RUN apt-get install -y curl wget unzip
RUN rm -rf /var/lib/apt/lists/*
```

#### 3. 使用 .dockerignore 排除无用文件

在项目根目录创建 `.dockerignore`：

```gitignore
# 排除不必要的文件，减小上下文体积
node_modules
.git
.env
*.md
.DS_Store
src/
dist/
```

#### 4. 选择轻量基础镜像

| 镜像类型 | 大小 | 适用场景 |
|---------|------|---------|
| `alpine` | ~5MB | 追求最小体积 |
| `slim` | ~100MB | 平衡体积与兼容性 |
| `latest`（完整） | ~1GB | 需要完整工具链 |

#### 5. 多阶段构建减小体积（核心技巧）

```dockerfile
# 编译阶段用完整版 Maven
FROM maven:3.9-eclipse-temurin-17 AS builder
RUN mvn package -DskipTests

# 运行阶段只用 JRE，镜像从 800MB 降到 200MB
FROM eclipse-temurin:17-jre-alpine
COPY --from=builder /app/target/*.jar app.jar
```

#### 6. 使用 --no-install-recommends 减少包体积（Debian/Ubuntu）

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl wget \
    && rm -rf /var/lib/apt/lists/*
```

---

## 六、构建与推送流程

### 本地构建

```bash
# 基础构建
docker build -t myapp:1.0.0 .

# 指定 Dockerfile 路径（非默认位置）
docker build -f Dockerfile.prod -t myapp:1.0.0 .

# 指定构建上下文（父目录）
docker build -t myapp:1.0.0 -f Dockerfile.prod .

# 不使用缓存（强制重新构建）
docker build --no-cache -t myapp:1.0.0 .

# 传入构建参数
docker build --build-arg APP_VERSION=1.0.0 -t myapp:1.0.0 .
```

### 登录仓库并推送

```bash
# 登录 Docker Hub（或私有仓库）
docker login
# 输入用户名和密码

# 给镜像打标签（格式：仓库名/镜像名:标签）
docker tag myapp:1.0.0 root/myapp:1.0.0

# 推送到仓库
docker push vansye/myapp:1.0.0

# 推送 latest 标签
docker tag myapp:1.0.0 root/myapp:latest
docker push root/myapp:latest
```

### 从私有仓库拉取

```bash
# 登录私有仓库
docker login registry.example.com

# 拉取
docker pull registry.example.com/myapp:1.0.0

# 运行
docker run -d -p 8080:8080 registry.example.com/myapp:1.0.0
```

### 常见 tag 命名规范

```
myapp:latest          # 最新稳定版（临时）
myapp:1.0.0           # 精确版本号（推荐生产使用）
myapp:1.0             # 主版本标签
myapp:dev             # 开发分支
myapp:20260804        # 日期标签（CI/CD 常用）
```

---

## 小结

| 要点                    | 说明                                                   |
| --------------------- | ---------------------------------------------------- |
| **Dockerfile 是镜像的蓝图** | 每行指令对应镜像的一层，顺序很重要                                    |
| **FROM 选对基础镜像**       | Alpine/slim 体积小，完整镜像兼容性好                             |
| **COPY 在 RUN 之前**     | 利用层缓存，依赖文件变化才重新 install                              |
| **多阶段构建是核心技巧**        | 编译和运行分开，最终镜像体积可缩小 5~10 倍                             |
| **.dockerignore 不能少** | 排除 node_modules/.git 等大目录，加快构建                       |
| **CMD 用 Exec 格式**     | `["java", "-jar", "app.jar"]` 而非 `java -jar app.jar` |
| **版本号固定**             | 生产环境不用 `latest`，用精确版本号便于回滚                           |

> Dockerfile 写好的标志：同一条 `docker build` 命令，在任何机器上都能构建出**完全相同**的镜像。
