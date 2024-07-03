---
title: Maven的安装与配置
date: 2024-04-08
categories: 
  - 开发工具
  - Maven
tags:
  - Maven
---

# 前置环境

- windows 10 
- jdk 1.8

# Maven 下载

https://maven.apache.org/download.cgi

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/企业微信截图_20240408082830.2667so4c9wzk.webp)

或者

https://archive.apache.org/dist/maven/maven-3/

# Maven 的安装与配置

1. 解压至非中文无空格目录，比如：D:\Programs

## Windows下的环境变量配置

1. 环境变量 ，

   1）名称：M2_HOME 值：D:\Programs\apache-maven-3.9.6

   2）Path里追加：%M2_HOME%\bin 

2. 验证maven是否可用，打开cmd , 输入命令：mvn -v 

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/企业微信截图_20240408083756.2ulsmzl8d1a0.webp)

## MAC下的环境变量配置

在终端打开配置环境变量到文件：

1）在终端输入  vim ~/.bash_profile，进入到环境变量配置文件里面；

2）进入后，是read模式，按下 i (编辑)键，进入insert模式；

3）将环境变量加入其实，环境变量如下：

export MAVEN_HOME=/Users/mac/maven/apache-maven-3.8.1

export PATH=$PATH:$MAVEN_HOME/bin

4）按下 ESC，退出insert模式；

5）输入 :wq (保存修改)退出当前文件；

6）使修改的环境变量bash_profile文件生效，输入 source ~/.bash_profile，按下Enter键即可.

## setting.xml配置文件设置

maven的安装包中conf文件夹下的setting.xml文件指定正确的仓库

4. 配置本地仓库位置

​		1）打开 D:\Programs\apache-maven-3.9.6\conf\settings.xml

​		2）把注释中的 <localRepository>/path/to/local/repo</localRepository> 复制出来，修改路径：D:/mvn_repository ，大概在 **53行** 左右

5. 配置阿里云镜像加速

​		1）查看配置指南 https://developer.aliyun.com/mvn/guide

​		2）打开 D:\Programs\apache-maven-3.9.6\conf\settings.xml

​		3）复制下面的代码到 <mirrors></mirrors> 标签里,大概在 **159行** 左右

```xml
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

6. 配置 jdk 版本

​		1）打开 D:\Programs\apache-maven-3.9.6\conf\settings.xml

​		2）复制下面的代码到 <profiles></profiles> 标签里，大概在 **196行** 左右

```xml
<profile>
    <id>jdk-1.8</id>
    <activation>
        <jdk>1.8</jdk>
        <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```

# IDEA中配置Maven插件

1. 打开设置-》构建、执行、部署-》构建工具-》Maven
2. 配置 Maven 主路径，用户配置文件、本地仓库

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/企业微信截图_20240408092153.29xlrix3usro.webp)