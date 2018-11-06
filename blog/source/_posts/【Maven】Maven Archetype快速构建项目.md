---
title: 【Maven】Maven Archetype快速构建项目
tags: [Maven]
date: 2018-11-07
---

## Maven模块快速构建项目
> 官方: http://maven.apache.org/guides/introduction/introduction-to-archetypes.html  
http://maven.apache.org/archetype/maven-archetype-plugin/plugin-info.html  
> 参考: https://www.yiibai.com/maven/create-a-project-with-maven-template.html

## 常用模板

- `maven-archetype-webapp -> Java Web Project`  
- `maven-archetype-quickstart -> Java Project`

## 构建指令


### 根据提示构建

```bash
$ mvn archetype:generate

# 输出如下,根据提示创建即可 常用为 7 、 10
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] >>> maven-archetype-plugin:3.0.1:generate (default-cli) > generate-sources @ standalone-pom >>>
[INFO]
[INFO] <<< maven-archetype-plugin:3.0.1:generate (default-cli) < generate-sources @ standalone-pom <<<
[INFO]
[INFO]
[INFO] --- maven-archetype-plugin:3.0.1:generate (default-cli) @ standalone-pom ---
[INFO] Generating project in Interactive mode
[WARNING] No archetype found in remote catalog. Defaulting to internal catalog
[INFO] No archetype defined. Using maven-archetype-quickstart (org.apache.maven.archetypes:maven-archetype-quickstart:1.0)
Choose archetype:
1: internal -> org.apache.maven.archetypes:maven-archetype-archetype (An archetype which contains a sample archetype.)
2: internal -> org.apache.maven.archetypes:maven-archetype-j2ee-simple (An archetype which contains a simplifed sample J2EE application.)
3: internal -> org.apache.maven.archetypes:maven-archetype-plugin (An archetype which contains a sample Maven plugin.)
4: internal -> org.apache.maven.archetypes:maven-archetype-plugin-site (An archetype which contains a sample Maven plugin site.
      This archetype can be layered upon an existing Maven plugin project.)
5: internal -> org.apache.maven.archetypes:maven-archetype-portlet (An archetype which contains a sample JSR-268 Portlet.)
6: internal -> org.apache.maven.archetypes:maven-archetype-profiles ()
7: internal -> org.apache.maven.archetypes:maven-archetype-quickstart (An archetype which contains a sample Maven project.)
8: internal -> org.apache.maven.archetypes:maven-archetype-site (An archetype which contains a sample Maven site which demonstrates
      some of the supported document types like APT, XDoc, and FML and demonstrates how
      to i18n your site. This archetype can be layered upon an existing Maven project.)
9: internal -> org.apache.maven.archetypes:maven-archetype-site-simple (An archetype which contains a sample Maven site.)
10: internal -> org.apache.maven.archetypes:maven-archetype-webapp (An archetype which contains a sample Maven Webapp project.)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 7: 

```

### 快速构建jar模板项目

```bash
$ mvn archetype:generate -DgroupId=com.test -DartifactId=test -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

### 快速构建war模板项目
```bash
$ mvn archetype:generate -DgroupId=com.test -DartifactId=test -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
```




