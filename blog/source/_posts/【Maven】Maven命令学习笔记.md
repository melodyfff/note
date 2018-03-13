---
title: 【Maven】Maven命令学习笔记
tags: [Maven]
date: 2018-3-13
---

> usage: mvn [options] [<goal(s)>] [<phase(s)>]

|Options|Description|Note
|:-----|:---------------|:-------|
|-am,--also-make| If project list is specified, also build projects required by the list|同时处理选定模块所依赖的模块 [参考](https://www.jianshu.com/p/29bb16755883)|
|-amd,--also-make-dependents|If project list is specified, also build projects that depend on projects on the list|同时处理依赖选定模块的模块 [参考](https://www.jianshu.com/p/29bb16755883)|
|-B,--batch-mode| Run in non-interactive (batch) mode|在非交互（批处理）模式下运行,能够避免一些需要人工参与交互而造成的挂起状态 [参考](http://blog.csdn.net/lovesomnus/article/details/50150011)|
|-b,--builder `<arg>`|The id of the build strategy to use|被使用的构建策略的id|
|-C,--strict-checksums|Fail the build if checksums don't match|如果校验(checksums)不匹配的话,构建失败 [参考](https://www.baidu.com/link?url=_xfOc7VywHu2QNuxroSiMgRm3qCKDXA7K7Wm2v8Br6m10sOR6_QXEvxlnewN6t1m44y_5V5cv4mGkuY4gowab_&wd=&eqid=ea3cc4b80004b659000000035aa76309)|
|-c,--lax-checksums|Warn if checksums don't match|如果校验(checksums)不匹配的话,产生告警 [参考](https://www.baidu.com/link?url=_xfOc7VywHu2QNuxroSiMgRm3qCKDXA7K7Wm2v8Br6m10sOR6_QXEvxlnewN6t1m44y_5V5cv4mGkuY4gowab_&wd=&eqid=ea3cc4b80004b659000000035aa76309)|
|-cpu,--check-plugin-updates|Ineffective, only kept for backward compatibility|对任何相关的注册插件,强制进行最新检查(即使项目POM里明确规定了Maven插件版本,还是会强制更新)|
|-D,--define `<arg>`|Define a system property|定义一个系统属性|
| -e,--errors| Produce execution error messages|控制Maven的日志级别,产生执行错误消息|
|-emp,--encrypt-master-password `<arg>`|Encrypt master security password|加密主安全密码 加密主安全密码,存储到Maven settings文件里|
|-ep,--encrypt-password `<arg>`|Encrypt server password|加密服务器密码,存储到Maven settings文件里|
|-f,--file `<arg>` |Force the use of an alternate POM file (or directory with pom.xml).|强制使用备用POM文件（或使用pom.xml的目录）|
|-fae,--fail-at-end |Only fail the build afterwards;allow all non-impacted builds to continue|仅影响构建结果,允许不受影响的构建继续|
|-ff,--fail-fast|Stop at first failure in reactorized builds|遇到构建失败就直接退出|
|-fn,--fail-never |NEVER fail the build, regardless of project result|不管项目结果如何，永远不要让构建失败|
| -gs,--global-settings `<arg>`|Alternate path for the global settings file|全局设置文件的备用路径|
|-h,--help|Display help information|显示帮助信息|
|-l,--log-file `<arg>`| Log file to where all build output will go|配置构建日志输出路径|
|-llr,--legacy-local-repository|Use Maven 2 Legacy Local Repository behaviour, ie no use of remote.repositories. Can also be activated by using -Dmaven.legacyLocalRepo=true|使用Maven 2 Legacy Local Repository行为，即不使用remote.repositories,也可使用 -Dmaven.legacyLocalRepo = true|
|-N,--non-recursive|Do not recurse into sub-projects|不递归子模块|
| -npr,--no-plugin-registry |Ineffective, only kept for backward compatibility|对插件版本不使用~/.m2/plugin-registry.xml(插件注册表)里的配置|
|-npu,--no-plugin-updates|Ineffective, only kept for backward compatibility||
| -o,--offline|Work offline|不联网更新依赖|
|-P,--activate-profiles `<arg>`|Comma-delimited list of profiles to activate|用逗号分隔的激活配置文件列表|
|-pl,--projects `<arg>`| Comma-delimited list of specified reactor projects to build instead of all projects. A project can be specified by [groupId]:artifactId or by its relative path|在指定模块上执行命令|
|-q,--quiet|Quiet output - only show errors|只输出错误日志|
|-rf,--resume-from `<arg>`|Resume reactor from specified project||
|-s,--settings `<arg>`|Alternate path for the user settings file|用户配置文件的备用路径|
|-T,--threads `<arg>`|Thread count, for instance 2.0C where C is core multiplied| 多线程构建,线程数，例如2.0C，其中C是内核的乘积|
|-t,--toolchains `<arg>`|Alternate path for the user toolchains file||
|-U,--update-snapshots|Forces a check for missing releases and updated snapshots on remote repositories|强制检查远程存储库中的缺失版本和更新快照|
|-up,--update-plugins| Ineffective, only kept for backward compatibility|强制更新插件|
|-V,--show-version|Display version information WITHOUT stopping build|显示版本信息而不停止构建|
|-v,--version|Display version information|显示版本信息|
|-X,--debug| Produce execution debug output|输出DEBUG日志|




