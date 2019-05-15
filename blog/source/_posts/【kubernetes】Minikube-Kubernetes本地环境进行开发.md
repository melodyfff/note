---
title: 【kubernetes】Minikube-Kubernetes本地环境进行开发
tags: [kubernetes]
date: 2019-5-15
---

# Minikube-Kubernetes本地环境进行开发

## 使用Minikube

**启动Minikube**
```bash
# 启动
minkube start

# 检查状态
minikube status

host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100


# 访问面板
$ minikube dashboard

# 访问web前端
$ kubectl proxy

```


**获取命名空间**
```bash
# kubectl get namespaces
$ kubectl get ns
NAME              STATUS   AGE
default           Active   156m
kube-node-lease   Active   156m
kube-public       Active   156m
kube-system       Active   156m
```

**获取集群信息**
```bash
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-
dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

**获取节点信息**
```bash
$ kubectl get node

NAME       STATUS   ROLES    AGE    VERSION
minikube   Ready    master   162m   v1.14.1
```

**运行nginx**
```bash
# 运行nginx
$ kubectl run nginx --image=nginx:1.16 --port=80 --labels="app=nginx,env=dev"

# 增加新标签
# kubectl label pod $POD_NAME app2=test

# 发布服务采用对外暴露节点
$ kubectl expose deployment nginx --type=NodePort
```

**获取pods**
```bash

$ kubectl get pods
# 根据标签查询
# kubectl get pods -l app=nginx

NAME                    READY   STATUS    RESTARTS   AGE
nginx-fcb945956-t8bqq   1/1     Running   0          67s

# 获取pod详细信息
# kubectl describe pods -l app=nginx
```

**查看Pod日志**
```bash
kubectl logs $POD_NAME
```

**进入Pod中**
```bash
# 查看环境
# kubectl exec $POD_NAME env

$ kubectl exec -ti nginx-fcb945956-t8bqq /bin/bash

```

**获取deployment**
```bash
$ kubectl get deployment

NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           18m


# 获取描述
$ kubectl describe deployment

Name:                   nginx
Namespace:              default
CreationTimestamp:      Wed, 15 May 2019 23:29:23 +0800
Labels:                 app=nginx
                        env=dev
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx,env=dev
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
           env=dev
  Containers:
   nginx:
    Image:        nginx:1.16
    Port:         8081/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-fcb945956 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  18m   deployment-controller  Scaled up replica set nginx-fcb945956 to 1
```

**获取svc(services)**
```bash
# kubectl get svc
$ kubectl get services

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        3h8m
nginx        NodePort    10.105.192.17   <none>        80:31199/TCP   5m9s

# 获取详情
$ kubectl describe services

Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        8443/TCP
Endpoints:         192.168.99.100:8443
Session Affinity:  None
Events:            <none>


Name:                     nginx
Namespace:                default
Labels:                   app=nginx
                          env=dev
Annotations:              <none>
Selector:                 app=nginx,env=dev
Type:                     NodePort
IP:                       10.105.192.17
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31199/TCP
Endpoints:                172.17.0.5:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

## 使用Minikube获取服务访问地址
```bash
# 获取服务url
$ minikube service nginx --url

# 访问测试
$ curl $(minikube service nginx --url)
```

## 删除服务
```bash
# kubectl delete deployments --all
$ kubectl delete deployments -l app=nginx

# kubectl delete pods --all
$ kubectl delete pods -l app=nginx

# 删除service
kubectl delete service -l app=nginx
```

## 停止Minikube
```bash
minikube stop
```

## 参考

[K8S官网文档](https://kubernetes.io/docs/concepts/services-networking/service/)

[Minikube - Kubernetes本地实验环境](https://yq.aliyun.com/articles/221687)

[Minikube：使用 Kubernetes 进行本地开发](https://linux.cn/article-8847-1.html)

[Kubernetes基础：查看状态、管理服务](https://www.cnblogs.com/cocowool/p/k8s_describe_node_pod_and_service.html)