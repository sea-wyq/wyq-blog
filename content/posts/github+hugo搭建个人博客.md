---
title: "github pages + hugo搭建个人博客"
date: 2022-06-22
categories:
- github pages
- hugo
tags:
- 其他
thumbnailImagePosition: right
thumbnailImage: img/main/th.jpg
---

在windows10上通过github pages和hugo实现个人博客的快速搭建。

<!--more-->

## 1. 安装choco
以管理员身份执行cmd ,终端下执行
```
@powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
choco help
```


## 2. 安装hugo 
```
choco install hugo -confirm
hugo version   #判断是否安装成功
```


## 3. 新建 Hugo 网站 

##### 3.1 新建一个目录，用于存放 Hugo 网站的文件
```
hugo new site blog(网站文件夹名)
cd blog
```
##### 3.2 进入blog站点目录，默认文件目录结构如下：

![Tranquilpeak](/img/main-blog/blog-1/mm.png)
config.toml : 网站的配置文件，可配置网站名称、关键字、插件等。  
content : 文章存放目录。  
themes : 网站主题存放目录。  
static : 静态资源存放目录, 如：图片、样式文件、脚本文件等。  

## 4. 选择 Hugo 主题
打开 Hugo Themes: https://themes.gohugo.io 。  
选择合适的主题，以Tranquilpeak 举例。 将选中的主题下载到thems文件夹中。
```
cd themes
git clone https://github.com/kakawait/hugo-tranquilpeak-theme.git
```

## 5. 配置Hugo站点并本地预览

##### 5.1 将主题 hugo-tranquilpeak-theme 中示例站点 exampleSite 文件夹的内容，全部复制到你的blog站点根目录。  

##### 5.2 修改站点配置文件 config.toml,

修改站点的 baseurl，可修改为你的 Github Pages 地址，如：https://sea-wyq.github.io  
##### 5.3 启动 Hugo server。

```
hugo server -D   #在站点根目录bolg下运行
```
##### 5.4 在浏览器中打开 http://localhost:1313 预览。  


![Tranquilpeak](/img/main-blog/blog-1/mm-1.png)


## 6. 自动化站点部署（github pages）
为了确保个人站点文章（markdown文件）的相对安全，采取将站点源码仓库和站点分开，即：站点源码仓库设置为私有，站点 GitHub Pages 仓库设为公开。  

##### 6.1 新建github仓库用来存放网站源码，如：wyq-blog。
注：wyq-blog为示例仓库，故设置为公开仓库，实践中建议设置为私有仓库。  

![Tranquilpeak](/img/main-blog/blog-1/mm-2.png)


##### 6.2  新建一个 GitHub repository，库名为<username>.github.io，其中 username 是你的 GitHub 账号。例如：sea-wyq.github.io。

![Tranquilpeak](/img/main-blog/blog-1/mm-3.png)


##### 6.3 克隆wyq-blog仓库到本地目录，然后将blog文件下的文件拷贝到仓库根目录中

```
git clone https://github.com/sea-wyq/wyq-blog.git
```
注：1. 将wyq-blog目录下 .gitignore文件中中的thems/删除。
    2. 将wyq-blog\themes\hugo-tranquilpeak-theme目录下的.git文件删除。


##### 6.4 构建 Hugo 网站

在本地仓库（wyq-blog）下执行 hugo 命令构建  

![Tranquilpeak](/img/main-blog/blog-1/mm-4.png)

##### 7.4  创建github token 并在wyq-blog和sea-wyq.github.io 添加secret  

github token创建流程：github用户画像-> Settings->Developer settings ->Personal 
access tokens ->Generate new token  

![Tranquilpeak](/img/main-blog/blog-1/mm-5.png)


secret创建：仓库Settings ->Secret ->actions -> new respository secret

![Tranquilpeak](/img/main-blog/blog-1/mm-6.png)


##### 7.5 利用 Github Actions实现将站点源文件（如：wyq-blog）自动化部署到 GitHub Pages （如：sea-wyq.github.io ）上。  

在站点源文件根目录创建 .github/workflows/deploy.yml 文件：

```
name: github pages # 名字自取

on:
  push:
    branches:
      - main
jobs:
  deploy: # 任务名自取
    runs-on: ubuntu-18.04 # 在什么环境运行任务
    steps:
      - uses: actions/checkout@v2 # 引用actions/checkout这个action，与所在的github仓库同名
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive) 获取submodule主题
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo  # 步骤名自取
        uses: peaceiris/actions-hugo@v2 # hugo官方提供的action，用于在任务环境中获取hugo
        with:
          hugo-version: 'latest'  # 获取最新版本的hugo
          # extended: true

      - name: Build
        run: hugo --minify  # 使用hugo构建静态网页

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3 # 一个自动发布github pages的action
        with:
          # github_token: ${{ secrets.HUGO_TOKEN }} 该项适用于发布到源码相同repo的情况，不能用于发布到其他repo
          external_repository: sea-wyq/sea-wyq.github.io # 发布到哪个repo
          personal_token: ${{ secrets.HUGO_TOKEN }} # 发布到其他repo需要提供上面生成的personal access token
          publish_dir: ./public # 注意这里指的是要发布哪个文件夹的内容，而不是指发布到目的仓库的什么位置，因为hugo默认生成静态网页到public文件夹，所以这里发布public文件夹里的内容
          publish_branch: main  # 发布到哪个branch
```

##### 6.5. 将本地仓库wyq-blog的修改进行提交
```
git add .
git commit -m "first commit"
git push origin main
```

##### 6.6 查看wyq-log仓库，显示代码提交成功

![Tranquilpeak](/img/main-blog/blog-1/mm-9.png)

访问https://sea-wyq.github.io 

![Tranquilpeak](/img/main-blog/blog-1/mm-10.png)

搭建成功！！!






