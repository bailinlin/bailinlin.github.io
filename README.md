# blog

博客系统基于[Hexo](http://hexo.io/)搭建，使用主题[next.Pisces](https://github.com/iissnan/hexo-theme-next)。采用[多说](http://duoshuo.com/)评论系统。

## 环境搭建
* 基本环境
    * [git](https://github.com/)
    * [Node&npm](https://nodejs.org/en/)
    * [Hexo](http://hexo.io/)

	``` bash
	npm install -g hexo-cli
	```
* 获取项目代码

	``` bash
	git clone https://github.com/bailinlin/bailinlin.github.io.git blog
	cd blog
	npm install
	```
* 写博客

	``` bash
	hexo s [-p 3000] // 启动博客本地环境，默认4000端口，供预览使用
	```
  
	新增博客 
	``` bash
	$ hexo new [layout] <title> // 可以在命令中指定文章的布局（layout），默认为 post
	```

## 博客规范
头部“---”行以前的部分为博文信息定义部分

```
title: 博客说明书                             // 博文名称
date: 2015-09-24 00:00:00                   // 博文创建时间
updated: 2015-09-25 00:00:00                // 博文修改时间【可选】
categories:                                 // 博文分类，可多级，有层级之分
tags:                                       // 标签，可多个，无层级之分
---
```
