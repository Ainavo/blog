---
title: Git 的一些常用命令~
date: 2023-07-24 11:31:55
tags:
  - git
  - github
---

![Git 常用命令速查表](/imgs/Git%20相关操作/Git%20常用命令速查表.png)

<!--more-->

## 什么是 Git ？

Git 是一个分布式版本控制系统，用于跟踪文件的变化并协调多人之间的文件更改。它允许多个用户在不同的分支上独立工作，并能够合并不同分支上的更改。Git 可以存储文件的完整历史记录，方便回溯和恢复文件的不同版本。它广泛用于软件开发中，特别是用于团队合作开发和开源项目。通过使用 Git，开发人员可以更好地管理和追踪代码的变化，确保团队成员之间的协作无缝进行。(来自 ChatGPT)

## Git 的安装

- Git 支持 Linux/Unix、macOS、Windows 平台使用，安装包地址：http://git-scm.com/downloads
- Git 安装之后，在终端输入 git version 或 git --version 查看版本信息（顺便确保安装成功）。

## Git 的配置

- 设置个人用户名和电子邮件

```bash
  $ git config --global user.name "miaomiaomiao"  # 设置个人用户名
  $ git config --global user.email miaomiao@163.com  # 设置电子邮件
```

- 设置提交文件大小的上限

```bash
  git config http.postBuffer 524288000
```

- Git 查看已有配置

```bash
  git config --list
  # http.postbuffer=2M   // 这个什么
  # user.name=miaomiaomiao
  # user.email=miaomiaomiao@163.com
```

## Git 的工作流程和原理

（待补充~）

## Git 创建和克隆仓库

- git init

```bash
  git init  # 初始化空的 Git 存储库
  git init miaomiao # 指目录作为 git repo
```

- git clone xxxx

```bash
  git clone https://github.com/yutto-dev/yutto.git  # 想 clone 到本地的 repo，以一个B站下载器为例
```

![repo 链接如下](/imgs/Git%20相关操作/repo%20链接.png)

## Git 基本操作

- git init 初始化仓库
- git add . 添加所有文件到暂存区
- git commit -m "xxxx" 添加 message 将暂存区提交到本地仓库，形成一个新的版本，生成一个指定 commit ID 标识号

```bash
  git commit -a # 不需要 git add 直接提交
```

- git status 查看仓库当前状态，显示有变更的文件
- git diff 比较文件的不同，即暂存区和工作区的差异
- git reset 回退版本

```bash
  git reset HEAD^ # 回退到上一个版本
  git reset HEAD^2 # 回退到上上一个版本
  git reset --hard HEAD^ #（强）回退到上一个版本，会删除之前所有的提交信息

  git reset  d4d0b27adcc07b4c1af2aa945157b196b98c8feb # 回退到指定版本号
```

![commit记录](/imgs/Git%20相关操作/提交记录.png)
![指定commit id](/imgs/Git%20相关操作/指定commit%20id.png)

- git rm 将文件从暂存区和工作区中删除
- git mv 移动或重命名工作区文件
- git remote 用于管理远程仓库
  - git remote：列出当前仓库中的远程仓库
  - git remote -v：列出当前仓库中已配置的远程仓库，并显示它们的 URL
  - git remote add <remote_name> <remote_url>：添加一个新的远程仓库，指定一个远程仓库的名称和 URL，将其添加到当前的仓库中。
  - git remote rename <old_name> <new_name>：将已配置的远程仓库重命名。
  - git remote remove <remote>：从当前仓库删除指定远程仓库
  - git remote set-url <remote_name> <new_url>：修改指定远程仓库的 URL
  - git remote show <remote_name>：显示指定远程仓库的详细信息，包括 URL 和跟踪分支

```bash
  git remote
  git remote -v
  git remote add miaomiao https://github.com/Ainavo/yutto.git
  git remote rename miaomaio wangwang
  git remote remove miaomiao
  git remote set-url miaomiao https://github.com/Ainavo/miaomaio.git
  git remote show miaomiao
```

- git fetch 从远程获取代码

```bash
  git fetch # 从远程获取代码
```

- git pull 拉取并合并远程 repo 的最新代码:git pull = git fetch + git merge

```bash
  git pull <远程主机名> <远程分支名>:<本地分支名>
  git pull origin master:brantest #将远程主机 origin 的 master 分支拉取过来，与本地的 brantest 分支合并

```

- git push 将本地分支推送到远程并合并

```bash
  git push <远程主机名> <本地分支名>:<远程分支名>  # 如果本地分支名和远程分支名相同则可以忽略省略号
  git push --force-with-lease # 强制推送（最小差异）
```

## Git 开发简单流程

- 首先，fork 一个想要贡献代码的 repo，然后 clone 到本地。

```bash
  git clone https://github.com/Ainavo/yutto.git  # 还是以某个B站下载器为例
```

![fork别人repo](/imgs/Git%20相关操作/fork别人repo.png)  
![fork第二步](/imgs/Git%20相关操作/fork第二步.png)  
![repo 链接如下](/imgs/Git%20相关操作/repo%20链接.png)

- git 创建一个新分支

```bash
  git checkout -b miaomiao  # 创建一个新分支叫 miaomiao，至于为什么要创建新分支--因为不能直接在 develop/master 等主分支直接提交
  git switch -c xxxx # 同样创建一个新分支 xxxx，建议用这种
  git branch # 查看所有本地分支
```

- git 提交修改内容

```bash
  git add .  # 添加所有更改内容到暂存区
  git commit -m "miaomiaomiao" # 将暂存区提交到本地仓库，形成一个新的版本，生成一个指定 commit ID 标识号
  git push
```

## 遇到的一些小问题

- 在与别人协作开发时，有时常常会因为别人先合入 repo，产生冲突，对于简单的冲突：

```bash
  # 假设目前处在 miaomiao 分支
  git checkout master  # 以master为例，切入主分支
  git pull # 拉取最新的 develop 代码
  git checkout miaomiao # 切回 miaomiao 分支
  git merge master # 合并 master 分支
  git push --force-with-lease # 强制推送最小改变到远程分支
```

- 某些冲突太多，不能直接解决冲突(直接重组)

```bash
  git   # 假设目前处在 miaomiao 分支
  git checkout master  # 以master为例，切入主分支
  git pull # 拉取最新的 develop 代码
  git checkout miaomiao # 切回 miaomiao 分支
  git reset --hard master # 将 miaomiao 分支上的代码回退到 master 上
  git push --force-with-lease # 强制推送最小改变到远程分支
```

## 参考文献

- [Git Book](https://git-scm.com/book/zh/v2)
- [Git 基本操作](https://www.runoob.com/git/git-branch.html)
