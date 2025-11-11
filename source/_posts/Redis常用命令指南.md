---
title: Redis常用命令指南
date: 2025-11-11
categories:
  - 学习笔记
  - 04-Redis相关笔记
tags:
  - redis
  - redis命令
---

# 常用指令

## 连接Redis并查看配置

```sh
# 连接到Redis服务器
redis-cli
# 查看timeout配置
CONFIG GET timeout
# 查看最大客户端连接数
CONFIG GET maxclients
# 查看所有相关配置
CONFIG GET *
```

## 查看连接状态

```sh
# 查看当前连接数
CLIENT LIST
# 查看服务器统计信息
INFO clients
INFO server
```

## 在线重载配置

```sh
# 重新加载配置文件
CONFIG REWRITE
# 或者直接设置配置项
CONFIG SET timeout 0
```



