---
type: 学习笔记
title: Linux 项目部署流程
created: 2026-08-03
updated: 2026-08-03
tags: [Linux, 部署, Nginx, SpringBoot, MySQL, 前后端分离]
subject: 部署/Linux
---

> 完整部署一个前后端分离项目，需要依次完成：服务器环境搭建 → 后端打包部署 → 前端构建 + Nginx 托管 → 数据库初始化 → 服务联调与验证。本文按这个顺序展开，每步都给出可直接执行的命令。

## 目录

- [一、部署前准备](#一部署前准备)
- [二、服务器环境搭建](#二服务器环境搭建)
- [三、后端部署（Spring Boot）](#三后端部署spring-boot)
- [四、前端部署（Nginx 托管）](#四前端部署nginx-托管)
- [五、数据库部署与初始化](#五数据库部署与初始化)
- [六、联调与常见问题排查](#六联调与常见问题排查)
- [小结](#小结)

---

## 一、部署前准备

### 部署流程图

```
开发环境 → 前端 npm run build → dist/（静态文件）
         → 后端 Maven clean package → target/*.jar
         → 上传到服务器 → Nginx + SpringBoot + MySQL 联调
```

### 需要准备的文件

| 文件 | 来源 | 说明 |
|------|------|------|
| `dist/` 目录 | 前端构建产物 | 包含 index.html、CSS、JS 等静态文件 |
| `*.jar`（Spring Boot 可执行 jar） | 后端 Maven 打包 | 内嵌 Tomcat，可直接 java -jar 运行 |
| `schema.sql` / 初始化脚本 | 数据库初始化 | 建表 + 初始数据 |
| `application.yml` 生产版本 | 后端配置文件 | 数据库地址、端口等改成服务器实际值 |

### 打包命令

```bash
# 前端打包（在本地开发机执行）
cd my-frontend
npm run build
# 产物在 dist/ 目录，整个目录上传到服务器

# 后端打包（在本地开发机执行）
cd my-backend
mvn clean package -DskipTests
# 产物在 target/my-app.jar
```

---

## 二、服务器环境搭建

### 2.1 更新系统并安装基础工具

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git unzip
```

### 2.2 安装 Java（后端运行环境）

```bash
# 查看可用版本
apt search openjdk

# 安装 JDK 17（Spring Boot 3.x 推荐）
sudo apt install -y openjdk-17-jdk

# 验证
java -version
# 应输出：openjdk version "17.x.x"

# 设置 JAVA_HOME（写入 ~/.bashrc）
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
source ~/.bashrc
```

### 2.3 安装 MySQL

```bash
# 安装 MySQL 8.0
sudo apt install -y mysql-server

# 启动并设置开机自启
sudo systemctl start mysql
sudo systemctl enable mysql

# 安全初始化（设置 root 密码、移除匿名账户等）
sudo mysql_secure_installation

# 登录 MySQL
sudo mysql -u root -p
```

**MySQL 常用配置**：

```sql
-- 创建项目数据库
CREATE DATABASE myapp_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 创建专用用户（不要用 root）
CREATE USER 'myapp'@'%' IDENTIFIED BY 'StrongPassword123!';

-- 授权
GRANT ALL PRIVILEGES ON myapp_db.* TO 'myapp'@'%';
FLUSH PRIVILEGES;

-- 查看用户
SELECT user, host FROM mysql.user;
```

> 生产环境 MySQL 默认只监听 `127.0.0.1`，如需远程访问需修改 `/etc/mysql/mysql.conf.d/mysqld.cnf` 中 `bind-address`。

### 2.4 安装 Redis（可选，用于缓存）

```bash
sudo apt install -y redis-server
sudo systemctl start redis
sudo systemctl enable redis
redis-cli ping   # 应返回 PONG
```

### 2.5 安装 Nginx

```bash
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx   # 确认运行状态
```

### 2.6 开放防火墙端口

```bash
# Ubuntu 使用 ufw
sudo ufw allow 22/tcp        # SSH
sudo ufw allow 80/tcp        # HTTP
sudo ufw allow 443/tcp       # HTTPS
sudo ufw allow 8080/tcp      # 后端 API 端口
sudo ufw enable              # 启用防火墙
sudo ufw status              # 查看规则
```

> 如果是云服务器（阿里云/腾讯云），还需在**控制台**的安全组中开放对应端口，仅配置服务器内防火墙是不够的。

---

## 三、后端部署（Spring Boot）

### 3.1 修改生产环境配置

后端打包前，修改 `src/main/resources/application-prod.yml`：

```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/myapp_db?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8
    username: myapp
    password: StrongPassword123!
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: none    # 生产环境禁用自动建表
    show-sql: false

server:
  port: 8080

# 日志配置
logging:
  file:
    name: /var/log/myapp/app.log
  level:
    root: INFO
    com.myapp: INFO
```

### 3.2 上传 jar 到服务器

```bash
# 本地执行，将 jar 传输到服务器
scp target/myapp.jar user@你的服务器IP:/opt/myapp/myapp.jar
```

### 3.3 创建启动脚本

在服务器 `/opt/myapp/` 目录下创建启动脚本：

```bash
sudo mkdir -p /opt/myapp /var/log/myapp
sudo cp myapp.jar /opt/myapp/
sudo chown -R $USER:$USER /opt/myapp /var/log/myapp
```

创建 `start.sh`：

```bash
#!/bin/bash
cd /opt/myapp
nohup java -jar myapp.jar --spring.profiles.active=prod > /var/log/myapp/app.log 2>&1 &
echo "后端服务已启动，PID: $!"
```

创建 `stop.sh`：

```bash
#!/bin/bash
PID=$(ps aux | grep 'myapp.jar' | grep -v grep | awk '{print $2}')
if [ -n "$PID" ]; then
    kill -15 $PID
    sleep 3
    kill -9 $PID 2>/dev/null
    echo "后端服务已停止"
else
    echo "后端服务未运行"
fi
```

```bash
chmod +x start.sh stop.sh
```

### 3.4 注册为 systemd 服务（推荐，可开机自启）

创建 `/etc/systemd/system/myapp.service`：

```ini
[Unit]
Description=My Application Backend
After=network.target mysql.service

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/java -jar /opt/myapp/myapp.jar --spring.profiles.active=prod
ExecStop=/bin/kill -15 $MAINPID
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start myapp
sudo systemctl enable myapp     # 开机自启
sudo systemctl status myapp     # 查看状态
journalctl -u myapp -f          # 实时查看日志
```

### 3.5 验证后端是否正常运行

```bash
curl http://localhost:8080/actuator/health
# 应返回 {"status":"UP"}

# 或查看进程
ps aux | grep myapp.jar
```

---

## 四、前端部署（Nginx 托管）

### 4.1 上传静态文件

```bash
# 本地执行
scp -r dist/ user@你的服务器IP:/opt/myapp/frontend/
```

### 4.2 配置 Nginx

编辑 `/etc/nginx/sites-available/default`（或新建 `/etc/nginx/sites-available/myapp`）：

```nginx
server {
    listen 80;
    server_name 你的域名或服务器IP;

    # 前端静态资源
    root /opt/myapp/frontend/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
        # try_files 解决 Vue Router History 模式刷新 404 的问题
    }

    # API 反向代理（解决跨域）
    location /api/ {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 文件上传接口（如需）
    location /uploads/ {
        proxy_pass http://127.0.0.1:8080/uploads/;
        client_max_body_size 50m;
    }

    # 静态资源缓存（CSS/JS/图片）
    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }
}
```

### 4.3 测试并重载 Nginx

```bash
sudo nginx -t              # 测试配置语法
sudo nginx -s reload       # 平滑重载配置
# 或
sudo systemctl reload nginx
```

### 4.4 验证前端是否可访问

```bash
curl http://localhost/index.html
# 应返回 HTML 内容
```

浏览器访问 `http://你的服务器IP`，应看到前端页面。

---

## 五、数据库部署与初始化

### 5.1 导入数据

```bash
# 将本地导出的 SQL 文件传到服务器
scp schema.sql user@你的服务器IP:/opt/myapp/

# 登录 MySQL 并执行
sudo mysql -u myapp -p myapp_db < /opt/myapp/schema.sql
```

### 5.2 常见数据库问题排查

**问题 1：连接超时（Communications link failure）**

```bash
# 检查 MySQL 是否在监听
sudo netstat -tlnp | grep 3306

# 检查防火墙
sudo ufw status

# 检查 MySQL 配置（是否绑定 127.0.0.1）
sudo cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep bind-address
```

> 前后端分离部署时，如果后端和数据库在同一台服务器，`127.0.0.1` 没问题。若分开部署，需将 `bind-address` 改为 `0.0.0.0` 并开放端口。

**问题 2：时区不一致导致时间错乱**

```sql
-- 查看 MySQL 时区
SHOW VARIABLES LIKE 'time_zone';

-- 修改时区（临时）
SET GLOBAL time_zone = '+8:00';

-- 永久修改：编辑 /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
default-time-zone = '+8:00'
```

**问题 3：字符集乱码**

```sql
-- 检查字符集
SHOW VARIABLES LIKE 'character_set%';

-- 确保数据库和表都是 utf8mb4
ALTER DATABASE myapp_db CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
ALTER TABLE your_table CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**问题 4：SQL 模式报错（ONLY_FULL_GROUP_BY 等）**

```sql
-- 查看当前 SQL 模式
SELECT @@sql_mode;

-- 临时修改
SET SESSION sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
```

**问题 5：磁盘满导致服务崩溃**

```bash
df -h                 # 查看磁盘使用
du -sh /var/log/mysql # 查看 MySQL 日志占用
mysqladmin processlist # 查看是否有长查询卡住
```

---

## 六、联调与常见问题排查

### 6.1 端到端验证流程

```bash
# 1. 确认所有服务运行中
sudo systemctl status nginx
sudo systemctl status myapp
sudo systemctl status mysql

# 2. 验证后端 API（直接访问后端）
curl http://localhost:8080/api/health

# 3. 验证前端页面（访问 Nginx）
curl http://localhost/

# 4. 验证 API 代理（通过 Nginx 访问后端）
curl http://localhost/api/health

# 5. 查看 Nginx 错误日志
sudo tail -f /var/log/nginx/error.log

# 6. 查看应用日志
sudo journalctl -u myapp -f
```

### 6.2 常见部署问题与解决

| 现象 | 可能原因 | 排查方法 |
|------|---------|---------|
| 前端页面空白/404 | Nginx 配置 root 路径错误 | `nginx -t` 检查；确认 dist 目录存在且含 index.html |
| 刷新页面 404 | Vue Router History 模式未配置 | 在 Nginx 加 `try_files $uri $uri/ /index.html` |
| 前端请求跨域报错 | 后端未配置 CORS，且 Nginx 未代理 | 在 Nginx 的 `/api/` location 加 `proxy_pass` |
| 后端报数据库连接失败 | 账号密码错误、数据库未启动、端口被拦截 | `mysql -u myapp -p` 本地验证；检查 `application.yml` |
| 后端启动后立即退出 | 内存不足、端口被占用、配置文件错误 | 查看日志 `journalctl -u myapp -n 100 --no-pager` |
| 端口被占用 | 之前进程未彻底关闭 | `sudo lsof -i :8080` 找到占用进程并 kill |
| 静态资源缓存不更新 | 浏览器缓存了旧版本 | Nginx 加了 `expires 7d`，清空浏览器缓存或加版本号 |
| 上传文件失败 | `client_max_body_size` 默认 1m | 在 Nginx 配置中改为 `client_max_body_size 50m` |

### 6.3 端口被占用排查

```bash
# 查看端口占用
sudo lsof -i :8080
sudo netstat -tlnp | grep 8080

# 杀掉占用进程
sudo kill -9 <PID>
```

### 6.4 日志查看速查

```bash
# Nginx 访问日志
sudo tail -f /var/log/nginx/access.log

# Nginx 错误日志
sudo tail -f /var/log/nginx/error.log

# 应用日志（systemd 方式）
sudo journalctl -u myapp -f

# 应用日志（nohup 方式）
tail -f /var/log/myapp/app.log
```

### 6.5 部署脚本（一键部署）

开发完成后，可以把部署流程封装成一个脚本：

```bash
# deploy.sh（在本地开发机执行）
#!/bin/bash
echo "1. 打包前端..."
cd my-frontend && npm run build && cd ..

echo "2. 打包后端..."
cd my-backend && mvn clean package -DskipTests && cd ..

echo "3. 上传到服务器..."
scp target/myapp.jar user@服务器IP:/opt/myapp/
scp -r dist/ user@服务器IP:/opt/myapp/frontend/dist/

echo "4. 重启后端服务..."
ssh user@服务器IP "sudo systemctl restart myapp"

echo "5. 重载 Nginx..."
ssh user@服务器IP "sudo nginx -s reload"

echo "部署完成！"
```

---

## 小结

1. **部署顺序**：环境搭建 → 后端部署 → 前端部署 → 数据库初始化 → 联调验证
2. **Spring Boot**：打成可执行 jar，用 `java -jar` 或 systemd 管理，推荐注册为 systemd 服务实现开机自启
3. **Nginx 前端**：`root` 指向 `dist/` 目录，`try_files` 解决 SPA 刷新 404，`proxy_pass /api/` 解决跨域
4. **数据库**：创建专用账户而非用 root，初始化脚本用 `source` 或 `<` 导入，时区和字符集是常见坑
5. **排查思路**：先看日志（`journalctl` / `error.log`）→ 再查服务状态 → 最后确认网络和端口

> 部署的核心逻辑就一句话：**让浏览器能访问前端，让前端能访问后端 API，让后端能访问数据库**，三者打通就部署成功了。
