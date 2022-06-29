---
title: "对象存储工具-S3cmd"
date: 2022-06-29
categories:
- 其它
tags:
- 其它
autoThumbnailImage: false
thumbnailImagePosition: right
thumbnailImage: img/main/th-5.jpeg
coverImage: /img/main-blog/blog-5/img.jpg
metaAlignment: center
---

**s3cmd** 是用于创建S3桶，上传，检索和管理数据到对象存储命令行实用程序。

<!--more-->

{{< toc >}}  

## 1. 安装s3cmd （以ubuntu系统为例）

```
  apt-get install s3cmd
```

## 2. 配置s3cmd
执行$ s3cmd --configure生成配置文件，一路Enter，注意跳过认证并保存配置

```
......
...
Test access with supplied credentials? [Y/n] n
Save settings? [y/N] y
Configuration saved to '/root/.s3cfg'
```
修改一下几项：
```
$ vim /root/.s3cfg
access_key = xxx
secret_key = xxx
host_base = ip:port
host_bucket = ip/kucketname
use_https = False
```
* 其中，access_key和secret_key是在本地创建S3用户时获得，host_base是S3服务所使用的ip地址（包括端口 号）,host_bucket为S3用户下的一个bucket(可在配置之后再创建，但该字段不能为空)

## 3. 常用命令

### 3.1 列举所有buckets(bucket相当于根文件夹)

```shell
root@node4:/home# s3cmd ls
2016-09-18 03:51  s3://my-bucket
2016-09-18 02:02  s3://my-new-bucket-node4
2016-09-18 07:17  s3://zhangbo
```

### 3.2 创建bucket(bucket名称唯一，不能重复)
```shell
命令：s3cmd mb s3://{$BUCKETNAME}
root@node4:/home# s3cmd mb s3://zhangbo1
Bucket 's3://zhangbo1/' created
```

### 3.3 删除空bucket
```shell
命令：s3cmd rb s3://{$BUCKETNAME}
root@node4:/home# s3cmd rb s3://zhangbo1
Bucket 's3://zhangbo1/' removed
```

### 3.4 上传某个文件到bucket
```shell
命令：s3cmd put {$FILENAME} s3://{$BUCKETNAME}
root@node4:~# s3cmd put s3cmd-1.5.2.tar.gz s3://zhangbo
s3cmd-1.5.2.tar.gz -> s3://zhangbo/s3cmd-1.5.2.tar.gz  [1 of 1]
94760 of 94760   100% in    0s   598.02 kB/s  done
```
### 3.5 列举bucket中的内容
```shell
命令：s3cmd ls s3://{$BUCKETNAME}
root@node4:~# s3cmd ls s3://zhangbo
2016-09-18 07:30     94760   s3://zhangbo/s3cmd-1.5.2.tar.gz
2016-09-18 07:30         8   s3://zhangbo/test.txt
```

### 3.6 下载文件
```shell
命令：s3cmd get s3://{路径+文件名}
root@node4:~# s3cmd get s3://zhangbo/s3cmd-1.5.2.tar.gz
s3://zhangbo/s3cmd-1.5.2.tar.gz -> ./s3cmd-1.5.2.tar.gz  [1 of 1]
s3://zhangbo/s3cmd-1.5.2.tar.gz -> ./s3cmd-1.5.2.tar.gz  [1 of 1]
94760 of 94760   100% in    0s     9.24 MB/s  done
```

### 3.7 删除文件
```shell
命令：s3cmd del/rm s3://{路径+文件名}
root@node4:~# s3cmd del s3://zhangbo/test.txt
File s3://zhangbo/test.txt deleted
```

### 3.8 获取对应的bucket所占用的的空间大小
```shell
命令：s3cmd du -H s3://{目录}
root@node4:~# s3cmd du -H s3://zhangbo
185k     s3://zhangbo/
root@node4:~# s3cmd du -H s3://zhangbo/hehe
92k      s3://zhangbo/hehe
```

### 3.9 查看更多关于bucket和文件的信息
```shell
命令：s3cmd info s3://BUCKET[/OBJECT]
root@node4:~# s3cmd info s3://zhangbo/s3cmd-1.5.2.tar.gz
s3://zhangbo/s3cmd-1.5.2.tar.gz (object):
File size: 94760
Last mod:  Sun, 18 Sep 2016 08:33:39 GMT
MIME type: application/gzip
MD5 sum:   3153116dc62c817a724ea58080968383
SSE:       NONE
policy:
ACL: zhangbo: FULL_CONTROL
```

### 3.10 复制bucket或文件
```shell
命令：s3cmd cp [--recursive] s3://BUCKET1/OBJECT1 s3://BUCKET2[/OBJECT2]
root@node4:~# s3cmd cp --recursive s3://zhangbo  s3://zhangbo1
File s3://zhangbo/hehe/s3cmd-1.5.2.tar.gz copied to s3://zhangbo1/hehe/s3cmd-1.5.2.tar.gz
File s3://zhangbo/s3cmd-1.5.2.tar.gz copied to s3://zhangbo1/s3cmd-1.5.2.tar.gz
root@node4:~# s3cmd cp s3://zhangbo/s3cmd-1.5.2.tar.gz s3://zhangbo2
File s3://zhangbo/s3cmd-1.5.2.tar.gz copied to s3://zhangbo2/s3cmd-1.5.2.tar.gz
```

### 3.11 移动文件
```shell
命令：s3cmd mv [--recursive] s3://BUCKET1/OBJECT1 s3://BUCKET2[/OBJECT2]
root@node4:~# s3cmd mv --recursive s3://zhangbo s3://zhangbo2
File s3://zhangbo/hehe/s3cmd-1.5.2.tar.gz moved to s3://zhangbo2/hehe/s3cmd-1.5.2.tar.gz
File s3://zhangbo/s3cmd-1.5.2.tar.gz moved to s3://zhangbo2/s3cmd-1.5.2.tar.gz
root@node4:~# s3cmd mv s3://zhangbo2/s3cmd-1.5.2.tar.gz s3://zhangbo
File s3://zhangbo2/s3cmd-1.5.2.tar.gz moved to s3://zhangbo/s3cmd-1.5.2.tar.gz
```

### 3.12 同步当前目录下所有的文件
```shell
命令：s3cmd sync ./ s3://{BUCKETNAME}
root@node4:~# s3cmd sync ./ s3://zhangbo2
./.bash_history -> s3://zhangbo2/.bash_history  [1 of 8]
4446 of 4446   100% in    0s    82.48 kB/s  done
./.bashrc -> s3://zhangbo2/.bashrc  [2 of 8]
3106 of 3106   100% in    0s    58.82 kB/s  done
./.cache/motd.legal-displayed -> s3://zhangbo2/.cache/motd.legal-displayed  [3 of 8]
0 of 0     0% in    0s     0.00 B/s  done
./.profile -> s3://zhangbo2/.profile  [4 of 8]
140 of 140   100% in    0s     2.44 kB/s  done
./.s3cfg -> s3://zhangbo2/.s3cfg  [5 of 8]
1769 of 1769   100% in    0s    31.63 kB/s  done
./.viminfo -> s3://zhangbo2/.viminfo  [6 of 8]
11480 of 11480   100% in    0s   221.65 kB/s  done
./test.txt -> s3://zhangbo2/test.txt  [7 of 8]
8 of 8   100% in    0s   160.86 B/s  done
./hehe/s3cmd-1.5.2.tar.gz -> s3://zhangbo2/hehe/s3cmd-1.5.2.tar.gz  [8 of 8]
94760 of 94760   100% in    0s  1677.16 kB/s  done
remote copy: hehe/s3cmd-1.5.2.tar.gz -> s3cmd-1.5.2.tar.gz
Done. Uploaded 115709 bytes in 1.0 seconds, 113.00 kB/s. Copied 1 files saving 94760 bytes transfer.
```

### 3.13 列出需要同步的项目，但不进行同步
```shell
命令：s3cmd sync --dry-run ./ s3://{BUCKETNAME}
root@node4:~# s3cmd sync --dry-run ./ s3://zhangbo2
upload: ./aaa -> s3://zhangbo2/aaa
WARNING: Exiting now because of --dry-run
```

### 3.14 在bucket中删除本地不存在的文件
```python
命令：s3cmd sync --delete-removed ./ s3://{BUCKETNAME}
root@node4:~# s3cmd sync --delete-removed ./ s3://zhangbo2
File s3://zhangbo2/aaa deleted
```