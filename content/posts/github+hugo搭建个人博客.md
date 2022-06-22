---
title: "github+hugo搭建个人博客"
date: 2022-06-22
categories:
- github
- hugo
tags:
- 其他
thumbnailImagePosition: right
thumbnailImage: images/th.jpg
---

在windows10上通过github和hugo实现个人博客的快速搭建。

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

![Tranquilpeak](/img/mm.png)
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


![Tranquilpeak](/img/mm-1.png)


## 6. 自动化站点部署（github pages）
为了确保个人站点文章（markdown文件）的相对安全，采取将站点源码仓库和站点分开，即：站点源码仓库设置为私有，站点 GitHub Pages 仓库设为公开。  

##### 6.1 新建github仓库用来存放网站源码，如：wyq-blog。
注：wyq-blog为示例仓库，故设置为公开仓库，实践中建议设置为私有仓库。  

![Tranquilpeak](/img/mm-2.png)


##### 6.2  新建一个 GitHub repository，库名为<username>.github.io，其中 username 是你的 GitHub 账号。例如：sea-wyq.github.io。

![Tranquilpeak](/img/mm-3.png)


##### 6.3 克隆wyq-blog仓库到本地目录，然后将blog文件下的文件拷贝到仓库根目录中

```
git clone https://github.com/sea-wyq/wyq-blog.git
```
注：1. 将wyq-blog目录下 .gitignore文件中中的thems/删除。
    2. 将wyq-blog\themes\hugo-tranquilpeak-theme目录下的.git文件删除。


##### 6.4 构建 Hugo 网站

在本地仓库（wyq-blog）下执行 hugo 命令构建  

![Tranquilpeak](/img/mm-4.png)

##### 7.4  创建github token 并在wyq-blog和sea-wyq.github.io 添加secret  

github token创建流程：github用户画像-> Settings->Developer settings ->Personal 

access tokens ->Generate new token




































## Paragraph

Lorem ipsum dolor sit amet, [test link]() consectetur adipiscing elit. **Strong text** pellentesque ligula commodo viverra vehicula. *Italic text* at ullamcorper enim. Morbi a euismod nibh. <u>Underline text</u> non elit nisl. ~~Deleted text~~ tristique, sem id condimentum tempus, metus lectus venenatis mauris, sit amet semper lorem felis a eros. Fusce egestas nibh at sagittis auctor. Sed ultricies ac arcu quis molestie. Donec dapibus nunc in nibh egestas, vitae volutpat sem iaculis. Curabitur sem tellus, elementum nec quam id, fermentum laoreet mi. Ut mollis ullamcorper turpis, vitae facilisis velit ultricies sit amet. Etiam laoreet dui odio, id tempus justo tincidunt id. Phasellus scelerisque nunc sed nunc ultricies accumsan.

Interdum et malesuada fames ac ante ipsum primis in faucibus. `Sed erat diam`, blandit eget felis aliquam, rhoncus varius urna. Donec tellus sapien, sodales eget ante vitae, feugiat ullamcorper urna. Praesent auctor dui vitae dapibus eleifend. Proin viverra mollis neque, ut ullamcorper elit posuere eget.


## List Types

### Definition List (dl)

<dl><dt>Definition List Title</dt><dd>This is a definition list division.</dd></dl>

### Ordered List (ol)

1. List Item 1
2. List Item 2
3. List Item 3

### Unordered List (ul)

- List Item 1
- List Item 2
- List Item 3

## Table

|  Header 1  | Header 2   | Header 3   |
|:----------:|------------|------------|
| Division 1 | Division 2 | Division 3 |
| Division 1 | Division 2 | Division 3 |
| Division 1 | Division 2 | Division 3 |
| Division 1 | Division 2 | Division 3 |

## Misc Stuff - abbr, acronym, sub, sup, etc.

Lorem <sup>superscript</sup> dolor <sub>subscript</sub> amet, consectetuer adipiscing <kdb>ctrl + c</kdb>. Nullam dignissim convallis est. Quisque aliquam. <cite>cite</cite>. Nunc iaculis suscipit dui.
 Nam
sit amet sem. Aliquam libero nisi, imperdiet at, tincidunt nec, gravida vehicula, nisl. Praesent mattis, massa quis luctus fermentum, turpis mi volutpat justo, eu volutpat enim diam eget metus. Maecenas ornare tortor. Donec sed tellus eget sapien fringilla nonummy. <acronym title="National Basketball Association">NBA</acronym> Mauris a ante. Suspendisse quam sem, consequat at, commodo vitae, feugiat in, nunc. Morbi imperdiet augue quis tellus.  <abbr title="Avenue">AVE</abbr>
