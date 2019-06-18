---
title: 【Vue】Vue快速构建项目
tags: [vue]
date: 2019-6-14
---

# Vue快速构建项目

## 安装vue-cli

```bash
npm install -g vue-cli
```

## 初始化项目

### 方式一

**vue init**
```bash
$ vue init --help
Usage: init [options] <template> <app-name>

generate a project from a remote template (legacy API, requires @vue/cli-init)

Options:
  -c, --clone  Use git clone when fetching remote template
  --offline    Use cached template
  -h, --help   output usage information
```

**初始化实例**

```bash
# 剩下的跟着提示走就行
vue init webpack ${project_name}
```

### 方式二
**vue create**
```bash
用法：create [options] <app-name>

创建一个由 `vue-cli-service` 提供支持的新项目

选项：

  -p, --preset <presetName>       忽略提示符并使用已保存的或远程的预设选项
  -d, --default                   忽略提示符并使用默认预设选项
  -i, --inlinePreset <json>       忽略提示符并使用内联的 JSON 字符串预设选项
  -m, --packageManager <command>  在安装依赖时使用指定的 npm 客户端
  -r, --registry <url>            在安装依赖时使用指定的 npm registry
  -g, --git [message]             强制 / 跳过 git 初始化，并可选的指定初始化提交信息
  -n, --no-git                    跳过 git 初始化
  -f, --force                     覆写目标目录可能存在的配置
  -c, --clone                     使用 git clone 获取远程预设选项
  -x, --proxy                     使用指定的代理创建项目
  -b, --bare                      创建项目时省略默认组件中的新手指导信息
  -h, --help                      输出使用帮助信息
```

**初始化实例**

```bash
# 剩下的跟着提示走就行
vue create ${project_name}
```


### 方式三

**vue ui**

```bash
vue ui
```

## 拉取 2.x 模板 (旧版本)

```bash
npm install -g @vue/cli-init
# `vue init` 的运行效果将会跟 `vue-cli@2.x` 相同
vue init webpack ${project_name}
```