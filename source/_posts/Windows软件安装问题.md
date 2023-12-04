---
title: Windows软件安装报2502、2503问题
date: 2023-12-04
tags:
  - 2502
  - 2503
categories:
  - 杂文
  - 实用方法
---

windows 安装软件时，时常出现2502，2503错误代码报错。

此现象是**安装权限不足**所致，此时，调用管理者权限cmd终端，终端内命令行安装即可。步骤如下：

第一步：右键开始菜单，选择Windows PowerShell（管理员）

![](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20231204/2023-12-04_221928.1wqm3vvlwoao.webp)

第二步：进入安装包安装目录，执行安装命令即可

```sh
msiexec /package xxx.msi
```

![](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20231204/2023-12-04_222054.2jmgnb8qk2m0.webp)