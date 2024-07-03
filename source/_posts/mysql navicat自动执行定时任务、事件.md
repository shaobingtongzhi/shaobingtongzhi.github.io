---
title: mysql navicat 自动执行定时任务/事件
date: 2024-04-30
categories:
  - 实战
  - Mysql服务器
tags: 
  - mysql事件
  - 定时任务
  - 自动执行
---

# 查看mysql是否开启了定时任务

```sql
show variables like 'event_scheduler';
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240430/Snipaste_2024-04-30_16-16-31.dcf7w6rwpio.webp)

查看event_scheduler如果为OFF或0就表示关闭 

```sql
#开启命令
set global event_scheduler = on;
```

这个时候，如果重启了mysql服务，发现 event_scheduler 又成 off 了，所以需要在mysql的配置文件my.ini中进行配置

只需要在my.ini配置文件的**[mysqld]**部分加上**event_scheduler=ON** 即可

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240430/Snipaste_2024-04-30_16-20-16.6wg6wcq9zvo0.webp)



参考链接：https://blog.csdn.net/zhangjunli/article/details/129928308