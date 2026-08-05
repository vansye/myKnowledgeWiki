---
type: knowledge
title: DNS 域名系统
created: 2026-08-05
updated: 2026-08-05
tags: [DNS, 网络, 运维, Linux]
source: 无
conclusion: DNS 是互联网的电话簿，将人类易记的域名（如 google.com）转换为机器可读的 IP 地址，是网络通信的基础服务。
---

## 详细

### 概念
DNS（Domain Name System，域名系统）是一种分布式数据库系统，负责将域名解析为对应的 IP 地址。它采用层次化的树状结构（根域、顶级域、二级域等），通过递归或迭代查询完成域名到 IP 的转换。

### 重点
- **核心功能**：正向解析（域名 → IP）；反向解析（IP → 域名）
- **常见记录类型**：
  - **A 记录**：域名指向 IPv4 地址
  - **AAAA 记录**：域名指向 IPv6 地址
  - **CNAME 记录**：域名别名（指向另一个域名）
  - **MX 记录**：邮件交换记录（指向邮件服务器）
  - **TXT 记录**：文本验证（如 SPF、DKIM）
- **查询流程**：浏览器缓存 → 系统 hosts 文件 → 本地 DNS 缓存 → 递归 DNS 服务器 → 根/顶级/权威 DNS 服务器
- **常用公共 DNS**：114.114.114.114（国内）、8.8.8.8（谷歌）、223.5.5.5（阿里）
- **Linux 相关命令**：
  - `dig domain.com` 查询域名的详细解析信息
  - `nslookup domain.com` 查询域名对应的 IP
  - `host domain.com` 简化版查询
  - 本地 hosts 文件：`/etc/hosts`（优先级高于 DNS 服务器）
  - 系统 DNS 配置：`/etc/resolv.conf`（指定 DNS 服务器地址）
- **Docker 环境注意**：容器默认继承宿主机 DNS 配置；可通过 `docker run --dns 8.8.8.8` 指定；自定义桥接网络支持容器名自动 DNS 解析