---
title: Windows 服务器安装 MQTT 服务
date: 2024-07-23
categories:
  - 实战
  - Windows服务器
tags:
  - windows server 2022
  - emqx
  - mqtt服务
---

# 简介

**EMQX** 是基于 Erlang/OTP 平台开发的开源物联网 **MQTT 消息服务器**，目前广泛应用于全球各行业物联网平台建设中。其设计目标是实现高可靠承载海量物联网终端的 MQTT 连接，支持在海量物联网设备间低延时消息路由

**MQTT** 是一种基于标准的消息传递协议或规则集，用于机器对机器的通信。智能传感器、可穿戴设备和其他物联网（IoT）设备通常必须通过带宽有限的资源受限网络传输和接收数据。这些物联网设备使用 MQTT 进行数据传输，因为它易于实施，并且可以有效地传输物联网数据。MQTT 支持设备到云端和云端到设备之间的消息传递

下图展示了 MQTT 发布/订阅过程。温度传感器作为客户端连接到 MQTT Broker，并通过发布操作将温度数据发布到一个特定主题（例如 `Temperature`）。MQTT Broker 接收到该消息后会负责将其转发给订阅了相应主题（`Temperature`）的订阅者客户端。

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240606/Snipaste_2024-07-23_09-18-54.ts3ckf40sb4.jpg)

# MQTT Broker 搭建

## 环境说明

- Windows Server 2022 Datacenter

- emqx-5.0.26-windows-amd64.zip

## 下载emqx

下载地址：https://www.emqx.com/zh/downloads/broker/5.0.26/emqx-5.0.26-windows-amd64.zip

文档地址：https://docs.emqx.com/zh/emqx/v5.0/deploy/install-windows.html

## 安装部署

### 1. 解压

解压 emqx-5.0.26-windows-amd64.zip 到服务器的指定路径，注意：<font color=red>指定路径不能有空格</font> ,我这里把它解压到了C：/emqx

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240723/Snipaste_2024-07-23_09-30-26.5ihnc2hdr280.jpg)

### 2. 启动

通过 cmd 进入 bin 目录下执行启动命令

```bat
cd bin
emqx start
```

```sh
C:\emqx\bin>emqx start
EMQX_LOG__CONSOLE_HANDLER__ENABLE [log.console.enable]: true
EMQX_NODE__DB_ROLE [node.role]: core
EMQX_NODE__DB_BACKEND [node.db_backend]: mnesia
```

通过 URL：http://localhost:18083/ 访问 EMQX Broker 对其进行配置。默认用户名：admin 密码：public

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240723/Snipaste_2024-07-23_09-45-23.6ta9offhdrk0.jpg)

### 3. 停止

```bat
emqx stop
```

## 常用命令

### 查看服务状态

```sh
emqx ctl status
# 正常情况
C:\emqx\bin>emqx ctl status
Node 'emqx@127.0.0.1' 5.0.26 is started

# 非正常情况
C:\emqx\bin>emqx ctl status
Node 'emqx@127.0.0.1' not responding to pings.
```

### 查看报错信息

```sh
emqx console
# 可以看到是什么原因导致的非正常情况
[error] tcp:default failed to listen on 1883 - eaddrinuse (address already in use)
# 我这里报错说是 1883 端口被占用了
# 解决办法：找到被占用的 1883 端口的进程，关闭即可，若该进程是其他有用的进程，则考虑 更换其他服务的端口或者更换emqx默认的1883端口
# 我通过修改emqx默认的1883端口为2883后启动正常了
```

### 端口占用

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240723/Snipaste_2024-07-23_10-03-28.4zoiekmfpw00.jpg)

端口配置文档：https://docs.emqx.com/zh/emqx/latest/configuration/listener.html#%E9%85%8D%E7%BD%AE-tcp-%E7%9B%91%E5%90%AC%E5%99%A8

### 查看端口监听

```sh
#查看监听器
emqx ctl listeners

#停止监听ws端口
emqx ctl listeners stop ws:default
emqx ctl listeners stop wss:default

#开启监听端口
emqx ctl listeners start ws:default
```



## 问题处理

### 修改默认 1883 端口

etc/emqx.conf

```sh
listeners.tcp.default {
  bind = "0.0.0.0:2883"
  max_connections = 1024000
}
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240723/Snipaste_2024-07-23_09-54-56.7kfbs0431k80.jpg)

### 修改 UI 18083 端口

etc/emqx.conf

```sh
dashboard {
    listeners.http {
        bind = 28083
    }
}
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240723/Snipaste_2024-07-23_09-59-01.5s16t37lnrk0.jpg)



# MQTT 客户端

MQTTX 全功能 MQTT 客户端工具

下载地址：https://mqttx.app/zh/downloads

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240723/Snipaste_2024-07-23_10-11-30.2i9q0xtnv520.jpg)

下载对应的版本即可

通过客户端可以测试连接到服务端！！！

至此完成了 MQTT 服务器的搭建！！！



# docker 安装EMQX

```sh
docker pull emqx:5.0.26

docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 -v E:/docker/emqx/data:/opt/emqx/data -v E:/docker/emqx/log:/opt/emqx/log emqx:5.0.26
```

