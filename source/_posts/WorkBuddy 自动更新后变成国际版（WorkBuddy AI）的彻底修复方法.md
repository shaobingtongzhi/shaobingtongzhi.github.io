---
title: WorkBuddy 自动更新后变成国际版（WorkBuddy AI）的彻底修复方法
date: 2026-10-21 10:00:00
tags:
  - WorkBuddy
  - WorkBuddy AI
  - 自动更新
  - 软件故障
  - Windows
categories:
  - 杂文
  - 问题处理
---

# 问题现象

安装的是 WorkBuddy 国内版，程序自动更新后却变成了 **WorkBuddy AI 国际版**：图标、界面语言和登录入口都变了。卸载后重新安装国内版，问题依然复现。

需要先明确一点：这不是简单的"版本切换"，而是**两个版本的更新程序和安装目录发生了混用**。只重装、不清缓存，基本无法恢复。

# 原因分析

WorkBuddy 国内版与国际版本是两个独立产品，但在 Windows 上，它们的自动更新机制会互相干扰，主要有三个环节：

| 环节 | 发生了什么 | 结果 |
| --- | --- | --- |
| 更新包缓存 | 国内版检查更新时，从共用缓存中读到了国际版安装包 | 国际版被当作"新版本"下载执行 |
| 安装目录 | 更新程序默认安装到同名 `WorkBuddy` 目录 | 国内版核心文件（如 `resources`）被覆盖 |
| 重装过程 | 旧缓存仍在，安装时又使用默认目录 | 新装的国内版再次被覆盖 |

因此，修复的关键不是"多装几次"，而是**清干净缓存 + 隔离安装目录**。

# 修复步骤

> 操作前建议先备份聊天记录和配置。下文只删除程序与更新缓存，不删除用户数据目录。

## 第一步：完全退出 WorkBuddy

1. 关闭 WorkBuddy 主窗口。
2. 在任务栏右下角托盘区找到 WorkBuddy 图标，右键退出。
3. 打开任务管理器，确认没有残留的 `WorkBuddy.exe` 进程。

```powershell
Get-Process WorkBuddy -ErrorAction SilentlyContinue | Stop-Process -Force
```

## 第二步：清理更新缓存

打开文件资源管理器，在地址栏分别输入下面的路径，删除对应内容：

| 路径 | 需要删除的内容 |
| --- | --- |
| `%TEMP%` | `workbuddy-update-x64*` 文件夹 |
| `%LOCALAPPDATA%` | `@genieworkbuddy-desktop-updater` 文件夹 |

也可以直接用 PowerShell 清理：

```powershell
Remove-Item "$env:TEMP\workbuddy-update-x64*" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "$env:LOCALAPPDATA\@genieworkbuddy-desktop-updater" -Recurse -Force -ErrorAction SilentlyContinue
```

## 第三步：卸载并删除残留目录

1. 打开「设置 → 应用 → 已安装的应用」，卸载 WorkBuddy。
2. 卸载完成后，检查并删除残留的安装目录，常见位置：
   - `C:\Users\<用户名>\AppData\Local\Programs\WorkBuddy`
   - 安装时自定义的目录
3. 如果目录删不掉，说明进程仍在运行，回到第一步重新确认。

> 想保留聊天记录，请不要删除 `%LOCALAPPDATA%\CodeBuddyExtension` 以及 `.workbuddy` 之类的用户数据目录，只删程序安装目录即可。

## 第四步：隔离安装两个版本

1. 从官网分别下载国内版和国际版的**最新完整安装包**，不要使用软件内置的更新功能。
2. 安装国内版时手动修改安装路径，装到一个独立的文件夹，例如 `D:\Apps\WorkBuddy-CN`。
3. 如需同时使用国际版，装到另一个完全不同的文件夹，例如 `D:\Apps\WorkBuddy-AI`。
4. 安装完成后先不要启动，进入安装目录，为主程序 `WorkBuddy.exe` 创建桌面快捷方式，以后统一用这个快捷方式启动。

| 版本 | 建议安装路径 |
| --- | --- |
| 国内版 | `D:\Apps\WorkBuddy-CN` |
| 国际版 | `D:\Apps\WorkBuddy-AI` |

## 第五步：阻断自动更新（可选）

官方暂未提供关闭自动更新的开关。如果希望彻底避免再次被覆盖，可以在安装完成后选择以下任一方式：

- 删除或重命名安装目录下的 `updates` 文件夹，并移除该目录的写入权限；
- 在 Windows 防火墙中为更新程序添加出站阻止规则；
- 需要升级时，手动下载安装包覆盖安装到原目录。

# 验证修复结果

启动后确认当前运行的是国内版：

```powershell
Get-Process WorkBuddy | Select-Object -ExpandProperty Path
```

输出应为 `D:\Apps\WorkBuddy-CN\WorkBuddy.exe`。如果仍指向默认的 `...\Programs\WorkBuddy\WorkBuddy.exe`，说明快捷方式或安装目录仍有问题。

再确认版本信息：

```powershell
(Get-Item 'D:\Apps\WorkBuddy-CN\WorkBuddy.exe').VersionInfo | Format-List FileVersion,ProductName
```

# 常见问题

**卸载重装后还是国际版？**

更新缓存没有清干净。回到第二步，重点检查 `%TEMP%` 和 `%LOCALAPPDATA%` 下的更新器目录。

**两个版本能共存吗？**

可以，前提是安装目录、快捷方式和数据目录都相互独立，不要共用默认的 `WorkBuddy` 目录。

**聊天记录会丢吗？**

只删除程序安装目录不会丢。不要删除 `%LOCALAPPDATA%` 下的用户数据目录。

**更新后快捷方式又变回国际版？**

说明自动更新仍在生效。按第五步阻断更新，并重新为国内版主程序创建快捷方式。

**数据目录也需要清理吗？**

如果以上步骤无效，检查电脑中是否同时存在 `.workbuddy` 和 `.workbuddy-ai` 两个数据文件夹。数据目录错乱也可能导致客户端行为异常，建议先备份再处理。

# 总结

这个问题的根源是**更新缓存共用 + 安装目录覆盖**，而不是安装包本身有问题。按"退出进程 → 清缓存 → 卸载清目录 → 隔离安装 → 阻断更新"的顺序处理，基本可以一次修复；只做卸载重装，通常会再次复发。