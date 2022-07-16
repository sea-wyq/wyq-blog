---
title: "k8s-StatefulSet基础组件解析"
date: 2022-07-16
categories:
- k8s
- docker
thumbnailImagePosition: right
thumbnailImage: img/main/th-9.jpeg
---

应用场景
- 稳定的不共享持久化存储：即每个pod的存储资源是不共享的，且pod重新调度后还是能访问到相同的持久化数据，基于pvc实现。
- 稳定的网络标志：即pod重新调度后其PodName和HostName不变，且PodName和HostName是相同的，基于Headless Service来实现的。
- 有序部署，有序扩展：即pod是有顺序的，在部署或者扩展的时候是根据定义的顺序依次依序部署的（即从0到N-1,在下一个Pod运行之前所有之前的pod必都是Running状态或者Ready状态），是基于init containers来实现的。
- 有序收缩：在pod删除时是从最后一个依次往前删除，即从N-1到0.

<!--more-->

{{< toc >}}


# 创建yaml文件

```
[root@vm21 opt]# vi statefulset-demo.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None                                 # 要点1，可以不用指定clusterIP
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"                           # 要点2，指定serviceName名称
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
          name: web
```

## 应用yaml文件
```
[root@vm21 opt]# kubectl apply -f statefulset-demo.yaml
service/nginx created
statefulset.apps/web created
```

## 查看资源清单
```
[root@vm21 opt]# kubectl get pods,svc,sts
NAME        READY   STATUS    RESTARTS   AGE
pod/web-0   1/1     Running   0          10m
pod/web-1   1/1     Running   0          10m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   109m
service/nginx        ClusterIP   None         <none>        80/TCP    10m

NAME                   READY   AGE
statefulset.apps/web   2/2     10m
```

## 扩缩容
```
# 增加一个pod
[root@vm21 opt]# kubectl scale --replicas=3 sts web
statefulset.apps/web scaled
```
## 查看资源清单

```
[root@vm21 opt]# kubectl get pods,svc,sts
NAME        READY   STATUS    RESTARTS   AGE
pod/web-0   1/1     Running   0          10m
pod/web-1   1/1     Running   0          10m
pod/web-2   1/1     Running   0          5m51s                             # 扩容后增加的pod

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   109m
service/nginx        ClusterIP   None         <none>        80/TCP    10m

NAME                   READY   AGE
statefulset.apps/web   3/3     10m
```
## 小结

- 1)有序索引
对于具有 N 个副本的 StatefulSet，StatefulSet 中的每个 Pod 将被分配一个整数序号， 从 0 到 N-1，该序号在 StatefulSet 上是唯一的。比如sts name=nginx，replicas=3则对应的pod名称依次为nginx-0、nginx-1、nginx-2。
- 2)稳定的网络ID
StatefulSet 可以使用 "无头服务" 控制它的 Pod 的网络域。管理域的这个服务的格式为： $(服务名称).$(命名空间).svc.cluster.local
其中 cluster.local 是集群域。 一旦每个 Pod 创建成功，就会得到一个匹配的 DNS 子域，格式为： $(pod 名称).$(所属服务的 DNS 域名) 其中所属服务由 StatefulSet 的 serviceName 域来设定。
