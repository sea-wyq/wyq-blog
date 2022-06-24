---
title: "使用GitHub Actions自动构建DockerHub镜像"
date: 2022-06-24
categories:
- github action
- docker hub
tags:
- 其他
thumbnailImagePosition: right
thumbnailImage: img/main/th-3.jpeg
---

GitHub Actions持续集成服务自动构建发布镜像到DockerHub，目前GitHub Actions是免费开放的，所以Github上的项目都可以使用它来发布、测试、部署等等，非常方便。

<!--more-->

## 1. 新建一个github仓库

![Tranquilpeak](/img/main-blog/blog-3/img-1.png)

## 2. 注册登录docker hub并进行仓库创建 
注册登录docker hub官网(https://hub.docker.com/), 需要翻墙访问...，然后创建自己的镜像仓库。  

![Tranquilpeak](/img/main-blog/blog-3/img.png)

## 3. 在github仓库下的建立.github/workflows/release.yml文件

```yaml
name: Docker Image CI

on:
  push:
    branches: [ main ]
    tags:
      - '*'
    
    

jobs:

  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Set up QEMU
      uses: docker/setup-qemu-action@v2

    - name: Set up Docker Buildx
      id: buildx
      uses: docker/setup-buildx-action@v2            #使用buildx进行构建镜像
    
    - name: Log in to Docker Hub
      if: github.event_name != 'pull_request'
      uses: docker/login-action@v2                   #登录docker hub 
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}  #用户名
        password: ${{ secrets.DOCKERHUB_TOKEN }}     #密码
        
    - name: Extract metadata (tags, labels) for Docker
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: seawyq/aiges-gpu                      # docker hub下建立的镜像仓库名
        
    - name: Build and push Docker image
      uses: docker/build-push-action@v3
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
```

## 4. 在当前仓库Settings下建立secret

![Tranquilpeak](/img/main-blog/blog-3/img-3.png)

## 5. 编写dockerfile文件并进行上传

在本地仓库的根目录下创建dockerfile文件  


## 6. 查看镜像是否自动化构建成功


![Tranquilpeak](/img/main-blog/blog-3/img-4.png)


## 7. 查看docker search 能否正常使用


![Tranquilpeak](/img/main-blog/blog-3/img-5.png)


呼叫 "over over over" .


