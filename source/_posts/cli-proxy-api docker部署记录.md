---
title: cli-proxy-api docker部署记录
date: 2026-05-11
categories:
  - 实战
  - docker
tags:
  - docker
  - cli-proxy-api
  - AI
---

# 简介

cli-proxy-api 是一个可以通过 Docker 快速部署的代理服务，部署后通过宿主机端口对外提供访问能力。

本文记录通过 Docker 部署 cli-proxy-api 的过程，方便后续迁移和复用。

# 准备目录

这里统一把配置文件和数据目录放到宿主机的 `/Users/mac/Docker/CLIProxyAPI` 目录下。

```sh
mkdir -p /Users/mac/Docker/CLIProxyAPI
```

# 准备配置文件

`config.yaml` 可以从官方示例配置文件复制一份作为初始配置：

```sh
curl -o /Users/mac/Docker/CLIProxyAPI/config.yaml https://raw.githubusercontent.com/router-for-me/CLIProxyAPI/main/config.example.yaml
```

如果下载失败，也可以先手动创建一个最小配置文件：

```yaml
host: ""
port: 8317

remote-management:
  allow-remote: false
  secret-key: ""
  disable-control-panel: false

auth-dir: "~/.cli-proxy-api"

api-keys:
  - "your-api-key"

debug: false
logging-to-file: false
usage-statistics-enabled: false
proxy-url: ""
request-retry: 3

quota-exceeded:
  switch-project: true
  switch-preview-model: true
```

其中 `api-keys` 是客户端访问代理服务时使用的密钥，建议改成自己的随机字符串。

容器启动时会将宿主机的配置文件挂载到容器内：

```sh
/Users/mac/Docker/CLIProxyAPI/config.yaml:/CLIProxyAPI/config.yaml
```

同时将宿主机目录挂载到容器内的数据目录：

```sh
/Users/mac/Docker/CLIProxyAPI:/root/.cli-proxy-api
```

# 拉取镜像

```sh
docker pull eceasy/cli-proxy-api:v7.0.2
```

# 启动服务

```sh
docker run -d --name cli-proxy-api \
  -p 8317:8317 \
  -v /Users/mac/Docker/CLIProxyAPI/config.yaml:/CLIProxyAPI/config.yaml \
  -v /Users/mac/Docker/CLIProxyAPI:/root/.cli-proxy-api \
  eceasy/cli-proxy-api:v7.0.2
```

参数说明：

- `-d`：后台运行容器
- `--name cli-proxy-api`：指定容器名称
- `-p 8317:8317`：将容器的 8317 端口映射到宿主机的 8317 端口
- `-v /Users/mac/Docker/CLIProxyAPI/config.yaml:/CLIProxyAPI/config.yaml`：挂载配置文件
- `-v /Users/mac/Docker/CLIProxyAPI:/root/.cli-proxy-api`：挂载数据目录，方便持久化保存
- `eceasy/cli-proxy-api:v7.0.2`：使用的镜像版本

# 查看运行状态

```sh
docker ps | grep cli-proxy-api
```

查看端口映射：

```sh
docker port cli-proxy-api
```

查看日志：

```sh
docker logs -f cli-proxy-api
```

# 访问服务

服务启动成功后，可以通过下面的地址访问：

```sh
http://localhost:8317
```

如果部署在服务器上，需要将 `localhost` 替换成服务器 IP 或域名：

```sh
http://服务器IP:8317
```

# 常用维护命令

## 停止服务

```sh
docker stop cli-proxy-api
```

## 启动服务

```sh
docker start cli-proxy-api
```

## 重启服务

```sh
docker restart cli-proxy-api
```

## 删除容器

```sh
docker stop cli-proxy-api
docker rm cli-proxy-api
```

## 重新部署

如果修改了挂载目录或镜像版本，可以删除旧容器后重新执行启动命令：

```sh
docker stop cli-proxy-api
docker rm cli-proxy-api

docker run -d --name cli-proxy-api \
  -p 8317:8317 \
  -v /Users/mac/Docker/CLIProxyAPI/config.yaml:/CLIProxyAPI/config.yaml \
  -v /Users/mac/Docker/CLIProxyAPI:/root/.cli-proxy-api \
  eceasy/cli-proxy-api:v7.0.2
```

# 注意事项

1. 宿主机的 8317 端口不能被其它程序占用。
2. `config.yaml` 文件路径要提前创建，否则 Docker 可能会把它当成目录处理。
3. 配置文件修改后，建议重启容器使配置生效。
4. 生产环境部署时，建议根据实际情况配置防火墙和反向代理。
