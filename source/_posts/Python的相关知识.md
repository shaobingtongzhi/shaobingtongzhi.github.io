---
title: Python的相关知识
date: 2024-08-09
categories:
  - 学习笔记
  - 11-Python相关笔记
tags:
  - Python
---



# 常用命令

## pip

```sh
# 查看安装模块列表
pip list
# 安装指定模块
pip install xxxx

```



# 常用的第三方模块

## requests

作用：用requests获取URL资源，就是这么简单！

```sh
# 安装命令
pip install requests
```

# 问题汇总

## pip install 速度慢

1. 更换镜像源

   ```sh
   pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
   # 或者
   pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
   ```

2. 安装时指定

   ```sh
   # 可以在使用pip install命令时使用-i选项指定镜像源。例如：
   pip install package_name -i https://pypi.tuna.tsinghua.edu.cn/simple
   ```

   