---
title: Docker常用命令
date: 2020-05-01
categories:
  - 学习笔记
  - 08-docker相关笔记
tags:
  - docker
  - save
  - load
---

# 镜像的导入导出

导出

```sh
docker save -o <文件路径> <镜像名称>:<标签>
```

导入

```sh
docker load -i <镜像文件路径>
```

# 容器导入、导出为镜像

导出

```sh
docker export <容器ID或容器名称> > <输出文件>
#示例：
# 导出为tar
docker export my_container > my_container.tar
# 导出并压缩
docker export my_container | gzip > my_container.tar.gz
```

导入

```sh
docker import <归档文件路径> [<镜像名称>:<标签>]
#示例：
#
docker import my_container.tar my_new_image:latest

docker import https://example.com/my_container.tar my_new_image:latest

docker import my_container.tar my_custom_image:v1

```

