---
title: 【Java】Java IO 总结
tags: [java]
date: 2018-3-6
---

# Java IO 总结

 Java的IO模型使用Decorator(装饰者)模式，按功能划分Stream，可以动态装配这些Stream，以便获得需要的功能。
 
 例如，需要一个具有缓冲的文件输入流，则应当组合使用`FileInputStream`和`BufferedInputStream`。

## Java IO 体系的层次

| Part | Description  |  
| :--------   | :----------  |
| 流式部分     | 字节流(`InputStream`,`OutputStream`) 、字符流(`Writer`,`Reader`) |  
| 非流式部分     |主要包含一些辅助流式部分的类，如: `File`、`RandomAssessFile`、 `FileDescriptor`...|  
| 其他类     | 文件读取部分的与安全相关的类,如：`SerializablePermission`类，以及与本地操作系统相关的文件系统的类，如：`FileSystem`类和`Win32FileSystem`类和`WinNTFileSystem`类。 |  

## Java IO 流的分类

- 根据处理数据类型的不同分为：字符流和字节流
- 根据数据流向不同分为：输入流和输出流
- 按数据来源（去向）分类：
    - File（文件）： FileInputStream, FileOutputStream, FileReader, FileWriter 
    - byte[]：ByteArrayInputStream, ByteArrayOutputStream
    - Char[]: CharArrayReader,CharArrayWriter 
    - String(字符):StringBufferInputStream, StringReader, StringWriter 
    - 网络数据流：InputStream,OutputStream, Reader, Writer 

### 字符流和字节流

流序列中的数据既可以是未经加工的原始二进制数据，也可以是经一定编码处理后符合某种格式规定的特定数据。

因此Java中的流分为两种：

- `字节流`：数据流中最小的数据单元是字节
- `字符流`：数据流中最小的数据单元是字符， Java中的字符是Unicode编码，一个字符占用两个字节
    - 字符流的由来： Java中字符是采用Unicode标准，一个字符是16位，即一个字符使用两个字节来表示,为此，JAVA中引入了处理字符的流。
    - 因为数据编码的不同，而有了对字符进行高效操作的流对象。`本质其实就是基于字节流读取时，去查了指定的码表`。

### 输入流和输出流

采用数据流的目的就是使得输出输入独立于设备。

`输入流( Input  Stream )/输出流( Output Stream )`不关心数据源`来自/流向`何种设备（键盘，文件，网络）。

- `输入流(Source 2 Program)`：程序从输入流读取数据源。数据源包括外界(键盘、文件、网络…)，即是将数据源读入到程序的通信通道
- `输出流(Program 2 device)`：程序向输出流写入数据。将程序中的数据输出到外界（显示器、打印机、文件、网络…）的通信通道。

相对于程序来说，输出流是往存储介质或数据通道写入数据，而输入流是从存储介质或数据通道中读取数据，一般来说关于流的特性有下面几点：

- 先进先出，最先写入输出流的数据最先被输入流读取到
- 顺序存取，可以一个接一个地往流中写入一串字节，读出时也将按写入顺序读取一串字节，不能随机访问中间的数据。`RandomAccessFile可以从文件的任意位置进行存取（输入输出）操作`
- 只读或只写，每个流只能是输入流或输出流的一种，不能同时具备两个功能，输入流只能进行读操作，对输出流只能进行写操作。在一个数据传输通道中，如果既要写入数据，又要读取数据，则要分别提供两个流。


## Java IO 中常用的类

| Class | Description  |  
| :--------   | :----------  |
| `File`     | 文件类（文件特征与管理） |  
| `RandomAccessFile`     | 随机存取文件类 |  
| `InputStream`     | 字节输入流（二进制格式操作） 抽象类，基于字节的输入操作，所有输入流的父类 |  
| `OutputStream`    | 字节输出流（二进制格式操作） 抽象类，基于字节的输出操作，所有输出流的父类 |  
| `Reader`     | 字符输入流 （文件格式操作） 抽象类，基于字符的输入操作|  
| `Writer`     | 字符输出流 （文件格式操作） 抽象类，基于字符的输出操作|




## File类

> 在Java语言的java.io包中，由File类提供了描述文件和目录的操作与管理方法。但File类不是InputStream、OutputStream或Reader、Writer的子类，因为它不负责数据的输入输出，而专门用来管理磁盘文件与目录。

### Fiels 常量 


| Modifier and Type| Field  |  Description |
| :--------   | :----------  | :----  |
| static `String`     | `pathSeparator` |   The system-dependent path-separator character, represented as a string for convenience.     |
| static char        |  `pathSeparatorChar`   |   The system-dependent path-separator character.  |
| static `String`       |    `separator`    | The system-dependent default name-separator character, represented as a string for convenience.  |
| static char       |    `separatorChar`    | The system-dependent default name-separator character.  |

- `File.separator` -> 系统相关的默认名称分隔符字符------>`\`
- `File.pathSeparator` -> 系统相关的路径分隔符字符----->`;`

### Constructors 构造器

| Constructors | Description  | Note |
| :--------   | :----------  |:-------|
|File(File parent,String child)|Creates a new File instance from a parent abstract pathname and a child pathname string.|从父抽象路径名和子路径名字符串创建一个新的File实例|
|File(String pathname)|Creates a new File instance by converting the given pathname string into an abstract pathname.|通过将给定的路径名字符串转换为抽象路径名创建一个新的File实例|
|File(String parent,String child)|Creates a new File instance from a parent pathname string and a child pathname string.|从父路径名字符串和子路径名字符串创建一个新的File实例|
|File(URI url)|Creates a new File instance by converting the given file: URI into an abstract pathname.|通过将给定文件：URI转换为抽象路径名创建一个新的File实例。|

