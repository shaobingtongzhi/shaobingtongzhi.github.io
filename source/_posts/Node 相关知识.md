---
title: Node 相关知识
date: 2020-02-01
tags: 
	- node
	- npm 
	- nvm 
	- 依赖关系
categories:
	- 学习笔记 
	- 05-Node相关知识
---

# node与npm 知识总结

## node 版本更新

### 简介

node.js是javascript的一种运行环境，是对Google V8引擎进行的封装，是一个服务器端的javascript的解释器。

### 使用 n 

注意：n命令不支持windows系統，只支持mac

#### 简介

一个简易的node版本管理工具



#### 安装

```sh
# 查看当前版本
node -v
# 清理本地包缓存
npm cache clean -f
# 安装
npm i -g n
# 查看n是否安装成功
n -V
```

#### 使用

```sh
n stable // 把当前系统的 Node 更新成最新的 “稳定版本”
n lts // 长期支持版
n latest // 最新版
n 16.13.1 // 指定安装版本
n ls-remote --all //查看仓库中所有版本清单
```

### 使用 nvm 进行更新

#### 简介

nvm全名node.js version management，是一个node的版本管理工具。通过它可以安装和切换不同版本的nodejs

#### 安装

- windows

下载地址：https://github.com/coreybutler/nvm-windows/releases 下载其中的 nvm-step.exe 安装即可

- mac、linux

下载地址：https://github.com/nvm-sh/nvm

安装步骤参考：[https://blog.csdn.net/sebeefe/article/details/126773937](https://blog.csdn.net/sebeefe/article/details/126773937)

#### 基本使用


```sh
# 列出所有可用的node版本
nvm list available
# 列出安装了node的哪些版本
nvm list
# 安装指定版本的node
nvm install xx.xx.xx
# 切换到指定版本的node
nvm use xx.xx.xx
```

### 查看 node 安装路径

```sh
# mac、linux 使用
which node
```


## npm 命令

### 简介

npm是nodejs的包管理器（package manager）。
nodejs和npm是包含关系，nodejs中含有npm，安装好nodejs，cmd输入npm -v会发现npm的版本号，说明npm已经安装好。

其实我们在Node.js上开发时，会用到很多别人已经写好的javascript代码，如果每当我们需要别人的代码时，都根据名字搜索一下，下载源码，解压，再使用，会非常麻烦。于是就出现了包管理器npm。大家把自己写好的源码上传到npm官网上，如果要用某个或某些个，直接通过npm安装就可以了，不用管那个源码在哪里。并且如果我们要使用模块A，而模块A又依赖模块B，模块B又依赖模块C和D，此时npm会根据依赖关系，把所有依赖的包都下载下来并且管理起来

### 基本使用

#### 查看当前源

```sh
npm get registry
```

#### 设置镜像源

```sh
# 设置淘宝镜像源
npm config set registry https://registry.npm.taobao.org 
# 设置官方镜像源
npm config set registry https://registry.npmjs.org
```

#### 升级指定版本

```sh
npm install npm@6.14.13 -g
```

#### 清理缓存

```sh
npm cache clean --force
```

## depcheck 命令

### 简介

Depcheck 是一款用于分析项目中依赖关系的工具，它可以帮助我们找出以下问题：在 package.json 中，每个依赖包如何被使用、哪些依赖包没有用处、哪些依赖包缺失。它是解决前端项目中依赖包清理问题的一个常用工具

### 安装

```sh
npm i -g depcheck
```

### 基本使用

```SH
# 在package.json所在目录执行如下命令
depcheck
```

