---
title: Docker部署Emqx服务
date: 2025-09-09
categories:
  - 实战
  - Emqx服务器
tags:
  - mqtt
  - emqx
---

# 拉取镜像

```sh
docker pull emqx/emqx:5.8.8
```
# 快速运行

```sh
docker run -d --name emqx \
  -p 1883:1883 \    # MQTT 协议端口
  -p 8883:8883 \    # MQTT SSL 端口
  -p 8083:8083 \    # WebSocket 协议端口
  -p 8084:8084 \    # WebSocket SSL 端口
  -p 18083:18083 \  # Dashboard 管理控制台
  emqx/emqx:5.8.8
```

# 访问管理控制台

在浏览器打开：

```
http://localhost:18083
```

默认账号密码：

```
用户名：admin
密码：public
```

# 配置和数据持久化

## 启动时不变的配置

启动时不变的配置放在 etc/emqx.conf



## 运行时更新的配置

直接在控制台进行修改



## 持久化

etc/emqx.conf 在容器里是只读的，无法修改，要映射到宿主机

控制台修改的配置存在 data/configs/ 

### 配置持久化步骤

1. 复制 emqx.conf 到宿主机目录

```sh
docker cp emqx:/opt/emqx/etc/emqx.conf .
```

2. 重启一个容器

```sh
# 暂停并删除旧容器
doker stop emqx
docke rm emqx
# 启动新容器
docker run -d --name emqx -p 1883:1883 -p 8883:8883 -p 8083:8083 -p 8084:8084 -p 18083:18083 -v E:/docker/emqx/data:/opt/emqx/data -v E:/docker/emqx/conf/emqx.conf:/opt/emqx/etc/emqx.conf -v E:/docker/emqx/log:/opt/emqx/log emqx/emqx:5.8.8
```

3. 使用默认cookie有安全风险，修改

     ![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20250909/Snipaste_2025-09-09_14-15-33.6pcoen10py80.webp)

4. 管理面板修改报文大小

  ![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20250909/Snipaste_2025-09-09_14-26-09.2lowgovv6ru0.webp)

5. 检验是否生效
```sh
# 暂停并删除容器
docker stop emqx
docker rm emqx

# 重启新容器
docker run -d --name emqx -p 1883:1883 -p 8883:8883 -p 8083:8083 -p 8084:8084 -p 18083:18083 -v E:/docker/emqx/data:/opt/emqx/data -v E:/docker/emqx/conf/emqx.conf:/opt/emqx/etc/emqx.conf -v E:/docker/emqx/log:/opt/emqx/log emqx/emqx:5.8.8

# docker logs emqx 查看没有了告警提示
```

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20250909/Snipaste_2025-09-09_14-31-33.778mrs9f3cs0.webp)

### 数据持久化步骤

除了需要挂载/opt/emqx/data目录外还需要固定hostname，因为 emqx 在运行时，数据存储的目录是 /opt/emqx/data/mnesia/节点名

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20250909/Snipaste_2025-09-09_14-36-23.1uogyr7z6ug0.webp)

```sh
# 指定hostname
docker run --hostname emqx.dev -e EMQX_HOST=emqx.dev -d --name emqx -p 1883:1883 -p 8883:8883 -p 8083:8083 -p 8084:8084 -p 18083:18083 -v E:/docker/emqx/data:/opt/emqx/data -v E:/docker/emqx/conf/emqx.conf:/opt/emqx/etc/emqx.conf -v E:/docker/emqx/log:/opt/emqx/log emqx/emqx:5.8.8

# 保持节点名不变是数据持久化不丢失的关键
```

# 扩展

## 进入容器内

```sh
#通过emqx用户进入容器内
docker exec -u emqx -it emqx bash
#可参考官方文档使用命令行进行操作
emqx ctl help
```

# 场景

A 通过主题发布了一条消息，B订阅了主题，但是过程中，emqx服务挂了，相当于消息已经到了emqx了，只是B还没收到，需要开启会话持久化

```conf
node {
  name = "emqx@127.0.0.1"
  cookie = "emqxsecretadfjkclkecookie"
  data_dir = "data"
}

cluster {
  name = emqxcl
  discovery_strategy = singleton
}

dashboard {
    listeners {
        http.bind = 18083
        # https.bind = 18084
        https {
            ssl_options {
                certfile = "${EMQX_ETC_DIR}/certs/cert.pem"
                keyfile = "${EMQX_ETC_DIR}/certs/key.pem"
            }
        }
    }
}
durable_sessions {
  enable = true
}
```

两个地方需要注意：

1. discovery_strategy = singleton
2. 开启会话持久化
```conf
durable_sessions {
	enable = true
}
```
参考文档：

https://docs.emqx.com/zh/emqx/v5.8/admin/cli.html

https://docker.emqx.dev 可生成docker-compose文件
