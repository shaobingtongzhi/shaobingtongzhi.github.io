---
title: Web 服务器部署
date: 2019-06-01
tags:
	- apache
	- php
	- memcache
	- 服务器部署
categories:
    - 实战
    - Web服务器部署
---


# 安装 Apache

```sh
cd /usr/local/src
wget http://archive.apache.org/dist/apr/apr-1.6.5.tar.gz
tar -zxvf apr-1.6.5.tar.gz


cd apr-1.6.5
./configure --prefix=/usr/local/webserver/apr
make && make install

cd /usr/local/src

wget http://archive.apache.org/dist/apr/apr-util-1.6.1.tar.gz

tar -zxvf apr-util-1.6.1.tar.gz

cd apr-util-1.6.1
./configure --prefix=/usr/local/webserver/apr-util --with-apr=/usr/local/webserver/apr/bin/apr-1-config
make && make install

cd /usr/local/src

wget https://netactuate.dl.sourceforge.net/project/pcre/pcre/8.42/pcre-8.42.tar.gz

tar -zxvf pcre-8.42.tar.gz
cd pcre-8.42

./configure
make && make install


cd /usr/local/src
wget https://archive.apache.org/dist/httpd/httpd-2.4.37.tar.gz

tar -zxvf httpd-2.4.37.tar.gz
cd httpd-2.4.37

./configure --prefix=/usr/local/webserver/apache --with-apr=/usr/local/webserver/apr --with-apr-util=/usr/local/webserver/apr-util --with-mpm=prefork
make clean
make && make install
echo 'export PATH=$PATH:/usr/local/webserver/apache/bin' >> /etc/profile

cp -Rf /usr/local/webserver/apache/bin/apachectl /etc/rc.d/init.d/httpd 

mkdir /web
chown daemon.daemon -R /web #更改目录所有者

chmod 755 /web -R #更改apache网站目录权限

chkconfig httpd on #设置开机启动

# 如果报错service httpd does not support chkconfig
service httpd does not support chkconfig
# 解决
# 打开 vi /etc/rc.d/init.d/httpd 添加(#!/bin/sh下面)

#chkconfig: 2345 10 90
#description: Activates/Deactivates Apache Web Server
```

# 启动apache
```sh
service httpd start
```

启动apache遇到错误:httpd: Could not reliably determine the server's fully qualified domain name 

1)进入apache的安装目录:(视个人安装情况而不同)
```sh
cd /usr/local/webserver/apache/conf
```

2)编辑httpd.conf文件，搜索"#ServerName"，添加ServerName localhost:80 
```sh
vim httpd.conf
# 添加:ServerName localhost:80
```

3)再重新启动apache 即可。
```sh
service httpd restart
```

# 配置防火墙
```sh
systemctl start firewalld
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
```

# 安装PHP
```sh
yum -y install libmcrypt-devel mhash-devel libxslt-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel

yum install libzip
yum groupinstall "Development Tools"

cd /usr/local/src
wget https://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.15.tar.gz 
tar -zxvf libiconv-1.15.tar.gz
cd libiconv-1.15
./configure --prefix=/usr/local/libiconv
make
make install

ln -s /usr/local/lib/libiconv.so.2.6.0 /usr/local/lib64/libiconv.so 
ln -s /usr/local/lib/libiconv.so.2.6.0 /usr/local/lib64/libiconv.so.2

cd /usr/local/src/
wget http://cn.php.net/distributions/php-5.6.36.tar.gz
wget https://www.php.net/distributions/php-5.6.36.tar.gz
tar -zxvf php-5.6.36.tar.gz
cd php-5.6.36
./configure --prefix=/usr/local/webserver/php --with-config-file-path=/usr/local/webserver/php/etc --with-apxs2=/usr/local/webserver/apache/bin/apxs --enable-soap --with-openssl --with-mcrypt --with-pcre-regex --with-zlib --enable-bcmath --with-bz2 --enable-calendar --with-curl --with-cdb --enable-fileinfo --with-gd --with-openssl-dir --with-jpeg-dir --with-png-dir --with-freetype-dir --with-gettext --with-gmp --with-mhash --enable-json --enable-mbstring --disable-mbregex --enable-pdo --with-pdo-mysql --with-zlib-dir --enable-session --enable-shmop --enable-sockets --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-wddx --with-libxml-dir --with-xsl --enable-zip --enable-mysqlnd-compression-support --with-pear --with-mysqli --with-iconv=/usr/local/libiconv --with-mysql

make && make install
# 报错
# configure: error: mcrypt.h not found. Please reinstall libmcrypt
# 处理

# wget ftp://mcrypt.hellug.gr/pub/crypto/mcrypt/attic/libmcrypt/libmcrypt-2.5.7.tar.gz 
# tar -zxvf libmcrypt-2.5.7.tar.gz
# cd libmcrypt-2.5.7
# ./configure --prefix=/usr/local
# make && make install

#报错
# Don't know how to define struct flock on this system, set --enable-opcache=no
# vim /etc/ld.so.conf.d/local.conf # 编辑库文件
# /usr/local/lib # 添加该行
# :wq # 保存退出
# ldconfig # 使之生效


# 把附件中的 php.ini 拷贝到 /etc/php.ini
# 并软连接到 /usr/local/webserver/php/etc/php.ini
ln -s /etc/php.ini /usr/local/webserver/php/etc/php.ini

```


# 配置apache+php

```sh
cd /usr/local/webserver/apache/conf

# 编辑httpd.conf
vi httpd.conf

# 1. 配置网站根目录
DocumentRoot "/web"
<Directory "/web">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>

# 2. 配置支持php
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
<IfModule mime_module>
    TypesConfig conf/mime.types
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
    AddType application/x-httpd-php .php .html
</IfModule>
```

# 安装Memcache

```sh
cd /usr/local/src
wget https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz

tar -zxvf libevent-2.1.8-stable.tar.gz

cd libevent-2.1.8-stable 
./configure --prefix=/usr 
make clean
make && make install


wget http://memcached.org/files/memcached-1.5.12.tar.gz 

tar zxvf memcached-1.5.12.tar.gz

cd memcached-1.5.12
./configure --with-libevent=/usr
make clean
make && make install

# 安装php扩展
cd /usr/local/src/
wget http://pecl.php.net/get/memcache-3.0.8.tgz

tar -zxvf memcache-3.0.8.tgz

cd memcache-3.0.8

/usr/local/webserver/php/bin/phpize -- clean
/usr/local/webserver/php/bin/phpize
./configure --enable-memcache --with-php-config=/usr/local/webserver/php/bin/php-config --with-zlib-dir 

make clean
make && make install
# vi /etc/php.ini 添加扩展
# 在php.ini文件末尾添加如下代码
# extension=memcache.so

```

# 升级apache到2.4.57
原apache版本：2.4.37
新apache版本：2.4.57

## 下载
```sh
cd /usr/local/src

wget https://archive.apache.org/dist/httpd/httpd-2.4.57.tar.gz
```

## 备份

```sh
service httpd stop

cd /usr/local/webserver

mv apache apache_bak
```

## 安装

```sh
cd /usr/local/src

tar -zxvf httpd-2.4.57.tar.gz

cd httpd-2.4.57

./configure --prefix=/usr/local/webserver/apache --with-apr=/usr/local/webserver/apr --with-apr-util=/usr/local/webserver/apr-util --with-mpm=prefork
make clean
make && make install

cp /usr/local/webserver/apache_bak/modules/libphp5.so /usr/local/webserver/apache/modules
# 恢复配置
cp /usr/local/webserver/apache_bak/conf/httpd.conf /usr/local/webserver/apache/conf/
```
## 查看版本
```sh
httpd -v
```
## 启动apache
```sh
service httpd start
```
