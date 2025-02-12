---
title: Ollama+deepseek搭建本地AI大模型
date: 2025-02-12
categories:
  - 杂文
  - 推荐
tags:
  - 大模型
  - deepseek
  - ollama
  - AI编程
---

# 简介

虽然deepseek官方的价格已经很便宜了，但是考虑到数据泄露风险问题，在本地搭建一个可用的大模型就很有必要了！本文介绍如何通过Ollama搭建deepseek-r1:7b模型，实现可以在phpstorm里编程，可以通过chatbox进行聊天！！！

# Ollama

## Ollama是干啥的

Get up and running with large language models. 

1. 下载安装

官网地址：https://ollama.com/

没啥说的，download -> install

2. 验证

打开cmd，输入 ollama help 命令查看结果，有结果表示安装成功！

3. 拉取模型

在Ollama官网找到模型页，选择对应的模型进行拉取运行即可,这里拉取的是deepseek-r1:7b

```sh
ollama pull deepseek-r1:7b
# 或直接执行
ollama run deepseek-r1:7b
```

由于网络问题，拉取经常失败可使用下面的脚本进行自动拉取

- 脚本名称 ollama.ps1 后缀要求是ps1 
- 在powershell中执行如下命令

```sh
# 执行权限
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process -Force
# 执行脚本文件
.\ollama.ps1
```

脚本内容:

```sh
$ModelName = "deepseek-r1:7b"

while ($true) {
    # 检查模型是否下载完成
    $models = ollama list
    if ($models -match $ModelName) {
        Write-Host "模型已下载完成！"
        break
    }

    # 启动ollama进程
    Write-Host "开始下载模型..."
    $process = Start-Process -FilePath "ollama" -ArgumentList "run", $ModelName -PassThru -NoNewWindow
    
    # 等待60秒
    Start-Sleep -Seconds 60

    # 确保进程仍然运行，然后终止
    if ($process -and (Get-Process -Id $process.Id -ErrorAction SilentlyContinue)) {
        try {
            Stop-Process -Id $process.Id -Force
            Write-Host "已中断本次下载，准备重新尝试..."
        }
        catch {
            Write-Host "进程已结束，无需中断。"
        }
    }
}

```

4. 效果如下

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250212/Snipaste_2025-02-12_09-54-25.u6kfd70tulc.webp)

# chatbox

顾名思义，一个AI客户端，支持众多先进的AI模型和API

1. 下载安装

官网地址：https://chatboxai.app/zh

2. 配置

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250212/Snipaste_2025-02-12_10-14-18.2uxpk2814nu0.webp)

# Phpstorm

## 安装插件 Continue

设置-插件-Continue

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250212/Snipaste_2025-02-12_10-20-52.6mokromgtk80.webp)

## 配置

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20250212/Snipaste_2025-02-12_10-24-44.4jyjg99bwm00.webp)