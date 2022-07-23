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
apiVersion: v1       #必选，版本号，例如v1
kind: Pod       #必选，Pod
metadata:       #必选，元数据
  name: string       #必选，Pod名称
  namespace: string    #必选，Pod所属的命名空间
  labels:      #自定义标签
    - name: string     #自定义标签名字
  annotations:       #自定义注释列表
    - name: string
spec:         #必选，Pod中容器的详细定义
  containers:      #必选，Pod中容器列表
  - name: string     #必选，容器名称
    image: string    #必选，容器的镜像名称
    imagePullPolicy: [Always | Never | IfNotPresent] #获取镜像的策略 Alawys表示下载镜像 IfnotPresent表示优先使用本地镜像，否则下载镜像，Nerver表示仅使用本地镜像
    command: [string]    #容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]     #容器的启动命令参数列表
    workingDir: string     #容器的工作目录
    volumeMounts:    #挂载到容器内部的存储卷配置
    - name: string     #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string    #存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean    #是否为只读模式
    ports:       #需要暴露的端口库号列表
    - name: string     #端口号名称
      containerPort: int   #容器需要监听的端口号
      hostPort: int    #容器所在主机需要监听的端口号，默认与Container相同
      protocol: string     #端口协议，支持TCP和UDP，默认TCP
    env:       #容器运行前需设置的环境变量列表
    - name: string     #环境变量名称
      value: string    #环境变量的值
    resources:       #资源限制和请求的设置
      limits:      #资源限制的设置
        cpu: string    #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string     #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests:      #资源请求的设置
        cpu: string    #Cpu请求，容器启动的初始可用数量
        memory: string     #内存清楚，容器启动的初始可用数量
    livenessProbe:     #容器探测。对Pod内个容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
      exec:      #对Pod容器内检查方式设置为exec方式
        command: [string]  #exec方式需要制定的命令或脚本
      httpGet:       #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:     #对Pod内个容器健康检查方式设置为tcpSocket方式
        host: string
        port: number
      initialDelaySeconds: 0  #容器启动完成后等待多少秒首次探测，单位为秒
      timeoutSeconds: 0   #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
      periodSeconds: 0    #对容器监控检查的探测频率，单位秒，默认每10秒一次
      successThreshold: 0  #连续探测失败多少次才被认定为失败，默认是3，最小值1
      failureThreshold: 0  #连续探测失败多少次才被认定为失败，默认是3，最小值1
      securityContext:
        privileged: false
    restartPolicy: [Always | Never | OnFailure] #Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
    nodeName: node1 #指定调度到node1上
    nodeSelector: obeject  #设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定
    imagePullSecrets:    #Pull镜像时使用的secret名称，以key：secretkey格式指定
    - name: string
    hostNetwork: false      #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
    volumes:       #在该pod上定义共享存储卷列表
    - name: string     #共享存储卷名称 （volumes类型有很多种）
      emptyDir: {}     #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
      hostPath: string     #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
        path: string     #Pod所在宿主机的目录，将被用于同期中mount的目录
      nfs:     #类型为nfs的存储卷
        server: IPstring  #nfs服务器地址
        path: /root/data/nfs   #共享文件路径
      secret:      #类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
        scretname: string  
        items:     
        - key: string
          path: string
      configMap:     #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
        name: string
        items:
        - key: string
      persistentVolumeClaim:
        claimName: pvc2
        readOnly: false
    lifecycle: 管理系统应对容器生命周期事件应采取的操作。无法更新。钩子函数。
      postStart:
        exec: #在容器启动时执行一个命令
          command:
      preStop: 
        exec: #在容器停止之前执行一个命令
          command:
  affinity:  #亲和性
    nodeAffinity:     #node亲和性
      requiredDuringSchedulingIgnoredDuringExecution:   #硬限制，node节点必须满足以下所有条件才可以
        nodeSelectorTerms: #节点选择列表
          matchFields: string        #按节点字段列出的节点选择器要求列表
          matchExpressions:          #按节点标签列出的节点选择器要求列表(推荐)
            key: string       #键
            values: string    #值
            operator: string  #关系符 支持Exists, DoesNotExist, In, NotIn, Gt, Lt
      preferredDuringSchedulingIgnoredDuringExecution:  #软限制，优先调度到指定规则的node中
        preference:  #一个节点选择器项，与权重相关联
          matchFields:       #按节点字段列出的节点选择器要求列表
          matchExpressions:  #按节点标签列出的节点选择器要求列表(推荐)
            key: string      #键
            values: string   #值
            operator: string #关系符 支持In, NotIn, Exists, DoesNotExist, Gt, Lt
        weight: number     #与匹配相应节点SelectorTerm关联的权重，在范围1-100。
    podAffinity:      #pod亲和性
      requiredDuringSchedulingIgnoredDuringExecution:   #硬限制
        namespaces: string      #指定参照pod的namespace
        topologyKey: [kubernetes.io/hostname,beta.kubernetes.io/os]   #指定调度作用域
                       #kubernetes.io/hostname，以Node节点为区分范围
                       #beta.kubernetes.io/os,以Node节点的操作系统类型来区分
        labelSelector:    #标签选择器
          matchExpressions:  #按节点标签列出的节点选择器要求列表(推荐)
            key: string   #键
            values: string #值
            operator: string #关系符 支持In, NotIn, Exists, DoesNotExist.
          matchLabels: string   #指多个matchExpressions映射的内容
      preferredDuringSchedulingIgnoredDuringExecution:  #软限制
        podAffinityTerm:  #选项
          namespaces: string     
          topologyKey: string
          labelSelector: 
            matchExpressions:  
              key: string    #键
              values: string #值
              operator: string
            matchLabels: string
        weight: number  #倾向权重，在范围1-100
    podAntiAffinity:  #pod反亲和性，参数与pod亲和性一样
      requiredDuringSchedulingIgnoredDuringExecution:   #硬限制
        namespaces: string      #指定参照pod的namespace
        topologyKey:   [kubernetes.io/hostname,beta.kubernetes.io/os]
        labelSelector:    #标签选择器
          matchExpressions:  #按节点标签列出的节点选择器要求列表(推荐)
            key: string      #键
            values: string   #值
            operator: string #关系符 支持In, NotIn, Exists, DoesNotExist.
          matchLabels: string   #指多个matchExpressions映射的内容
      preferredDuringSchedulingIgnoredDuringExecution:  #软限制
        podAffinityTerm:  #选项
          namespaces: string     
          topologyKey: string
          labelSelector:
            matchExpressions:  
              key: string   #键
              values: string #值
              operator: string
            matchLabels: string
        weight: number #倾向权重，在范围1-100
  tolerations:      #添加容忍
  - key: string        #要容忍的污点的key，空意味着匹配所有的键
    value: string      #容忍的污点的value
    operator: string   #key-value的运算符，支持Equal和Exists（默认）
    effect: string     #添加容忍的规则，这里必须和标记的污点规则相同，空意味着匹配所有影响
    tolerationSeconds: number   # 容忍时间, 当effect为NoExecute时生效，表示pod在Node上的停留时间
```

