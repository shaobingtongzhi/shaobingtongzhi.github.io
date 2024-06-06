---
title: Jenkins实现自动化部署
date: 2024-05-28
categories:
  - 实战
  - Web服务器部署
tags:
  - Jenkins
  - 持续集成
  - 自动化部署
---

# 安装

```sh
docker pull jenkins/jenkins

docker run -d -p 10240:8080 -p 10241:50000 -v E:/docker/jenkins/jenkins_home:/var/jenkins_home --name jenkins jenkins/jenkins:latest

# 查看初始密码
docker logs jenkins
```

# 插件管理

插件地址替换：https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

在容器内安装mvn工具

参考连接：[Maven的安装与配置](/202404/15e9783b8316.html)

给容器中的mvn命令设置软连接

```sh
docker exec -it -u root jenkins bash

ln -s /var/jenkins_home/env/apache-maven-3.8.1/bin/mvn /usr/bin/mvn
ln -s /var/jenkins_home/env/apache-maven-3.8.1/bin/mvn /usr/local/bin/mvn
```



stop.bat

```bat
@echo off
net stop SpringBoot_BackItems
exit
```

start.bat

```bat
@echo off
net start SpringBoot_BackItems
exit
```

restart.bat

这个是后端自动脚本

```bat
@echo off
rem rem表示注释，不会执行
rem 关闭服务
net stop SpringBoot_BackItems
rem 设置操作目录
set dir= C:\JavaWorkspace\
del %dir%app-old.jar
rem 重命名文件
ren %dir%app.jar app-old.jar
ren %dir%api-zhiboredian-net-1.0-SNAPSHOT.jar app.jar
rem 开启服务
net start SpringBoot_BackItems
exit
```

> 实际环境调整 服务名称和操作目录即可

回滚脚本设置

```sh
case $deploy_env in
  deploy)
    echo "deploy: $deploy_env"
    mvn clean package -DskipTests
    # path="${WORKSPACE}/bak/${BUILD_NUMBER}"      #创建每次要备份的目录
    #if [ -d $path ];
    #then
    #    echo "The files is already  exists "
    #else
    #    mkdir -p  $path
    #fi
    #\cp -f ${WORKSPACE}/target/*.war $path        #将打包好的war包备份到相应目录,覆盖已存在的目标
    #echo "Completing!"
    ;;
  rollback)
      echo "rollback: $deploy_env"
      echo "version: $version"
      rm -rf target
      cp -R ${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${version}/archive/target .
      # cd ${WORKSPACE}/bak/$Version            #进入备份目录
      # \cp -f *.war ${WORKSPACE}/target/       #将备份拷贝到程序打包目录中，并覆盖之前的war包
      pwd && ls
      ;;
  *)
  exit
      ;;
esac
```

deploy.bat

这个是前端自动部署脚本

```bat
::@echo off
set dir= C:\JavaWorkspace\frontend\
cd %dir%
rmdir /S /Q %dir%dist_old
rem 重命名文件
ren %dir%dist dist_old

set ZIP_FILE=C:\JavaWorkspace\frontend\dist.zip
set DESTINATION_DIR=C:\JavaWorkspace\frontend
powershell -Command "Expand-Archive -Path '%ZIP_FILE%' -DestinationPath '%DESTINATION_DIR%'"

del %dir%dist.zip
exit
```

# 详细步骤

## 新建任务

输入任务名称，选择：构建一个自由风格的软件项目 ，确定

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240529/Snipaste_2024-05-29_15-31-08.rm7hiew55c0.webp)

## 配置

### General

1. 勾选：参数化构建过程

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240529/Snipaste_2024-05-29_15-37-25.4zqkub4a3ao0.webp)

2. 添加参数-》选项参数

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240529/Snipaste_2024-05-29_15-39-39.2xcum1tr0jm0.webp)

3. 添加参数-》字符参数

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240529/Snipaste_2024-05-29_15-41-15.61lf3bblsjg0.webp)

### 源码管理

填写代码远程仓库地址

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240529/Snipaste_2024-05-29_15-46-33.4ffhg36vio8.webp)

### 构建触发器

可以不选，不选的话，需要手动构建

### 构建环境

不选

### Build Steps

选择执行 shell ,配置下面的命令

```sh
case $deploy_env in
  deploy)
    echo "deploy: $deploy_env"
    mvn clean package -DskipTests
    ;;
  rollback)
      echo "rollback: $deploy_env"
      echo "version: $version"
      rm -rf target
      cp -R ${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${version}/archive/target .
      pwd && ls
      ;;
  *)
  exit
      ;;
esac
```

> 注意：这里的参数，$deploy_env 要与第一步General里配置的参数一致

### 构建后操作

第一步：增加构建后操作步骤-》**归档成品**

内容填写：target/*.jar

选择高级-》勾选：只有构建成功时归档

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240529/Snipaste_2024-05-29_15-56-25.7gofj4tran40.webp)

第二步：增加构建后操作步骤-》**Send build artifacts over SSH**

选择 SSH Server中配置好的 Name，相当于是实际的服务器（最终代码部署所在的服务器）

> 这一步的意思是要把构建好的jar包通过ssh publishers 传到部署服务器上

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240529/Snipaste_2024-05-29_16-24-20.3s7pyqbkxq20.webp)

设置源地址、目标地址以及在目标服务器上执行的命令

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240529/Snipaste_2024-05-29_16-18-21.5qewcal04480.webp)

> 注意：在目标服务器上执行的指令需要采用全路径（上面的第四步在生产环境有问题），例如直接填写即可：C:/JavaWorkspace/restart.bat

# docker 容器

在 jenkins 的容器内安装nvm

由于jenkins容器默认用户是jenkins，且没有用户根目录，安装完nvm后，无法使用设置环境变量，当前把环境变量设置在了 /etc/bash.bashrc

在这个文件末尾追加即可

```sh
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
```

