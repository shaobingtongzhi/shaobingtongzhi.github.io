---
title: CentOS 安装 JDK 环境
date: 2016-03-01
tags:
	- java
	- java运行环境
	- jdk
categories:
	- 学习笔记
	- 09-java相关笔记
---

# centos安装jdk源码
## 下载jdk源码包
> https://download.oracle.com/java/19/latest/jdk-19_linux-x64_bin.tar.gz

## 安装
### 解压

```sh
#在目录usr/local下新建目录jdk,并将下载的jdk源码包传到当前目录下
cd /usr/local
mkdir jdk
cd jdk
#解压
tar -zxvf xxxx.tar.gz
```
### 配置环境变量
```sh
#进入配置目录并创建配置文件
cd /etc/profile.d
touch java_env.sh
vi java_env.sh
#输入如下内容并保存退出

JAVA_HOME=/usr/local/jdk/jdk-19.0.2
JAVA_PATH=${JAVA_HOME}/bin
PATH=$PATH:${JAVA_PATH}
export PATH
#保存退出
：wq

#动态加载配置文件
source ./java_env.sh
```

## 验证
```sh
java -version 
javac -version
```
## 编写HelloWorld.java,编译并执行
> HelloWorld.java

```sh
public class HelloWorld{
	public static void main(String[] args){
		System.out.println("hello world");
	}
}
```
> 编译

```sh
javac HelloWorld.java
```
> 执行

```sh
java HelloWorld
```

