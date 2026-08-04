---
type: 学习笔记
title: Linux 软件安装方式
created: 2026-08-03
updated: 2026-08-03
tags: [Linux, 安装, 软件包, apt, yum, 编译, Docker]
subject: 部署/Linux
---

> Linux 软件安装主要有三种方式：包管理器（apt/yum）、源码编译安装、容器化部署（Docker），各有适用场景，掌握它们即可覆盖绝大多数软件部署需求。

## 目录

- [一、APT（Debian/Ubuntu 系）](#一aptdebianubuntu-系)
- [二、YUM/DNF（Red Hat/CentOS/Fedora 系）](#二yumdnfred-hatcentosfedora-系)
- [三、源码编译安装](#三源码编译安装)
- [四、Docker 容器化安装](#四docker-容器化安装)
- [五、其他方式（快照、第三方仓库）](#五其他方式快照第三方仓库)
- [小结](#小结)

---

## 一、APT（Debian/Ubuntu 系）

APT（Advanced Package Tool）是 Debian 系发行版最标准的包管理工具，使用 `.deb` 格式的压缩包，依赖管理自动解决。

### 基础命令

```bash
sudo apt update           # 刷新本地软件包索引（每次操作前建议先执行）
sudo apt upgrade          # 升级所有已安装的软件包
sudo apt install <包名>   # 安装软件包
sudo apt remove <包名>    # 卸载软件包（保留配置文件）
sudo apt purge <包名>     # 卸载软件包（同时删除配置文件）
sudo apt autoremove       # 清理不再被依赖的孤立包
sudo apt full-upgrade     # 升级时允许更换依赖包（更激进的升级）
```

### 搜索与查看

```bash
apt search <关键词>       # 搜索软件包（匹配名称和描述）
apt show <包名>           # 查看包的详细信息（版本、依赖、描述）
apt list --installed      # 列出所有已安装的包
apt list --upgradable     # 列出可升级的包
```

### 清理缓存

```bash
apt cache clean           # 清空已下载的 .deb 缓存
apt autoclean             # 只清理过期的缓存包
```

### 实际示例

```bash
# 安装 Nginx
sudo apt update
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl enable nginx    # 开机自启

# 安装 Java JDK
sudo apt install openjdk-17-jdk
java -version    # 验证安装

# 卸载并保持配置
sudo apt remove nginx
# 完全卸载（含配置）
sudo apt purge nginx
```

> **注意**：`apt` 是面向用户的命令（简洁），`apt-get` 是底层命令（脚本中使用）。日常推荐使用 `apt`。

---

## 二、YUM/DNF（Red Hat/CentOS/Fedora 系）

RHEL 系发行版使用 RPM 包格式，`yum` 是老版本（CentOS 7 及以下），`dnf` 是新版本（CentOS 8/Fedora/RHEL 8+），命令语法基本相同。

### 基础命令

```bash
sudo yum check-update        # 检查可更新的包（yum）
sudo dnf check-update        # 检查可更新的包（dnf）

sudo yum update              # 更新所有包（yum）
sudo dnf upgrade             # 更新所有包（dnf）

sudo yum install <包名>      # 安装软件包（yum）
sudo dnf install <包名>      # 安装软件包（dnf）

sudo yum remove <包名>       # 卸载软件包（yum）
sudo dnf remove <包名>       # 卸载软件包（dnf）

sudo yum autoremove          # 清理孤立依赖（yum）
sudo dnf autoremove          # 清理孤立依赖（dnf）
```

### 搜索与查看

```bash
yum search <关键词>          # 搜索软件包（yum）
dnf search <关键词>          # 搜索软件包（dnf）

yum info <包名>              # 查看包信息（yum）
dnf info <包名>              # 查看包信息（dnf）

yum list installed           # 列出已安装包（yum）
dnf list installed           # 列出已安装包（dnf）
```

### 实际示例

```bash
# 安装 EPEL 扩展源（RHEL/CentOS 默认仓库软件较少，需先装 EPEL）
sudo yum install epel-release      # yum
sudo dnf install epel-release      # dnf

# 安装 Nginx
sudo dnf install nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# 安装 Java
sudo dnf install java-17-openjdk
```

> CentOS 7 已停止维护，推荐使用 Rocky Linux、AlmaLinux 或 Ubuntu 作为替代。

---

## 三、源码编译安装

当包管理器没有所需软件、需要特定版本、或需要自定义编译选项时，使用源码编译安装。

### 通用步骤

```bash
# 1. 获取源码（以 Nginx 为例）
wget https://nginx.org/download/nginx-1.24.0.tar.gz
tar -xzvf nginx-1.24.0.tar.gz
cd nginx-1.24.0

# 2. 配置编译选项（生成 Makefile）
./configure --prefix=/usr/local/nginx \
            --with-http_ssl_module \
            --with-http_v2_module

# 3. 编译（仅编译，不安装）
make -j$(nproc)    # 使用所有 CPU 核心加速编译

# 4. 安装
sudo make install

# 5. 验证
/usr/local/nginx/sbin/nginx -v
```

### 常见依赖问题

编译前通常需要先安装构建工具和依赖库：

```bash
# Debian/Ubuntu
sudo apt install build-essential autoconf automake libtool \
                 libpcre3 libpcre3-dev zlib1g zlib1g-dev \
                 libssl-dev wget curl

# CentOS/RHEL
sudo yum groupinstall "Development Tools"
sudo yum install pcre-devel zlib-devel openssl-devel \
                 autoconf automake libtool wget curl
```

### 清理

```bash
make clean       # 清理编译产物（保留 Makefile）
make distclean   # 彻底清理（删除 Makefile 和配置）
sudo make uninstall  # 卸载（部分项目支持，需看源码）
```

### 适用场景

- 需要特定版本（包管理器版本太旧或太新）
- 需要自定义编译参数（如启用/禁用某个模块）
- 软件没有提供二进制包
- **缺点**：耗时、依赖管理需手动处理、升级麻烦

---

## 四、Docker 容器化安装

Docker 将软件及其依赖打包进容器镜像，不污染宿主机系统，是现代 Linux 部署的主流方式。

### 安装 Docker（Ubuntu 示例）

```bash
# 官方脚本一键安装
curl -fsSL https://get.docker.com | sudo sh

# 或手动安装
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker

# 将当前用户加入 docker 组（避免每次 sudo）
sudo usermod -aG docker $USER
newgrp docker    # 立即生效，无需重新登录
```

### 基本命令

```bash
# 镜像管理
docker pull nginx:latest          # 拉取镜像
docker images                     # 查看所有本地镜像
docker rmi nginx                  # 删除镜像
docker pull mysql:8.0             # 拉取指定版本

# 容器管理
docker run --name mynginx -d -p 80:80 nginx    # 启动容器（后台，端口映射）
docker ps                                    # 查看运行中的容器
docker ps -a                                 # 查看所有容器（含已停止）
docker stop mynginx                          # 停止容器
docker start mynginx                         # 启动容器
docker rm mynginx                            # 删除容器
docker logs mynginx                          # 查看容器日志
docker exec -it mynginx bash                 # 进入容器交互模式
```

### 实际示例：用 Docker 部署 MySQL

```bash
# 一键启动 MySQL
docker run --name mymysql -d \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=my-secret-pw \
  -v /data/mysql:/var/lib/mysql \
  mysql:8.0

# 进入容器
docker exec -it mymysql bash
mysql -u root -p

# 数据持久化在宿主机的 /data/mysql 目录
```

### Docker Compose（多容器编排）

```yaml
# docker-compose.yml
version: '3.8'
services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: my-secret-pw
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

```bash
docker-compose up -d          # 后台启动所有服务
docker-compose down           # 停止并删除所有容器
docker-compose logs -f nginx  # 查看指定服务日志
```

---

## 五、其他方式（快照、第三方仓库）

### AppImage（跨发行版桌面应用）

```bash
# 下载 .AppImage 文件后直接运行，无需安装
chmod +x application.AppImage
./application.AppImage
```

### Snap（Canonical 推出的通用包）

```bash
sudo snap install slack          # 安装
sudo snap list                   # 查看已安装的 snap 包
sudo snap refresh                # 更新所有 snap 包
```

### Flatpak（通用桌面应用格式）

```bash
flatpak install flathub org.gimp.GIMP   # 安装 GIMP
flatpak run org.gimp.GIMP               # 运行
```

### 第三方 APT 源（PPA）

```bash
# 添加第三方仓库（以 Node.js 官方源为例）
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs

# 添加 GPG 密钥（常见操作，防止警告）
curl -fsSL https://keys.openpgp.org/vks/v1/by-fingerprint/... | sudo gpg --dearmor -o /etc/apt/keyrings/keyring.gpg
```

---

## 小结

| 安装方式 | 适用场景 | 优点 | 缺点 |
|---------|---------|------|------|
| **APT** | Debian/Ubuntu 系的常规软件 | 简单、自动解决依赖 | 版本可能较旧 |
| **YUM/DNF** | RHEL/CentOS/Fedora 系的常规软件 | 简单、自动解决依赖 | 版本可能较旧，需 EPEL 扩展 |
| **源码编译** | 需要特定版本或自定义功能 | 灵活、版本可控 | 复杂、耗时、依赖手动处理 |
| **Docker** | 服务部署、微服务、隔离环境 | 环境一致、快速部署、可回滚 | 需要学习 Docker 概念 |
| **Snap/Flatpak** | 桌面应用、跨发行版通用 | 一键安装、版本最新 | 包体积大、启动慢 |

> 优先使用包管理器（apt/dnf），遇到版本不够或需要特定配置时用 Docker，特殊情况才考虑源码编译。
