---
title: Git 学习记录
date: 2022-09-06 09:26:12
categories: 版本控制
tags:
- Git
toc: true
---
Git 是目前世界上最先进、最流行的分布式版本控制系统。Git 采用 C 语言开发，并完全免费开源，由 [Linus Torvalds](https://zh.wikipedia.org/wiki/林纳斯·托瓦兹) 发起，并作为主要开发者。他同时还是 Linux 内核的最早作者，担任 Linux 内核的首要架构师与项目协调者。
<!--more-->

## 配置用户
安装完 Git 后一般要配置用户名和邮箱，以便在每次提交中记录下来，方便查找每次提交的用户。Git 的配置一共有三个级别：system（系统级）、global（用户级）、local（版本库）。system 的配置整个系统只有一个，global 的配置每个账号只有一个，local 的配置存在于 Git 版本库中，可以对不同的版本库配置不同的 local 信息。这三个级别的配置是逐层覆盖的关系，当用户提交修改时，首先查找 system 配置，其次查找 global 配置，最后查找 local 配置，逐层查找的过程中，若查询到配置信息，则会覆盖上一层配置，记录在提交记录中。
当有多个账号信息时，为了区分不同账户提交的记录。可以配置 global 为常用的用户和邮箱信息。对于不常用的，可以在对应的版本库里配置单独的用户和邮箱信息。
```bash
git config --global user.name "username"
git config --global user.email "email address"
```
## 创建版本库
1. 新建一个空文件夹，并切换至目录下
```bash
mkdir test_git
cd test_git
```
2. 初始化版本库，此后该目录下的所有文件都将被 Git 管理
```bash
git init
```
## 添加并提交文件到版本库
新建/修改/删除文件的行为都可以被 Git 管理。
1. 对文件进行了以上操作后，将所有的文件变动添加至 Git 暂存区，用于后续提交到版本库
```bash
git add .
```
2. 提交到版本库
```bash
git commit -m '提交信息'
```
## 版本管理
### 概念解释
#### 工作区
工作区是指用户新建的可见目录，其下存放着用户自己创建和修改的工作文件。
#### 版本库
版本库就是使用 `git init` 创建出 `.git` 隐藏目录，称为 Git 版本库。
Git 的版本库里存了很多东西，其中最重要的就是称为 stage（或者叫 index）的**暂存区**，还有 Git 为我们自动创建的第一个分支 master，以及指向 master 的一个指针叫 HEAD。
当用户完成文件修改后，使用 `git add` 命令就可以将文件变动添加至 Git 暂存区，如果用户发现不想添加本次修改，可以使用 `git checkout --<file>` 撤销指定文件的添加。此时还没有生成新的版本库，如果确认添加无误，使用 `git commit` 提交本次所有修改，生成新的版本库，并且清空所有暂存区的文件变动。`git status` 可以时刻观察当前仓库的状态，`git log` 可以查看每次 `commit` 的相关信息。提交后，用 `git diff HEAD -- <file>` 命令可以查看工作区和版本库里面最新版本的区别。

![Git 概念图](/img/git_1.png)
###  撤销修改
把文件在工作区的修改全部撤销，让这个文件回到最近一次 `git commit` 或 `git add` 时的状态
```bash
git checkout -- <file>
```
在 `git add`后，把暂存区的修改撤销掉（unstage），重新放回工作区
```bash
git reset HEAD <file>
```
总结：
- 场景 1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令 `git checkout -- <file>`。
- 场景 2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令 `git reset HEAD <file>`，就回到了场景 1，第二步按场景 1 操作。
### 删除文件
当删除工作区的文件时，工作区和版本库的文件就不一致了，`git status` 命令会立刻告诉你哪些文件被删除了。
现在你有两个选择，一是确实要从版本库中删除该文件，那就用命令 `git rm` 删掉，并且 `git commit`：
```bash
git rm <file>
git commit -m "remove file"
```
另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：
```bash
git checkout -- <file>
```
`git checkout` 其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。
### 版本退回
退回到上个版本
```bash
git reset --hard HEAD^
```
退回到指定版本
```bash
# 查看版本号
git log
# 退回到指定版本
git reset --hard <commit id>
```
当找不到目标 `commit id` 时，Git 提供了一个命令 `git reflog` 用来记录你的每一次命令：
```bash
git reflog
```
总结：
- HEAD 指向的版本就是当前版本，因此，Git 允许我们在版本的历史之间穿梭，使用命令 `git reset --hard <commit id>`。
- 穿梭前，用 `git log` 可以查看提交历史，以便确定要回退到哪个版本。
- 要重返未来，用 `git reflog` 查看命令历史，以便确定要回到未来的哪个版本。
## 远程仓库
### 添加远程库
以 GitHub 为例，创建一个空的 GitHub 仓库，然后将此仓库添加至本地远程库：
```bash
git remote add origin git@github.com:imaginefish/blog.git
```
这里远程库的名字就是 `origin`，这是 Git 默认的叫法，也可以改成别的。
### 推送本地库到远程库
```
git push -u origin main
```
以上命令会把本地的 `main` 分支推送到远程库。

由于远程库是空的，我们第一次推送 mian 分支时，加上了`-u` 参数，Git 不但会把本地的 main 分支内容推送的远程新的 mian 分支，还会把本地的 main 分支和远程的 main 分支关联起来，在以后的推送或者拉取时就可以简化命令，之后推送就可以省略 `-u` 参数。
### 查看远程库
```bash
git remote -v
```
### 删除远程库
```bash
git remote rm origin
```
### clone 远程库
Git 支持 `ssh` 和 `https` 等协议，`ssh` 协议速度快，`https` 速度慢，并且每次推送都必须输入口令，但是出于安全考虑，有些网络环境下没有开放 `ssh 22` 端口，则只能使用 `https` 协议。
```bash
git clone git@github.com:imaginefish/blog.git
```
## 分支管理
### 创建与合并分支
创建分支：
```bash
git branch main
```
切换分支：
```bash
git checkout main
# 新版命令
git switch main
```
创建并切换分支（替代以上两条命令）：
```bash
git checkout -b main
# 新版命令
git switch -c main
```
查看当前分支：
```bash
git branch
```
合并指定分支到当前分支：
```bash
git merge <name>
```
删除分支：
```bash
git branch -d main
```
### 分支冲突
- 当被合并分支的修改内容与当前分支不一致时，合并分支会出现分支冲突。
- 当 Git 无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。
- 解决冲突就是把 Git 合并失败的文件手动编辑为我们希望的内容，再提交。
- 用 `git log --graph` 命令可以看到分支合并图。
### Rebase
- `git rebase` 操作可以把本地未 push 的分叉提交历史整理成直线。
- `git rebase` 的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。
## 使用 GitHub
- 在 GitHub 上，可以自己创建自己的公开和私有仓库
- 可以任意 Fork 开源仓库
- 自己拥有 Fork 后的仓库的读写权限
- 可以推送 pull request 给官方仓库来贡献代码

将本机生成的公钥内容添加至个人的 GitHub 账户 `SSH Keys` 中，便能实现本地访问 GitHub 仓库，并使用以下命令测试连接是否成功：
```bash
ssh -T git@github.com
```
## 搭建 Git 服务器
一般在公司内部，还会搭建 Git 服务器，托管公司自己的代码，提升访问速度和安全性，防止代码泄露。
1. 安装 `git`
```bash
sudo apt-get install git
```
2. 创建 `git` 用户，用于运行 `git` 服务
```bash
sudo adduser git
```
3. 创建 ssh 证书登录
创建 SSH Key：
```bash
ssh-keygen -t rsa -C "youremail@example.com"
```
之后可以在用户主目录里找到 `.ssh` 目录，里面有 `id_rsa` 和 `id_rsa.pub` 两个文件，这两个就是 `SSH Key` 的秘钥对，`id_rsa` 是私钥，不能泄露出去，`id_rsa.pub` 是公钥，可以放心地告诉任何人。
收集所有需要登录的用户的公钥，把所有公钥导入到 `/home/git/.ssh/authorized_keys` 文件里，一行一个。
4. 初始化 Git 仓库
```bash
git init --bare test.git
```
Git 就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的 Git 仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的 Git 仓库通常都以 .git 结尾。然后，把 owner 改为 git：
```bash
sudo chown -R git:git test.git
```
5. 禁用 bash 登录
出于安全考虑，第二步创建的 git 用户不允许登录 bash，这可以通过编辑 `/etc/passwd` 文件完成。找到类似下面的一行：
```
git:x:1001:1001:,,,:/home/git:/bin/bash
```
改为：
```
git:x:1001:1001:,,,:/home/git:/usr/bin/git-bash
```
这样，git 用户可以正常通过 ssh 使用 git，但无法登录 bash，因为我们为 git 用户指定的 git-bash 每次一登录就自动退出。
6. 克隆远程仓库
```bash
git clone git@server:/xxx/test.git
```