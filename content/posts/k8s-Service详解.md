---
title: "k8s-Service基础组件解析"
date: 2022-07-16
categories:
- k8s
- docker
thumbnailImagePosition: right
thumbnailImage: img/main/th-11.jpeg
---

Kubernetes 中Service有以下3种类型：
- ClusterIP：默认类型，自动分配一个仅Cluster内部可以访问的虚拟IP
- NodePort：通过每个 Node 上的 IP 和静态端口（NodePort）暴露服务。以ClusterIP为基础，NodePort 服务会路由到 ClusterIP 服务。通过请求 <NodeIP>:<NodePort>, 可以从集群的外部访问一个集群内部的 NodePort 服务。
- LoadBalancer：使用云提供商的负载均衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 NodePort 服务和 ClusterIP 服务。

<!--more-->

{{< toc >}}

>需要注意的是：Service 能够将一个接收 port 映射到任意的 targetPort。默认情况下，targetPort 将被设置为与 port 字段相同的值。Service域名格式：$(service name).$(namespace).svc.cluster.local，其中 cluster.local 为指定的集群的域名


# Deoployment的资源清单

```
[root@master mainfest]# vim deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:latest
        ports:
        - containerPort: 80
        
[root@master mainfest]# kubectl apply -f deploy.yaml 
deployment.apps/deploy created
[root@master mainfest]# kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
deploy-585449566-kdwq5   1/1     Running   0          55s   10.244.1.58   node1   <none>           <none>
deploy-585449566-p5fdj   1/1     Running   0          55s   10.244.1.57   node1   <none>           <none>
deploy-585449566-wcj5r   1/1     Running   0          55s   10.244.1.56   node1   <none>           <none>


# curl访问
[root@master mainfest]# curl 10.244.1.58
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
[root@master mainfest]# curl 10.244.1.57
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
[root@master mainfest]# curl 10.244.1.56
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
```

## ClusterIP类型示例
创建cluster IP类型的service
```
[root@master mainfest]# vim svc-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  type: ClusterIP   //可以不写，默认为ClusterIP
  selector:
    app: nginx
  ports:
  - name: nginx
    port: 80
    targetPort: 80
    
[root@master mainfest]# kubectl apply -f svc-clusterip.yaml 
service/svc-clusterip created
[root@master mainfest]# kubectl get svc
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP   6d23h
svc-clusterip   ClusterIP   10.105.207.162   <none>        80/TCP    2m39s


# curl访问
[root@master mainfest]# curl 10.105.207.162
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>

```

## NodePort类型示例
如果将 type 字段设置为 NodePort，则 Kubernetes 控制层面将在 --service-node-port-range 标志指定的范围内分配端口（默认值：30000-32767）。
```
[root@master mainfest]# vim svc-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - name: nginx1
    port: 80      //service对提供服务的端口
    targetPort: 80    //请求转发后端pod所用的端口
    nodePort: 30001   //可选字段，默认情况下，为了方便起见，k8s控制层面会从某个范围内随机分配一个端口号(默认：30000-32767)
    
[root@master mainfest]# kubectl apply -f svc-nodeport.yaml 
service/svc-nodeport created
[root@master mainfest]# kubectl get svc
svc-nodeport    NodePort    10.105.10.125    <none>        80:30001/TCP   7s


[root@master mainfest]# curl 10.105.10.125
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
[root@master mainfest]# curl 192.168.100.110:30001    //访问本机的30001端口
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>

```
## LoadBalancer类型示例
在使用支持外部负载均衡器的云提供商的服务时，设置 type 的值为 "LoadBalancer"， 将为 Service 提供负载均衡器。 负载均衡器是异步创建的，关于被提供的负载均衡器的信息将会通过 Service 的 status.loadBalancer 字段发布出去。
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
      - ip: 192.0.2.127
```
来自外部负载均衡器的流量将直接重定向到后端 Pod 上，不过实际它们是如何工作的，这要依赖于云提供商。
