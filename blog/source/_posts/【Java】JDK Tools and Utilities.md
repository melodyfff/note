---
title: 【Java】JDK Tools and Utilities
tags: [java]
date: 2019-7-21
---

# JDK Tools and Utilities

`JDK`提供了一系列工具给开发者使用

这里介绍一些常用，其他部分可参考官网

---

## Basic Tools

**jar**

```
```bash
用法: jar {ctxui}[vfmn0PMe] [jar-file] [manifest-file] [entry-point] [-C dir] files ... 
选项:
-c  创建新档案
-t  列出档案目录                                                                                                                                                                      
-x  从档案中提取指定的 (或所有) 文件                                                                                                                                                  
-u  更新现有档案                                                                                                                                                                      
-v  在标准输出中生成详细输出                                                                                                                                                          
-f  指定档案文件名                                                                                                                                                                    
-m  包含指定清单文件中的清单信息                                                                                                                                                      
-n  创建新档案后执行 Pack200 规范化                                                                                                                                                   
-e  为捆绑到可执行 jar 文件的独立应用程序 指定应用程序入口点                                                                                                                                                                
-0  仅存储; 不使用任何 ZIP 压缩
-P  保留文件名中的前导 '/' (绝对路径) 和 ".." (父目录) 组件                                                                                                                           
-M  不创建条目的清单文件                                                                                                                                                              
-i  为指定的 jar 文件生成索引信息                                                                                                                                                     
-C  更改为指定的目录并包含以下文件如果任何文件为目录, 则对其进行递归处理。清单文件名,档案文件名和入口点名称的指定顺序与 'm', 'f' 和 'e'标记的指定顺序相同。

                                                                                                                    
示例 1: 将两个类文件归档到一个名为 classes.jar 的档案中:                                                                                                                                     
		jar cvf classes.jar Foo.class Bar.class                                                                                                                                        
示例 2: 使用现有的清单文件 'mymanifest' 并将 foo/ 目录中的所有文件归档到 'classes.jar' 中:                                                                                                                                  
		jar cvfm classes.jar mymanifest -C foo/ .   


# 常用方法

# 解压`jar/war`到当前目录
jar -xvf source.jar
# 打包不进行压缩
jar -cvfM0 source.jar ./              
```

**javap**

```bash
# -c 对代码进行反编译，-l 输出行号和本地变量表
$ javap -c -l $class
```



## Java Troubleshooting, Profiling, Monitoring and Management Tools

**jcmd**
```bash

# 列出当前运行的所有虚拟机
$ jcmd -l 

# 针对每一个虚拟机，可以使用help命令列出该虚拟机支持的所有命令
$ jcmd [pid] help

# 查看虚拟机启动时间VM.uptime
$ jcmd [pid] VM.uptime

# 打印线程栈信息Thread.print
$ jcmd [pid] Thread.print

# 查看系统中类统计信息GC.class_histogram
$ jcmd [pid] GC.class_histogram

# 导出堆信息GC.heap_dump  这个命令功能和 jmap -dump 功能一样
$ jcmd [pid] GC.heap_dump /tmp/dump

# 获取系统Properties内容VM.system_properties
$ jcmd [pid] VM.system_properties

# 获取启动参数VM.flags
$ jcmd [pid] VM.flags

# 获取所有性能相关数据PerfCounter.print
$ jcmd [pid] PerfCounter.print



```


## Monitoring Tools

**jps**

`jps（JVM Process Status Tool）`可以列出正在运行的虚拟机进程，并显示虚拟机执行主类`（Main Class,main()函数所在的类）`名称以及这些进程的本地虚拟机唯一ID`（Local Virtual Machine Identifier,LVMID）`。

虽然功能比较单一，但它是使用频率最高的JDK命令行工具，因为其他的JDK工具大多需要输入它查询到的LVMID来确定要监控的是哪一个虚拟机进程。

对于本地虚拟机进程来说，LVMID与操作系统的进程ID（Process Identifier,PID）是一致的，使用Windows的任务管理器或者UNIX的ps命令也可以查询到虚拟机进程的LVMID，但如果同时启动了多个虚拟机进程，无法根据进程名称定位时，那就只能依赖jps命令显示主类的功能才能区分了。
```bash
# 命令格式 
$ jps [options] [hostid]

# 输出JVM启动时显示指定的JVM参数
$ jps -v

# 输出主类全名或jar路径
$ jps -l

# 输出JVM启动时传递给main()的参数
$ jps -m

# 只输出LVMID
$ jps -q

```

**jstat**

`jstat(JVM statistics Monitoring)`是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。

```bash
# 命令格式
# [option] : 操作参数
# LVMID : 本地虚拟机进程ID，如果是远程虚拟机进程，那VMID的格式应当是：protocol://lvmid@hostname:port/servername
# [interval] : 连续输出的时间间隔
# [count] : 连续输出的次数
$  jstat [option] LVMID [interval] [count]


# 每隔250ms查询一次GC情况，查询20次
$ jstat -gc [pid] 250 20


# 监视类装载卸载数量、总空间以及耗时
$ jstat -class [pid] $interval $count

# 输出JIT编译过的方法数量耗时等
$ jstat -compiler [pid] $interval $count


```

## Troubleshooting Tools

**jinfo**

`jinfo(JVM Configuration info)`这个命令作用是实时查看和调整虚拟机运行参数。 

之前的`jps -v`口令只能查看到显示指定的参数，如果想要查看未被显示指定的参数的值就要使用`jinfo`口令

```bash

$ jinfo [option] [args] LVMID # 命令格式

$ jinfo -flag [+|-]<name> LVMID # 输出指定args参数的值,或者设置VM参数

$ jinfo -flags LVMID # 不需要args参数，输出所有JVM参数的值

$ jinfo -sysprops LVMID # 输出系统属性，等同于System.getProperties()
```

**jmap**

`jmap（Memory Map for Java）`命令用于生成堆转储快照（一般称为heapdump或dump文件）。

如果不使用jmap命令，要想获取Java堆转储快照，还有一些比较“暴力”的手段：譬如加`-XX：+HeapDumpOnOutOfMemoryError`参数，可以让虚拟机在OOM异常出现之后自动生成dump文件，通过`-XX：+HeapDumpOnCtrlBreak`参数则可以使用[Ctrl]+[Break]键让虚拟机生成dump文件，又或者在Linux系统下通过`Kill-3`命令发送进程退出信号“吓唬”一下虚拟机，也能拿到dump文件。

jmap的作用并不仅仅是为了获取dump文件，它还可以查询finalize执行队列、Java堆和永久代的详细信息，如空间使用率、当前用的是哪种收集器等。和jinfo命令一样，jmap有不少功能在Windows平台下都是受限的，除了生成dump文件的-dump选项和用于查看每个类的实例、空间占用统计的-histo选项在所有操作系统都提供之外，其余选项都只能在Linux/Solaris下使用。

```bash
# 用法
# -dump : 生成堆转储快照，格式为:-dump:[live, ] format=b,file=<filename>,其中live子参数说明是否只dump出存活的对象。
# -finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
# -heap : 显示Java堆详细信息
# -histo : 显示堆中对象的统计信息，GC使用的算法，heap的配置及wise heap的使用情况,可以用此来判断内存目前的使用情况以及垃圾回收情况
# -permstat : to print permanent generation statistics
# -F : 当-dump没有响应时，强制生成dump快照
 $ jmap [option] <pid>


```



## 参考
[Orcale - JDK Development Tools](https://docs.oracle.com/javase/8/docs/technotes/tools/)

 
