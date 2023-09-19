---
title: Linux 下 java 输出中文乱码问题
date: 2016-05-01
tags: 
	- java中文乱码
categories:
	- 实战
	- 问题处理
---


1.检查java文件编码是否是UTF-8
2.检查系统环境是否安装了中文库
```sh
locale -a |grep zh_CN
```
如果没有中文库，则先安装中文库
> 安装中文库

```sh
sudo yum install kde-l10n-Chinese
```
> 添加UTF-8字符集

```sh
localedef -c -f UTF-8 -i zh_CN zh_CN.utf8
```
> 配置环境变量

```sh
vi /etc/profile.d/java_env.sh
LANG=zh_CN.UTF-8
export LANG
```