---
title: Linux 磁盘空间爆满，服务无法访问
date: 2021-06-07
tags: 
	- docker
	- 磁盘爆满
	- 容器
	- Linux
categories:
    - 实战
    - 问题处理
---

# 原因：
docker引擎默认是安装在系统盘下 /var/lib/docker 一般来说系统盘比较小，当空间不够时会造成docker运行不正常


造成空间不足的原因：

> 1. 没有用的镜像没有及时删除
> 2. 没有用的容器没有及时删除
> 3. docker容器运行中产生的各种垃圾缓存
> 4. 系统产生的垃圾
> 5. 人为。。。

# 解决方案：
把docker迁移到数据盘，所以根本问题是保证磁盘的空间充足

将docker的目录改到 /data  下
```sh

# 创建docker目录
mkdir -p /data/docker

# 关停docker服务
service docker stop 

# 移动docker到新建的目录
mv /var/lib/docker/* /data/docker/

# 删除旧的doker目录
rm -rf /var/lib/docker

# 建立软连接 
ln -s /data/docker /var/lib/docker

# 重启docker
service docker start
```
