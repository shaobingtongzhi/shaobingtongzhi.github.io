---
title: ZyFun 视频源整理：GitHub 上还能用的订阅地址
date: 2026-11-03
categories:
  - 杂文
  - 实用方法
tags:
  - ZyFun
  - 视频源
  - IPTV
  - GitHub
---

# 写在前面

ZyFun（原 ZY Player）是个跨平台桌面端视频资源播放器，本身不带任何内容，所有片源都靠外部配置的「源」——解析接口、CMS 采集站、IPTV 直播列表、Alist 网盘。它的 GitHub 仓库在 [Hiram-Wong/zyfun](https://github.com/Hiram-Wong/zyfun)，约 9k Star，Electron 写的，界面确实干净。

问题在于源这个东西生命周期很短：域名一换、接口一改，昨天还能用的配置今天就报错。所以这里把当前还活着的几个仓库和订阅地址做一次整理，顺便把几个已经失效的知名仓库标出来，省得再花时间去找。

> 导入路径统一是：设置 → 基础配置 → 数据管理 → 数据导入 → 远端导入 / 本地导入。

# 播放器本体

- 仓库：https://github.com/Hiram-Wong/zyfun
- 定位：跨平台（Windows / macOS / Linux）桌面端视频资源播放器，免费无广告
- 特色：支持解析接口、CMS 采集站、IPTV、Alist 网盘、弹幕

装好之后是空壳，必须自己导源，这是它和其他播放器最大的区别。

# 推荐源仓库

| 仓库 | 说明 | 订阅地址 |
|---|---|---|
| [kongjie0325-art/zyfun-sources](https://github.com/kongjie0325-art/zyfun-sources) | 103 Star，目前最全的合集：17 个解析接口 + 11 个 IPTV 源 + 48 个直播频道 | `https://raw.githubusercontent.com/kongjie0325-art/zyfun-sources/master/zyfun-all-in-one.json` |
| 同上（精简版） | 去掉直播频道，只留源站与解析 | `https://raw.githubusercontent.com/kongjie0325-art/zyfun-sources/master/zyfun-sources.json` |
| [Bilusi/zyfun_sources](https://github.com/Bilusi/zyfun_sources) | ZyFun 完整配置导出，约 60 个源站，site / analyze 结构齐全 | `https://raw.githubusercontent.com/Bilusi/zyfun_sources/main/zy_config_1790133094297.json` |
| [bhsherry/vod-source-feed](https://github.com/bhsherry/vod-source-feed) | GitHub Actions 每天 03:17 自动逐个探活，只保留 category 接口返回正常的 24 个 CMS 源 | `https://raw.githubusercontent.com/bhsherry/vod-source-feed/main/public/zyfun-vod-import.json` |
| [cuiocean/ZY-Player-Resources](https://github.com/cuiocean/ZY-Player-Resources) | ZY Player 官方老资源库，源站列表 + 影视推荐 | `https://raw.githubusercontent.com/cuiocean/ZY-Player-Resources/main/Sites/Sites.json` |
| [SeqCrafter/zyfun_danmu](https://github.com/SeqCrafter/zyfun_danmu) | 弹幕接口，不是视频源，用来补弹幕 | 见仓库 README |

## 一个必须注意的坑

`kongjie0325-art/zyfun-sources` 的 README 里，所有链接写的都是 `main` 分支：

```
https://raw.githubusercontent.com/kongjie0325-art/zyfun-sources/main/zyfun-all-in-one.json
```

但这个仓库的默认分支是 `master`，`main` 那条链接实测返回 **404**。直接照 README 复制粘贴，远端导入必然失败，而且 ZyFun 给出的报错信息很不明确，很容易误判成格式问题。把 `main` 换成 `master` 就能正常下载了。

# 已经失效的仓库

这两个在各类教程里被引用得最多，但仓库本身已经没了，不用再试：

- `chao921125/source` —— 常被称为「最全第三方源」，仓库已 404 删除
- `ls125781003/dmtg` —— 里面的 `zy.json` 聚合源曾广泛流传，仓库同样 404

# CMS 采集站

CMS 采集站不能通过 JSON 批量导入，ZyFun 里需要在「快捷配置 → 此软件」逐条添加。常用的这些：

| 名称 | API 地址 |
|---|---|
| 量子资源 | `https://cj.lziapi.com/api.php/provide/vod/` |
| 非凡资源 | `https://cj.ffzyapi.com/api.php/provide/vod/` |
| 无尽资源 | `https://api.wujinapi.me/api.php/provide/vod/` |
| 光速资源 | `https://api.guangsuapi.com/api.php/provide/vod/` |
| 闪电资源 | `https://sdzyapi.com/api.php/provide/vod/` |
| 暴风资源 | `https://bfzyapi.com/api.php/provide/vod/` |
| 极速资源 | `https://jszyapi.com/api.php/provide/vod/` |
| 红牛资源 | `https://www.hongniuzy2.com/api.php/provide/vod/` |
| 电影天堂 | `https://caiji.dyttzyapi.com/api.php/provide/vod/` |
| 天涯资源 | `https://tyyszy.com/api.php/provide/vod` |
| 新浪资源 | `https://api.xinlangapi.com/xinlangapi.php/provide/vod/` |
| 索尼 / 率率资源 | `https://suoniapi.com/api.php/provide/vod/` |
| 百度资源 | `https://api.apibdzy.com/api.php/provide/vod` |
| 优质资源 | `https://api.1080zyku.com/inc/api_mac10.php` |
| CK 资源 | `https://ckzy.me/api.php/provide/vod/` |
| IKUN 资源 | `https://ikunzyapi.com/api.php/provide/vod/from/ikm3u8/at/json` |

注意 `type` 字段：返回 JSON 的填 `1`，返回 XML 的填 `0`，填错会直接解析失败。像量子资源同时提供两套接口，`/at/xml/` 结尾的是 XML 版本。

# 解析接口

这类接口主要给网页播放用，适合临时抓某个站的会员视频。已包含在 all-in-one 那套 JSON 里：

| 名称 | 地址 |
|---|---|
| 咸鱼 | `https://jx.xyflv.cc/?url=` |
| 虾米 | `https://jx.xmflv.com/?url=` |
| 夜幕 | `https://www.yemu.xyz/?url=` |
| 冰豆 | `https://bd.jx.cn/?url=` |
| 爱狗 | `https://jx.i2g.cn/?url=` |
| OK 解析 | `https://okjx.cc/?url=` |
| 诺讯 | `https://www.nxflv.com/?url=` |

解析接口的存活率比 CMS 站更低，失效是常态，多备几个就行。

# 导入顺序建议

ZyFun 对远端 JSON 的校验比较严格，直接远端导入经常失败，推荐按这个顺序来：

1. 浏览器打开 all-in-one 的 raw 地址，右键另存为本地 JSON
2. ZyFun 里「设置 → 基础配置 → 数据管理 → 数据导入 → **本地导入**」，选中文件
3. 导入完成后重启客户端，源列表才会刷新
4. 用 `bhsherry/vod-source-feed` 的每日探活列表做补充，它只收录当前接口正常的源，可以拿来替换掉失效项
5. CMS 采集站单独在「快捷配置 → 此软件」逐条添加

另外 `bhsherry/vod-source-feed` 仓库里有个 `public/vod-source-health.json`，记录了每个源最近一次探活的结果和失败原因。源突然不能用了，先去这里看看是接口挂了还是自己网络的问题，比一个个手测快得多。

# 一点提醒

这些源全部是第三方聚合，域名随时可能变更，接口随时可能失效，也没人保证内容质量。整理出来只是省掉重复搜索的时间，不保证长期可用。

仅供个人学习与技术测试，请支持正版。
