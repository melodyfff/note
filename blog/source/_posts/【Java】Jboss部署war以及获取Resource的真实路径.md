---
title: 【Java】Jboss部署war以及获取Resource的真实路径
tags: [java]
date: 2019-7-19
---

# Jboss部署war以及获取Resource的真实路径

最近在将一个`SpringBoot`项目打成`war`包部署到`Jboss`中，中途遇到一些问题记录。

## Jboss上部署war


普通的`SpringBoot`项目目录结构如下

```bash
.
├── src
     └── main
        ├── java
        └── resources
```

当我们打出`war`包后，想在`Jboss`中部署时需要添加`jboss-deployment-structure.xml`文件

关于此文件的配置可参考[Jboss as 7 Developer Guide](https://docs.jboss.org/author/display/AS7/Developer+Guide)

加入后目录结构如下

```bash
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   ├── resources
│   │   │   ├── application.yml
│   │   │   └── META-INF
│   │   └── webapp
│   │       └── WEB-INF
│   │           ├── jboss-deployment-structure.xml
│   │           └── jboss-web.xml

```

**jboss-deployment-structure.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jboss-deployment-structure>
	<deployment>
            <!-- 需要排除的 -->
            <exclusions>
                <module name="javax.validation.api" />
                <module name="org.hibernate.validator" />
                <!--Log4j exclude-->
                <module name="org.slf4j" />
                <module name="org.slf4j.impl" />
            </exclusions>

            <!-- 需要依赖的模块 -->
            <dependencies>
                <!-- This one always goes last. -->
                <module name="javax.api" export="true"/>
            </dependencies>
	</deployment>
</jboss-deployment-structure>
```

**jboss-web.xml**
```xml
<!DOCTYPE jboss-web PUBLIC "-//JBoss//DTD Web Application 5.0//EN"  
"http://www.jboss.org/j2ee/dtd/jboss-web_5_0.dtd"> 
<jboss-web>
	<context-root>app</context-root>
</jboss-web>

```

## Jboss中获取Resource的真实路径

在使用过程中，因为在`Resouce`中放了一些文件，需要去获取文件内容

最开始使用如下方法去获取

```java
// vfs:/content/app.war/WEB-INF/classes/data/data.yaml
new ClassPathResource("data/data.yaml")).getURI()
```

当我尝试创建一个`File`时报错找不到

因此借助`JBoss VFS`去获取当前资源的真实路径


`MAVEN`中添加
```xml
        <!-- JBoss is using Virtual File System (VFS) -->
        <dependency>
            <groupId>org.jboss</groupId>
            <artifactId>jboss-vfs</artifactId>
            <version>3.2.14.Final</version>
        </dependency>
```


具体使用：

```java

 VirtualFile content = (VirtualFile) this.getClass().getClassLoader().getResource("data/data.yaml").getContent();

// $JBOSS_HOME/tmp/vfs/temp/tempc755413fe36e407c/app.war-64dfd9c1b9e1463e/WEB-INF/classes/data/data.yaml
 String realPath = content.getPhysicalFile().getPath()
```




## 参考

[JBoss VFS](https://github.com/jbossas/jboss-vfs/blob/master/src/main/java/org/jboss/vfs/VFSUtils.java)

[VFS3 User Guide](https://developer.jboss.org/wiki/VFS3UserGuide)

[Jboss as 7 Developer Guide](https://docs.jboss.org/author/display/AS7/Developer+Guide)

[StackOverflow:Not getting absolute file path from resources](https://stackoverflow.com/questions/22567174/not-getting-absolute-file-path-from-resources)

