---
type: 学习笔记
title: Linux 系统概述
created: 2026-08-03
updated: 2026-08-03
tags: [Linux, 操作系统, 基础, 内核, 文件系统]
subject: 部署/Linux
---

> Linux 是一个基于 Unix 的多用户、多任务操作系统，核心由内核和 GNU 工具链组成，采用树形文件系统结构和严格的用户权限模型，广泛应用于服务器、云计算和嵌入式设备。

## 目录

- [一、Linux 核心组成](#一linux-核心组成)
- [二、文件系统结构](#二文件系统结构)
- [三、用户与权限模型](#三用户与权限模型)
- [四、进程管理](#四进程管理)
- [五、网络基础](#五网络基础)
- [六、包管理](#六包管理)
- [七、系统服务（systemd）](#七系统服务systemd)
- [八、环境变量](#八环境变量)
- [小结](#小结)

---

## 一、Linux 核心组成

Linux 是一套类 Unix 操作系统，以**内核（Kernel）**为核心，配合 GNU 工具链和用户空间程序，构成完整的操作系统。Linux 并非指单一系统，而是内核名称——市面上各种 Linux 发行版（Ubuntu、CentOS 等）都是基于同一内核的不同"套壳"组合。

### 核心组成

| 组成部分 | 说明 |
|---------|------|
| **内核（Kernel）** | 操作系统最底层，管理 CPU、内存、进程、设备驱动等硬件资源 |
| **Shell** | 用户与内核交互的接口，接收命令并转发给内核执行（如 bash、zsh） |
| **文件系统** | 以 `/` 为根的树形结构，所有设备和文件都是文件系统中的节点 |
| **用户空间程序** | 各类工具和软件（编译器、文本编辑器、网络工具等） |

```
用户 → Shell（bash） → 系统调用 → 内核 → 硬件驱动 → 硬件
```

---

## 二、文件系统结构

Linux 文件系统从根目录 `/` 开始，所有文件都在一棵树形结构中。

### 常见目录及用途

| 路径 | 说明 | 存放内容 |
|------|------|---------|
| `/` | 根目录 | 整个文件系统的起点 |
| `/bin` / `/usr/bin` | 基本命令 | 普通用户可用的可执行程序（ls、cp、grep 等） |
| `/sbin` / `/usr/sbin` | 系统管理命令 | 管理员才能执行的命令（fdisk、ifconfig 等） |
| `/etc` | 配置文件 | 系统和服务的配置文件 |
| `/home` | 用户家目录 | 普通用户的个人文件（`/home/用户名`） |
| `/root` | root 用户的家目录 | root 用户的个人文件 |
| `/var` | 可变数据 | 日志文件（`/var/log`）、缓存、邮件等频繁变化的数据 |
| `/tmp` | 临时文件 | 系统临时目录，重启后内容清空 |
| `/proc` | 进程信息 | 伪文件系统，反映内核和进程状态（如 `/proc/cpuinfo`） |
| `/dev` | 设备文件 | 硬件设备的抽象（如 `/dev/sda` 代表第一块硬盘） |
| `/opt` | 可选软件 | 第三方大型软件的安装目录 |
| `/boot` | 启动文件 | 内核镜像、引导加载器配置文件 |
| `/lib` / `/usr/lib` | 共享库 | 程序运行时依赖的动态链接库 |

> **记忆技巧**：`/etc` = configuration，`/var` = variable，`/proc` = process，`/dev` = device。

---

## 三、用户与权限模型

Linux 是**多用户**系统，每个文件/目录都有明确的所有者和权限。

### 三类用户角色

| 角色 | 说明 |
|------|------|
| **root**（超级用户） | UID=0，拥有系统最高权限，可以执行任何操作 |
| **普通用户** | 日常使用，权限受限制 |
| **系统用户** | 运行特定服务（如 www-data 运行 Nginx），通常不能登录 |

### 权限类型

| 权限 | 符号 | 对文件的作用 | 对目录的作用 |
|------|------|------------|------------|
| 读 | `r`（4） | 查看文件内容 | 列出目录内容 |
| 写 | `w`（2） | 修改文件内容 | 在目录中创建/删除文件 |
| 执行 | `x`（1） | 运行文件为程序 | 进入该目录（cd） |

### 权限表示方式

```
-rwxr-xr--  user  group  size  date  filename
 │││  │││  │││
 │││  │││  └── 其他人（other）：读
 │││  └──── 组用户（group）：读+执行
 ││└─────── 所有者（user）：读+写+执行
 │└──────── 文件类型（- 普通文件，d 目录，l 软链接）
 └───────── 无特殊权限
```

### 修改权限

```bash
chmod 755 script.sh       # 所有者 rwx，组 r-x，其他人 r-x
chmod +x app              # 给所有用户添加执行权限
chmod 600 private.key     # 仅所有者可读写（常用于密钥文件）
```

> 权限数字：`4`=读、`2`=写、`1`=执行，格式为 `所有者 组用户 其他人`，如 `755` = `rwx r-x r-x`。

---

## 四、进程管理

Linux 中的一切都是进程，每个进程都有唯一 PID。

### 进程状态

| 状态 | 说明 |
|------|------|
| 运行中（Running） | 正在 CPU 上执行或等待执行 |
| 休眠（Sleeping） | 等待某个事件（如 I/O） |
| 停止（Stopped） | 被暂停（如收到 SIGSTOP 信号） |
| 僵尸（Zombie） | 进程已结束但父进程未回收 |

### 常用命令

```bash
ps aux                     # 查看所有进程列表
ps -ef                     # 另一种进程查看格式
top                        # 实时查看进程 CPU/内存占用（按 q 退出）
htop                       # top 的增强版，彩色可视化（需安装）
kill PID                   # 发送默认 SIGTERM 终止进程
kill -9 PID                # 强制终止（SIGKILL，不可忽略）
pkill -f "python"          # 按名称终止进程
jobs                       # 查看当前 shell 的后台任务
fg %1                      # 将后台任务调回前台
```

**进程优先级**：`nice` 和 `renice` 调整进程优先级（-20 最高，19 最低）。

---

## 五、网络基础

```bash
# 查看网络配置
ip addr                  # 查看 IP 地址（替代 ifconfig）
ip route                 # 查看路由表
ss -tlnp                 # 查看监听中的端口
ping 8.8.8.8             # 测试网络连通性
curl http://example.com  # 发送 HTTP 请求

# 查看网络连接
netstat -tlnp            # 查看 TCP 连接（旧命令，被 ss 替代）
```

---

## 六、包管理

不同发行版的包管理器不同：

| 发行版系列 | 包管理器 | 安装命令 | 说明 |
|----------|---------|---------|------|
| Debian/Ubuntu | `apt` | `apt install pkg` | 使用 `.deb` 包，依赖管理优秀 |
| Red Hat/CentOS | `yum` / `dnf` | `dnf install pkg` | 使用 `.rpm` 包，RHEL 系标准 |
| Arch | `pacman` | `pacman -S pkg` | 滚动更新，速度快 |
| openSUSE | `zypper` | `zypper install pkg` | SUSE 系 |

```bash
# Ubuntu/Debian
sudo apt update           # 刷新软件包索引
sudo apt upgrade          # 升级所有已安装包
sudo apt install nginx    # 安装软件
sudo apt remove nginx     # 卸载软件（保留配置）
sudo apt autoremove       # 清理不再需要的依赖

# CentOS/RHEL
sudo dnf install nginx
sudo dnf remove nginx
```

---

## 七、系统服务（systemd）

现代 Linux 使用 `systemd` 管理服务，取代了旧的 init 系统：

```bash
# 服务管理
sudo systemctl start nginx     # 启动服务
sudo systemctl stop nginx      # 停止服务
sudo systemctl restart nginx   # 重启服务
sudo systemctl status nginx    # 查看服务状态
sudo systemctl enable nginx    # 设置开机自启
sudo systemctl disable nginx   # 取消开机自启

# 查看系统信息
systemctl list-units --type=service   # 列出所有服务
journalctl -u nginx                   # 查看服务日志
```

---

## 八、环境变量

| 概念 | 说明 |
|------|------|
| **环境变量** | 系统级别的键值对，影响所有进程的行为（如 `$PATH`、`$HOME`） |
| **配置文件** | `~/.bashrc`（交互式 shell 启动时读取）、`~/.profile`（登录时读取） |

```bash
echo $HOME            # 查看家目录
echo $PATH            # 查看命令搜索路径
export JAVA_HOME=/usr/local/java   # 设置环境变量（仅当前 shell 生效）
source ~/.bashrc      # 重新加载配置（使修改立即生效）
env                   # 查看所有环境变量
```

---

## 小结

1. **Linux = 内核 + GNU 工具 + 用户程序**，内核是资源管理器，Shell 是交互接口
2. **一切皆文件**：设备、进程、配置都以文件形式存在于树形目录 `/` 中
3. **权限三要素**：读（r=4）、写（w=2）、执行（x=1），分所有者/组/其他人三层
4. **多用户系统**：root 权限极高，日常操作应使用普通用户 + sudo
5. **进程是核心概念**：每个任务都是一个进程，`top`/`ps`/`kill` 是进程管理三板斧
6. **包管理因发行版而异**：Debian 系用 `apt`，RHEL 系用 `dnf`/`yum`
7. **systemd 管理服务**：`systemctl` 是 systemd 的核心命令，管理服务的启停和开机自启

> Linux 的知识体系庞大，以上内容是入门必读的核心骨架，后续围绕这些概念展开即可。

<!-- KB:ANNOTATIONS -->
