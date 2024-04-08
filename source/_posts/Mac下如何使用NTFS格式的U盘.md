---
title: Mac下如何使用NTFS格式的U盘
date: 2023-12-13
tag:
  - Mac
  - UTFS
  - U盘
categories:
  - 杂文
  - 实用方法
---

> 在Mac上使用U盘时，发现只能读取，不能写入，因为U盘的格式时Windows的NTFS格式，有什么办法可以让我们能正常读写U盘呢？

# 方案一

**步骤**

第一步：插入U盘

第二步：打开终端输入下面命令，记住U盘的名字

```sh
diskutil list
```

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20231213/%E6%88%AA%E5%B1%8F2023-12-13-%E4%B8%8B%E5%8D%8812.14.46.5e8muw6vjcg0.png)

第三步：在/etc目录下创建一个文件fstab，并写入下面的内容

```sh
#创建fstab文件
sudo touch /etc/fstab
sudo vim /etc/fstab
#写入下面内容
LABEl=U启动U盘 none ntfs rw,auto,nobrowse
#保存退出
:wq
```

第四步：从桌面退出U盘，重新插入U盘，这个时候桌面上已经看不到U盘了

第五步：在 **启动台** 里找到 **磁盘工具** ，找到U盘，右键在访达显示即可

# 方案二

安装软件，推荐使用Mounty

官网地址：https://mounty.app

问题：每次**拔掉U盘前**，一定要**先卸载U盘再退出软件**，否则Mac就无法再识别U盘了

解决办法：把U盘再windows系统的电脑上重新退出一下就可以了