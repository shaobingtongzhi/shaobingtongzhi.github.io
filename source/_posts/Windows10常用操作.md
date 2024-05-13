---
title: Window10常用操作汇总
date: 2024-01-24
tags:
  - Windows
  - 常用操作
categories:
  - 杂文
  - 实用方法
---



1. 打开类似于Win7的计算机属性

**控制面板\系统和安全\系统**

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240124/Snipaste_2024-01-24_18-56-16.1v4bn7fvyjeo.webp)

2. 桌面显示 计算机、控制面板等

右键-》个性化-》主题-》桌面图标设置

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240124/Snipaste_2024-01-24_18-58-22.17gwl5u19l1c.webp)

3. 关闭Windows defender拦截

Windows安全中心-》病毒和威胁防护-》管理设置-》添加和删除排除项

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240507/Snipaste_2024-05-07_14-36-50.z3lso0qbfhs.png)

4. 防火墙开启后无法 ping 通？

要允许ICMP数据包（例如ping请求）通过Windows防火墙，您需要修改入站规则来允许ICMP通信。

控制面板-》系统和安全-》Windows Defender防火墙-》高级设置-》入站规则

在右侧，滚动找到和选择名为“文件和打印机共享 (Echo Request - ICMPv4-In)”的规则，右键启用规则

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240509/Snipaste_2024-05-09_11-01-23.2myzdbtw9li0.webp)