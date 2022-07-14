---
title: "k8s-Pod基础组件解析"
date: 2022-07-10
categories:
- k8s
- docker
thumbnailImagePosition: right
thumbnailImage: img/main/th-6.jpeg
---

Pod是Kubernetes中能够创建和部署的最小单元，是Kubernetes集群中的一个应用实例。
- 单容器Pod，最常见的应用方式。
- 多容器Pod，对于多容器Pod，Kubernetes会保证所有的容器都在同一台物理主机或虚拟主机中运行。

<!--more-->

{{< toc >}}

# 1. Pod概述

- 最小部署的单元。
- Pod里面是由一个或多个容器组成【一组容器的集合】。
- 一个pod中的容器是共享网络命名空间。
- Pod是短暂的。
- 每个Pod包含一个或多个紧密相关的用户业务容器。

# 2. Pod 两种机制

## 2.1 Pod 共享网络

容器本身之间相互隔离的，一般是通过 namespace 和 group 进行隔离，那么Pod里面的容器如何实现通信？

- 首先需要满足前提条件，也就是容器都在同一个namespace之间


关于Pod实现原理，首先会在Pod会创建一个根容器： pause容器，然后我们在创建业务容器 【nginx，redis 等】，在我们创建业务容器的时候，会把它添加到 info容器中,而在 info容器 中会独立出 ip地址，mac地址，port 等信息，然后实现网络的共享

![img.png](/img/main-blog/blog-7/img.png)

## 2.1 Pod 共享存储
Pod持久化数据，专门存储到某个地方中
![img.png](/img/main-blog/blog-7/img-1.png)

使用 Volumn数据卷进行共享存储，案例如下所示
![img.png](/img/main-blog/blog-7/img-2.png)

# 3. Pod.yaml字段详解

```
apiVersion: v1 #指定api版本，此值必须在kubectl apiversion中
kind: Pod #指定创建资源的角色/类型
metadata: #资源的元数据/属性
  name: web04-pod #资源的名字，在同一个namespace中必须唯一
  labels: #设定资源的标签，详情请见http://blog.csdn.net/liyingke112/article/details/77482384
    k8s-app: apache
    version: v1
    kubernetes.io/cluster-service: "true"
  annotations:            #自定义注解列表
    - name: String        #自定义注解名字
spec:#specification of the resource content 指定该资源的内容，定义，规范，目标状态，用户定义
  restartPolicy: Always #表明该容器一直运行，默认k8s的策略，在此容器退出后，会立即创建一个相同的容器
  nodeSelector:     #节点选择，先给主机打标签kubectl label nodes kube-node1 zone=node1
    zone: node1
  containers:
  - name: web04-pod #容器的名字
    image: web:apache #容器使用的镜像地址
    imagePullPolicy: Never #三个选择Always、Never、IfNotPresent，每次启动时检查和更新（从registery）images的策略，
                           # Always，每次都检查
                           # Never，每次都不检查（不管本地是否有）
                           # IfNotPresent，如果本地有就不检查，如果没有就拉取
    command: ['sh'] #启动容器的运行命令，将覆盖容器中的Entrypoint,对应Dockefile中的ENTRYPOINT
    args: ["$(str)"] #启动容器的命令参数，对应Dockerfile中CMD参数
    env: #指定容器中的环境变量
    - name: str #变量的名字
      value: "/etc/run.sh" #变量的值
    resources: #资源管理，请求请见http://blog.csdn.net/liyingke112/article/details/77452630
      requests: #容器运行时，最低资源需求，也就是说最少需要多少资源容器才能正常运行
        cpu: 0.1 #CPU资源（核数），两种方式，浮点数或者是整数+m，0.1=100m，最少值为0.001核（1m）
        memory: 32Mi #内存使用量
      limits: #资源限制
        cpu: 0.5
        memory: 32Mi
    ports:
    - containerPort: 80 #容器开发对外的端口
      name: httpd  #名称
      protocol: TCP
    livenessProbe: #pod内容器健康检查的设置，详情请见http://blog.csdn.net/liyingke112/article/details/77531584
      httpGet: #通过httpget检查健康，返回200-399之间，则认为容器正常
        path: / #URI地址
        port: 80
        #host: 127.0.0.1 #主机地址
        scheme: HTTP
      initialDelaySeconds: 180 #表明第一次检测在容器启动后多长时间后开始
      timeoutSeconds: 5 #检测的超时时间
      periodSeconds: 15  #检查间隔时间
      #也可以用这种方法
      #exec: 执行命令的方法进行监测，如果其退出码不为0，则认为容器正常
      #  command:
      #    - cat
      #    - /tmp/health
      #也可以用这种方法
      #tcpSocket: //通过tcpSocket检查健康
      #  port: number
    lifecycle: #生命周期管理
      postStart: #容器运行之前运行的任务
        exec:
          command:
            - 'sh'
            - 'yum upgrade -y'
      preStop:#容器关闭之前运行的任务
        exec:
          command: ['service httpd stop']
    volumeMounts:  #详情请见http://blog.csdn.net/liyingke112/article/details/76577520
    - name: volume #挂载设备的名字，与volumes[*].name 需要对应
      mountPath: /data #挂载到容器的某个路径下
      readOnly: True
  volumes: #定义一组挂载设备
  - name: volume #定义一个挂载设备的名字
    #meptyDir: {}
    hostPath:
      path: /opt #挂载设备类型为hostPath，路径为宿主机下的/opt,这里设备类型支持很多种
```

