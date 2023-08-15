---
title: PhpStorm整理
date: 2017-02-01 10:21:20
tags:
	- IDEA
	- PHP 
	- 编辑器
	- 激活码
categories:
	- 学习笔记 
	- 03-PhpStorm相关笔记

---
# PhpStorm整理
> 获取临时激活码：http://jets.idejihuo.com
> IDEA临时激活码：http://www.idejihuo.com
## 试用期重置

[插件地址](https://gitee.com/pengzhile/ide-eval-resetter)

[备用插件地址](https://59-47-225-167.d.cjjd15.com:30443/download-cdn.123pan.cn/123-123/84a4999a/1654118-0/84a4999aed34aada39b3f728de2e247b?v=5&t=1682911840&s=16829118404c47510be209231c8e0520fc2046e9db&r=FYGSSR&filename=ide-eval-resetter-2.3.5-c80a1d.zip&x-mf-biz-cid=c4c15598-dcad-41c7-b5ef-f3d3c8a75a5a-584000&auto_redirect=0&xmfcid=698e5495-9736-479e-81c6-0188f331745b-cd8a62355-2430-24)

[教程](https://zhile.io/2020/11/18/jetbrains-eval-reset.html)

## 常用插件

- IdeaVim 使用vim的操作

- CodeGlance 代码迷你地图
- Chinese(Simplified) Language 中文语言包，汉化

## 常用配置

### 字体 Font

首选字体 JetBrains Mono

### 颜色和字体

方案 Monokai

### 活动模板

**a) pre + Tab**

设置缩写：pre      

模板文本：

```php
echo "<pre>";
print_r($arr);
echo "</pre>";
```

使用位置设置：php

**b) zhu + Tab 设置带名字的注释**

缩写：zhu   

模板文本：

```php
$DATE$ $TIME$ By 爱拆东西的程序员     
```

编辑变量：

![001.jpg](https://cdn.staticaly.com/gh/hfshaobing/picx-images-hosting@master/20230815/image.37l0939a45k0.webp)

 图1

**c) zhushi + Tab 设置详细信息注释**

缩写：zhushi 

模板文本：

```php
/** 
* 
* @param 
* @return 
* $DATE$ $TIME$ By 爱拆东西的程序员 
*/
```

编辑变量：如图1

## 常用快捷键

### mac版本

- command + shift + o 根据文件名，快速查询文件

- command + o 根据类名，快速查询文件

- command + f 查找当前文件

- command + r 查找替换
- command + shift + f 关键字全局查找
- command + shift + r 高级替换
- command + e 打开最近文件列表
- command + shift + [ 向左切换tab页
- command +shift + ] 向右切换tab页
- command + shift + +,- 展开或缩起
- command + 1切换左侧项目树结构
- command + 7 切换左侧类树结构
- command + F12 查看当前文件的结构