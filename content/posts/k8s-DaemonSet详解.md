---
title: "k8s-DaemonSet基础组件解析"
date: 2022-07-16
categories:
- k8s
- docker
thumbnailImagePosition: right
thumbnailImage: img/main/th-10.jpeg
---

DaemonSet 是一个确保全部或者某些节点上必须运行一个Pod的工作负载资源（守护进程），当有节点加入集群时，也会为他们新增一个Pod, 通过创建DaemonSet 可以确保守护进程pod 被调度到每个可用节点上运行。  
下面是常用的使用案例:
- 集群守护进程，如Kured、node-problem-detector
- 日志收集守护进程，如fluentd、logstash
- 监控守护进程，如promethues node-exporter



<!--more-->

{{< toc >}}

# 创建一个DaemonSet

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: mirrorgooglecontainers/fluentd-elasticsearch:v2.4.0
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## 运行并查看运行情况
```
$ kubectl apply -f ds-els.yaml

$ kubectl get pod -n kube-system -l name=fluentd-elasticsearch                ## 单节点，所以只有一个pod运行了

NAME                          READY   STATUS    RESTARTS   AGE
fluentd-elasticsearch-nwqph   1/1     Running   0          4m11s

$ kubectl get ds -n kube-system fluentd-elasticsearch
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd-elasticsearch   1         1         1       1            1           <none>          27m

```

## 仅在某些节点上运行Pod
如果想让DaemonSet在某个特定的Node上运行，可以使用nodeAffinity。
```
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: metadata.name
            operator: In
            values:
            - node1
```

## Taints and Tolerations
在k8s集群中，我们可以给Node打上污点，这样可以让pod避开那些不合适的node。在node上设置一个或多个Taint后，除非pod明确声明能够容忍这些污点，否则无法在这些node上运行。
```
kubectl taint nodes node1 key=value:NoSchedule           //node1打上了一个污点，这将阻止pod调度到node1这个节点上。
kubectl taint nodes node1 key:NoSchedule-                //移除污点
```
如果我们想让pod运行在有污点的node节点上，我们需要在pod上声明Toleration，表明可以容忍具有该Taint的Node。

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-taints
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
    - name: pod-taints
      image: busybox:latest
```
