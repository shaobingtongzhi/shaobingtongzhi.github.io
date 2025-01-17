---
title: PowerToys让Windows像Mac一样优雅
date: 2025-01-17
categories:
  - 杂文
  - 推荐
tags:
  - PowerToys
  - 快捷访问
  - Alt + 空格
  - 实用工具
---

# 简介

是 Microsoft 开发的一组实用工具,可帮助高级用户调整和简化其 Windows 体验,从而提高工作效率

下载地址：https://github.com/microsoft/PowerToys

# 安装

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_14-28-00.5cobase1y880.webp)

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_14-28-35.47t9f3vvn2g0.webp)

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_14-29-15.24jxc8y8b3ls.webp)



# 插件

## ChatGPT PowerToys

下载地址：https://github.com/ferraridavide/ChatGPTPowerToys

1. 解压 [Community.PowerToys.Run.Plugin.ChatGPT.x64.zip](https://github.com/ferraridavide/ChatGPTPowerToys/releases/download/v0.87.1/Community.PowerToys.Run.Plugin.ChatGPT.x64.zip)
2. 拷贝至 PowerToys 的插件目录

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_14-57-26.75x9rffxa280.webp)

3. 重启 PowerToys 进入 PowerToys Run 查看

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_14-59-37.4c3nikfy1d60.webp)

4. Alt + 空格进行 搜索查看

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_15-01-49.hvochfqkmxc.webp)

5. 将 ChatGPT 结果设置到第一位，然后重新启用一下 PowerToys Run

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_15-06-06.332cf0pt92q0.webp)

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_16-31-15.6jfb0x4vo2w0.webp)

## WebSearchShortcut

下载地址：https://github.com/Daydreamer-riri/PowerToys-Run-WebSearchShortcut

实现通过 PowerToys 进行ChatGPT搜索时，可以不用在浏览器里打开

1. 解压并复制到 %LOCALAPPDATA%\Microsoft\PowerToys\PowerToys Run\Plugins 目录下

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_16-47-44.6bjc3k1ezxs0.webp)

重启PowerToys。

在PowerToys Run的设置界面，将“包括在全局结果中”打开。

%LOCALAPPDATA%\Microsoft\PowerToys\PowerToys Run\Settings\Plugins\Community.PowerToys.Run.Plugin.WebSearchShortcut\WebSearchShortcutStorage.json

（把地址复制到资源管理器的地址栏即可打开）

然后修改里面的参数。

把https://chat.openai.com/?q=%s

修改为

https://chatgpt.com/?q=%s&temporary-chat=true&hints=search

其中：

&temporary-chat=true 代表使用临时聊天模式

&hints=search 代表开启联网搜索功能

大家可以根据自己的需要，选择是否添加这两个参数，或者配置多个搜索引擎。

Keyword（关键词）建议设为数字（不受输入法状态影响）。

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_16-50-52.wn3455raw68.webp)

2. 设置独立窗口打开， 经测试仅支持Microsoft Edge浏览器，Chrome不支持

   - Edge或Chrome浏览器封装ChatGPT应用
   - 在地址栏输入edge://flags，开启Enable opening supported links with installed web apps
   - 设置封装的ChatGPT应用，进入应用设置，开启“链接处理“

   ![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250117/Snipaste_2025-01-17_16-53-22.60j47j8p7hs0.webp)



