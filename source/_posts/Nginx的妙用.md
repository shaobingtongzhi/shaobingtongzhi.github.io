---
title: Nginx的妙用
date: 2024-11-13
categories: 
  - 实战
  - Nginx服务器
tags:
  - Nginx
  - 代理服务器
  - 压缩
  - gzip
---

# 文件压缩

如果我们租用了一个带宽很低的服务器，网站访问速度会很慢，这时我们可以通过让nginx开启GZIP压缩来提高网站的访问速度。

首先对nginx进行限速操作，限制每个连接的访问速度为128K来建立一个比较慢的访问场景

修改配置文件，进行限速操作

```conf
server {
    listen       80;
    server_name  xxxx.xxx.xxx;
    
    limit_rate 128k; #限制网速为128K

    location / {
        root   /usr/share/nginx/html/mall;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```



nginx返回请求头信息如下：

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241113/Snipaste_2024-11-13_15-49-53.3jwaivuktg80.webp)

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241113/Snipaste_2024-11-13_15-49-26.27pgdr8vt64g.webp)



修改`nginx.conf`配置文件，开启GZIP压缩

```conf
http {
    gzip on; #开启gzip
    gzip_disable "msie6"; #IE6不使用gzip
    gzip_vary on; #设置为on会在Header里增加 "Vary: Accept-Encoding"
    gzip_proxied any; #代理结果数据的压缩
    gzip_comp_level 6; #gzip压缩比（1~9），越小压缩效果越差，但是越大处理越慢，所以一般取中间值
    gzip_buffers 16 8k; #获取多少内存用于缓存压缩结果
    gzip_http_version 1.1; #识别http协议的版本
    gzip_min_length 1k; #设置允许压缩的页面最小字节数，超过1k的文件会被压缩
    gzip_types application/javascript text/css; #对特定的MIME类型生效,js和css文件会被压缩

    include /etc/nginx/conf.d/*.conf;
}
```

再次进行访问，我们可以发现js文件已经被压缩，加载时间缩短到29.26s，且文件大小也确实被压缩了

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241113/Snipaste_2024-11-13_15-53-31.6da22n251000.webp)



nginx返回请求头中添加了`Content-Encoding: gzip`的信息

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241113/Snipaste_2024-11-13_15-53-05.644083n5s3s0.webp)

