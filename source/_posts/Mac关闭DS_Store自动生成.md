---
title: Mac关闭DS_Store自动生成
date: 2024-01-22
tags:
  - Mac
  - DS_Store
categories:
  - 杂文
  - 实用方法
---
在终端执行并重启后生效
```sh
# 关闭自动生成
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE

# 开启自动生成
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool FALSE
```

