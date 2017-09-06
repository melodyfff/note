# Markdown&GitBook 简单介绍

[toc]

## Markdown

### 简介

![cmd-markdown-logo](https://www.zybuluo.com/static/img/logo.png)

> `Markdown`是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式 \(类似`HTML` 超文本标记语言\)

### 优点

*  `Markdown`的语法简洁明了、学习容易，而且功能比纯
     文本更强，因此有很多人用它写博客。
*  用于编写说明文档，并且以“README.MD”的文件名保存在软件的目录下面。
*  转换成`HTML`、`PDF`等格式。
*  随时修改你的文章版本，不必像字处理软件生成若干文件版本导致混乱。
*  可读、直观、学习成本低。

**不同客户端对 Markdown 的定制对高阶功能可能有不同的显示效果**

### 基础语法

#### 标题:

标题能显示出文章的结构。行首插入1-6个 \# ，每增加一个 \# 表示更深入层次的内容，对应到标题的深度由 1-6 阶。

# Header 1

## Header 2

### Header 3

#### Header 4

##### Header 5

###### Header 6

#### 列表：

* 一级条目1
* 一级条目2
  * 二级条目1
  * 二级条目2
     * 三级条目1
     * 三级条目2
     * 三级条目3
  * 二级条目3
* 一级条目3

_斜体_

**加粗**

#### 引用:

> 真的猛士，敢于直面惨淡的人生，敢于正视淋漓的鲜血。 —— 鲁迅


#### 代码块:

```python
@requires_authorization
class SomeClass:
    pass

if __name__ == '__main__':
    # A comment
    print 'hello world'
```
#### 图片与链接：

![cmd-markdown-logo](https://www.zybuluo.com/static/img/logo.png)

在线[Markdown编辑器](https://www.zybuluo.com/mdeditor "Markdown编辑器")

#### 表格：

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |


#### 角标注：
get 10 times more traffic from [Google][1] than from [Yahoo][2] or [MSN][3].  

[1]: http://google.com/        "Google" 
[2]: http://search.yahoo.com/  "Yahoo Search" 
[3]: http://search.msn.com/    "MSN Search"

#### 分割线

***
---
## GitBook

### 简介

> GitBook于2014年创建，旨在为文档，数字写作和出版创建一个现代化和简单的解决方案。  
> 我们已经开始构建开源格式。 理念是简单的优雅点，消除内容创作者的分心和担忧，让他们自由地写作。  
> 从那时起，我们已经发展成为帮助超过25万人共同为每个月向20万游客提供15万本书籍。

GitBook 是一个基于 Node.js 的命令行工具，可使用 Github/Git 和 Markdown 来制作精美的电子书，GitBook 并非关于 Git 的教程。

一般来说，你的书得有一个 README.md 和一个 SUMMARY.md。   
其中 SUMMARY.md 是最重要的，它代表了整个书的框架，也是我们主要需要修改的地方

### 前提条件

`Git`的安装和配置

```bash
git config --global user.name  test 
git config --global user.email test
```

### 目录结构

```
├── _book                     // gitbook serve自动生成的文件夹，不用理
├── designer                  // 各章节目录
│   ├── README.md             // 各章节的简介
│   ├── images                // 存放各章节的图片
│   └── designer_standard.md  // 各章节内容的具体展开
├── node_modules              // npm包的安装目录
├── product                   // 各章节目录
│   ├── README.md             // 各章节的简介
│   ├── images                // 存放各章节的图片
│   └── product_standard.md   // 各章节内容的具体展开
├── web                       // 各章节目录
│   ├── README.md             // 各章节的简介
│   ├── images                // 存放各章节的图片
│   └── web_standard.md       // 各章节内容的具体展开
├── .gitignore                // git 的忽略文件
├── book.json                 // 整个gitbook的配置文件
├── README.md                 // 整本书的简介
└── SUMMARY.md                // 整本书的目录索引
```


#### 插件的使用

Book.json

```json
{

    //样式风格配置格式
    "styles": {
        "website": "styles/website.css",
        "ebook": "styles/ebook.css",
        "pdf": "styles/pdf.css",
        "mobi": "styles/mobi.css",
        "epub": "styles/epub.css"
     },

    //插件安装配置格式
    "title": "Markdown&GitBook",
    "description": "Markdown&GitBook",
    "language": "zh",
    "plugins": [ "theme-faq","timeline" ,"disqus"],
    "pluginsConfig": {
        "disqus": {
            "shortName": "webpack-handbook"
        }
     }    
}
```
#### 发布到gitlab

.gitlab-ci.yml

```yml
# requiring the environment of NodeJS 4.2.2
image: node:4.2.2

# add 'node_modules' to cache for speeding up builds
cache:
  paths:
    - node_modules/ # Node modules and dependencies

before_script:
  - npm install gitbook-cli -g # install gitbook
  - gitbook fetch latest # fetch latest stable version
  - gitbook install
  #- gitbook fetch pre # fetch latest pre-release version
  #- gitbook fetch 2.6.7 # fetch specific version

# the 'pages' job will deploy and build your site to the 'public' path
pages:
  stage: deploy
  script:
    - gitbook build . public # build to public path 
  artifacts:
    paths:
      - public
  only:
    - master # this job will affect only the 'master' branch
```

### 本地部署GitBook
`Node.js`的安装和配置

`GitBook-Cli`的安装配置

```bash
npm install gitbook-cli -g
mkdir test
cd test
gitbook init    # 初始化
gitbook install # 下载插件
gitbook build   # 构建生成HTML
gitbook serve   # 本地发布  --port指定端口  默认4000
```
