---
title: Linux部署Nginx-UI
date: 2024-12-19
categories:
  - 实战
  - Nginx服务器
tags:
  - 服务器管理
  - nginx管理
  - nginx-ui
  - nginx
---

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241219/Snipaste_2024-12-19_15-56-51.1pqg4repl940.webp)

项目地址：https://github.com/0xJacky/nginx-ui

# 简介

通过这个开源的项目，可以在线查看服务器 CPU、内存、系统负载、磁盘使用率等指标；在线编辑 Nginx 配置文件；在线查看 Nginx 日志

# 步骤 

## 1. 下载运行

根据服务器情况下载对应版本的软件包

- 查看服务器架构

```sh
uname -m 
# x86_64 表示 64 位架构；i386 表示 32 位架构
```

- 解压

```sh
tar zxvf nginx-ui-linux-64.tar.gz
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241219/Snipaste_2024-12-19_16-08-32.1wh2a7541xpc.webp)

首次解压下来只有 nginx-ui 这个可执行文件

- 运行

```sh
./nginx-ui -config app.ini

# 会自动生成app.ini,根据里面的设置通过 ip:port就可以访问了，然后设置完后会自动生成 database.db
```

- 配置nginx的相关内容到  app.ini 里

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241219/Snipaste_2024-12-19_16-12-33.19y43lgwdqn4.webp)

- 关闭服务

```sh
kill -9 $(ps -aux | grep nginx-ui | grep -v grep | awk '{print $2}')
```

## 2. 设置 systemd 服务

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241219/Snipaste_2024-12-19_16-14-59.25sxirds3kr.webp)

- 新建 nginx-ui.service 文件

```sh
cd /usr/li/systemd/system
touch nginx-ui.service
```

内容如下：

```sh
[Unit]
Description=Yet another WebUI for Nginx
Documentation=https://github.com/0xJacky/nginx-ui
After=network.target

[Service]
Type=simple
ExecStart=/home/nginx-ui/nginx-ui -config /home/nginx-ui/app.ini
Restart=on-failure
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

- 新建 nginx-ui.service.d 目录

```sh
cd /usr/li/systemd/system
mkdir -p nginx-ui.service.d
```

- 重新加载配置

```sh
systemctl daemon-reload
```

- 启动服务

```sh
systemctl start nginx-ui
```

- 查看服务状态

```sh
systemctl status nginx-ui
```

