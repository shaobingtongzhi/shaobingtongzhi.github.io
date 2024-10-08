---
title: Git 相关知识
date: 2023-08-10 20:19:48
tags: 
	- git
categories:
	- 学习笔记
	- 07-Git相关笔记
---



# Git 相关知识

## git常用命令

### git add 命令

#### 将修改文件提交到暂存区

- git add 

例如：

```SH
# 将所有文件提交到暂存区
git add .

# 将指定文件提交暂存区
git add a.txt
```

### git restore 命令

#### 撤销提交到暂存区的文件

```sh
git restore --staged add.txt
```



### git remote 命令

#### 显示所有远程仓库

- git remote -v 显示所有远程仓库

例如：

```sh
$ git remote -v
origin	https://gitee.com/hf_app/spring-shop.git (fetch)
origin	https://gitee.com/hf_app/spring-shop.git (push)
```

这里 origin 为远程地址的别名

#### 显示某个远程仓库

- git remote show xxx（xxx为远程地址的别名）显示某个远程仓库的信息

例如：

```sh
$ git remote show origin
* remote origin
  Fetch URL: https://gitee.com/hf_app/spring-shop.git
  Push  URL: https://gitee.com/hf_app/spring-shop.git
  HEAD branch: master
  Remote branches:
    dev    tracked
    master tracked
  Local branches configured for 'git pull':
    dev    merges with remote dev
    master merges with remote master
  Local refs configured for 'git push':
    dev    pushes to dev    (up to date)
    master pushes to master (up to date)
    
    
```

#### 添加远程版本库

- git remote add [name] [url] 作用是添加远程版本库

name 是自己取的仓库的名字 

url 是地址

> 这个也是经常用到的，添加之后 一般都是使用 git fetch --all 拉取代码
>
> 然后 git push name HEAD:refs/for/分支名  提交代码
>
> 这里的 name 就是刚才取的名字

#### 删除远程仓库

- git remote rm name 删除远程仓库

#### 修改仓库名

- git remote rename old_name new_name 修改仓库名

### git branch 命令

#### 查看分支

```sh
# 查看本地分支
git branch

# 查看远程分支
git branch -r

# 查看所有分支（远程+本地）
git branch -a
```

#### 追踪远程分支

```sh
git branch 本地分支名称 --track remotes/origin/source
```

### git checkout 命令

#### 切换分支

```sh
git checkout [branch name]
```

#### 创建新分支并切换到新分支

```sh
git checkout -b [branch name]
# 从指定分支创建新分支
# 1.切换到指定分支
# 2.拉取代码
# 3.创建分支
# 4.推送

git checkout --orphan [branch name]
```

#### 删除分支内的所有内容

```sh
git rm -rf .
```



### git merge 命令

```sh
# 比如从master 合并到 dev
# 1. 切换到 master
git checkout master
# 2. 拉取代码到最新
git pull
# 3. 切换到 dev
git checkout dev
# 4. 合并，可以用 git status 查看合并状态检查是否有冲突
git merge master
# 5. 提交
git push
```

#### 恢复文件，撤销修改

```sh
# 恢复所有文件，撤销所有修改
git checkout .

# 恢复某个文件，如a.txt
git checkout a.txt
```

### git stash 贮藏

#### 贮藏

```sh
# 保存当前修改不提交，且仓库状态恢复至上一次提交
git stash

# 实际应用中推荐给每个stash加一个message，用于记录版本，使用git stash save取代git stash命令
git stash save "test-stash"
```

#### 应用贮藏

```sh
# 将缓存堆栈中的第一个stash删除，并将对应修改应用到当前的工作目录下
git stash pop
# 将缓存堆栈中的stash多次应用到工作目录中，但并不删除stash拷贝
git stash apply
```

#### 查看现有贮藏

```sh
git stash list
# 结果示例
stash@{0}: WIP on main: ff1853b aaa
stash@{1}: On main: hexo配置
```

#### 移除贮藏

```sh
git stash drop stash@{1}
```

#### 查看贮藏里具体内容

```sh
git stash show -p 
```

#### 清空贮藏

```sh
git stash clear
```

### git diff 比较

#### 比较差异

```sh
git diff [文件]
```

## git配置

```sh
git config -l
#列出配置文件中设置的所有变量及其值

git config -e 
#  Opens an editor to modify the specified config file; either --system, --global, or repository (default).
#  打开编辑器以修改指定的配置文件；系统、全局或存储库（默认）。

```

### 设置代理

```sh
# 代理地址是可用的才行
git config http.proxy socks5://127.0.0.1:1081
git config https.proxy socks5://127.0.0.1:1081

# 取消代理
git config unset http.proxy
git config --local unset http.proxy
git config --global unset http.proxy
```



## .gitignore文件

### 作用

在任何当前工作的 Git 仓库中，每个文件都是这样的：

- **追踪的（tracked）**- 这些是 Git 所知道的所有文件或目录。这些是新添加（用 `git add` 添加）和提交（用 `git commit` 提交）到主仓库的文件和目录。
- **未被追踪的（untracked）** - 这些是在工作目录中创建的，但还没有被暂存（或用 `git add` 命令添加）的任何新文件或目录。
- **被忽略的（ignored）** - 这些是 Git 知道的要全部排除、忽略或在 Git 仓库中不需要注意的所有文件或目录。本质上，这是一种告诉 Git 哪些未被追踪的文件应该保持不被追踪并且永远不会被提交的方法。

所有被忽略的文件都会被保存在一个 `.gitignore` 文件中。

`.gitignore` 文件是一个纯文本文件，包含了项目中所有指定的文件和文件夹的列表，这些文件和文件夹是 Git 应该忽略和不追踪的。

在 `.gitignore` 中，你可以通过提及特定文件或文件夹的名称或模式来告诉 Git 只忽略一个文件或一个文件夹。你也可以用同样的方法告诉 Git 忽略多个文件或文件夹。

### 创建

通常，一个 `.gitignore` 文件会被放在仓库的根目录下。根目录也被称为父目录和当前工作目录。根目录包含了组成项目的所有文件和其他文件夹。

也就是说，你可以把它放在版本库的任何文件夹中。你甚至可以有多个 `.gitignore` 文件。

要在基于 Unix 的系统（如 macOS 或 Linux）上用命令行创建一个 `.gitignore` 文件，打开终端程序（如 macOS 上的 Terminal.app）。然后，用 `cd` 命令导航到包含项目的根文件夹，并输入以下命令，为你的目录创建一个 `.gitignore` 文件：

```bash
touch .gitignore
```

名字前面有点（`.`）的文件默认是隐藏的。

当单独使用 `ls` 命令时，隐藏的文件是不可见的。要从命令行查看所有的文件--包括隐藏的文件--请在 `ls` 命令中使用 `-a` 标志，如图所示：

```sh
ls -a
```

***在 .gitignore 文件中应包括什么？***

你应该考虑添加到 `.gitignore` 文件中的文件类型是任何不需要被提交的文件。

你可能出于安全原因不想提交它们，或者因为它们是你的本地文件，因此对与你在同一项目上工作的其他开发者来说是不必要的。

其中一些可能包括：

- 操作系统文件。每个操作系统（如 macOS、Windows 和 Linux）都会生成系统特定的隐藏文件，其他开发者不需要使用这些文件，因为他们的系统也会生成这些文件。例如，在 macOS 上，Finder 会生成一个 `.DS_Store` 文件，其中包括用户对文件夹的外观和显示的偏好，如图标的大小和位置。
- 由代码编辑器和 IDE（IDE 代表集成开发环境）等应用程序生成的配置文件。这些文件是为你、你的配置和你的偏好设置定制的。
- 从你的项目中使用的编程语言或框架自动生成的文件，以及编译后的代码特定文件，如 `.o` 文件。
- 由软件包管理器生成的文件夹，如 npm 的 `node_modules` 文件夹。这是一个用于保存和跟踪你在本地安装的每个软件包的依赖关系的文件夹。
- 包含敏感数据和个人信息的文件。这类文件的一些例子是含有你的凭证（用户名和密码）的文件和含有环境变量的文件，如 `.env` 文件（`.env` 文件含有需要保持安全和隐私的 API 密钥）。
- 运行时文件，如 `.log` 文件。它们提供关于操作系统的使用活动和错误的信息，以及在操作系统中发生的事件的历史。

### 设置忽略

如果你想只忽略一个特定的文件，你需要提供该文件在项目根目录下的完整路径。

例如，如果你想忽略位于根目录下的 `text.txt` 文件，你可以做如下操作：

```bash
/text.txt
```

而如果你想忽略一个位于根目录下的 `test` 目录中的 `text.txt` 文件，你要做的是：

```bash
/test/text.txt
```

你也可以这样写上述内容：

```bash
test/text.txt
```

如果你想忽略所有具有特定名称的文件，你需要写出该文件的字面名称。

例如，如果你想忽略任何 `text.txt` 文件，你可以在 `.gitignore` 中添加以下内容：

```bash
text.txt
```

在这种情况下，你不需要提供特定文件的完整路径。这种模式将忽略位于项目中任何地方的具有该特定名称的所有文件。

要忽略整个目录及其所有内容，你需要包括目录的名称，并在最后加上斜线 `/`：

```bash
test/
```

这个命令将忽略位于你的项目中任何地方的名为 `test` 的目录（包括目录中的其他文件和其他子目录）。

需要注意的是，如果你只写一个文件的名字或者只写目录的名字而不写斜线 `/`，那么这个模式将同时匹配任何带有这个名字的文件或目录：

```bash
# 匹配任何名字带有 test 的文件和目录
test
```

如果你想忽略任何以特定单词开头的文件或目录怎么办？

例如，你想忽略所有名称以 `img` 开头的文件和目录。要做到这一点，你需要指定你想忽略的名称，后面跟着 `*` 通配符选择器，像这样：

```bash
img*
```

这个命令将忽略所有名字以 `img` 开头的文件和目录。

但是，如果你想忽略任何以特定单词结尾的文件或目录呢？

如果你想忽略所有以特定文件扩展名结尾的文件，你需要使用 `*` 通配符选择器，后面跟你想忽略的文件扩展名。

例如，如果你想忽略所有以 `.md` 文件扩展名结尾的 markdown 文件，你可以在你的 `.gitignore` 文件中添加以下内容：

```bash
*.md
```

这个模式将匹配位于项目中任何地方的以 `.md` 为扩展名的任何文件。

前面，你看到了如何忽略所有以特定后缀结尾的文件。当你想做一个例外，而有一个后缀的文件你不想忽略的时候，会发生什么？

假设你在你的 `.gitignore` 文件中添加了以下内容：

```bash
.md
```

这个模式会忽略所有以 `.md` 结尾的文件，但你不希望 Git 忽略一个 `README.md` 文件。

要做到这一点，你需要使用带有感叹号的否定模式，即 `!`，来排除一个本来会被忽略的文件：

```bash
# 忽略所有 .md 文件
.md

# 不忽略 README.md 文件
!README.md
```

在 `.gitignore` 文件中使用这两种模式，所有以 `.md` 结尾的文件都会被忽略，除了 `README.md` 文件。

需要记住的是，如果你忽略了整个目录，这个模式就不起作用。

例如，你忽略了所有的 `test` 目录：

```bash
test/
```

假设在一个 `test` 文件夹内，你有一个文件，`example.md`，你不想忽略它。

你不能像这样在一个被忽略的目录内排除一个文件：

```bash
# 忽略所有名字带有 test 的目录
test/

# 试图在一个被忽略的目录内排除一个文件是行不通的
!test/example.md
```

### 补充

***<font color=red>如何忽略以前提交的文件</font>***

当你创建一个新的仓库时，最好的做法是创建一个 `.gitignore` 文件，包含所有你想忽略的文件和不同的文件模式--在提交之前。

Git 只能忽略尚未提交到仓库的未被追踪的文件。

如果你过去已经提交了一个文件，但希望没有提交，会发生什么？

比如你不小心提交了一个存储环境变量的 `.env` 文件。

<font color=red>第一步：你首先需要更新 `.gitignore` 文件以包括 `.env` 文件</font>：

```bash
# 给 .gitignore 添加 .env 文件
echo ".env" >> .gitignore
```

<font color=red>第二步：现在，你需要告诉 Git 不要追踪这个文件，把它从索引中删除：</font>

```bash
git rm --cached .env
```

`git rm` 命令，连同 `--cached` 选项，从版本库中删除文件，但不删除实际的文件。这意味着该文件仍然在你的本地系统和工作目录中作为一个被忽略的文件。

`git status` 会显示该文件已不在版本库中，而输入 `ls` 命令会显示该文件存在于你的本地文件系统中。

如果你想从版本库和你的本地系统中删除该文件，省略 `--cached` 选项。

<font color=red>第三步：接下来，用 `git add` 命令将 `.gitignore` 添加到暂存区：</font>

```bash
git add .gitignore
```

<font color=red>第四步：最后，用 `git commit` 命令提交 `.gitignore` 文件：</font>

```bash
git commit -m "update ignored files"
```

***如何检查被忽略的文件是被.gitignore中的哪一行生效的***

```sh
# 例如查询 Readme.md是被哪一行生效的
git check-ignore -v Readme.md
```

## 其它

***git中文件名中文乱码问题***

```sh
 git config --global core.quotepath false
```

# 问题汇总

## git status 时有Untracked files 的文件，原因分析及解决方案

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240606/Snipaste_2024-04-03_08-46-39.6lzv1e1c30o0.1xsx05m3cbgg.jpg)

### 原因分析

我们要真正弄明白问题的原因，我们就要先知道文件的几个状态。
git在未 **commit** 之前有三种状态：

1. Untracked files 未跟踪
2. Changes not staged for commit 未提交的更改
3. Changes to be committed 提交的更改

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240606/Snipaste_2024-04-03_08-50-19.1tcm818hkshs.1ii0dug5j75s.jpg)

什么文件会是未跟踪的呢？
那些新创建的或者从未add过的文件就是未跟踪的。
此时有几种情况：

**<font color=red>1. 我们创建了准备提交上去的，这种好办只要add了就可以了</font>**。

**<font color=red>2. 必须放在git工具目录中，但又不能提交的，比如保存了数据库密码的配置文件等的东西。</font>**

**<font color=red>3. 我们不准备提交又没用的。</font>**

### 问题解决

#### 第一种情况

直接add

```sh
git add xxx.txt
```

#### 第二种情况

在git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，git就会自动忽略这些文件

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240606/Snipaste_2024-04-03_08-56-07.7fejp4587tw0.27t72rk5jdhc.jpg)

.gitignore 文件

```
personal_tax.access_log
personal_tax.error_log
~$管理大屏显示接口文档.docx
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240606/企业微信截图_20240403085725.1ichvmdhnhls.4nxicsakg360.jpg)

#### 第三种情况

我们不需要就删除就可以了：clean
git clean 是从你的工作目录种删除所有没有tracked(未跟踪)过的文件。
要知道这个命令很危险，删除了就找不到了。但是如果已经git add . 就不会被删除。
参数说明：
n：显示将要被删除的文件以及目录
d：删除未被添加到git路径中的文件以及目录（将.gitignore文件标记的文件全部删除）。
f：强制执行（只会删除文件）
x：删除没有被track的文件

**使用**

```sh
## 是一个演习，告诉你那些文件会被删除，不会真正删除
git clean -n
## 删除当前目录下所有没有track过的文件，.gitignore文件里指定的不会删除。
git clean -f
## 删除指定路径下的没有被track过的文件
git clean -f <path>
## 强制删除所有没有被track过的文件和文件夹，
git clean -df
## 强制删除所有没有被track过的文件（.gitignore文件里指定的也不能避免）
git clean -fx

```

## 如何将本地项目推送到远程仓库

第一步：把在本地项目文件夹下执行 git init 

第二步：设置忽略文件内容 .gitignore

第三步：执行 git add .  

第四步：git commit -m "初始化msg"

第五步：**在gitee 或者github 建立远程仓库**

第六步：将本地仓库和远程仓库建立关联（git remote add origin https://gitee.com/xxx/xxx.git）

第七步：推送到远程仓库（git push -u origin master）

## git clone 报错：error: xxxx bytes of body are still expected

可能原因：项目过大

解决办法：

```sh
git config --global http.lowSpeedLimit 0 

git config --global http.lowSpeedTime 999999 
```

