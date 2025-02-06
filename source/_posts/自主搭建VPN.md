---
title: 使用Cloudflare Workers搭建Shiny应用
date: 2025-02-06
categories: 
  - 杂文
  - 推荐
tags:
  - Cloudflare
  - Shiny
  - Workers
---

在这篇博客中，我记录了如何使用Cloudflare搭建Shiny应用的过程，具体步骤如下：

# 在Cloudflare创建Worker

首先，你需要登录到Cloudflare，在左侧菜单栏中选择【计算（Workers）】。接着进入你的Worker项目设置，点击【设置】按钮。

![Cloudflare Workers设置](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250206/Snipaste_2025-02-06_08-07-37.32hrkor3p2m0.webp)

在设置页面，点击【添加】按钮，创建新的变量并设置你的UUID变量。UUID是你所需要的一些信息，可以使用.env文件来快速配置多个环境变量。

# 配置Worker环境变量

接下来，你需要设置Worker的环境变量。点击【添加】后，选择文本类型，并命名为`uuid`。然后填入你的UUID值。

![Cloudflare Workers环境变量](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250206/Snipaste_2025-02-06_08-13-51.1y0sl56fvp9c.webp)

配置完成后，点击保存。

# 生成自定义URL

通过上面的步骤，你将获得一个类似这样的URL：`https://xxxx.ssss.workers.dev/`。在此URL后，你只需要添加你的UUID信息即可访问你的Shiny应用。

![Cloudflare Worker自定义URL](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250206/Snipaste_2025-02-06_08-49-00.glec40xekaw.webp)

# 结语 

通过以上步骤，你成功地使用Cloudflare的Workers服务搭建了Shiny应用。你可以将此Worker链接分享给其他人，或者根据自己的需求修改配置。

# 关键点

- https://github.com/yonggekkk/Cloudflare_vless_trojan

- 复制_worker.js

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250206/Snipaste_2025-02-06_08-54-06.kyclexmz8hs.webp)