---
title: "k8s-Secret&ConfigMap基础组件解析"
date: 2022-07-16
categories:
- k8s
- docker
thumbnailImagePosition: right
thumbnailImage: img/main/th-12.jpeg
---

- ConfigMap是一个K8s存储资源，用于存储应用程序配置文件。
- Secret主要存储敏感数据。

<!--more-->

{{< toc >}}

# configmap
## 通过环境变量引入：使用 configMapKeyRef

```
vim mysql-configmap.yaml                                            # 创建一个存储 mysql 配置的 configmap

apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql
  labels:
    app: mysql
data:
  log: "1"
  lower: "1"

kubectl apply -f mysql-configmap.yaml                              # 应用configmap

apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
  - name: mysql
    image: busybox
    command: [ "/bin/sh", "-c", "sleep 3600" ]
    env:
    - name: log_bin                                                # 定义环境变量 log_bin
      valueFrom: 
        configMapKeyRef:
        name: mysql                                                # 指定 configmap 的名字
        key: log                                                   # 指定 configmap 中的 key
    - name: lower                                                  # 定义环境变量 lower
      valueFrom:
        configMapKeyRef:
          name: mysql
          key: lower
  restartPolicy: Never

```

## 通过环境变量引入：使用 envfrom
```
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod-envfrom
spec:
  containers:
  - name: mysql
    image: busybox
    command: [ "/bin/sh", "-c", "sleep 3600" ]
    envFrom: 
    - configMapRef:
      name: mysql                                                   # 指定 configmap 的名字
  restartPolicy: Never
```

## 把configmap做成 volume，挂载到 pod
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql
  labels:
  app: mysql
data:
  log: "1"
  lower: "1"
  my.cnf: |
    [mysqld]
    Welcome=hello


apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod-volume
spec:
  containers:
  - name: mysql
    image: busybox
    command: [ "/bin/sh","-c","sleep 3600" ]
    volumeMounts:
    - name: mysql-config
      mountPath: /tmp/config
  volumes:
  - name: mysql-config
    configMap:
      name: mysql
  restartPolicy: Never
```

# Secret

## 通过环境变量引入 Secret
```
kubectl create secret generic mysql-password --from-literal=password=pod**lucky66
kubectl describe secret mysql-password                              # 查看secret详细信息


apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
  labels:
    app: myapp
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80
    env:
    - name: MYSQL_ROOT_PASSWORD                                     # 它是 Pod 启动成功后,Pod 中容器的环境变量名.
      valueFrom:
        secretKeyRef:
          name: mysql-password                                      # 这是 secret 的对象名
          key: password                                             # 它是 secret 中的 key 名


```
进入pod查看环境变量 MYSQL_ROOT_PASSWORD 的值为pod**lucky66
## 通过 volume 挂载 Secret

创建 Secret

```
# 手动加密，基于 base64 加密
echo -n 'admin' | base64
YWRtaW4=

echo -n 'xuegod123456f' | base64
eHVlZ29kMTIzNDU2Zg==

#解密
echo eHVlZ29kMTIzNDU2Zg== | base64 -d

apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: eHVlZ29kMTIzNDU2Zg==


apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-volume
spec:
  containers:
  - name: myapp
    image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: mysecret
```
进入pod可以看到/etc/secret下有password和username两个文件，查看内容和我们创建的secret内容吻合。
