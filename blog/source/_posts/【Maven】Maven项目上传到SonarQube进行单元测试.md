---
title: 【Maven】Maven项目上传到SonarQube进行单元测试
tags: [Maven]
date: 2017-9-26
---

搭建完SonarQube环境后，将`Maven Multi-Module`上传至Sonar,发现单元测试和覆盖率为空。  

可用以下命令： 

```bash
mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install -Dmaven.test.failure.ignore=true

mvn sonar:sonar -Dsonar.host.url=http://ip:port

```




