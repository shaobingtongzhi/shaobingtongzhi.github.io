---
title: Apache 服务器中.htaccess文件的使用
date: 2018-09-06
tags: 
  - Apache
  - .htaccess
categories:
  - 实战
  - Apache服务器
---

# .htaccess是什么

.htaccess文件(或者分布式配置文件）,全称是Hypertext Access(超文本入口)。 提供了针对目录改变配置的方法， 即在一个特定的文档目录中放置一个包含一个或多个指令的文件， 以作用于此目录及其所有子目录。 



# 使用

1. apache服务器中开启配置

httpd.conf
```conf
# 取消#号
LoadModule rewrite_module modules/mod_rewrite.so

<Directory "/web">
    Options Indexes FollowSymLinks
    AllowOverride All    # 将none改为All
    Require all granted
</Directory>

```

2. 编写.htaccess

```
RewriteEngine on
RewriteCond %{REQUEST_METHOD} ^(TRACE|TRACK)
RewriteRule .* - [F]
 

#RewriteBase    
RewriteCond %{REQUEST_METHOD} !POST

### openapi ###
RewriteRule ^/?openapi+([\S\s]*)$                         index.php?app=index&mod=Openapi&act=index&%{QUERY_STRING} [L]

### shortUrl ###
RewriteRule ^/?url\/+([\S]*)$                      index.php?app=public&mod=Index&act=shorturl&url=$1 [L]


### topic ###
RewriteRule ^/?t\/+([\S\s]*)$                         index.php?app=public&mod=Index&act=topic&t=$1&%{QUERY_STRING} [L]

### avatr & img ###
RewriteRule ^/?avatar-([0-9a-zA-Z_]+)-([0-9]+)-([0-9]+)\.([jpg|gif]+)$                index.php?app=public&mod=Index&act=avatar&uid=$1&type=$2&t=$3&ext=$4 [L]

RewriteRule ^/?was-([0-9]+)-([0-9a-zA-Z_]+)-([0-9a-zA-Z_]+)\.([jpg|gif]+)$     index.php?app=widget&mod=Attach&act=showImage&attach_id=$1&hash=$2&type=$3 [L]
RewriteRule ^/?wav-([0-9]+)-([0-9]+)\.([pdf|xml|txt]+)$     index.php?app=widget&mod=ShowDocument&act=view&attach_id=$1&file_id=$2&type=$3 [L]
RewriteRule ^/?fimg-([0-9]+)-([0-9]+)-([0-9a-zA-Z_]+)-([0-9a-zA-Z_]+)\.([jpg|gif]+)$     index.php?app=file&mod=Do&act=showImage&attach_id=$1&file_id=$2&hash=$3&type=$4 [L]
RewriteRule ^/?fvideo-([0-9]+)-([0-9]+)-([0-9a-zA-Z_]+)\.([flv|mp4|mp3|mov]+)$     index.php?app=file&mod=Do&act=showVieo&attach_id=$1&file_id=$2&hash=$3 [L]

### office web apps ###
RewriteRule ^/?wopi\/files\/([0-9a-zA-Z_=\.\-\s+]+)$   index.php?app=widget&mod=Attach&act=getFileInfo&code=$1&%{QUERY_STRING} [L]
RewriteRule ^/?wopi\/files\/([0-9a-zA-Z_=\.\-\s+]+)\/contents$  index.php?app=widget&mod=Attach&act=getFileContent&code=$1&%{QUERY_STRING} [L]


############## For Sapce Domain| must just before the Channel  #############
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^/?user\/+([0-9a-zA-Z_]+)$             index.php?app=public&mod=Profile&act=shorturl&domain=$1 [L]

############### For Channel | must be end end of doc ###############
### Subject ###
RewriteRule ^/?subjects$                         index.php?app=portal&mod=Show&act=subject&%{QUERY_STRING} [L]

RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^/?([0-9a-zA-Z_]+)\/([0-9a-zA-Z_]+)\.html$      index.php?app=portal&mod=Show&channel=$1&page=$2&%{QUERY_STRING}        [L]

RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^/?([0-9a-zA-Z_]+)(\/)?$                        index.php?app=portal&mod=Show&channel=$1&page=index&%{QUERY_STRING}     [L]
```