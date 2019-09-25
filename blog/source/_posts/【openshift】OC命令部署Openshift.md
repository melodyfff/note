---
title: 【openshift】OC命令部署Openshift
tags: [openshift]
date: 2019-9-25
---

# OC命令部署Openshift

```bash
# install openshift
wget -c https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz 
tar -zxvf openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
mv openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit/oc /usr/local/bin/
mv openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit/kubectl /usr/local/bin/

# 生成配置文件,改了hosts之后最好重启一下在进行后面的操作
cat << EOF >> /etc/hosts
172.26.252.34 okd.xc
127.0.0.1 okd.xc
EOF

# 删除本地可能之前生成的配置文件,并生成配置文件
rm -rf openshift.local.clusterup
oc cluster up --skip-registry-check=true --public-hostname=okd.xc --write-config=true

# 替换 subdomain
sed -i 's/router\.default\.svc\.cluster\.local/pod\.xc/g' openshift.local.clusterup/openshift-controller-manager/master-config.yaml
sed -i 's/router\.default\.svc\.cluster\.local/pod\.xc/g' openshift.local.clusterup/openshift-apiserver/master-config.yaml
sed -i 's/127\.0\.0\.1\.nip\.io/pod\.xc/g' openshift.local.clusterup/openshift-controller-manager/master-config.yaml
sed -i 's/127\.0\.0\.1\.nip\.io/pod\.xc/g' openshift.local.clusterup/openshift-apiserver/master-config.yaml

# 启动 OKD 集群
oc cluster up --skip-registry-check=true --public-hostname=okd.xc
oc login -u system:admin
oc adm policy add-cluster-role-to-user admin xinchen
```