---
title: Memcache 漏洞修复
date: 2019-06-03
tags: 
	- memcache
	- 漏洞
	- Memcached未授权访问漏洞
categories: 
    - 实战
    - 漏洞修复
---
# memcache漏洞修复

背景：Memcached未授权访问漏洞

## 查看memcache进程

```sh
ps -ef|grep memcache
```
## 杀死进程
```sh
kill -9 进程号
```

## 启动
```sh
# 绑定只有本机能访问
memcached -d -m 1024 -u root -l 127.0.0.1 -p 11211 -c 200 -P /tmp/memcached.pid
```