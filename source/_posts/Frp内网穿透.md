---
title: Frp 内网穿透
date: 2024-08-26
categories:
  - 实战
  - 内网穿透
tags:
  - Frp
  - 内网穿透
---

# 简介

官网：https://gofrp.org/zh-cn/

FRP(Fast Reverse Proxy)是一个专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。

基本原理：

- 在带有公网 ip 的云服务器上部署 frp 的服务端 frps
- 在需要穿透的内网服务器上部署 frp 的客户端 frpc
- 每个客户端都会有一个配置文件用于和服务器连接
- 公网服务器充当代理服务器，用户访问 公网ip + 端口时，公网服务器的 frps 服务会根据端口号，自动转发到对应的内网服务器上，从而访问到内网服务
  

 # 下载服务端docker镜像

```sh
# 服务器端
docker pull snowdreamtech/frps

# 准备配置文件 frps.toml

# 启动一个容器
docker run --restart=always --network host -d -v /docker/frp/conf/frps.toml:/etc/frp/frps.toml -v /docker/frp/html:/frp/html/ --name frps snowdreamtech/frps:latest
```

# 下载客户端docker镜像

```sh
docker pull snowdreamtechamd64/frpc:latest
# 准备配置文件 frpc.toml

# 运行一个容器
docker run --restart=always --network host -d -v E:/docker/frp/frpc.toml :/etc/frp/frpc.toml --name frpc snowdreamtechamd64/frpc:latest
```

# 通过域名访问内网服务

通过域名访问内网服务，大体有以下两个方案：

- 直接通过 配置 frps.toml 实现（简单推荐）
- 通过 nginx 做转发

## 方案一

服务端配置文件 ***frps.toml***

```toml
[common]
bind_port = 7000
# 自定义404页面
custom_404_page = /frp/html/404.html
# 启用面板
dashboard_port = 7500
# 面板登录名和密码
dashboard_user = admin
dashboard_pwd = xxxxx
# 使用http代理并使用80端口进行穿透
vhost_http_port = 80

# 认证超时时间
authentication_timeout = 900
# 认证token，客户端需要和此对应
token=shaobingtongzhi

# 定义一个名称为 jenkins 的隧道，隧道类型 http 
# 该隧道将本地的 12040 端口映射到远程
[jenkins]
type = http
local_ip = 127.0.0.1
local_port = 10240
# 配置域名，通过下面的域名可以访问 本地的服务,注意：已设置该域名的A记录指向了当前服务器ip
custom_domains = xxx.your_domain.com
```

客户端配置文件 ***frpc.toml***

```toml
[common]
server_addr = xxx.xx.xx.xx # 公网服务器IP
server_port = 7000
token = shaobingtongzhi

[jenkins]
type = http
# local_ip = 127.0.0.1
local_port = 10240
custom_domains = xxx.your_domain.com
```

## 方案二

1. 由于加入了nginx代理，所以这里列出了nginx的配置

```conf
server {
    listen      80;
    server_name xxx.your_domain.com;
    location / {
        proxy_pass http://127.0.0.1:10240/; 
        # 由于使用了docker容器,故127.0.0.1 需要改成 容器内可以访问的宿主机ip 
        # 查询命令：ip -4 addr show docker0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
    }
}
```

2. 服务端配置文件 ***frps.toml***

```toml
[common]
bind_addr = 0.0.0.0
bind_port = 7000
custom_404_page = /frp/html/404.html
# 启用面板
dashboard_port = 7500
# 面板登录名和密码
dashboard_user = admin
dashboard_pwd = xxxxx
# 认证token，客户端需要和此对应
token=shaobingtongzhi

[jenkins]
type = tcp
local_ip = 127.0.0.1
local_port = 10240
remote_port = 10240
```

3. 客户端配置文件 ***frpc.toml***

```toml
[common]
server_addr = xxx.xxx.xxx.xxx #公网服务器IP
server_port = 7000
token = shaobingtongzhi

[jenkins]
type = tcp
local_ip = 127.0.0.1
local_port = 10240
remote_port = 10240
```



# 自定义404页面

404.html

```html
<!DOCTYPE html>
<html>
<head>
<title>Not Found</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>The page you requested was not found.</h1>
<p>Sorry, the page you are looking for is currently unavailable.<br/>
Please try again later.</p>
</body>
</html>
```

# 实战配置

## 服务端

frps.toml

```toml
bindPort = 7010
webServer.addr = "127.0.0.1"
webServer.port = 7500
webServer.user = "admin"
webServer.password = "admin12345+"
auth.token = "authToken"
```

启动命令

```sh
frps.exe -c frps.toml
```





## 客户端

frpc.toml

```toml
serverAddr = "xx.xx.xx.xx"  # 服务端所在的服务器IP
serverPort = 7010
auth.token = "authToken" #与服务端一致
webServer.addr = "127.0.0.1"
webServer.port = 7500
webServer.user = "admin"
webServer.password = "admin12345+"
[[proxies]]
name = "xxxx" # 名称随意
type = "tcp"
localIP = "127.0.0.1"
localPort = 80 # 客户端开放的端口
remotePort = 30001 # 映射到服务器的端口, 如果不通过端口直接访问的话，服务端可不开启这个端口，因为客户端和服务端是通过7010通信的，所以说开了7010就可以了

[[proxies]]
name = "xxxx1"
type = "tcp"
localIP = "127.0.0.1"
localPort = 81
remotePort = 30002

[[proxies]]
name = "xxxx2"
type = "tcp"
localIP = "127.0.0.1"
localPort = 82
remotePort = 30003
transport.bandwidthLimit = "30MB"

[[proxies]]
name = "xxxx3"
type = "tcp"
localIP = "127.0.0.1"
localPort = 8080
remotePort = 2321
```

启动命令

```sh
frpc.exe -c frpc.toml
```

# 与Nginx的结合使用

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250512/网络结构.3lpvawqy05e0.webp)

## 内网服务器

### nginx

项目E的配置

```conf
server {
    listen        8083;
    server_name  localhost;
    root         "C:/Project/qinghuaci/frontend/dist";
    location / {
        index index.php index.html error/index.html;
        error_page 400 /error/400.html;
        error_page 403 /error/403.html;
        error_page 404 /error/404.html;
        error_page 500 /error/500.html;
        error_page 501 /error/501.html;
        error_page 502 /error/502.html;
        error_page 503 /error/503.html;
        error_page 504 /error/504.html;
        error_page 505 /error/505.html;
        error_page 506 /error/506.html;
        error_page 507 /error/507.html;
        error_page 509 /error/509.html;
        error_page 510 /error/510.html;
        include C:/Project/headQ/nginx.htaccess;
        autoindex  off;
    }
    location /api/something {
        proxy_pass http://localhost:8082/;
        proxy_cache mycache;
        proxy_cache_valid 200 302 100s;
        add_header cache $upstream_cache_status;
    }
	# websocket转发
    location /device/alarm {
        proxy_pass http://127.0.0.1:8082;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```





### frpc

```toml
serverAddr = "xxx.xx.xx.xx" #外网服务器IP
serverPort = 7010
auth.token = "123123"
webServer.addr = "127.0.0.1"
webServer.port = 7500
webServer.user = "admin"
webServer.password = "admin12345+"
[[proxies]]
name = "项目A"
type = "tcp"
localIP = "127.0.0.1"
localPort = 80
remotePort = 30001

[[proxies]]
name = "项目B"
type = "tcp"
localIP = "127.0.0.1"
localPort = 81
remotePort = 30002

[[proxies]]
name = "项目C"
type = "tcp"
localIP = "127.0.0.1"
localPort = 82
remotePort = 30003
transport.bandwidthLimit = "30MB"

[[proxies]]
name = "项目D"
type = "tcp"
localIP = "127.0.0.1"
localPort = 8080
remotePort = 2321

[[proxies]]
name = "项目E"
type = "tcp"
localIP = "127.0.0.1"
localPort = 8083
remotePort = 2322
```

## 外网服务器

### nginx

项目E的配置

```conf
server {
        listen        80;
        server_name  域名;
        location / {
            proxy_pass http://127.0.0.1:2322; # 转发到 FRP 的 HTTP 端口
			proxy_set_header Host $host;      # 保持原始 Host 信息
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        # websocket请求
		location /device/alarm {
			proxy_pass http://127.0.0.1:2322;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "upgrade";
			proxy_set_header Host $host;
		}
}

```

### frps

没有变化

```toml
bindPort = 7010
webServer.addr = "127.0.0.1"
webServer.port = 7500
webServer.user = "admin"
webServer.password = "admin12345+"
auth.token = "123123"
```

