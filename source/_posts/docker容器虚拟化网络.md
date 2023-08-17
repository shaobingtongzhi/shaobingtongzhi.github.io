---
title: docker 容器虚拟化网络
date: 2023-03-28 03:11
tag: 
	- docker
	- 容器
	- docker网络
categories:
	- 08-docker相关笔记
	- docker容器网络
---
# 容器虚拟化网络
## 查看docker支持的网络模型
```sh
docker network ls
```
## 查看桥接式网络元数据
```sh
docker network inspect bridge|host|none
```
## 自定义桥接网络模型
```sh
docker network create custom_net
```
## 把容器加入自定义网络中
```sh
docker network connect [网络名] [容器名]
```
## 把容器从自定义网络中移除
```sh
docker network disconnect [网络名] [容器名]
```
# 容器内安装ifconfig、ping等
## 容器内提取最新包信息
```sh
apt-get update
```
> E: List directory /var/lib/apt/lists/partial is missing. - Acquire (13: Permission denied)报错表示权限不足
> 解决办法：docker exec -u 0 -it [容器] bash
## 容器内安装net-tools（ifconfig命令）
```sh
apt-get install net-tools
```
## 容器内安装iputils-ping(ping命令)
```sh
apt-get install iputils-ping
```
## 容器内安装iproute2(ip addr命令)
```sh
apt-get install -y iproute2
```
## 容器内安装vim
```sh
apt-get install vim
```
## 容器内安装telnet
```sh
apt-get install -y telnet
```
# 补充

## 容器内部无法联通外部时使用
```sh
host.docker.internal
```