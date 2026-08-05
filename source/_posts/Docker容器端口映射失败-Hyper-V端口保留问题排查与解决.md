---
title: Docker容器端口映射失败：Hyper-V端口保留问题排查与解决
date: 2026-08-05
categories:
  - 运维
  - Docker
tags:
  - Docker
  - Windows
  - Hyper-V
  - 端口冲突
  - WSL2
---

# 问题背景

Windows 上使用 Docker Desktop 运行 MySQL 容器，配置了端口映射 `-p 8306:3306`，容器状态显示 `Up`，MySQL 日志也正常输出 `ready for connections`，但外部应用始终无法连接到数据库。

# 排查过程

## 1. 容器状态正常，但端口映射为空

首先查看容器状态：

```bash
docker ps -a --filter "name=mysql57"
```

```
CONTAINER ID   NAMES     STATUS              PORTS      IMAGE
25c6b3ccc4cd   mysql57   Up About a minute   3306/tcp   mysql:5.7.44
```

注意 `PORTS` 列只显示 `3306/tcp`，而不是正常的 `0.0.0.0:8306->3306/tcp`。这说明端口虽然被 EXPOSE 了，但并没有被成功 PUBLISH 到宿主机。

## 2. 确认端口绑定配置存在但未生效

```bash
# HostConfig 中端口绑定配置是存在的
docker inspect mysql57 --format "{{json .HostConfig.PortBindings}}"
# 输出：{"3306/tcp":[{"HostIp":"","HostPort":"8306"}]}

# 但 NetworkSettings 中端口绑定为空
docker port mysql57
# 输出：（空）

docker inspect mysql57 --format "{{json .NetworkSettings.Ports}}"
# 输出：{"3306/tcp":[]}
```

配置里写了 `8306->3306`，但实际绑定是空的。`docker restart` 重启容器无效，问题依旧。

## 3. MySQL 本身正常

```bash
docker exec mysql57 mysql -uroot -p123456 -e "SHOW DATABASES;"
```

```
Database
information_schema
algorithm_auth_service
algorithm_platform
llxy
...
```

容器内部 MySQL 完全正常，数据库都在。问题出在 Docker 的端口转发层。

## 4. 重建容器暴露真正错误

删除旧容器，用相同参数重新创建：

```bash
docker stop mysql57 && docker rm mysql57
docker run -d --name mysql57 --restart always -p 8306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v E:/docker/mysql/conf/my.cnf:/etc/my.cnf \
  -v E:/docker/mysql/data:/var/lib/mysql \
  -v E:/docker/mysql/log:/var/log/mysql \
  mysql:5.7.44
```

这次 Docker 给出了明确的错误信息：

```
docker: Error response from daemon: ports are not available: exposing port TCP 0.0.0.0:8306 -> 127.0.0.1:0:
listen tcp 0.0.0.0:8306: bind: An attempt was made to access a socket in a way forbidden by its access permissions.
```

**端口 8306 被系统禁止绑定。**

# 根本原因：Hyper-V 动态端口保留

## 查看 Windows TCP 排除端口范围

```bash
netsh int ipv4 show excludedportrange protocol=tcp
```

```
协议 tcp 端口排除范围

开始端口    结束端口
----------    --------
      8204        8303
      8304        8403      ← 8306 在这个范围内！
      8404        8503
      8504        8603
      8604        8703
      8704        8803
      8804        8903
      8904        9003
      9004        9103
      9104        9203
      9204        9303
      9304        9403
      9404        9503
      9801        9900
     50000       50059     *
```

**端口 8306 落在 8304-8403 这个排除范围内，被 Hyper-V 保留了。**

## Hyper-V 为什么会保留这些端口？

Windows 的 Hyper-V（包括 WSL2 依赖的虚拟化层）在启动时会从系统的**动态端口范围**（dynamic port range）中随机圈定几段端口给自己用，被圈定的端口会加入 `excludedportrange` 列表，任何应用都无法绑定。

Windows 默认的动态端口范围是 `49152-65535`，但在某些系统配置下（安装了 Hyper-V、WSL2、Docker Desktop 后），这个范围会被扩大到 `1024-65535`，导致 Hyper-V 可能在很低的端口范围（如 8000-9000）就保留端口。

这就是为什么 `docker restart` 不管用——每次重启容器，Docker 都尝试绑定 8306，每次都被系统拒绝。

# 解决方案

## 方案一：换一个不在排除范围内的端口（最简单）

选择一个不在排除列表中的端口，比如 3306、13306 等：

```bash
docker run -d --name mysql57 --restart always -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v E:/docker/mysql/conf/my.cnf:/etc/my.cnf \
  -v E:/docker/mysql/data:/var/lib/mysql \
  -v E:/docker/mysql/log:/var/log/mysql \
  mysql:5.7.44
```

**缺点**：需要修改应用的数据库连接配置。

## 方案二：用 winnat 抢占端口（不重启）

在管理员 PowerShell 中执行：

```powershell
# 1. 停掉 NAT 服务，释放 Hyper-V 占用的端口
net stop winnat

# 2. 把 8306 标记为专属端口
netsh int ipv4 add excludedportrange protocol=tcp startport=8306 numberofports=1

# 3. 重新启动 NAT 服务
net start winnat
```

**缺点**：重启 Windows 后可能失效，需要重新执行。

## 方案三：缩小动态端口范围（推荐，一劳永逸）

把 Windows 动态端口范围缩小到 50000 附近，Hyper-V 就只能从这个范围里抢端口，不会碰 8000-9000 段了。

在管理员 PowerShell 中执行：

```powershell
# 缩小 TCP 和 UDP 动态端口范围到 50000-50999
netsh int ipv4 set dynamicport tcp start=50000 num=1000
netsh int ipv4 set dynamicport udp start=50000 num=1000
```

然后**重启 Windows**，重启后验证：

```bash
netsh int ipv4 show excludedportrange protocol=tcp
```

```
开始端口    结束端口
----------    --------
     50000       50059     *
     50165       50264
     50265       50364
     50463       50562
     50563       50662
     50663       50762
     50763       50862
```

所有排除范围都缩到了 50000 段，8306 彻底解放。

最后重新创建容器：

```bash
docker run -d --name mysql57 --restart always -p 8306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v E:/docker/mysql/conf/my.cnf:/etc/my.cnf \
  -v E:/docker/mysql/data:/var/lib/mysql \
  -v E:/docker/mysql/log:/var/log/mysql \
  mysql:5.7.44
```

验证：

```bash
docker ps --filter "name=mysql57"
```

```
NAMES     STATUS          PORTS
mysql57   Up 13 seconds   0.0.0.0:8306->3306/tcp, [::]:8306->3306/tcp
```

端口映射正常生效，问题彻底解决。

# 总结

| 方案 | 操作 | 优点 | 缺点 |
|------|------|------|------|
| 换端口 | `-p 3306:3306` | 最快，无需重启 | 需改应用配置 |
| winnat 抢占 | `net stop/start winnat` | 不需重启 | 重启后可能失效 |
| **缩小动态范围** | `netsh set dynamicport` | **一劳永逸** | 需重启一次 Windows |

**排查要点**：当 Docker 容器状态正常但端口不通时，先检查 `docker port` 输出是否为空，再查 `netsh int ipv4 show excludedportrange` 确认端口是否被 Hyper-V 保留。
