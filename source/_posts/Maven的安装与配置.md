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

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/企业微信截图_20240408082830.2667so4c9wzk.webp)

或者

https://archive.apache.org/dist/maven/maven-3/

# Maven 的安装与配置

1. 解压至非中文无空格目录D:\Programs

2. 配置环境变量 ，

   1）名称：M2_HOME 值：D:\Programs\apache-maven-3.9.6

   2）Path里追加：%M2_HOME%\bin 

3. 验证maven是否可用，打开cmd , 输入命令：mvn -v 

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/企业微信截图_20240408083756.2ulsmzl8d1a0.webp)

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
        <maven.compiler.compilerVersion>1.8</xmaven.compiler.compilerVersion>
    </properties>
</profile>
```

# IDEA中配置Maven插件

1. 打开设置-》构建、执行、部署-》构建工具-》Maven
2. 配置 Maven 主路径，用户配置文件、本地仓库

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/企业微信截图_20240408092153.29xlrix3usro.webp)