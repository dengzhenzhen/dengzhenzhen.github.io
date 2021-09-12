---
layout:     post
title:      vscode插件的helloworld
subtitle:   json文件校验小工具
date:       2021-09-12
author:     dengzhenzhen
header-img: img/home-bg-geek.jpg
catalog: 	 true
tags:
    - vscode
---

# 从零开始制作一个自己的vscode插件

## 1. Quick Start

最近工作有需求可能做一个vscode插件会更方便用户使用，所以这个周末看了下基本的vscode插件开发。

顺便做了个校验json字段的功能，可以用在v2ray配置文件的校验上。

## 2. 开发环境配置

### 2.1 环境准备

如果以前配置过前端开发或者nodejs的环境，就直接跳到2.2去就好了。

没有的话先到2.1配置nodejs吧

2.1 Node.js 安装配置

点这个 [Node.js 安装配置 | 菜鸟教程](https://www.runoob.com/nodejs/nodejs-install-setup.html) 看nodejs配置


2.2 安装vscode开发环境

先安装脚手架
```shell
npm install -g yo generator-code
```

然后
```shell
yo code
```

这里会出来一堆选项，按照提示选就好了，注意有个选项是选用typescript还是javascript，这按照喜好选吧。

这次我选的js，带类型检查的ts可能效果会更好点，下次开别的项目试试。

### todo

其他的下次再补吧