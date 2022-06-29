---
title: "使用sealos在阿里云服务器上部署k8s集群"
date: 2022-06-20
categories:
- docker
- sealos
tags:
- 其他
thumbnailImagePosition: right
thumbnailImage: img/main/th-2.jpeg
---

使用sealos在阿里云上进行k8s集群快捷部署。

<!--more-->

{{< toc >}}

## 1. 申请阿里云服务器（3台）
![img.png](/img/main-blog/blog-2/img.png)

## 2. 集群环境初始化
 点击服务器实例的远程链接按钮进入WorkBench控制台。

### 2.1 关闭防火墙和selinux
```
systemctl stop firewalld 
systemctl disable firewalld 
setenforce 0
```

### 2.2 在各节点配置主机名
```
hostnamectl set-hostname master 
hostnamectl set-hostname node1
hostnamectl set-hostname node2
```

### 2.3 在master节点上配置hosts文件
各节点的ip地址可以通过ifconfig命令获取并修改
```
cat <<EOF >> /etc/hosts  
172.24.253.78 master   
172.24.253.80 node1  
172.24.253.79 node2  
EOF
```

### 2.4 配置master免密登录node1和node2
```
ssh-keygen   #一路回车即可
ssh-copy-id root@master 
ssh-copy-id root@node1 
ssh-copy-id root@node2 
```
## 3. Sealos集群搭建
### 3.1 各节点下载并安装sealos
```
wget -c https://sealyun-home.oss-cn-beijing.aliyuncs.com/sealos-4.0/latest/sealos-amd64 -O sealos && \
    chmod +x sealos && mv sealos /usr/bin   
```

### 3.2 在master上创建集群（一主俩从）
```
 sealos run labring/kubernetes:v1.24.0 labring/calico:v3.22.1 \
     --masters 172.24.253.78 \
     --nodes 172.24.253.80,172.24.253.79 -p “自定义密码”
```

### 3.3 在master上安装helm
```
 sealos run labring/helm:v3.8.2 # install helm
```

## 4. k8s服务器集群搭建成功

![img_1.png](/img/main-blog/blog-2/img-1.png)









