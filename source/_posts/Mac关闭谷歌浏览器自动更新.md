---
title: Mac 关闭谷歌浏览器自动更新
date: 2023-06-19 20:19:48
tags: 
    - 浏览器
    - chrome
    - 自动更新
categories:
    - 杂文
    - 实用方法
---

# Mac 关闭谷歌浏览器自动更新

```sh
# 进入浏览器安装目录
cd /Users/xxxx/Library/Google

# 查看更新程序的权限组 并记录
aichaidngxuyuan:Google xxxx$ ls -l
total 8
-rw-------   1 xxxx  staff   61 10 25  2018 Google Chrome Brand.plist
drwxr-xr-x@ 10 xxxx  staff  320  6 19 11:33 GoogleSoftwareUpdate

# 修改更新程序的权限组 即可 
sudo chown root:wheel GoogleSoftwareUpdate

# 还原自动更新，把权限还原就行了
sudo chown xxxx:staff GoogleSoftwareUpdate
```
