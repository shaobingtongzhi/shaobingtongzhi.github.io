---
title: Win10右键菜单设置
date: 2023-12-25
tags:
  - Windows
  - 右键菜单
categories:
  - 杂文
  - 实用方法
---



# 简介

Windows操作系统下自定义右键菜单功能有的时候可以实实在在的提高工作的效率

# 使用场景

打开注册表Win + R ,然后输入regedit

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240115/2024-01-15_103633.n3qpxosn1pc.webp)

## 情况一

桌面上右键弹窗里自定义

```
计算机\HKEY_CLASSES_ROOT\Directory\Background\shell
```

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231225/2023-12-25_145938.69tzqi3urs00.webp)

## 情况二

在某个文件夹上右键

```
计算机\HKEY_CLASSES_ROOT\Folder\shell\
```

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231225/2023-12-25_153818.5r2bz5ao0vg0.webp)

## 情况三

在文件上右键

```
计算机\HKEY_CLASSES_ROOT\*\shell\
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240429/Snipaste_2024-04-29_11-35-19.3jvem6jy35i0.webp)

实际效果：

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240429/Snipaste_2024-04-29_11-39-53.64s14kawhjo0.webp)

# 其他

关于 <font color=red>**鼠标单击右键时，菜单移动到鼠标左侧的问题**</font>

1. 如果想将菜单移动到图标的右侧，请按 Windows 键 + R 打开"运行"框。

2. 在运行框里输入 <font color=red>**shell:::{80F3F1D5-FECA-45F3-BC32-752C152E456E}** </font>并按 Enter。

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240310/2024-03-10_105710.66e4i0wdkpo0.webp)



3. 选择“其他”选项卡，切换“惯用右手”惯用左手“按钮，点击确定即可

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240310/2024-03-10_105817.1bbfzkextfc0.webp)