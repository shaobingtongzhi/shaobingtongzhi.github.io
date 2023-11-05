---
title: TVBox打包记录
date: 2023-11-05
tags:
	- TVBox
	- Android 打包
	- APK
categories:
	- 杂文
    - 推荐
---

# TVBox 观影神器

## 介绍

一款开源的、可玩性极强的多端影音工具，用户可以自定义资源，市面上大部分影音工具都基于此二次开发。

开源地址：

[https://github.com/CatVodTVOfficial/TVBoxOSC](https://github.com/CatVodTVOfficial/TVBoxOSC)

![](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20231105/2023-11-05_174108.jfb3a6k9e9c.webp)

## 特点

- 支持各类爬虫源、XP源、采集源等
- 无限制，无广告
- 支持本地功能，聚合模式
- 支持去片头、片尾等

## 接口地址

https://raw.liucn.cc/box/m.json



## TVBox打包记录

### 步骤

第一步：下载打包工具

Android Studio  

版本：

![](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20231105/AndroidStudio版本.4wkupa6lafm0.webp)

第二步：拉取源代码

```sh
git clone https://github.com/CatVodTVOfficial/TVBoxOSC.git
```

第三步：修改gradle和gradle的匹配版本

原因：打包工具带的java版本有点高了，java17

参照链接：[https://blog.csdn.net/crasowas/article/details/130002017](https://blog.csdn.net/crasowas/article/details/130002017)

![](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20231105/动画.7l1z1v638ng0.gif)



### 问题

手动下载gradle

https://services.gradle.org/distributions/