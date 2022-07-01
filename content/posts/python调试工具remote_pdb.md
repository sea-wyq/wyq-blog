---
title: "python远程调试工具remote-pdb"
date: 2022-06-29
categories:
- 其它
tags:
- 其它
autoThumbnailImage: false
thumbnailImagePosition: right
thumbnailImage: img/main/th-1.jpeg
coverImage: /img/main-blog/blog-5/img.jpeg
metaAlignment: center
---

使用remote-pdb + telent实现在远程服务器上进行Python文件调试

<!--more-->

{{< toc >}}  

## 1. 安装remote-pdb

```
  pip install remote-pdb
  pip install telent
```
## 2. 常用指令
![img.png](/img/main-blog/blog-6/img.png)


## 3. 使用流程

### 3.1 在需要调试的python文件添加代码块

```python
  import pdb
  pdb.set_trace(port=4444)     
```

### 3.2 执行python文件

### 3.3 使用telent进行调试

```shell
telent 127.0.0.1 4444
```
