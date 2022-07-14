---
title: "k8s-Deployment基础组件解析"
date: 2022-07-14
categories:
- k8s
- docker
thumbnailImagePosition: right
thumbnailImage: img/main/th-6.jpeg
---

Deployment对象，顾名思义，是用于部署应用的对象。它使Kubernetes中最常用的一个对象，它为ReplicaSet和Pod的创建提供了一种声明式的定义方法

<!--more-->

{{< toc >}}


# nginx-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment       # deployment的名称，全局唯一
spec:
  replicas: 1     # Pod副本的期待数量
  selector:
    matchLabels:
      app: nginx    # 符合目标的Pod拥有此标签
  template:       # 根据此模板创建Pod的副本
    metadata:
      labels:     # Pod副本拥有的标签
        app: nginx
    spec:
      containers:           # Pod内容器的定义部分
      - name: nginx         # 容器对应的名称
        image: nginx        # 容器对应的Docker镜像
        ports:
        - containerPort: 80      # 容器应用监听的端口号
```

## 创建Deployment
```
[root@k8s-node1 mytestyaml]# kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment configured
```

## 查看Deployment状态
```
[root@k8s-node1 mytestyaml]# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           84s
```

## 查看Pod
```
[root@k8s-node1 mytestyaml]# kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
nginx-deployment-85ff79dd56-8hbb9   1/1     Running   0          21s   10.244.2.60   k8s-node3   <none>           <none>
nginx-deployment-85ff79dd56-gx2dl   1/1     Running   0          20s   10.244.1.68   k8s-node2   <none>           <none>

```
- deployment 相对于直接创建Pod而言，他会自动的保持pod副本的数量维持在指定的replicas数目。
- 即使手动将该Pod删除，k8s仍然会自动再次创建这个Pod。

```
[root@k8s-node1 mytestyaml]# kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE    IP            NODE        NOMINATED NODE   READINESS GATES
nginx-deployment-85ff79dd56-8v76n   1/1     Running   0          7s     10.244.2.61   k8s-node3   <none>           <none>
nginx-deployment-85ff79dd56-gx2dl   1/1     Running   0          6m1s   10.244.1.68   k8s-node2   <none>           <none>
[root@k8s-node1 mytestyaml]# kubectl delete pod nginx-deployment-85ff79dd56-8v76n
pod "nginx-deployment-85ff79dd56-8v76n" deleted
[root@k8s-node1 mytestyaml]# kubectl get pod -o wide
NAME                                READY   STATUS              RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
nginx-deployment-85ff79dd56-42mh9   0/1     ContainerCreating   0          14s     <none>        k8s-node3   <none>           <none>
nginx-deployment-85ff79dd56-gx2dl   1/1     Running             0          6m45s   10.244.1.68   k8s-node2   <none>           <none>
[root@k8s-node1 mytestyaml]# kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
nginx-deployment-85ff79dd56-42mh9   1/1     Running   0          24s     10.244.2.62   k8s-node3   <none>           <none>
nginx-deployment-85ff79dd56-gx2dl   1/1     Running   0          6m55s   10.244.1.68   k8s-node2   <none>           <none>
```
可以看到，在一个Pod被删除后，另外一个Pod立马又被创建了。