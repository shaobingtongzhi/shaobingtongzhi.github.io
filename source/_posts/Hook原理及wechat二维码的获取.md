---
title: Hook原理及wechat二维码的获取
date: 2023-09-19
tags: 
	- Wechat
	- Hook
	- wechat.exe 3.7.0.29
categories:
	- 学习笔记
	- 10-Wechat Hook相关笔记
---

# Hook原理介绍

## 什么是hook 

英文意思：钩子

## 原理

如下图：

![](https://cdn.statically.io/gh/hfshaobing/picx-images-hosting/raw/master/20230919/Hook原理.6uz0r235zl80.webp)

# 微信二维码获取

## 使用工具

OD 全称 OllyDebug，一种反汇编软件，动态追踪工具，Ring 3 级的调试器

CE 全称 Cheat Engine，是一款专注于游戏的修改器。它可以用来扫描游戏中的内存，并允许修改它们。

## 前置知识

IHDR 在PNG格式图像中，文件头数据块（IHDR）是第一个出现的数据块，用于描述整个图像的基本属性和特征。文件头数据块的标识符为IHDR，包含13个字节的数据。

## 获取相关信息

### 获取二维码图片偏移

1. 打开微信，保持停留在登录二维码界面
2. 打开CE，附加微信进程，附加后搜索IHDR，校正地址-c，显示出 png
![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20230919/CE附加微信.6b10rojxepg0.webp)
![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20230919/CE查询IHDR.5g6n51qoiwk0.webp)
![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20230919/CE寻找二维码地址.5cbt3v5vqj00.webp)
3. 打开OD，附件微信进程，dd找出来的指针地址
补充：
通过dc [地址] 查看内容

通过dm 命令打印内容
1. 新建一个文件 打印.txt
2. 输入如下内容
[064FB580] 表示上面找到的指针地址，在OD中查看
E78表示图片的大小 ，在OD中查看
"w_qrcode"表示保存的图片名称，可以自定义
```sh
dm [064FB580],E78,"w_qrcode.png"
ret
```
注意： 图片保存地址为微信的安装目录

### 获取二维码hook偏移

在数据窗口找到的地址上，下一个内存写入断点，通过刷新二维码进行单步调试，直到找到合适的hook地址，最好找到一个寄存器中的地址与数据窗口中的图片地址相近的。

```
6C2869FE    8B4F 30         mov ecx,dword ptr ds:[edi+0x30]
6C286A01    6A 01           push 0x1
6C286A03    6A 00           push 0x0
6C286A05    E8 2886FB00     call WeChatWi.6D23F032                   ; 这里也可以hook
6C286A0A    8B4F 38         mov ecx,dword ptr ds:[edi+0x38]
6C286A0D    6A 00           push 0x0
6C286A0F    8B01            mov eax,dword ptr ds:[ecx]               ; WeChatWi.6DFC13C8
```
![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20230919/二维码hook地址.2g1ke18by3f.webp)
二维码Hook偏移：  376A05 = 指令地址-WechatWin.dll基址 = 6C286A05-6BF10000
二维码图片地址偏移： 3230 = 064FB580-064F8350(ECX)   
二维码图片地址： ECX + 3230
二维码图片大小地址： ECX + 0x3230 + 0x4


WechatWin.dll的基址可以在OD上方工具栏的e中进行查看，也可以在CE中手动填加地址获取

### 获取指令A的偏移

call偏移： 132F032 = 6D23F032-6BF10000

### 获取指令B的偏移

要返回的地址偏移： 376A0A = 6C286A0A-6BF10000

