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

