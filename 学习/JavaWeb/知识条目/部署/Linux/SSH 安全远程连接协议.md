---
type: knowledge
title: SSH 安全远程连接协议
created: 2026-08-03
updated: 2026-08-03
tags: [SSH, Linux, 网络, 安全, 运维]
source: 无
conclusion: SSH 是一种加密网络协议，用于在不安全网络上安全地远程登录服务器并执行命令，是 Linux 运维的核心工具，基于客户端-服务器架构，默认端口 22。
---

## 详细

### 概念
SSH（Secure Shell）是一种加密网络协议，用于在不安全网络中实现客户端与服务器之间的安全通信。它替代了不加密的 Telnet 和 rlogin，广泛应用于远程服务器管理、命令执行和文件传输。

### 重点
- **核心用途**：安全远程登录服务器；执行远程命令；传输文件（配合 scp/sftp）；端口转发与隧道代理
- **认证方式**：密码认证（需输入密码）；公钥认证（更安全，配置免密登录）
- **默认端口**：22（建议修改为非默认端口增强安全性）
- **配置文件**：`/etc/ssh/sshd_config`（服务端配置）；`~/.ssh/config`（客户端配置）
- **常用命令**：`ssh user@host` 远程登录；`ssh -p port user@host` 指定端口；`ssh-keygen -t rsa` 生成密钥对
- **免密登录配置**：`ssh-copy-id user@host` 将公钥上传至服务器 `~/.ssh/authorized_keys`
- **文件传输**：`scp local.txt user@host:/remote/` 上传文件；`sftp user@host` 交互式文件传输
- **安全建议**：禁用 root 直接登录，使用普通用户 + sudo；禁用密码登录，仅允许密钥认证；修改默认端口