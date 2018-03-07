---
title: 【Java】 Maven多模块项目上传到Sonar扫描问题合集
tags: [Java]
date: 2018-3-7
---

> 上传到Soanr时，项目有单元测试数,但是覆盖率为0

### 修改pom.xml
```xml
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.4.2</version>
                    <configuration>
                        <skipTests>false</skipTests>
                        <testFailureIgnore>true</testFailureIgnore>
                        <includes>
                            <include>**/*Test.java</include>
                        </includes>
                        <excludes>
                            <!--<exclude>**/Abstract*.java</exclude>-->
                            <!--<exclude>**/*Service.java</exclude>-->
                        </excludes>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.jacoco</groupId>
                    <artifactId>jacoco-maven-plugin</artifactId>
                    <version>0.7.9</version>
                    <executions>
                        <execution>
                            <id>pre-test</id>
                            <goals>
                                <goal>prepare-agent</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
```
### 必要时带上参数

```bash
clean cobertura:cobertura -Dcobertura.report.format=xml package -Dmaven.test.failure.ignore=true sonar:sonar -Dsonar.language=java
```

> 多模块项目通过Jenkins构建扫描上传到Sonar时,代码覆盖率很低,只覆盖了单模块的代码,这时可以通过配置JaCoCo解决该问题。
> [参考资料](https://stackoverflow.com/search?q=maven+Multi-module+cover)
 
### 修改pom.xml
```xml
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.7.9</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>report</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

通过`clean org.jacoco:jacoco-maven-plugin:prepare-agent jacoco:report-aggregate install` 可在target下生成`jacoco-aggregate`覆盖相关报告

> 注：jacoco版本低于0.7.9 `jacoco:report-aggregate`参数可能不存在
> 多模块项目整合覆盖率还可以指定`report`目录

```xml
 <properties>
        <sonar.java.coveragePlugin>jacoco</sonar.java.coveragePlugin>
        <sonar.dynamicAnalysis>reuseReports</sonar.dynamicAnalysis>
        <sonar.jacoco.reportPath>${project.basedir}/../target/jacoco.exec</sonar.jacoco.reportPath>
 </properties>        

 <build>
         <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.jacoco</groupId>
                    <artifactId>jacoco-maven-plugin</artifactId>
                    <version>0.7.9</version>
                    <configuration>
                        <destFile>${sonar.jacoco.reportPath}</destFile>
                        <append>true</append>
                    </configuration>
                    <executions>
                        <execution>
                            <goals>
                                <goal>prepare-agent</goal>
                            </goals>
                        </execution>
                        <execution>
                            <id>report</id>
                            <phase>prepare-package</phase>
                            <goals>
                                <goal>report</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </pluginManagement>
 </build>
```

