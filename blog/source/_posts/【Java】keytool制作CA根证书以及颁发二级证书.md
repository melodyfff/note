---
title: 【Java】keytool制作CA根证书以及颁发二级证书
tags: [Java]
date: 2018-4-13
---
> keytool制作CA根证书以及颁发二级证书

```bash
[xinchen@localhost ~]# keytool -help 

 -certreq            生成证书请求  
 -changealias        更改条目的别名  
 -delete             删除条目  
 -exportcert         导出证书  
 -genkeypair         生成密钥对  
 -genseckey          生成密钥  
 -gencert            根据证书请求生成证书  
 -importcert         导入证书或证书链  
 -importkeystore     从其他密钥库导入一个或所有条目  
 -keypasswd          更改条目的密钥口令  
 -list               列出密钥库中的条目  
 -printcert          打印证书内容  
 -printcertreq       打印证书请求的内容  
 -printcrl           打印 CRL 文件的内容  
 -storepasswd        更改密钥库的存储口令 
```

- `keytool` 是jdk提供的工具，该工具名为”keytool“
- alias www.mydomain.com 此处”www.mydomain.com“为别名，可以是任意字符，只要不提示错误即可。因一个证书库中可以存放多个证书，通过别名标识证书。
- `keyalg` RSA 此处”RSA“为密钥的算法。可以选择的密钥算法有：RSA、DSA、EC。
– keysize 4096 此处”4096“为密钥长度。keysize与keyalg默认对应关系： 
    - 2048 (when using -genkeypair and -keyalg is “RSA”) 
    - 1024 (when using -genkeypair and -keyalg is “DSA”) 
    - 256 (when using -genkeypair and -keyalg is “EC”)
- `keypass` mypassword 此处”mypassword “为本条目的密码(私钥的密码)。最好与storepass一致。
- `sigalg` SHA256withRSA 此处”SHA256withRSA“为签名算法。keyalg=RSA时，签名算法有：MD5withRSA、SHA1withRSA、SHA256withRSA、SHA384withRSA、SHA512withRSA。keyalg=DSA时，签名算法有：SHA1withDSA、SHA256withDSA。此处需要注意：MD5和SHA1的签名算法已经不安全。
- `dname` “cn=www.mydomain.com,ou=xxx,o=xxx,l=Beijing,st=Beijing,c=CN” 在此填写证书信息。”CN=名字与姓氏/域名,OU=组织单位名称,O=组织名称,L=城市或区域名称,ST=州或省份名称,C=单位的两字母国家代码”
- `validity` 3650 此处”3650“为证书有效期天数。
- `keystore` `www.xinchen.com_keystore.jks` 此处`www.xinchen.com_keystore.jks`为密钥库的名称。此处也给出绝对路径。默认在当前目录创建证书库。
- `storetype` JKS 此处”JKS “为证书库类型。可用的证书库类型为：JKS、PKCS12等。jdk9以前，默认为JKS。自jdk9开始，默认为PKCS12。
- storepass mypassword 此处”mypassword “为证书库密码(私钥的密码)。最好与keypass 一致。


## 制作CAS证书
```bash
# 密钥生成
# 密码：123456 一路回车
keytool -genkeypair -keyalg RSA -keysize 2048 -sigalg SHA1withRSA -validity 36500 -alias xinchen.sso.com -keystore d:/tomcat.keystore -dname "CN=passport.sso.com,OU=xinchen,O=xc,L=ChonQing,ST=ChonQing,C=CN"

# 证书生成
# 密码：123456
keytool -exportcert -alias xinchen.sso.com -keystore d:/tomcat.keystore  -file d:/tomcat.cer -rfc

# 导入cacerts证书库
# 默认密码：changeit
keytool -import -alias xinchen.sso.com -keystore %JAVA_HOME%\jre\lib\security\cacerts -file d:/tomcat.cer -trustcacerts

# 检查是否导入成功
keytool -list -keystore "%JAVA_HOME%\jre\lib\security\cacerts" | findstr/i server
```