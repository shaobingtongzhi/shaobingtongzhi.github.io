---
title: 整合wb和qoder国际版中的免费模型到Codex
date: 2026-10-13
categories:
  - 杂文
  - 推荐
tags:
  - Codex
  - qoder
  - workbuddy
  - AI
  - docker
---

# 简介

Qoder 国际版和 WorkBuddy（以下简称 wb）都提供了一定的免费模型额度。本文记录如何把这两个平台分别通过网关转成标准 OpenAI 兼容接口，再统一聚合后接入 Codex 使用，从而在 Codex 里直接调用它们的免费模型。

整体流程：

1. 部署 qoder2api-hub，把 Qoder 账号转成 OpenAI 兼容接口（端口 8790）
2. 部署 workbuddy2api-hub，把 WorkBuddy 账号转成 OpenAI 兼容接口（端口 8788）
3. 在 EasyCLIProxyAPI 中把两个接口聚合，并一键写入 Codex 配置
4. 在 Codex 中选择对应模型使用

# 部署 qoder2api-hub

qoder2api-hub 把 Qoder 国际/国内双区多账号封装成标准 OpenAI 兼容接口，支持 Codex / Claude Code 与标准 OpenAI 客户端。

项目地址：https://github.com/shuishuipingan/qoder2api-hub

## 启动容器

```sh
docker run -d --name qoder2api --restart unless-stopped -p 8790:8790 -v E:/docker/qoder2api/accounts:/app/accounts -v E:/docker/qoder2api/usage:/app/usage -e API_KEY=123123 ghcr.io/shuishuipingan/qoder2api-hub:latest
```

参数说明：

- `-p 8790:8790`：把容器 8790 端口映射到宿主机
- `-v E:/docker/qoder2api/accounts:/app/accounts`：持久化账号凭证与出口设置
- `-v E:/docker/qoder2api/usage:/app/usage`：持久化请求流水与指标快照
- `-e API_KEY=123123`：客户端访问网关时使用的密钥，建议改成自己的随机字符串

## 添加账号

启动后浏览器访问 http://127.0.0.1:8790 打开看板（默认面板密码 `admin`），在「网关与运维」页点击「+ 添加账号 (OAuth)」完成设备授权，或点击「🔑 导入 PAT」粘贴令牌。

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20261013/image.q9690ecctkw.webp)

## 创建 API Key

进入「设置」页，在「API KEY 与出口绑定」里点击「+ 添加 API Key」，新建一个供 EasyCLIProxyAPI 使用的 Key，并按需绑定出口（这里固定到国际版出口）。

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20261013/image.1qjw0or1qjy8.webp)

# 部署 workbuddy2api-hub

workbuddy2api-hub 把 WorkBuddy 国际版（www.workbuddy.ai）与国内版（codebuddy.cn）的原生服务封装成标准 OpenAI 兼容接口，支持 Codex / Claude Code / DSH 与标准 OpenAI 客户端。

项目地址：https://github.com/ardeyouxipianyi/workbuddy2api-hub

## 启动容器

```sh
docker run -d --name workbuddy2api --restart unless-stopped -p 8788:8788 -v E:/docker/workbuddy2api/accounts:/app/accounts -v E:/docker/workbuddy2api/usage:/app/usage ghcr.io/ardeyouxipianyi/workbuddy2api-hub:latest
```

参数说明：

- `-p 8788:8788`：把容器 8788 端口映射到宿主机
- `-v E:/docker/workbuddy2api/accounts:/app/accounts`：持久化账号凭证与出口设置
- `-v E:/docker/workbuddy2api/usage:/app/usage`：持久化请求流水与指标快照

## 添加账号

启动后浏览器访问 http://127.0.0.1:8788 打开看板，在「网关与账号」页添加 WorkBuddy 账号。

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20261013/image.d9v31e1chk0.webp)

## 创建 API Key

进入「设置」页，在「API KEY」里点击「+ 添加 API Key」，新建一个供 EasyCLIProxyAPI 使用的 Key（这里跟随面板切换出口）。

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20261013/image.iq4n48a8z4.webp)

# 部署 EasyCLIProxyAPI

EasyCLIProxyAPI 是 CLIProxyAPI 的桌面控制台，可以把多个 OpenAI 兼容服务聚合成统一入口，并一键为 Codex 等客户端写入配置。

项目地址：https://github.com/router-for-me/EasyCLIProxyAPI

到 Releases 下载对应平台的安装包，直接运行即可。

## 添加 OpenAI 兼容服务

进入「API 接入」→「OpenAI 兼容」，点击「+ 添加服务」，把上面两个网关的地址与密钥分别填入：

| 名称 | 地址 | 密钥 |
| --- | --- | --- |
| Qoder2Api（Qoder国际版） | http://127.0.0.1:8790/v1 | qoder2api 网关中创建的 Key |
| Workbuddy2Api | http://127.0.0.1:8788/v1 | workbuddy2api 网关中创建的 Key |

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20261013/image.204m10pu62ww.png)

## 配置 Codex

在「智能体配置」页检测本机的 Codex CLI，一键把配置指向本网关，改前会自动备份原配置。

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20261013/image.27cuoj3kenk0.webp)

# 注意事项

1. 两个网关的端口（8790、8788）不能被其它程序占用。
2. 网关的 API Key 与面板访问密码相互独立，首次登录后建议修改默认密码。
3. EasyCLIProxyAPI 中填的是各网关的 Base URL，注意带上 `/v1` 后缀。