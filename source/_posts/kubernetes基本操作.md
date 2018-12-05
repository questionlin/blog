---
title: kubernetes基本操作
date: 2018-12-05 11:45:16
tags: 运维
id: 1543981565
---
kubernetes 官方提供了非常好的交互学习平台 https://kubernetes.io/docs/tutorials/ ，这篇文章是当作一个命令参考，毕竟这些命令不常用，容易忘。

# 基本概念
- Cluster 集群，一个集群里有一个 Master 和数个 Node
- Node 通常拿一台物理机座位图一个 Node，也可以用虚拟机
- Pod 是一个 docker 实例，一个 Node 里有一个或多个 Pod
- Deployment 一个发布，可以包含一个或多个 Pod
- Service deployment 暴露出来的服务，内置了负载分发

# 创建集群
```sh
kubernetes start # 创建集群

$ kubectl get nodes # 查看所有 nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    19s       v1.10.0
```

# 发布应用
```sh
$ kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --port=8080 # 启动一个应用
deployment.apps/kubernetes-bootcamp created

$ kubectl get deployments # 查看所有应用
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           50s

$ kubectl proxy # 转发本地端口到 pod 的端口
```

# 查看应用
```sh
$ kubectl get pods # 查看所有 pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5c69669756-85r9w   0/1       Pending   0          1s

$ kubectl describe pods # 查看 pods 详情
Name:           kubernetes-bootcamp-5c69669756-85r9w
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Wed, 05 Dec 2018 05:55:14 +0000
Labels:         pod-template-hash=1725225312
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.4
Controlled By:  ReplicaSet/kubernetes-bootcamp-5c69669756
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://a7a117005de01756ff3eb0800b91ef089810db413961c86d14fdf9cd8c451754
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
  .
  .
  .

$ kubectl logs kubernetes-bootcamp-5c69669756-85r9w # pod 的访问日志
$ kubectl exec kubernetes-bootcamp-5c69669756-85r9w env # 在 pod 里执行 shell（查看环境变量）
$ kubectl exec -ti kubernetes-bootcamp-5c69669756-85r9w bash # 登录到 pod 里
```

# 暴露应用
## 创建服务
```sh
$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080 # 暴露服务

$ kubectl get services
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP          5m
kubernetes-bootcamp   NodePort    10.102.235.226   <none>        8080:32115/TCP   4m

$ kubectl describe services/kubernetes-bootcamp # 查看服务详情
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.102.235.226
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32115/TCP
Endpoints:                172.18.0.4:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

$curl 172.17.0.49:32115 # 服务已经暴露在32115端口
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-hh4k7 | v=1
```

## 使用 label
```sh
$ kubectl describe deployment # 查看服务发布
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Wed, 05 Dec 2018 06:07:06 +0000
Labels:                 run=kubernetes-bootcamp
Annotations:            deployment.kubernetes.io/revision=1
Selector:               run=kubernetes-bootcamp
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=kubernetes-bootcamp
  Containers:
   kubernetes-bootcamp:
    Image:        gcr.io/google-samples/kubernetes-bootcamp:v1
    Port:         8080/TCP
.
.
.

$ kubectl get pods -l run=kubernetes-bootcamp # 只列出某个 label 的 pods
$ kubectl get services -l run=kubernetes-bootcamp # 只列出某个 label 的 services
$ kubectl label pod $POD_NAME app=v1 # 给 deployment 改 label
```

## 删除 services
```sh
$ kubectl delete service -l run=kubernetes-bootcamp # 删除 services
```
删除 services 只会删除转发，其对应的 pod 还在运行，还可以用 kubectl exec 进行交互

# 扩展应用
## 扩展发布
```sh
$ kubectl scale deployments/kubernetes-bootcamp --replicas=4 # 把 deployment 扩展为4个实例

$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4         4         4            4           3m

$ kubectl get pods -o wide
NAME                                   READY     STATUS    RESTARTS   AGE       IP           NODE
kubernetes-bootcamp-5c69669756-4lzwf   1/1       Running   0          2m        172.18.0.7   minikube
kubernetes-bootcamp-5c69669756-hbrb8   1/1       Running   0          3m        172.18.0.3   minikube
kubernetes-bootcamp-5c69669756-rr92t   1/1       Running   0          2m        172.18.0.6   minikube
kubernetes-bootcamp-5c69669756-z4ld6   1/1       Running   0          2m        172.18.0.5   minikube
```

## 缩减发布
```sh
$ kubectl scale deployments/kubernetes-bootcamp --replicas=2 # 把 deployment 缩减为2个实例
```

# 更新应用
## 替换镜像
```sh
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2 # 换镜像

$ kubectl rollout status deployments/kubernetes-bootcamp # 查看新镜像配置状态
deployment "kubernetes-bootcamp" successfully rolled out

$ kubectl describe pods
Name:           kubernetes-bootcamp-7799cbcb86-prj8w
Namespace:      default
Node:           minikube/172.17.0.81
Start Time:     Wed, 05 Dec 2018 06:33:29 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.9
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://76d31785dc318d21ba1c822c64f24c7f58701dfe761d2c1cb8df03002d568732
    Image:          jocatalin/kubernetes-bootcamp:v2 # 镜像已经替换
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
.
.
.
```

## 回滚
```sh
$ kubectl rollout undo deployments/kubernetes-bootcamp # 回滚
```