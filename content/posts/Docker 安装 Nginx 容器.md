---
title: "Docker 安装 Nginx 容器"
date: 2022-12-15
thumbnailImagePosition: right
thumbnailImage: img/main/th-4.jpeg
coverImage: /img/main-blog/blog-4/img-2.jpeg
metaAlignment: center
coverMeta: out
categories:
- nginx
tags:
- 其它
---

<!--more-->

{{< toc >}} 

# 1. 下载Nginx镜像

docker pull nginx下载最新版Nginx镜像 (其实此命令就等同于 : docker pull nginx:latest )
docker pull nginx:xxx下载指定版本的Nginx镜像 (xxx指具体版本号)

# 2. 创建Nginx配置文件
启动前需要先创建Nginx外部挂载的配置文件（ /home/nginx/conf/nginx.conf）
之所以要先创建 , 是因为Nginx本身容器只存在/etc/nginx 目录 , 本身就不创建 nginx.conf 文件
当服务器和容器都不存在 nginx.conf 文件时, 执行启动命令的时候 docker会将nginx.conf 作为目录创建 , 这并不是我们想要的结果 。
```bash
# 创建挂载目录
mkdir -p /home/nginx/conf
mkdir -p /home/nginx/log
mkdir -p /home/nginx/html
```
容器中的nginx.conf文件和conf.d文件夹复制到宿主机
```bash
# 生成容器
docker run --name nginx -p 9001:80 -d nginx
# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /home/nginx/conf/nginx.conf
# 将容器default.conf文件复制到宿主机
docker cp nginx:/etc/nginx/conf.d /home/nginx/conf/conf.d
# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /home/nginx/
# 删除正在运行的nginx容器
docker rm -f nginx
```
# 3. 创建Nginx容器并运行
```
docker run \
-p 9002:80 \
--name nginx \
-v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/nginx/log:/var/log/nginx \
-v /home/nginx/html:/usr/share/nginx/html \
-d nginx:latest
```
# 4. 结果检测
![img.png](/img/main-blog/blog-9/1.png)

# 5.修改内容进行展示
修改本地 html下的index.html
![img.png](/img/main-blog/blog-9/2.png)
测试
![img.png](/img/main-blog/blog-9/3.png)



