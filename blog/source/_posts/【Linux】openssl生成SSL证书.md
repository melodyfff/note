---
title: 【Linux】openssl生成SSL证书
tags: [Linux]
date: 2019-11-2
---
# openssl生成SSL证书

## 基础知识

`https = http + ssl`

| 密码体制  | 组成 | 加密| 解密|
| ------------- | ------------- |------------- |------------- |
| 公钥私钥  | 公钥+私钥+加密解密算法  |公钥+加密算法|私钥+解密算法|
| 对称加密  | 密匙+加密解密算法  |密匙+加密算法|密匙+解密算法|
| 非对称加密  | 加密匙+解密匙+加密解密算法  |加密匙+加密算法|解密匙+解密算法|
| RSA  | 公钥+私钥+加密解密算法  |公钥或私钥+加密算法|公钥或私钥+解密算法|
| 签名加加密  | 签名hash+内容+加密解密算法  |签名hash+内容+加密算法|签名hash+内容+解密算法|

## openssl生成SSL证书

### 生成自定义CA自签名证书(根证书)
`opesnssl`全局配置一般在`/usr/lib/ssl/openssl.cnf`

通常生成方法
```bash
openssl genrsa -des3 -out ca.key 1024
openssl rsa -in ca.key -out ca.key
# 生成10年有效期的CA证书, 可以加-config /etc/ssl/openssl.cnf 指定配置
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt 

# 合并证书
cat ca.key ca.crt > ca.pem
```

快速生成方法
```bash
openssl genrsa -out ca.com.key 2048
openssl req -new -x509 -nodes -days 3650 -key ca.com.key -out ca.com.crt -subj /C=CN/ST=Chongqing/L=Chongqing/O=org/OU=OrgUnit/CN=ca.com/emailAddress=test@test.com

```

### 服务端证书签发
通常方法

```bash
# 生成服务器端的私钥(key文件)
openssl genrsa -des3 -out server.key 1024
# 去除口令
openssl rsa -in server.key -out server.key

# 生成服务器端证书签名请求文件(csr文件) 生成的csr文件交给CA签名后形成服务端自己的证书
openssl req -new -key server.key -out server.csr

# 签发证书
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt

openssl x509 -req -days 3650 -in server.csr -CA ca.pem -CAkey ca.pem -CAcreateserial -out server.crt
```

快速方法
```bash
# 生成 *.x.com 泛域名证书 (set_serial: 101)
openssl req -newkey rsa:2048 -days 3650 -nodes -keyout x.com.key -out x.com.csr -subj /C=CN/ST=Chongqing/L=Chongqing/O=org/OU=OrgUnit/CN=*.x.com/emailAddress=test@test.com

cat << EOF > v3.ext
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = *.x.com
EOF

openssl x509 -req -in x.com.csr -days 3650 -CA ca.com.crt -CAkey ca.com.key -set_serial 101 -out x.com.crt -extfile v3.ext

```

## 根证书安装
### windows
直接双击`*.crt`文件安装,注意在选择证书存储的时候选择 `受信任的根证书颁发机构`
### ubuntu
```bash
sudo su -
# curl http://172.2.2.2/ca.com.crt -o /usr/local/share/ca-certificates/ca.com.crt
mv ca.com.crt /usr/local/share/ca-certificates/ca.com.crt
update-ca-certificates
```
### centos
```bash
# curl http://172.2.2.2/ca.com.crt -o /etc/pki/ca-trust/source/anchors/ca.com.crt
mv ca.com.crt /etc/pki/ca-trust/source/anchors/ca.com.crt
update-ca-trust
```
### Java 中导入根证书
```bash
cd $JAVA_HOME
./jre/bin/keytool -keystore jre/lib/security/cacerts -importcert -alias ca.com -file  ca.com.crt
#Java 证书管理keystore默认密码为 changeit
```