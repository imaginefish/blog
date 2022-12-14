---
title: 本地仓库推送到 GitHub 仓库 main 分支
date: 2022-09-05 19:40:24
categories: 版本控制
tags:
- Git
- GitHub
toc: true
---
>GitHub 目前已经将用户创建的版本库的默认分支从 `master` 更改为了 `main`，详情见[GitHub Change Blog](https://github.blog/changelog/2020-08-26-set-the-default-branch-for-newly-created-repositories)。而 Git 默认创建的还是 `master` 分支，所以在本地仓库同步到 GitHub 时，会出现 `error: failed to push some refs to 'github.com:imaginefish/blog.git'` 报错，需要切换至 `main` 分区后再进行操作。
<!--more-->
1. 初始化版本库
```shell
git init
```
2. 添加远程仓库
```shell
git remote add origin git@github.com:imaginefish/blog.git
```
3. 拉取 GitHub 仓库到本地同步
```shell
git pull --rebase origin main
```
4. 切换本地分支从 `master` 到 `main`
```shell
git checkout main
```
5. 添加所有修改到暂存区
```shell
git add .
```
6. 提交修改到版本库
```shell
git commit -m 'some messages'
```
7. 推送到 GitHub 的 `main` 分支
```shell
git push origin main
```