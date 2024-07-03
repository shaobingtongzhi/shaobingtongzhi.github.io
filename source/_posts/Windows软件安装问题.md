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

# 安装

**windows 安装软件时出现2502，2503报错**

此现象是**安装权限不足**所致，此时，调用管理者权限cmd终端，终端内命令行安装即可。步骤如下：

第一步：右键开始菜单，选择Windows PowerShell（管理员）

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231204/2023-12-04_221928.1wqm3vvlwoao.webp)

第二步：进入安装包安装目录，执行安装命令即可

```sh
msiexec /package xxx.msi
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231204/2023-12-04_222054.2jmgnb8qk2m0.webp)

# 卸载

**windows 卸载软件时出现2502，2503报错**

第一步：进入安装程序目录 C:\Windows\Installer

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231204/2023-12-04_231136.3b4m6xhqciy0.webp)

第二步：执行下面3个步骤就可以找到对应程序的msi了

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231204/2023-12-04_231255.10whr236xedc.webp)

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231204/2023-12-04_231427.375g6xpe0tu0.webp)

第三步：管理员进入Windows PowerSheel

```sh
cd C:\Windows\Installer
```

第四步：执行第二步找到的msi

```sh
xxx.msi
```

参考地址：[查看](https://www.php.cn/faq/515351.html)

# 补充

出现这个问题，大多是因为当前用户对**C:\Windows\Temp**这个文件夹的**权限不够**造成的，所以**提权**才是解决的关键

1. 找到Temp这个文件夹，然后右键属性-》高级-》添加

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231222/2023-12-22_164705.1a6aarsqzykg.webp)

2. 选择主体-》填入当前账号名称-》把权限都勾上即可

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231222/2023-12-22_164848.6zhs0hayz08.webp)

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231222/2023-12-22_164958.6548nzbrzmg0.webp)
