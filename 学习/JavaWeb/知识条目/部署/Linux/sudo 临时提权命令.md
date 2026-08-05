---
type: knowledge
title: sudo 临时提权命令
created: 2026-08-03
updated: 2026-08-03
tags: [Linux, 权限, 命令, 运维]
source: 无
conclusion: sudo 允许授权用户以超级用户（root）或其他用户身份执行命令，是 Linux 系统管理中的核心权限控制工具，避免长期使用 root 账户带来的安全风险。
---

## 详细

### 概念
`sudo`（Superuser DO）是一个 Linux/Unix 命令，允许普通用户以 root 或其他指定用户的权限执行特定命令，同时通过 `/etc/sudoers` 配置文件精确控制授权范围，并记录所有操作日志。

### 重点
- **核心作用**：以 root 身份执行管理任务（如安装软件、修改系统文件），而无需切换为 root 用户
- **基本用法**：`sudo <命令>`，首次执行需输入当前用户密码（非 root 密码）
- **关键参数**：`-u <用户>` 以指定用户身份执行；`-i` 模拟初始登录环境；`-l` 查看当前用户可执行的命令列表
- **与 su 的区别**：`su -` 切换为 root 用户（需 root 密码），`sudo` 仅临时提权执行单条命令（需当前用户密码），更安全可控
- **配置文件**：`/etc/sudoers` 中的规则如 `username ALL=(ALL:ALL) ALL`，需使用 `visudo` 命令编辑以防止语法错误
- **安全策略**：`secure_path` 限制命令路径，`log_output` 启用操作审计
- **常见场景**：软件包管理（`apt install`、`yum install`）；系统服务管理（`systemctl`）；编辑受保护配置文件（`sudo vim /etc/hosts`）
- **注意事项**：谨慎使用 `sudo rm -rf` 等高危命令，执行危险操作前建议先确认路径；不建议在 sudo 下运行图形界面程序