---
title: 【kubernetes】使用kubeadm从零开始搭建k8s集群
tags: [kubernetes]
date: 2019-4-27
---

# 使用kubeadm从零开始搭建k8s集群

参考文档：

[使用kubeadm安装Kubernetes 1.13](https://www.kubernetes.org.cn/4956.html)

[使用 kubeadm 创建一个 kubernetes 集群](https://yq.aliyun.com/articles/431059)

[kubernetes 1.5.1 安装 ( kubeadm centos7.2 阿里云源)](https://www.jianshu.com/p/4f5066dad9b4)

---

## 前期准备(针对所有节点)
前置条件两台虚拟机 `CentOS Linux release 7.6.1810 (Core)` 配置为 `2C 2G`

分别为
```bash
node1 192.168.201.140
node2 192.168.201.141
```

确保`/etc/hosts`路径下
```
192.168.201.140 node1
192.168.201.141 node2
```

分别在各个节点上设置`hostname`
```bash
# 节点1 master节点
hostnamectl set-hostname node1

# 节点2 node节点
hostnamectl set-hostname node2

# 检查名字是否修改成功
hostname

hostnamectl
```

#### 系统配置

替换`yum`源为阿里源
```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak

# curl -o CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

yum clean all
yum makecache
```

配置`kubernetes`源
```bash
cat /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
```

关闭防火墙
```bash
systemctl stop firewalld
systemctl disable firewalld
```

禁用SELINUX：
```bash
setenforce 0

vi /etc/selinux/config
SELINUX=disabled
```

创建`/etc/sysctl.d/k8s.conf`文件，添加如下内容
```bash
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

使其生效
```bash
modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```




#### kube-proxy开启ipvs的前置条件
由于ipvs已经加入到了内核的主干，所以为kube-proxy开启ipvs的前提需要加载以下的内核模块：
```bash
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack_ipv4
```

在所有的Kubernetes节点node1和node2上执行以下脚本:
```bash
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

确认一下`iptables filter`表中FOWARD链的默认策略(`pllicy`)为`ACCEPT`。
```bash
iptables -nvL
```
---
## 安装docker
参考 ：

[Docker CE 镜像源站](https://yq.aliyun.com/articles/110806?spm=5176.8351553.0.0.479a1991LkyttK)

[Docker 镜像加速器](https://yq.aliyun.com/articles/29941)

[阿里镜像加速](https://cr.console.aliyun.com)

分别在各个节点上安装，安装后`docker`版本信息如下

```bash
# docker version
Client:
 Version:           18.06.3-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        d7080c1
 Built:             Wed Feb 20 02:26:51 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.3-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       d7080c1
  Built:            Wed Feb 20 02:28:17 2019
  OS/Arch:          linux/amd64
  Experimental:     false

```

替换国内镜像库
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ID.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker.service
```

## 安装k8s相关

```bash
sudo yum install socat kubelet kubeadm kubectl kubernetes-cni -y

sudo systemctl enable kubelet.service && sudo systemctl start kubelet.service
```

#### 关闭Swap
```bash
swapoff -a
```

修改 `/etc/fstab` 文件，注释掉 `SWAP` 的自动挂载，使用`free -m`确认`swap`已经关闭。 swappiness参数调整，修改`/etc/sysctl.d/k8s.conf`添加下面一行：
```bash
vm.swappiness=0
```

执行`sysctl -p /etc/sysctl.d/k8s.conf`使修改生效。

如果还有其他服务在运行，关闭`swap`可能对其他服务造成影响，可采用`kubelet`的启动参数的形式
修改/etc/sysconfig/kubelet，加入
```bash
KUBELET_EXTRA_ARGS=--fail-swap-on=false
```
---
## 使用kubeadm init初始化集群
所有节点执行开机启动`kubelet`
```bash
systemctl enable kubelet.service
```

接下来使用kubeadm初始化集群，选择node1作为Master Node，在node1上执行下面的命令：

```bash
kubeadm init --kubernetes-version=v1.14.1 --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.201.140 --ignore-preflight-errors=Swap
```

当然可能因为镜像拉取不下来，可提前执行镜像拉取脚本:
```bash
#!/usr/bin/env bash

for image in `kubeadm config images list`
do
	image_mirror=gcr.akscn.io/google_containers/${image##*/}
	echo "pull image $image_mirror from dockerhub";
	docker pull $image_mirror;
	docker tag $image_mirror $image;
	docker rmi $image_mirror;
	echo "pull image $image done."
	docker images | grep "${image##*/}"
done

```

查看集群状态，确认个组件都处于healthy状态。
```bash
kubectl get cs

NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
```

集群初始化如果遇到问题，可以使用下面的命令进行清理：
```bash
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
```
---
## 安装Pod Network
接下来安装flannel network add-on：

注，可能需要提前修改
```bash
...
"Network": "192.168.0.0/16"
...
```

以及多网卡得指定–iface参数指定集群主机内网网卡的名称
```bash
...
containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.10.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=ens33
...
```


```bash
mkdir -p ~/k8s/
cd ~/k8s
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f  kube-flannel.yml

clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created
```


使用`kubectl get pod –all-namespaces -o wide`确保所有的Pod都处于Running状态。

```bash
[root@node1 k8s]# kubectl get pod --all-namespaces -o wide
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE    IP                NODE    NOMINATED NODE   READINESS GATES
kube-system   coredns-fb8b8dccf-6r4qr                 1/1     Running   1          135m   192.168.0.11      node1   <none>           <none>
kube-system   coredns-fb8b8dccf-6rnv2                 1/1     Running   1          135m   192.168.0.10      node1   <none>           <none>
kube-system   etcd-node1                              1/1     Running   1          133m   192.168.201.140   node1   <none>           <none>
kube-system   kube-apiserver-node1                    1/1     Running   1          134m   192.168.201.140   node1   <none>           <none>
kube-system   kube-controller-manager-node1           1/1     Running   5          134m   192.168.201.140   node1   <none>           <none>
kube-system   kube-flannel-ds-amd64-7bdf7             1/1     Running   1          135m   192.168.201.140   node1   <none>           <none>
kube-system   kube-proxy-lsxbd                        1/1     Running   0          124m   192.168.201.140   node1   <none>           <none>
kube-system   kube-scheduler-node1                    1/1     Running   4          134m   192.168.201.140   node1   <none>           <none>

```

#### master node参与工作负载

```bash

# 查看master节点的调度情况
kubectl describe node node1 | grep Taint
Taints:             node-role.kubernetes.io/master:NoSchedule

# 使node1参与工作负载
kubectl taint nodes node1 node-role.kubernetes.io/master-
node "node1" untainted

```

测试DNS
```bash
kubectl run curl --image=radial/busyboxplus:curl -it
kubectl run --generator=deployment/apps.v1beta1 is DEPRECATED and will be removed in a future version. Use kubectl create instead.
If you don't see a command prompt, try pressing enter.
[ root@curl-5cc7b478b6-r997p:/ ]$ 
```

进入后执行nslookup kubernetes.default确认解析正常:
```bash
nslookup kubernetes.default
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
```
---
## 向Kubernetes集群中添加Node节点
下面我们将node2这个主机添加到Kubernetes集群中，因为我们同样在node2上的kubelet的启动参数中去掉了必须关闭swap的限制，所以同样需要–ignore-preflight-errors=Swap这个参数。 在node2上执行:
```bash
kubeadm join 192.168.201.140:6443 --token kelv2x.u1ot0biiesbh5174 \
    --discovery-token-ca-cert-hash sha256:b2aec674ac027e03ed310f02a907f310075c41e1a14bc06038e73a836b03410e
```

查看集群节点
```bash
# kubectl get nodes
NAME    STATUS   ROLES    AGE    VERSION
node1   Ready    master   143m   v1.14.1
node2   Ready    <none>   14s    v1.14.1
```

#### 移除节点
在`master`上执行
```bash
kubectl drain node2 --delete-local-data --force --ignore-daemonsets
kubectl delete node node2
```

在`node2`上执行
```bash
kubeadm reset -f
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
```

## kube-proxy开启ipvs
修改ConfigMap的`kube-system/kube-proxy`中的`config.conf, mode: “ipvs”`：

```bash
kubectl edit cm kube-proxy -n kube-system
```

之后重启各个节点上的kube-proxy pod：
```bash
# kubectl get pod -n kube-system | grep kube-proxy | awk '{system("kubectl delete pod "$1" -n kube-system")}'
kube-proxy-pf55q                1/1     Running   0          9s
kube-proxy-qjnnc                1/1     Running   0          14s

# kubectl logs kube-proxy-pf55q -n kube-system
I1208 06:12:23.516444       1 server_others.go:189] Using ipvs Proxier.
W1208 06:12:23.516738       1 proxier.go:365] IPVS scheduler not specified, use rr by default
I1208 06:12:23.516840       1 server_others.go:216] Tearing down inactive rules.
I1208 06:12:23.575222       1 server.go:464] Version: v1.13.0
I1208 06:12:23.585142       1 conntrack.go:52] Setting nf_conntrack_max to 131072
I1208 06:12:23.586203       1 config.go:202] Starting service config controller
I1208 06:12:23.586243       1 controller_utils.go:1027] Waiting for caches to sync for service config controller
I1208 06:12:23.586269       1 config.go:102] Starting endpoints config controller
I1208 06:12:23.586275       1 controller_utils.go:1027] Waiting for caches to sync for endpoints config controller
I1208 06:12:23.686959       1 controller_utils.go:1034] Caches are synced for endpoints config controller
I1208 06:12:23.687056       1 controller_utils.go:1034] Caches are synced for service config controller
```

## Dashboard
参考

[Kubernetes Dashboard的安装与坑](https://www.jianshu.com/p/c6d560d12d50)
