---
title: 你需要知道的 gulp 自动化构建
date: 2016-03-30 20:37:03
tags: [自动化构建]
---
### 为什么要前端自动化

>什么是前端自动化构建就不说了，因为我不是写书的。在前端开发实践中，大公司都会有自己的基础前端架构，能容包括了开发环境、代码管理，代码质量，性能检测，命令行工具，开发规范，开发流程，前端架构及性能优化。相对而言，小公司或则是创业型的公司，前端架构这块做得就相对没有这么好，甚至于很不规范，而规范的目的在于提升工作效率。

而规范需要一定的过程，我们就先从代码质量，代码管理上入手。

 1. 对代码（html，css，js）进行语法检查
 2. 对图片，代码进行压缩
 3. 对sass。less 的css预处理器进行编译
 4. 期望代码有改动后，能自动刷新页面
 5. ...

这些操作，我们可以通过人工来完成，但是效率真的低到没朋友，难道语法检查你要自己一行一行的`review`，或则是拜托你的同事帮你一行一行的 `review` 么。如果你让我做这个，我肯定和你绝交...但是 `review` 的目的是帮助我们写出高质量的代码。这是必不可少的，所以我们期望能有一个自动帮我们实现代码检测压缩的工具。只要一个命令，你就能轻松的实现代码压缩，图片压缩，`css`预处理器编译等原来需要你去人工完成的任务，是不是爽到爆炸。

在项目自动化构建工具中，大家用得比较多的，分别是`grunt`，`gulp`。与这些自动化工具配套的包管理工具呢，通常还有`npm`。`node`包含了`npm`的包，所以只要你的系统里安装的 `node`，你就可以在你的控制台里通过 npm install 来安装你的项目依赖。还有的就是最近流行起来的 `webpack` 模块管理工具,大家对`webpack` 的反应也很好，所以我们打算在项目开发的时候把 gulp 和 webpack 一起用起来，并把研究后的搭建流程写成教程。这次分享的是gulp的搭建，下次等我的后台项目开始用 webpack 的时候，再来分享一篇。

### 从零开始搭建 gulp 前端自动化

 1. 安装node.js
 2. npm init 生成package文件，或则你可以自己手动生成
 3. 在控制台中输入` npm install --save-dev gulp`命令，在项目中安装gulp
 4. 配置gulp任务
 5. 在控制台中输入 `gulp`或则`gulp default`测试你的gulp任务
 6. 配置你真正需要的 gulp 任务，（压缩，代码质量检查，浏览器自动刷新）


    var gulp = require('gulp');
    gulp.task('default',function(){
        console.log("hello")
    });



    #####浏览器自动刷新

    1. 在你的谷歌浏览器里安装插件。关键字`livereload`
    2. 通过命令`mpn install gulp-livereload --save-dev`来安装依赖
    3. 在gulp文件中引入`livereload = require('gulp-livereload'),`
    4. 在gulp的`watch`任务中通过 `livereload.listen([options])`启动刷新服务
    5. 定义的任务在最后加入一个工作流`.pipe(livereload())`,
    6. 在启动后进入到这个任务后，开启谷歌插件，就能自动刷新浏览器了

    #gulpfile.js 文件

        var gulp = require('gulp'),
        uglify = require('gulp-uglify'),
        livereload = require('gulp-livereload'),

    gulp.task('test',function() {
        return gulp.src('js/test.js')
            .pipe(uglify())
            .pipe(gulp.dest('build'))
            .pipe(livereload())
    });

    gulp.task('watch',function(){
        livereload.listen();
        gulp.watch('js/test.js', ['test']);
    });

    当你修改你的test.js 文件之后，ctrl + s 保存，你就可以看到时时刷新。


 7.代码压缩

    1.通过命令`mpn install gulp-uglify --save-dev`来安装依赖(js 压缩)
    2.通过命令`mpn install gulp-concat --save-dev`来安装依赖(合并压缩后的文件到一个文件)

        #gulpfile.js 文件

        uglify = require('gulp-uglify'),

        gulp.task('compress',function(){
        return gulp.src('js/servers/*.js')
            .pipe(uglify())
            .pipe(concat('all.js'))
            .pipe(gulp.dest('dist/js'))
            .pipe(livereload())
    });

 8.同理css压缩，生成雪碧图等task，代码质量检查，都是同样的先安装依赖，再引用，编写task

**如果你想深入学习**

[我理想中的前端工作流](https://segmentfault.com/a/1190000004638228)
[gulp 中文网](http://www.gulpjs.com.cn/)
[livereload](https://scotch.io/tutorials/a-quick-guide-to-using-livereload-with-gulp)
[gulp-livereload](https://www.npmjs.com/package/gulp-livereload)ulp-livereload)