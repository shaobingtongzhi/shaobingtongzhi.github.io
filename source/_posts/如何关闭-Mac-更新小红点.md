---
title: 如何关闭 Mac 更新小红点
date: 2024-01-03
tag: 
  - Mac
  - 小红点
categories:
  - 杂文
  - 实用方法
---

# 如何关闭 Mac 更新小红点

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240103/截屏2024-01-03-上午10.02.08.7k3ev1rppn40.webp)

# 步骤

第一步：关闭自动更新

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240103/截屏2024-01-03-上午10.15.46.nygo1fatngg.webp)

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240103/截屏2024-01-03-上午10.17.52.23u4m6h2cphc.webp)

第二步：打开终端，输入以下命令即可

```sh
defaults write com.apple.systempreferences AttentionPrefBundleIDs 0 && killall Dock
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240103/截屏2024-01-03-上午10.19.58.6lsrtezs8b40.webp)

# 如何恢复小红点

点开设置，开启更新就好了

