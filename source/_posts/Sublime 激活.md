---
title: Sublime 的激活
date: 2023-03-17
tags:
	- Sublime
	- PHP
	- 编辑器
	- 激活 
categories:
	- 学习笔记 
	- 03-编辑器相关笔记 

---

# sublime4143激活教程
## windows
- 使用浏览器打开[hexed.it](https://hexed.it/)
![在这里插入图片描述](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20230824/68ba1c378b2847569d886b9551f5d49c.4cj0h2821fw0.webp)

- 点击“打开文件"，选择sublime text 安装目录中的“sublime_text.exe”
 ![在这里插入图片描述](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20230824/8c17dc04c4c64055bc801e7113498336.2eb1qmgwdyv4.webp)
- 在搜索框中输入807805000f94c1，找到后，替换为c64005014885c9
![在这里插入图片描述](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20230824/9fd749b60b2e40f4b49af2f75f3c37e8.12m8a09e6hu8.webp)

- 点击“另存为”，保存文件到本地，文件名设定为sublime_text.exe
- 备份原sublime_text.exe文件（修改为如sublime_text_bk.exe）
- 将新保存的sublime_text.exe复制到原sublime text 4安装目录中

## mac
- 下载并安装APP“Hex Fiend”，Hex Fiend, a fast and clever hex editor for macOS
- 打开Sublime Text所在的目录：
- Sublime Text -> (右键)Show Package ->Contents -> Mac OS -> Sublime_text
![在这里插入图片描述](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20230824/9fb737b0bf9844e8be5590539ded60ea.9wninzb9jgw.webp)

- 打开Hex Fiend，将Sublime_text拖至Hex Fiend中
- 在搜索框中输入807805000f94c1，找到后，替换为 c64005014885c9
![在这里插入图片描述](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20230824/dcfc4dbc79e943e19d3747a5a7da392f.3wq368nxf1g0.webp)
- 保存后，退出
- 在Terminal下，进入Sublime Text目录执行如下语句
![在这里插入图片描述](https://jsd.cdn.zzko.cn/gh/hfshaobing/picx-images-hosting@master/20230824/4a49a7d1af0041579174b28adc734815.33n0sf2muqy0.webp)
```sh
codesign --remove-signature Sublime\ Text.app/
```
- 打开sublime Text，激活成功。

相关软件下载地址：
百度云盘链接:[https://pan.baidu.com/s/1m_bKaPyxGmV-byDUSF47TQ](https://pan.baidu.com/s/1m_bKaPyxGmV-byDUSF47TQ)
提取码: x2ga