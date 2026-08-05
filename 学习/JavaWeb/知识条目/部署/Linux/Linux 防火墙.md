---
type: knowledge
title: Linux 防火墙
created: 2026-08-03
updated: 2026-08-03
tags: [Linux, 防火墙, firewalld, iptables, 运维, 安全]
source: 无
conclusion: Linux 防火墙通过包过滤规则控制网络流量，是系统安全的第一道防线，主流方案包括 iptables（底层）和 firewalld（动态管理），云环境常用安全组实现类似功能。
---

## 详细

### 概念
Linux 防火墙是一种基于 Netfilter 框架的网络流量控制机制，通过定义规则链对进出系统的数据包进行过滤、转发或丢弃，从而保护服务器免受未授权访问和网络攻击。

### 重点
- **核心功能**：允许/拒绝特定 IP 或端口的访问；实现 NAT 网络地址转换；防御 DDoS 等基础攻击
- **主流方案**：
  - **iptables**：传统命令行工具，规则即时生效，配置复杂但灵活，适用于精细控制
  - **firewalld**：动态防火墙管理工具（CentOS/RHEL 7+ 默认），支持运行时与永久配置分离，基于 zone（区域）管理规则
  - **ufw**：Ubuntu 简化版前端，适合初学者
- **常用 firewalld 命令**：
  - `firewall-cmd --add-port=8080/tcp --permanent` 开放端口
  - `firewall-cmd --reload` 重载配置
  - `firewall-cmd --list-all` 查看当前规则
- **常用 iptables 命令**：
  - `iptables -L -n -v` 查看规则
  - `iptables -A INPUT -p tcp --dport 22 -j ACCEPT` 允许 SSH
- **云环境补充**：云厂商（阿里云、AWS 等）的安全组（Security Group）工作在虚拟机外部，优先级高于系统防火墙，需同时配置
- **注意事项**：配置前确认 SSH 端口已放行，避免远程锁定；生产环境建议限制管理 IP 白名单；修改后测试连通性