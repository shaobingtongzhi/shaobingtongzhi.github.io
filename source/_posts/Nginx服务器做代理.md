---
title: Nginx服务器做代理服务器
date: 2024-04-30
categories: 
  - 实战
  - Web服务器部署
tags:
  - nginx
  - 代理
---

# 问题

如何通过nginx代理实现，从另一台电脑访问本地部署的开发环境项目

# 解决

## 1. 下载nginx服务软件

https://nginx.org/en/download.html

## 2. 进行配置

找到目录 conf，里面有文件 nginx.conf。可以通过这个文件配置Nginx。

```
server {
	# 监听要开放服务的主机的8000端口（也就是部署了项目的的主机）
    listen  8000;
    # 运行项目的主机ip
    server_name  172.16.10.101;
    location / {
    	# 代理的真实服务地址，可以是ip、域名
    	# proxy_pass http://localhost:8080;
    	proxy_pass http://ietest.zhiboredian.net/;
    }
}
```

## 3. 启动服务

```sh
# 找到nginx安装路径下的sbin目录下执行
# 启动
./nginx
# 停止
./nginx -s quit
# 暴力停止
./nginx -s stop
# 加载配置文件
./nginx -s reload
# 强制关闭nginx
taskkill /f /t /im nginx.exe
```

