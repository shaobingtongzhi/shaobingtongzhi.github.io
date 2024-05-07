---
title: VirtualBox安装Centos
date: 2018-06-01
tags:
	- VirtualBox
	- CentOS
	- Linux
categories:
    - 学习笔记
    - 06-Linux相关知识
---

# VirtualBox安装CentOS

## 准备工作

宿主机：macbook
VirtualBox版本：6.1.12 r139181 (Qt5.6.3)
CentOS版本：7.9.2009 x86_64

下载地址：http://mirrors.163.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2009.iso

## 创建虚拟机

### 新建虚拟机
名称自定义，建议写操作系统名称

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/新建虚拟机.28nx0juzzw9w.webp)
### 内存设置
内存大小建议选择 2048M

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/内存设置.6ig93fxywus0.webp)
### 创建虚拟硬盘
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/创建虚拟硬盘.hv6lvzggjc8.webp)
### 虚拟硬盘文件类型
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/选择硬盘文件类型.63xpcmihllg0.webp)
### 动态分配
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/动态分配.5oxmncdems00.webp)
### 文件位置和大小
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/文件位置和大小.uzm3jp1fncg.webp)
### 创建完成
完成后，进入首页进行盘片设置
### 安装盘片（镜像文件）
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/安装盘片_1.7klmmnczxb00.webp)

选择下载好的镜像文件
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/安装盘片_2.hyq7tp9edi8.webp)
### 网络设置

网卡1:NAT网络

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/网卡_1.4cqzlfn58nc0.webp)

网卡2:仅主机（Host-Only）网络
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/网卡_2.3ax4siszm5m0.webp)
## 安装系统

### 启动虚拟机
用上下键选择 Install CentOS7,回车

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/Install-CentOS7.4al7a1qt71m0.webp)
### 语言选择
选中文，点继续
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/语言选择.3vfwgfwdofw.webp)
### 安装信息摘要
必须将黄色感叹号的内容配置好后，才能开始安装
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/安装信息摘要.53ju5g6uexw0.webp)
点击安装位置，进入后直接点击完成，黄色叹号即可消失
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/安装位置.2x46o4vwedq0.webp)
### 配置
设置root密码，完成即可
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/设置root密码.txer4aw2di8.webp)
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/设置密码.6yyz3wvx0og0.webp)
### 安装完成，重启
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/完成重启.3rgg2nd8oaw0.webp)
## 网络设置

### 访问外网

enp0s3网卡设置
```sh
cd /etc/sysconfig/network-scripts/

vi ifcfg-enp0s3

```
按图中所示进行设置，完成后重启网卡，进行测试

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/enp0s3.66x4mfblqwo0.webp)

```sh
service network restart

ping baidu.com
```
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/访问外网.3t0h0da588a0.webp)

### 与宿主机互通
enp0s8网卡设置
```sh
cd /etc/sysconfig/network-scripts/

vi ifcfg-enp0s8

```
按图中所示进行设置，完成后重启网卡，进行测试

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/enp0s8.25123q59mmm8.webp)

```sh
service network restart

ping 192.168.0.112
```
![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20230907/访问宿主机.761lpovrls80.webp)
