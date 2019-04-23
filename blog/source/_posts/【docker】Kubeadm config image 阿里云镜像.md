---
title: 【docker】Kubeadm config image 阿里云镜像.md
tags: [docker]
date: 2019-4-23
---

# Kubeadm config image 阿里云镜像

参考：

[谷歌k8s.gcr.io镜像快速传入阿里云镜像源的解决方案（需浏览器科学上网）](https://zhuanlan.zhihu.com/p/52122243)

[kubeadm config image 阿里云镜像](https://www.jianshu.com/p/bd97c06bd5b0)

国内部署`k8s`时去拉取镜像的友好脚本

```bash
images=(
    kube-apiserver:v1.13.2
    kube-controller-manager:v1.13.2
    kube-scheduler:v1.13.2
    kube-proxy:v1.13.2
    pause:3.1
    etcd:3.2.24
    coredns:1.2.6
)
for imageName in ${images[@]} ; do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/${imageName}
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${imageName}
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${imageName}
done
```

