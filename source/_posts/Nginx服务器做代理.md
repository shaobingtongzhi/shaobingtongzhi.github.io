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

# 配置

首先我们要了解nginx进行转发代理的核心在于两处，一是入口，二是出口；

- 入口就是url路径匹配识别对应的路径，

- 出口就是转发映射对应的后台服务地址

在nginx的配置文件中通过location配置路由转发规则，配置语法如下：

```conf
location [=||*|^~] /uri/ {
	# ...
}
```

中括号中为路由匹配符号，常见的有：

- =：精确匹配
- ^~：模糊前缀匹配
- ~：区分大小写的正则匹配
- ~*：不区分大小写的正则匹配
- /uri：普通前缀匹配
- /：通用匹配



## 精确匹配

例子：

```conf
location = /api {
	....
}
# 表示只有路径为 /api 的才会匹配到，/api/或者 /apixxx 都匹配不到
```

## 前缀模糊匹配

例子：

```conf
location ^~ /api/ {
	....
}
# 表示只要以路径 /api/ 开头的都会匹配到
```

## 无符号的精确匹配

例子：

```conf
没有符号，按照路径开头精确匹配，但是匹配到后不会立即返回，还会继续匹配其他普通匹配，如果匹配到，则会舍弃之前匹配的路径
比如 location /user/ , 当访问/user/开头时会匹配到
比如 location /user/admin，当访问/user/admin时会匹配到
```

## 区分大小写的正则匹配

进行uri的模糊匹配，区分大小写，匹配到后不再进行其他匹配

例子1：

```conf
# 当路径包含 /api/ 时会匹配，比如 /admin/user/ 或者 /user/admin/
location ~ /api/ {
	...
}
```

例子2：

```conf
# 能够匹配以/aoi开头，admin结尾的路径，.*表示的是任意字符
location ~ ^/user(.*)admin$ {
   ...
}
```

## 不区分大小写的正则匹配

进行uri的模糊匹配，不区分大小写，匹配到后不再进行其他匹配

# 反向代理配置

反向代理是指代理服务器接收互联网上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给互联网上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器

- proxy_pass是Nginx中的一个重要指令，它属于Nginx的HTTP代理模块。
- proxy_pass 的主要作用是设置代理服务器的地址，实现 **反向代理** 功能

示例：

```conf
location / {
    proxy_pass http://localhost:8080;
}
```

这个配置表示，当用户访问例如 `http://your-nginx-server/anything` 的时候，Nginx 会在后台将请求转发到 `http://localhost:8080/anything`，并将从那里得到的响应返回给用户

## 路径转发

### location匹配路径末尾没有 /

此时proxy_pass后面的路径必须拼接location的路径

```conf
location /test{   
	proxy_redirect off;   
	proxy_set_header        Host $host;   
	proxy_set_header        X-Real-IP $remote_addr;   
	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;   	
	proxy_pass http://192.168.1.31/test;
}

```

外面访问：http://192.168.1.30/test/test1.html

相当于访问：http://192.168.1.31/test/test1.html

### location匹配路径末尾有 /

此时proxy_pass后面的路径需要分为以下四种情况讨论

- 情况1：proxy_pass后面的路径只有域名且最后没有 /

```conf
location /test/{   
	proxy_redirect off;   
	proxy_set_header        Host $host;   
	proxy_set_header        X-Real-IP $remote_addr;   
	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;   	
	proxy_pass http://192.168.1.31;
}
```

外面访问：http://192.168.1.30/test/test1.html

相当于访问：http://192.168.1.31/test/test1.html



- 情况2：proxy_pass后面的路径只有域名同时最后有 /

```conf
location /test/{   
	proxy_redirect off;   
	proxy_set_header        Host $host;   
	proxy_set_header        X-Real-IP $remote_addr;   
	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;   	
	proxy_pass http://192.168.1.31/;
}
```

外面访问：http://192.168.1.30/test/test1.html

相当于访问：http://192.168.1.31/test1.html



- 情况3：proxy_pass后面的路径还有其他路径但是最后没有 /

```conf
location /test/{   
	proxy_redirect off;   
	proxy_set_header        Host $host;   
	proxy_set_header        X-Real-IP $remote_addr;   
	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;   	
	proxy_pass http://192.168.1.31/abc;
}
```

外面访问：http://192.168.1.30/test/test1.html

相当于访问：http://192.168.1.31/abctest1.html



- 情况4：proxy_pass后面的路径还有其他路径同时最后有 /

```conf
location /test/{   
	proxy_redirect off;   
	proxy_set_header        Host $host;   
	proxy_set_header        X-Real-IP $remote_addr;   
	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;   	
	proxy_pass http://192.168.1.31/abc/;
}
```

外面访问：http://192.168.1.30/test/test1.html

相当于访问：http://192.168.1.31/abc/test1.html
