---
title: Oshi服务器信息采集
date: 2024-10-22
categories:
  - 备忘
tags:
  - maven
  - 获取服务器信息
  - 服务器
  - springboot
  - mvn
---

# 简介

在常规的开发中，能够监控服务器的运行情况至关重要

比如：

1. 监控CPU信息

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241022/Snipaste_2024-10-22_09-08-34.4bkcr1eos620.webp)

2. 监控磁盘状态

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241022/Snipaste_2024-10-22_09-09-09.1w141f3oplmo.webp)

3. 监控 jvm 信息

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241022/Snipaste_2024-10-22_09-08-59.144z59uya7cw.webp)

4. 监控服务器信息

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241022/Snipaste_2024-10-22_09-08-45.oiqvvzkleps.webp)



OSHI是一个免费的基于JNA的Java操作系统和硬件信息库。它<font color=red>**不需要安装任何额外的本机库**</font>，旨在提供<font color=red>**跨平台**</font>**实现来**<font color=red>**检索系统信息**</font>，如操作系统版本、进程、内存和CPU使用情况、磁盘和分区、设备、传感器等。

开源地址：https://github.com/oshi/oshi

使用方法：https://github.com/xkcoding/spring-boot-demo/tree/master/demo-websocket