---
title: Arthas的使用
date: 2024-04-23
categories:
  - 学习笔记
  - 09-java相关笔记
---

背景：项目采用的 Springboot + docker部署 的方式，需要在线排查问题

Arthas 官网地址：https://arthas.aliyun.com/

# 简介

Arthas 是一款线上监控诊断产品，通过全局视角实时查看应用 load、内存、gc、线程的状态信息，并能在不修改应用代码的情况下，对业务问题进行诊断，包括查看方法调用的出入参、异常，监测方法执行耗时，类加载信息等，大大提升线上问题排查效率

# 使用

将下载的 arthas 包导入到 docker 容器内

## 启动

```sh
java -jar arthas-boot.jar
```

## 常用命令

### jad 

反编译指定已加载类的源码，可以查看代码是否更新正确了

```sh
jad net.zhiboredian.controller.Device
```

### thread

查看当前线程信息，查看线程的堆栈

```sh
# 指定最忙的前 3 个线程并打印堆栈
thread -3
# 查看指定线程pid的堆栈
thread 17
```

### trace

可以用来排查性能问题，由于命令难写，可以在IDEA中安装 arthas 插件

```sh
# 查询接口informationList中执行耗时超过1秒的方法
trace net.zhiboredian.controller.Device informationList  -n 5 --skipJDKMethod false '#cost>1000'
```

### watch

能方便的观察到指定函数的调用情况，比如说入参、返回值，相当于本地的debug

```sh
watch net.zhiboredian.controller.Device getDeviceByWord '{params,returnObj,throwExp}'  -n 5  -x 3 
```

### tt

方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

```sh
# 记录
tt -t net.zhiboredian.controller.Device getDeviceByWord -n 5 
# 回看
tt -p -i 1000

# tt 相关功能在使用完之后，需要手动释放内存，否则长时间可能导致OOM
tt --delete-all
```



