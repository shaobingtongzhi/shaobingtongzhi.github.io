---
title: Windows的虚拟内存
date: 2023-12-16
tags:
  - 虚拟内存
  - C盘瘦身
categories:
  - 杂文
  - 实用方法
---

> C盘臃肿不堪，很有可能是因为虚拟内存占用的磁盘空间，本文将教你将虚拟内存设置在非系统盘符，在不影响性能的情况下达到瘦身的效果

1. 通过显示隐藏文件查看C盘根目录下虚拟内存文件占用大小

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231216/2023-12-15_235853.4yl8276g3bs0.webp)

2. 设置到非系统盘

​		计算机->属性->高级系统设置->性能->设置->高级->更改；一般设置为真实内存的1.5 ~ 2倍即可

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231216/2023-12-16_001519.119zdju73xfk.webp)