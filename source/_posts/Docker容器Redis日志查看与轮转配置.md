---
title: Docker容器Redis日志查看与轮转配置
date: 2025-06-18
categories:
  - 运维
  - Docker
tags:
  - Docker
  - Redis
  - 日志
  - 日志轮转
---

# 背景

在使用 Docker 部署 Redis 等服务时，容器产生的日志默认会以 `json-file` 日志驱动写入到宿主机的一个 JSON 日志文件中。随着服务长时间运行，该日志文件会持续增长，如果没有做任何限制，极有可能撑爆磁盘，进而导致 Redis 乃至宿主机上其他服务异常。

本文整理日常排查 Redis 容器日志、查看日志大小与内容，以及配置日志轮转（log rotation）的方法。

# 查看日志文件大小

先定位容器日志在宿主机上的实际路径，再查看其大小。

```sh
docker inspect --format='{{.LogPath}}' redis | xargs sudo du -sh
```

- `docker inspect --format='{{.LogPath}}' redis`：获取名为 `redis` 的容器日志文件在宿主机上的绝对路径，例如 `/var/lib/docker/containers/<id>/<id>-json.log`。
- `xargs sudo du -sh`：将该路径传给 `du` 命令，以人类可读的格式（如 `120M`、`3.2G`）输出大小。

> 提示：`redis` 是容器名或容器 ID，按实际情况替换。若当前用户没有读取 `/var/lib/docker` 的权限，需要配合 `sudo` 执行。

# 查看日志内容

拿到日志路径后，可直接用 `tail` 查看最近的日志。

```sh
sudo tail -n 100 $(docker inspect --format='{{.LogPath}}' redis)
```

该命令会输出最近 100 行日志。Docker `json-file` 驱动写出的每一条日志都是一个 JSON 对象，形如：

```json
{"log":"Ready to accept connections tcp\n","stream":"stdout","time":"2025-06-18T10:00:00.000000000Z"}
```

其中 `log` 字段才是实际的日志文本，如果需要只看文本内容，可以配合 `jq` 提取：

```sh
sudo tail -n 100 $(docker inspect --format='{{.LogPath}}' redis) | jq -r '.log'
```

当然，临时排查也可以直接用 `docker logs`：

```sh
docker logs --tail 100 redis
docker logs -f --since 10m redis
```

# 配置日志轮转

为防止日志无限增长，应给容器配置日志轮转策略。Docker 的 `json-file` 驱动支持以下几个参数：

| 参数 | 说明 |
| --- | --- |
| `max-size` | 单个日志文件的最大大小，超过即触发滚动（如 `10m`、`100m`） |
| `max-file` | 保留的滚动日志文件个数（如 `3`、`5`） |

## 方式一：docker run 启动时指定

```sh
docker run -d \
  --name redis \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  redis:7
```

## 方式二：docker-compose 中指定

```yaml
services:
  redis:
    image: redis:7
    container_name: redis
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

## 方式三：全局默认配置（/etc/docker/daemon.json）

如果希望对所有容器统一生效，修改 Docker 守护进程配置：

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

修改后重启 Docker 服务使其生效：

```sh
sudo systemctl restart docker
```

> 注意：`daemon.json` 的全局配置仅对修改后新建的容器生效，已存在的容器需重新创建才会应用新的日志策略。

# 小结

- 用 `docker inspect` 取 `LogPath` 配合 `du`/`tail`，可快速排查容器日志大小与内容。
- 无论是单容器还是全局，都建议配置 `max-size` 与 `max-file`，避免日志文件把磁盘吃满。
- 对已有容器修改日志策略，需要重建容器后才会生效。
