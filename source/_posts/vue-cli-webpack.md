---
title: vue-cli#4.7项目结构分析
date: 2018-05-07 14:11:04
tags: [vue, vue-cli, webpack]
---
### 前言

尝试过 vue 项目开发的同学一定使用过 vue-cli 脚手架创建项目，我们这次不讲如何使用 vue-cli 脚手架来初始化一个 vue 项目，我们来了解些 脚手架为我们生成的 webpack 配置，如果你没有使用过 vue-cli 搭建 vue 项目，你可以参考这篇文章 [《 使用vue-cli脚手架创建新项目 》](https://segmentfault.com/a/1190000007441374)

### 项目结构
<code>

        ├── build --------------------------------- webpack相关配置文件
        │   ├── build.js --------------------------webpack打包配置文件
        │   ├── check-versions.js ------------------------------ 检查npm,nodejs版本
        │   ├── logo.png ---------------------------------- 项目 logo
        │   ├── utils.js --------------------------------------- 配置资源路径，配置css加载器
        │   ├── vue-loader.conf.js ----------------------------- 配置css加载器等
        │   ├── webpack.base.conf.js --------------------------- webpack基本配置
        │   ├── webpack.dev.conf.js ---------------------------- 用于开发的webpack设置
        │   ├── webpack.prod.conf.js --------------------------- 用于打包的webpack设置
        ├── config ---------------------------------- 配置文件
        ├── node_modules ---------------------------- 存放依赖的目录
        ├── src ------------------------------------- 源码
        │   ├── assets ------------------------------ 静态文件
        │   ├── components -------------------------- 组件
        │   ├── main.js ----------------------------- 主js
        │   ├── App.vue ----------------------------- 项目入口组件
        │   ├── router ------------------------------ 路由
        ├── package.json ---------------------------- node配置文件
        ├── .babelrc--------------------------------- babel配置文件
        ├── .editorconfig---------------------------- 编辑器配置
        ├── .gitignore------------------------------- 配置git可忽略的文件

</code>

### 配置文件解析前你需要了解的 webpack 几个知识点

#### 1. path模块
path是node.js中的一个模块，用于处理目录的对象，提高开发效
<pre>

    常用方法：
    path.join():用于连接路径。该方法的主要用途在于，会正确使用当前系统的路径分隔符，Unix系统是”/“，Windows系统是”\“
    path.resolve()用于将相对路径转为绝对路径

    常使用的文件路径
    __dirname: 总是返回被执行的 js 所在文件夹的绝对路径
    __filename: 总是返回被执行的 js 的绝对路径
    process.cwd(): 总是返回运行 node 命令时所在的文件夹的绝对路径

</pre>

#### 2.process

process对象是Node的一个全局对象，提供当前Node进程的信息。
<pre>

    process对象提供一系列属性，用于返回系统信息
    process.argv：返回当前进程的命令行参数数组。
    process.env：返回一个对象，成员为当前Shell的环境变量，比如process.env.HOME
    process.pid：当前进程的进程号

</pre>

#### 3. ExtractTextWebpackPlugin

[ExtractTextWebpackPlugin](https://doc.webpack-china.org/plugins/extract-text-webpack-plugin/) 插件通常用来做样式文件的分离，被分离的文件不会被内嵌到  JS bundle 中，而会被放到一个单独的文件中，在样式文件比较大的时候，能够提前样式的加载,配置示例如下

<pre>
        const ExtractTextPlugin = require("extract-text-webpack-plugin");

        module.exports = {
          module: {
            rules: [
              {
                test: /\.css$/,
                use: ExtractTextPlugin.extract({
                  fallback: "style-loader",
                  use: "css-loader"
                })
              }
            ]
          },
          plugins: [
            new ExtractTextPlugin("styles.css"),
          ]
        }

</pre>

它会将所有的入口 chunk(entry chunks)中引用的 *.css，移动到独立分离的 CSS 文件。因此，你的样式将不再内嵌到 JS bundle 中，而是会放到一个单独的 CSS 文件（即 styles.css）当中。 如果你的样式文件大小较大，这会做更快提前加载，因为 CSS bundle 会跟 JS bundle 并行加载。

#### 4.html-webpack-plugin

如果你有多个 webpack 入口点， 他们都会在生成的HTML文件中的 script 标签内。如果你有任何 CSS assets 在 webpack 的输出中（例如， 利用ExtractTextPlugin提取CSS）， 那么这些将被包含在HTML head中的<link>标签内。通常在开发中，我们为了避免 CDN 和浏览器的缓存通常会个输出文件 bundle.js 加上一个hash 值例如 `[hash].bundle.js`，使用 [html-webpack-plugin](https://doc.webpack-china.org/plugins/html-webpack-plugin/) 能够在创建新的 html 文件的时候将我们把带有哈希值的 bundle.js 引用到 html 文件.

#### 5.CopyWebpackPlugin

[CopyWebpackPlugin](https://doc.webpack-china.org/plugins/copy-webpack-plugin/)从插件名称上我们不难看出他的作用，通常用来拷贝资源，对项目文件进行归类整合

#### 5.friendly-errors-webpack-plugin

[friendly-errors-webpack-plugin](https://www.npmjs.com/package/friendly-errors-webpack-plugin)能够更好在终端看到webapck运行的警告和错误，提高开发体验

#### 6.optimize-css-assets-webpack-plugin

用来优化从脚本里提炼出来的 css ，配置示例如下

<pre>
    var OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');
    module.exports = {
      module: {
        rules: [
          {
            test: /\.css$/,
            loader: ExtractTextPlugin.extract('style-loader', 'css-loader')
          }
        ]
      },
      plugins: [
        new ExtractTextPlugin('styles.css'),
        new OptimizeCssAssetsPlugin({
          assetNameRegExp: /\.optimize\.css$/g,
          cssProcessor: require('cssnano'),
          cssProcessorOptions: { discardComments: { removeAll: true } },
          canPrint: true
        })
      ]
    };
</pre>

#### 6.UglifyjsWebpackPlugin

[UglifyjsWebpackPlugin](https://doc.webpack-china.org/plugins/uglifyjs-webpack-plugin/)用来压缩 js 代码

#### 7.Source map

简单说，[Source map](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)就是一个信息文件，里面储存着位置信息。也就是说，转换后的代码的每一个位置，所对应的转换前的位置。有了它，出错的时候，debug 工具将直接显示原始代码，而不是转换后的代码。这无疑给开发者带来了很大方便。[webpack 的 devtool里有 7种 SourceMap 模式]( https://juejin.im/post/58293502a0bb9f005767ba2f)


| 模式     | 解释      |
| ------ | ------- | ------- | ----- |
| eval  | 每个 module 会封装到 eval 里包裹起来执行，并且会在末尾追加注释 //@ sourceURL    |
| source-map | 生成一个 SourceMap 文件. |
| hidden-source-map| 和 source-map 一样，但不会在 bundle 末尾追加注释. |
| inline-source-map| 生成一个 DataUrl 形式的 SourceMap 文件. |
| eval-source-map| 每个 module 会通过 eval() 来执行，并且生成一个 DataUrl 形式的 SourceMap . |
| cheap-source-map| 生成一个没有列信息（column-mappings）的 SourceMaps 文件，不包含 loader 的 sourcemap（譬如 babel 的 sourcemap） |
| cheap-module-source-map| 生成一个没有列信息（column-mappings）的 SourceMaps 文件，同时 loader 的 sourcemap 也被简化为只包含对应行的。 |


#### 8.开发中 Server(DevServer)

关键配置解析

| clientLogLevel     | historyApiFallback      | hot      | contentBase    |
| ------ | ------- | ------- | ----- |
| 1. 简洁  | 1.注释    | 1.常量    | 1.存储类 |
| 2.环境设置 | 2.数据类型  | 2.修饰符类型 | 2.运算符 |
| 3.基本语法 | 3.变量作用域 |         |       |


webpack-dev-server使用方法，看完还不会的来找我~ - JSer - SegmentFault 思否 https://segmentfault.com/a/1190000006670084

#### 9. webpack-merge

开发环境(development)和生产环境(production)的构建目标差异很大。在开发环境中，我们需要具有强大的、具有实时重新加载(live reloading)或热模块替换(hot module replacement)能力的 source map 和 localhost server。而在生产环境中，我们的目标则转向于关注更小的 bundle，更轻量的 source map，以及更优化的资源，以改善加载时间。由于要遵循逻辑分离，我们通常建议为每个环境编写彼此独立的 webpack 配置。通用的配置部分，我们抽象出一个公共文件，通过 [webpack-merge](https://doc.webpack-china.org/guides/production/) 工具的“通用”配置，我们不必在环境特定的配置中重复代码。

### 配置文件解析
>通过了解了上面的配置，我们应该对 webpack 的常用插件和工具有了一定了解，我们来看下 vue-cli 脚手架给我们生成的配置情况

####  build.js
####  check-versions.js
####  vue-loader.conf.js
####  build.js




### 扩展

webpack指南 https://doc.webpack-china.org/guides/production/
webpack文档 https://doc.webpack-china.org/concepts/
webpack配置 https://doc.webpack-china.org/configuration
HtmlWebpackPlugin https://doc.webpack-china.org/plugins/html-webpack-plugin
Manifest https://doc.webpack-china.org/concepts/manifest
开发 https://doc.webpack-china.org/guides/development/#%E4%BD%BF%E7%94%A8-source-map
开发中 Server(DevServer) https://doc.webpack-china.org/configuration/dev-server
模块热替换(Hot Module Replacement) https://doc.webpack-china.org/concepts/hot-module-replacement/
启用 HMR 非常简单，在大多数情况下也不需要设置选项。   https://doc.webpack-china.org/plugins/hot-module-replacement-plugin/
NamedModulesPlugin https://doc.webpack-china.org/plugins/named-modules-plugin/
webpack-merge: Merge designed for Webpack (MIT) https://github.com/survivejs/webpack-merge
DefinePlugin https://doc.webpack-china.org/plugins/define-plugin
[webpack] devtool里的7种SourceMap模式是什么鬼？ - 掘金 https://juejin.im/post/58293502a0bb9f005767ba2f
webpack-dev-server使用方法，看完还不会的来找我~ - JSer - SegmentFault 思否 https://segmentfault.com/a/1190000006670084


