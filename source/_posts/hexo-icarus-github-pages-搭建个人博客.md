---
title: Hexo + Icarus + GitHub Pages 搭建个人博客
date: 2022-09-02 17:27:20
categories:
- 个人博客
tags: 
- Hexo
- Icarus 
toc: true
---
## 先决条件
需要先安装以下程序：
- [Node.js](http://nodejs.org) (Node.js 版本需不低于 10.13，建议使用 Node.js 12.0 及以上版本)
- [Git](http://git-scm.com)
<!--more-->
### Node.js 版本限制
强烈建议永远安装最新版本的 Hexo，以及推荐的 Node.js 版本。
|Hexo 版本|最低兼容的 Node.js 版本|
|:--:|:--:|
|6.0+|12.13.0|
|5.0+|10.13.0|
|4.1 - 4.2|8.10|
|4.0|8.6|
|3.3 - 3.9|6.9|
|3.2 - 3.3|0.12|
|3.0 - 3.1|0.10 or iojs|
|0.0.1 - 2.8|0.10|
## 安装 Hexo
使用 npm 全局安装 Hexo。
```shell
npm install -g hexo-cli
```
## 安装 icarus
1. 指定路径初始化博客目录，并切换至该路径下，以blog路径为例
```shell
hexo init blog
cd blog
```
2. 使用 npm 安装 Hexo。
```shell
npm install hexo-theme-icarus
```
3. 配置 Hexo 主题
```shell
hexo config theme icarus
```
## 创建 GitHub 仓库
1. 创建 GitHub 仓库，并开启 ` Environments`，配置 url
2. 配置 ssh，确保可以 ssh 远程访问 GitHub，可以使用以下命令测试连接是否成功：
```shell
ssh -T git@github.com
```
如果出行以下信息，则说明连接成功：
```
Hi imaginefish! You've successfully authenticated, but GitHub does not provide shell access.
```
## 修改配置
### Hexo 配置
Hexo 的配置文件在 `blog` 目录下，名为 `_config.yml`
- 修改语言为中文简体
```yml
language: zh-CN
```
- 修改时区
```yml
timezone: 'Asia/Shanghai'
```
- 修改博客网址，如果不配置会出现文件路径引用错误问题，导致 js、css、图片无法加载
```yml
url: https://imaginefish.github.io/blog
```
- 修改 hexo 部署方式，推送至 Github 仓库的 gh-pages 分支，实现博客部署
```yml
deploy:
  type: git
  repository: git@github.com:imaginefish/blog.git
  branch: gh-pages
```
### Icarus 配置
Icarus 的配置文件在 `blog` 目录下，名为 `_config.icarus.yml`
- 该主题导航栏无法跟随 Hexo 语言本地化，需要手动修改配置文件
```yml
    menu:
        主页: /
        归档: /archives
        分类: /categories
        标签: /tags
        关于: /about
```
- 设置博主邮箱链接
```yml
Email:
    icon: fas fa-envelope
    url: mailto:imaginefishes@outlook.com
```
- 修改 `sidebar` 配置，使 toc 随文章下拉滚动
```yml
sidebar:
    left:
        sticky: true
```
- 用户访问量统计
```yml
busuanzi: true
```
- 布局配置文件
布局配置文件遵循着与主题配置文件相同的格式和定义。 `_config.post.yml` 中的配置对所有文章生效，而 `_config.page.yml` 中的配置对所有自定义页面生效。 这两个文件将覆盖主题配置文件中的配置。
例如，可以在 `_config.post.yml` 中把所有文章变为两栏布局：
```yml
widgets:
    -
        type: toc
        position: left
        index: true
        collapsed: true
        depth: 3
    -
        type: recent_posts
        position: left
    -
        type: categories
        position: left
    -
        type: tags
        position: left
        order_by: name
        amount: 
        show_count: true
```
在 `_config.icarus.yml` 中把其他页面仍保持三栏布局：
```yml
    -
        position: right
        type: recent_posts
    -
        position: right
        type: categories
    -
        position: left
        type: archives
    -
        position: right
        type: tags
        order_by: name
        amount: 
        show_count: true
```
以下给出个人的完整 `_config.icarus.yml` 配置：
```yml
version: 5.1.0
variant: default
logo: /img/logo.svg
head:
    favicon: /img/favicon.svg
    manifest:
        name:
        short_name: 
        start_url: 
        theme_color: 
        background_color: 
        display: standalone
        icons:
            -
                src: ''
                sizes: ''
                type: 
    structured_data:
        title: 梦鱼乡
        description: 个人博客
        url: https://github.com/imaginefish/blog
        author: 梦鱼
        publisher: 
        publisher_logo: 
        image: 
    meta:
        - ''
    rss: /atom.xml
navbar:
    menu:
        主页: /
        归档: /archives
        分类: /categories
        标签: /tags
        关于: /about
    links:
        GitHub:
            icon: fab fa-github
            url: https://github.com/imaginefish/blog
footer:
    links:
        Creative Commons:
            icon: fab fa-creative-commons
            url: https://creativecommons.org/
        Attribution 4.0 International:
            icon: fab fa-creative-commons-by
            url: https://creativecommons.org/licenses/by/4.0/
        Download on GitHub:
            icon: fab fa-github
            url: https://github.com/ppoffice/hexo-theme-icarus
article:
    highlight:
        theme: atom-one-light
        clipboard: true
        fold: unfolded
    readtime: true
    update_time: true
    licenses:
        Creative Commons:
            icon: fab fa-creative-commons
            url: https://creativecommons.org/
        Attribution:
            icon: fab fa-creative-commons-by
            url: https://creativecommons.org/licenses/by/4.0/
        Noncommercial:
            icon: fab fa-creative-commons-nc
            url: https://creativecommons.org/licenses/by-nc/4.0/
search:
    type: insight
    index_pages: true

sidebar:
    left:
        sticky: true
    right:
        sticky: false
widgets:
    -
        position: left
        type: profile
        author: 梦鱼
        author_title: 大数据开发工程师
        location: 中国.上海
        avatar: /img/avatar.jpg
        avatar_rounded: true
        gravatar: 
        follow_link: https://github.com/imaginefishes
        social_links:
            Github:
                icon: fab fa-github
                url: https://github.com/imaginefishes
            Email:
                icon: fas fa-envelope
                url: mailto:imaginefishes@outlook.com
            RSS:
                icon: fas fa-rss
                url: /atom.xml
    -
        position: left
        type: toc
        index: true
        collapsed: true
        depth: 3
    -
        position: left
        type: links
        links:
            Hexo: https://hexo.io
            Icarus: https://ppoffice.github.io/hexo-theme-icarus
    -
        position: right
        type: recent_posts
    -
        position: right
        type: categories
    -
        position: left
        type: archives
    -
        position: right
        type: tags
        order_by: name
        amount: 
        show_count: true
plugins:
    animejs: true
    back_to_top: true
    busuanzi: true
    cookie_consent:
        type: info
        theme: edgeless
        static: false
        position: bottom-left
        policyLink: https://www.cookiesandyou.com/
    gallery: true
    katex: true
    mathjax: true
    outdated_browser: false
    progressbar: true
providers:
    cdn: jsdelivr
    fontcdn: google
    iconcdn: fontawesome
```
## 其他问题
- 文章图片插入问题
若要插入本地图片，在博客根目录下找到 `source` 文件夹，在其下创建 `img` 子目录，将图片放置于此，通过 `/img/xxx.jpg` 路径引入。