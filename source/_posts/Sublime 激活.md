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
![在这里插入图片描述](https://github.com/hfshaobing/picx-images-hosting/raw/master/20230824/68ba1c378b2847569d886b9551f5d49c.4cj0h2821fw0.webp)

- 点击“打开文件"，选择sublime text 安装目录中的“sublime_text.exe”
 ![在这里插入图片描述](https://github.com/hfshaobing/picx-images-hosting/raw/master/20230824/8c17dc04c4c64055bc801e7113498336.2eb1qmgwdyv4.webp)
- 在搜索框中输入807805000f94c1，找到后，替换为c64005014885c9
![在这里插入图片描述](https://github.com/hfshaobing/picx-images-hosting/raw/master/20230824/9fd749b60b2e40f4b49af2f75f3c37e8.12m8a09e6hu8.webp)

- 点击“另存为”，保存文件到本地，文件名设定为sublime_text.exe
- 备份原sublime_text.exe文件（修改为如sublime_text_bk.exe）
- 将新保存的sublime_text.exe复制到原sublime text 4安装目录中

## mac
- 下载并安装APP“Hex Fiend”，Hex Fiend, a fast and clever hex editor for macOS
- 打开Sublime Text所在的目录：
- Sublime Text -> (右键)Show Package ->Contents -> Mac OS -> Sublime_text
![在这里插入图片描述](https://github.com/hfshaobing/picx-images-hosting/raw/master/20230824/9fb737b0bf9844e8be5590539ded60ea.9wninzb9jgw.webp)

- 打开Hex Fiend，将Sublime_text拖至Hex Fiend中
- 在搜索框中输入807805000f94c1，找到后，替换为 c64005014885c9
![在这里插入图片描述](https://github.com/hfshaobing/picx-images-hosting/raw/master/20230824/dcfc4dbc79e943e19d3747a5a7da392f.3wq368nxf1g0.webp)
- 保存后，退出
- 在Terminal下，进入Sublime Text目录执行如下语句
![在这里插入图片描述](https://github.com/hfshaobing/picx-images-hosting/raw/master/20230824/4a49a7d1af0041579174b28adc734815.33n0sf2muqy0.webp)
```sh
codesign --remove-signature Sublime\ Text.app/
```
- 打开sublime Text，激活成功。

相关软件下载地址：
百度云盘链接:[https://pan.baidu.com/s/1m_bKaPyxGmV-byDUSF47TQ](https://pan.baidu.com/s/1m_bKaPyxGmV-byDUSF47TQ)
提取码: x2ga



# 使用

## 查寻文件快捷键

```sh
ctrl + p
```

## 分屏

```sh
shift + alt + 2
# 2表示分两个栏
```

## 在控制器里找方法

```sh
ctrl + r
```

## 折叠代码块

```sh
# “ctrl + 1” 中的数字代码折叠级数
# 按住ctrl不松手，按下 k 松开，再按下 1
ctrl+k,ctrl+1
```

## 安装phpmd工具

phpmd工具可以用来查看php文件中哪些变量没有用到

**安装phpmd**

```sh
composer global require phpmd/phpmd
```

**创建自定义 Build System**

1. 打开 Sublime Text，点击 `Tools > Build System > New Build System`。

2. 在新打开的文件中，添加如下内容：

   ```
   {
       "cmd": ["cmd","/c","phpmd", "$file", "text", "cleancode,codesize,unusedcode"],
       "file_regex": "^(...*?):([0-9]*):?([0-9]*)",
       "selector": "source.php"
   }
   ```

3. 保存文件为 `phpmd.sublime-build`。

**使用 Build System 检查未使用的变量**

1. 打开 PHP 文件。
2. 点击 `Tools > Build System`，选择 `phpmd`。
3. 按 `Ctrl+B` 来运行 `phpmd`，会在 Sublime Text 下方的控制台中显示检测结果。

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240911/Snipaste_2024-09-11_16-30-38.5jzsx9lizbk0.webp)

## 恢复关闭的标签页

```sh
ctrl + shift + t
```



## 自定义插件

### 注释插件

Sublime Text 本身的代码片段 (`.sublime-snippet`) 语法没有内置功能来动态获取当前时间，因此不能直接在代码片段中插入自动更新的时间。不过，有一些变通的方案可以在 Sublime Text 中实现通过其他方式插入时间：

### 步骤 1: 创建自定义 Python 插件

1. 打开 `Tools` -> `Developer` -> `New Plugin...`。

2. 删除默认的代码，并粘贴以下代码：

   ```python
   import datetime
   import sublime
   import sublime_plugin
   
   class InsertCommentWithTimestampCommand(sublime_plugin.TextCommand):
       def run(self, edit):
           # 获取当前时间并格式化为指定格式
           timestamp = datetime.datetime.now().strftime("%Y/%m/%d %H:%M")
           # 插入注释模版，包含当前时间
           template = "/**\n 	* \n 	* @param \n 	* @return \n 	* {} By 爱拆东西的程序员\n 	*/".format(timestamp)
           self.view.insert(edit, self.view.sel()[0].begin(), template)
   ```

3. 保存文件，命名为 `insert_comment_with_timestamp.py`，并放置到 `Packages/User/` 目录下。

### 步骤 2: 设置键绑定

现在需要设置键绑定，让这个插件通过快捷键触发：

1. 点击 `Preferences` -> `Key Bindings`。

2. 在右侧的用户键绑定文件中添加以下代码：

   ```json
   [{
       "keys": ["ctrl+alt+/"],
       "command": "insert_comment_with_timestamp"
   }]
   ```

3. 这样，就可以通过 `Ctrl + Alt + / 快捷键来插入带有当前时间的注释。

### 使用效果：

当按下 `Ctrl + Alt + /，Sublime Text 会自动插入类似以下的注释：

```php
/**
 * 
 * @param 
 * @return 
 * 2024/10/14 09:02 By 爱拆东西的程序员
 */
```

