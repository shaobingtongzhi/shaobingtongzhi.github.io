---
title: Redis 集群部署
date: 2022-08-01
tags: 
	- redis
	- redis集群
	- 哨兵
categories:
	- 学习笔记 
	- 04-Redis相关笔记 
--- 

# 拉取镜像

```sh
docker pull redis
```
# 修改别名

```sh
docker tag [镜像ID] redis62
```
# 一主两从配置
## 步骤
### 创建主redis配置文件redis.conf

1) vi /Users/xxxx/Document/docker/redis/redis1/redis.conf

2) 配置内容如下：

```sh
port 6379
logfile "redis.log"
dir /data
appendonly yes
bind 0.0.0.0
appendfilename appendonly.aof
```

>配置说明：
logfile：日志文件在工作目录下
dir：工作目录
appendonly：是否需要持久化,yes为需要
slaveof：指明为主机一的从机
requirepass：redis客户端连接的认证密码,若不需要可不配置
masterauth：主从redis同步的认证密码,与连接密码同,若不需要可不用配置

启动服务

```sh

docker run -it -d -p 6371:6379 --name redis_master --privileged=true -v /Users/xxxx/Document/docker/redis/redis1:/data redis62 redis-server /data/redis.conf --appendonly yes

```

### 创建第一个从redis配置文件redis.conf

1) vi /Users/xxxx/Document/docker/redis/redis2/redis.conf

2) 配置文件内容如下：

```sh
port 6379
logfile "redis.log"
dir /data
appendonly yes
bind 0.0.0.0
appendfilename appendonly.aof
slaveof 172.17.0.5 6379
# 上一行 slaveof或者replicationof ip地址为宿主机ip地址(或者容器ip)
```

> 配置说明：
logfile：日志文件在工作目录下
dir：工作目录
appendonly：是否需要持久化,yes为需要
slaveof：指明为主机一的从机
requirepass：redis客户端连接的认证密码,若不需要可不配置
masterauth：主从redis同步的认证密码,与连接密码同,若不需要可不用配置

3. 启动服务

```sh
docker run -it -d -p 6372:6379 --name redis_slave --privileged=true -v /Users/xxxx/Document/docker/redis/redis2:/data redis62 redis-server /data/redis.conf --appendonly yes
```

### 创建第二个从redis配置文件redis.conf

> 说明：类似与第一个从redis

1) vi /Users/xxxx/Document/docker/redis/redis3/redis.conf

2) 配置文件内容如下：

```sh
port 6379
logfile "redis.log"
dir /data
appendonly yes
bind 0.0.0.0
appendfilename appendonly.aof
slaveof 172.17.0.5 6379
# 上一行 slaveof或者replicationof ip地址为宿主机ip地址(或者容器ip)
```

> 配置说明：
logfile：日志文件在工作目录下
dir：工作目录
appendonly：是否需要持久化,yes为需要
slaveof：指明为主机一的从机
requirepass：redis客户端连接的认证密码,若不需要可不配置
masterauth：主从redis同步的认证密码,与连接密码同,若不需要可不用配置

3. 启动服务
```sh
docker run -it -d -p 6373:6379 --name redis_slave2 --privileged=true -v /Users/xxxx/Document/docker/redis/redis3:/data redis62 redis-server /data/redis.conf --appendonly yes
```

### 查看状态

1. 查看主机状态

```sh
docker exec -it redis_master bash
redis-cli
info
```

> 查看相关信息
> #Replication
role:master
connected_slaves:1 (如果从机配置好了则为1,否则0)
...

2. 查看从机状态

```sh
docker exec -it redis_slave bash
redis-cli
info
```

>#Replication
role:slave
master_host:172.17.0.5
...

### 验证主从关系

master 写入

```sh
docker exec -it redis_master bash
redis-cli
set name lili
```

slave 读取

```sh
docker exec -it redis_slave bash
redis-cli
get name
```

> 结果：lili

# 三个哨兵配置

## 步骤

### 创建redis哨兵1配置文件sentinel.conf

1) vi /Users/xxxx/Document/docker/redis/sentinel1/sentinel.conf
2) 配置文件内容如下：

```sh
port 26379
daemonize no
protected-mode no
logfile "sentinel.log"
sentinel monitor mymaster 172.17.0.5 6379 1
sentinel down-after-milliseconds mymaster 6000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
```

> 配置说明:
daemonize yes 开启守护进程,以后台方式运行
protected-mode no 是否开启保护模式
logfile "sentinel.log"：输出日志目录
sentinel monitor mymaster 192.168.0.1 6379 1：哨兵监控的主redis名称、ip、端口
sentinel auth-pass mymaster 1234 ： 哨兵的认证密码(如果没有则不需要)
3) 启动哨兵服务

```sh
docker run -d -p 26371:26379 --name redis_sentinel1 -v /Users/xxxx/Document/docker/redis/sentinel1:/data redis62 redis-sentinel /data/sentinel.conf
```

### 创建redis哨兵2配置文件sentinel2.conf

> 类似哨兵1配置

1) vi /Users/xxxx/Document/docker/redis/sentinel2/sentinel.conf
2) 配置文件内容如下：

```sh
port 26379
daemonize no
protected-mode no
logfile "sentinel.log"
sentinel monitor mymaster 172.17.0.5 6379 1
sentinel down-after-milliseconds mymaster 6000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
```

> 配置说明:
daemonize yes 开启守护进程,以后台方式运行
protected-mode no 是否开启保护模式
logfile "sentinel.log"：输出日志目录
sentinel monitor mymaster 192.168.0.1 6379 1：哨兵监控的主redis名称、ip、端口
sentinel auth-pass mymaster 1234 ： 哨兵的认证密码(如果没有则不需要)

3) 启动哨兵服务

```sh
docker run -d -p 26372:26379 --name redis_sentinel2 -v /Users/xxxx/Document/docker/redis/sentinel2:/data redis62 redis-sentinel /data/sentinel.conf
```

### 创建redis哨兵3配置文件sentinel3.conf

> 类似哨兵1配置

1) vi /Users/xxxx/Document/docker/redis/sentinel3/sentinel3.conf

2) 配置文件内容如下：

```sh
port 26379
daemonize no
protected-mode no
logfile "sentinel.log"
sentinel monitor mymaster 172.17.0.5 6379 1
sentinel down-after-milliseconds mymaster 6000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
```

> 配置说明:
daemonize yes 开启守护进程,以后台方式运行
protected-mode no 是否开启保护模式
logfile "sentinel.log"：输出日志目录
sentinel monitor mymaster 192.168.0.1 6379 1：哨兵监控的主redis名称、ip、端口
sentinel auth-pass mymaster 1234 ： 哨兵的认证密码(如果没有则不需要)

3) 启动哨兵服务

```sh
docker run -d -p 26373:26379 --name redis_sentinel3 -v /Users/xxxx/Document/docker/redis/sentinel3:/data redis62 redis-sentinel /data/sentinel.conf
```

### 查看哨兵情况

```sh
docker exec -it redis_sentinel1 bash
redis-cli -h 127.0.0.1 -p 26379
info
```

> #Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.0.1:6379,slaves=2,sentinels=4

### 验证哨兵模式

> 关闭redis_master主服务,查看redis_slave从服务是否会自动切换为主服务

```sh
docker stop redis_master
docker exec -it redis_slave bash
redis-cli
info
```

> #Replication
role:master
connected_slaves:1 (如果从机配置好了则为1,否则0)
...

至此完成所有配置！！！