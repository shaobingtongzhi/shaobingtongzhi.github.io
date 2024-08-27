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
server_addr = 39.99.149.236
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

