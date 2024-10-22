---
title: Linux 相关知识
date: 2022-02-01
tags: 
	- linux 
	- 常用命令
categories:
	- 学习笔记 
	- 06-Linux相关知识
---

# Linux 相关知识

## 常用命令

查看当前目录占用空间大小

```sh
du -sh 

du -h --max-depth=0 
```

ls查看文件大小指定单位

```sh
ls -l --block-size=m
```

排序

```sh
du -sm ./* |sort -rn
```

查看硬盘空间

```sh
df -h
```
Linux 基本信息展示
```sh
# 可显示电脑以及操作系统的相关信息。
uname -a
# 显示正在运行内核版本信息
cat /proc/version
# 显示的是发行版本信息
cat /etc/issue
```

 



 

