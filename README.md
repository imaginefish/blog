# Blog
个人博客，利用 [`Hexo`](https://hexo.io) 构建，使用 [`Icarus`](https://ppoffice.github.io/hexo-theme-icarus/) 主题。
## 如何使用
1. Git 克隆到本地
```bash
git clone git@github.com:imaginefish/blog.git
```
2. 安装相关 node modules

使用 npm 全局安装 Hexo
```bash
npm install -g hexo-cli
```
进入仓库目录下，使用 npm 安装 node modules
```bash
cd blog
npm install
```
3. 使用 hexo_cli 进行写作和发布
```bash
# 新建文章
hexo new <title>
# 启动本地服务
hexo server
# 生成静态文件
hexo generate
# Hexo 部署到 GitHub Pages
hexo clean && hexo deploy
```
4. Git 推送到本远程 GitHub 仓库 `main` 分支，更新仓库
```bash
git add .
git commit -m 'some messages'
git push origin main
```