---
title: "终端复用工具-Tmux"
date: 2022-06-27
thumbnailImagePosition: right
thumbnailImage: img/main/th-4.jpeg
coverImage: /img/main-blog/blog-4/img-2.jpeg
metaAlignment: center
coverMeta: out
categories:
- 其它
tags:
- 其它
---


主要有两个作用：

- 有的可执行的程序或者脚本我们经常需要使用nohup 来再后台运行防止关掉终端后程序就断开，使用tmux 可以替换nohup使程序 在后台运行。

- 调试bug的时候，可能需要多个终端来回切换，使用tmux 我们可以在一个屏上，操作多个终端。详见开头使用效果。

<!--more-->

## 效果图

![Tranquilpeak](/img/main-blog/blog-4/img.png)

## 2. Linux下安装

```
  apt-get install tmux
```

## 3. 常用指令


```yaml
tmux   建立一个回话并进入回话
tmux new -s test 建立一个名字叫test的回话
Ctrl-b + %  将屏幕左右分为两个终端
ctrl-b + "  将屏幕分为上下两个终端
ctrl-b + o  切换终端
ctrl-b + x 关闭当前终端（会出现提示）
ctrl-b + &  关闭整个页面（会出现提示）

tmux ls  显示回话列表
tmux a  连接上一个回话
tmux a -t session_name 连接指定名称的回话
tmux kill-session -t s1   关闭回话s1
tmux kill-server   关闭所以回话
```



