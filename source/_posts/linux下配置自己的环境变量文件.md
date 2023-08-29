---
title: Linux 下自定义环境变量文件
date: 2016-04-01
tags: 
	- 环境变量配置
	- 自定义环境变量配置文件
categories:
	- 实战
	- 问题处理
---

# linux下配置自己的环境变量文件my_env.sh
配置环境变量分为系统级和用户级，系统级所有用户生效，用户级针对特定用户，现场环境根据职能不同，会通过用户限制操作范围，环境变量修改以实际需要为准，遵循权限最小原则。
## 系统级

一般添加系统环境变量，修改/etc/profile文件，如果操作失误，删除重要配置，影响系统运行。

centos7版本中 /etc/profile 默认扫描路径 /etc/profile.d/ 下sh文件，并添加内容到环境变量中，可以通过这种方式不操作/etc/profile增加环境变量。
在/etc/profile.d/下创建文件 my_env.sh，并设置环境变量，如jdk等，内容如下：

```sh
vi /etc/profile.d/my_env.sh
```
```sh
export JAVA_HOME=/usr/local/jdk/jdk-19.0.2
export CLASSPATH=$JAVA_HOME/lib/
export PATH=$JAVA_HOME/bin:$PATH
```
保存退出
```sh
:wq
```
加载配置文件
```sh
source /etc/profile.d/my_env.sh
```
输出环境变量
```sh
echo $PATH
```

## 用户级
编辑用户文件：~/.bash_profile，增加新path配置信息。如jdk：
```sh
vi ~/.bash_profile

# Java Home
export JAVA_HOME=/usr/local/jdk/jdk-19.0.2
export PATH=$PATH:$JAVA_HOME/bin
```