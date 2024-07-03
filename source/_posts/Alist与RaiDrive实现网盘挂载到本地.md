---
title: Alist与RaiDrive实现把网盘挂载在本地
date: 2023-11-14
tags:
  - Alist
  - RaiDrive
  - 网盘整合
categories:
  - 杂文
  - 实用方法
---



> 1. 解决网盘账号太多，需要下载不同的客户端，登录不同的账号问题
> 2. 让网盘变成类似我们系统的磁盘一样使用

# Alist

一个支持多种存储的文件列表程序

官网地址：https://alist.nn.ci/zh/

## 守护进程

用 **`.VBS`** 脚本启动和停止，分别创建两个脚本 分别是 启动.vbs 和 停止.vbs

直接在和Alist启动程序同级文件夹里面双击启动即可，不用担心没有反应 直接去 浏览器访问即可

启动.vbs

```vbscript
Dim ws
Set ws = Wscript.CreateObject("Wscript.Shell")
ws.run "alist.exe server",vbhide
Wscript.quit
```

停止.vbs

```vbscript
Dim ws
Set ws = Wscript.CreateObject("Wscript.Shell")
ws.run "taskkill /f /im alist.exe",0
Wscript.quit
```

## 实现自启动

1. 将启动.vbs设置快捷方式
2. win + R 输入：shell:startup
3. 将生成的快捷方式复制进文件夹即可

# RaiDrive

一款能够将各种云存储和网络存储设备（NAS）映射为本地磁盘的软件，让您可以在文件资源管理器中看到一个新的磁盘驱动器，并直接在这个磁盘上操作文件，就像在本地磁盘上一样

官网地址：[RaiDrive](https://www.raidrive.com/)

下载地址：[下载](https://app.raidrive.com/d86ea6fa40f74010914976063f94774b/release/stable/RaiDrive_2023.9.16.8_x64.msi)

## 安装配置

默认安装就可以了，由于网络问题可能一次安装不成功，多尝试吧

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231226/2023-12-26_164119.7kq5osta0q80.webp)

**实在是安装不上就采用下面的方式吧**

# rclone

除了RaiDrive以外，rclone也可以把Alist映射到本地，并且是开源免费的，不会有弹窗广告，但是对新手不太友好

## 准备软件

1. 下载rclone安装包，[下载地址](https://github.com/rclone)，根据情况下载对应的版本就可以了

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231226/2023-12-26_165752.3wwzb15g7zg0.webp)

2. 下载winfsp，[下载地址](https://winfsp.dev/rel/) 在windows操作系统下是必须的

## 安装配置

1. 解压rclone-v1.65.0-windows-amd64.zip，并把解压出来的文件直接拷贝到自己的安装目录即可。我一般是D:\Program

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231226/2023-12-26_170614.q51caic82jk.webp)

2. 安装winfsp，选择好安装目录，其他的默认安装就可以了
3. 进入rclone.exe所在目录进行配置

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231226/2023-12-26_171216.5nhxok1dvg80.webp)

默认生成的配置文件在：C:\Users\xxx\AppData\Roaming\rclone\rclone.conf

4. 进入rclone.exe所在目录执行挂载命令

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231226/2023-12-26_170911.4cx3dc2x6ly0.webp)

```sh
rclone mount 远程磁盘: k: --network-mode --header "Referer:" --multi-thread-streams 8 --buffer-size 512M  --vfs-fast-fingerprint --vfs-cache-mode full --no-modtime --file-perms 0777
```

5. 这个时候就可以看到已经挂载完成了

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20231226/2023-12-26_172306.38ulgd6nkxq0.webp)

## 开机自启动

1. 将配置文件拷贝到rclone安装目录

2. 新建 **start.bat**

```bat
D:\Program\rclone-v1.65.0-windows-amd64\rclone.exe mount alist:/ k: --config=D:/Program/rclone-v1.65.0-windows-amd64/rclone.conf --network-mode --header "Referer:" --multi-thread-streams 8 --buffer-size 512M  --vfs-fast-fingerprint --vfs-cache-mode full --no-modtime --file-perms 0777 --no-console --log-file D:/Program/rclone-v1.65.0-windows-amd64/start.log
```

解释：

alist:/  是配置文件中webdav的名称

k: 是映射到本地的磁盘盘符

--no-console 表示后台启动

--log-file 表示日志文件路径

3. 新建 **启动rclone.vbs**

```vbscript
Dim ws
Set ws = Wscript.CreateObject("Wscript.Shell")
ws.run "cmd /c D:/Program/rclone-v1.65.0-windows-amd64/start.bat",vbhide
Wscript.quit
```

4. 对 启动rclone.vbs 创建快捷方式，并放入 自启动目录（运行-》shell:startup）

5. 新建 **停止.vbs**

```vbs
Dim ws
Set ws = Wscript.CreateObject("Wscript.Shell")
ws.run "taskkill /f /im rclone.exe",0
Wscript.quit
```

