---
title: "Helm解析"
date: 2022-07-17
categories:
- k8s
- docker
thumbnailImagePosition: right
thumbnailImage: img/main/th-13.jpeg
---

Helm概念
- Chart 是一个Helm包，涵盖了需要在Kubernetes集群中运行应用，工具或者服务的资源定义。 把它想象成Kubernetes对应的Homebrew公式，Apt dpkg，或者是Yum RPM文件。
- 仓库（Repository）： 归集和分享chart的地方。
- 发布（Release）：在Kubernetes集群中运行的chart实例。一个chart经常在同一个集群中被重复安装。每次安装都会生成新的发布。比如MySQL，如果想让两个数据库运行在集群中，可以将chart安装两次。每一个都会有自己的发布版本，并有自己的发布名称。

<!--more-->

{{< toc >}}
