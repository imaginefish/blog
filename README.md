# blog
个人博客，利用 [`Hexo`](https://hexo.io) 构建，使用 [Icarus](https://ppoffice.github.io/hexo-theme-icarus/) 主题。
## 如何使用
1. Git 克隆到本地
```shell
git clone git@github.com:imaginefish/blog.git
```
2. 进入仓库目录下，使用 npm 安装 node modules
```shell
cd blog
npm install
```
3. 使用 hexo_cli 进行写作和发布
```shell
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
```shell
git add .
git commit -m 'some messages'
git push origin main
```