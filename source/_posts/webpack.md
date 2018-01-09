---
title: 你需要知道的 webpack 配置
date: 2017-06-09 21:27:52
tags: webpack
---

好久没有写文章，最近在做项目自动化构建工具的迁移，花了一点时间去研究 `webpack` ，`webpack` 的入门其实简单，但是现有的资料比较零碎，按照我的学习路径整理了下，希望对大家能有所帮助。

>接下来我将从3个部分来给大家介绍`webpack`,分别是 webpack 的基础配置，哪些常用的加载器，我在项目自动化构建工具改造的过程中雨大了那些问题

### webpack 基础配置
首先我们需要理解四个重要的概念
1. 入口（你需要打包的文件声明），你的项目需要什么依赖没在这里进行声明，require 你需要的依赖，webpack 会直接和间接的找到依赖文件进行打包，可传字符串，数组，对象
<pre>
        // 配置了3个入口文件
        entry: [
				'./config/dependencies.js',
				'./config/index.js',
				'./config/cssImport.js'
		]
</pre>
2. 出口（你打包资源后到哪个目录哪个文件），声明依赖打包后的文件输出的目录及命名方式，可传字符串，数组，对象
<pre>
        //声明了依赖压缩打包之后会被添加到 build 目录的 bundle.js 文件里
        output: {
				path: path.join(__dirname, '../build'),
				filename: 'bundle.js',
		},
</pre>
3. loader（模块加载器）能将各种资源的依赖模块打包成webpack 能够理解的 js 模块，从而进行你需要的操作，如 css 预编译，图片压缩，路径转换等
<pre>
        loaders: [
					{
						test: /\.js?$/,
						exclude: /(node_modules|bower_components)/,
						loader: 'babel-loader', // 'babel-loader' is also a legal name to reference
						query: {
								presets: ['es2015']
						}
				}]
</pre>
4. 插件（插件用于扩展 loader 的能力）你可以用插件进行定义环境变量对代码进行打包压缩。
<pre>
        //声明
        const ImageminPlugin = require("imagemin-webpack-plugin").default
        const CopyWebpackPlugin = require('copy-webpack-plugin')

        //从imgSrc 目录压缩图片，压缩完拷贝到 build/img 目录下
        plugins: [
				new CopyWebpackPlugin([
						{ from: 'imgSrc' ,to:'img'}
				]),
				new ImageminPlugin(
						{ test: /\.(jpe?g|png|gif|svg)$/i }
						)
		]
</pre>

[概念方面如果有不清晰的可以看下 webpack 的中文文档](https://doc.webpack-china.org/concepts/)
### webpack 的常用加载器

loader 用于常用的源代码进行装换，常用的有js编译，css 预编译，图片压缩等，这些都是项目中比较常见的，大家平时不需要记忆，只要能大概知道有这么一个东西，需要用到的时候去查阅就行，[loader 官方收录文档](https://doc.webpack-china.org/loaders/)

### 在项目迁移中遇到的问题

由于各种历史原因，我们的项目目录结构凌乱，项目依赖多，结构复杂，使用着 `angular + gulp` 进行开发，自动化构建工具还处于刀耕火种的年代，发版本的时候通过前端给包，不安全，不规范。新的一年我们尝试去改变这种现状，洗完通过运维直接拉去前端代码，这样能充分的保证前端代码的一致性。

#### 目前的项目结构的控诉
1. 项目所有问文件处于一个平级状态
2. 有3个依赖包文件夹分别是 framwork，lib，node_modules(各种历史遗留问题)
3. 源图片imgSrc和压缩后的img在同一个目录
4. 人工手动打包的时候要很小心的删除不必要的文件

#### 现有的改进方案

 1. 创建 build 文件夹，nginx 代理 到 build 目录
 2. config 中配置 webpack 构建打包任务，以及各种依赖的入口文件
 3. ib && framwork 移到 build 文件
 4. less 中对图片的引用路径需要变动，因为img 的路径变动了，tpls 也变动了，所以在视图中直接引用的不需要变化
 5. gulpfile 中的 js 压缩，less 编译，图片压缩内容迁移到 webpack 任务中
 6. 通过 npm run dev 进行编译构建
 8. 图片压缩
 9. 依赖及模块文件变化时实时构建项目
 10. 弃用 gulpfile 编译构建

创建 `build` 文件夹，`nginx` 代理 到 `build`
>目录，配置生产环境依赖目录，运维可以直接将代理指向该目录，不需要人工手动剔除多余的目录

config 中配置 webpack 构建打包任务，以及各种依赖的入口文件
>建立config 目录，存放 webpack 配置文件目录，及各种依赖声明目录，我们一共声明了3个依赖入口文件，分别是npm 管理的生产依赖文件，css 文件，项目逻辑js文件。css 和 生产依赖的内容比较烧，我们手动 require 一下，但是我们项目是已经经过近两年的迭代开发，逻辑代码 js 文件繁多，目录结构负责，所以我们需要写一个简单的 node 脚本，递归查询项目目录结构，动态 require 写入到我们的js入口文件中，node 脚本 如下

<pre>
        var files = fs.readdirSync('js')
        var jsPath = 'js'

        fs.unlink('config/index.js')

        var getFileName = function (files,dirPath) {

		files.forEach(function (filename) {
				var fullname = path.join(dirPath,filename)
				var stats = fs.statSync(fullname)

				if (stats.isDirectory()){
						var subFiles = fs.readdirSync(fullname)
						getFileName(subFiles,fullname)
				} else {
						let file = './../'+dirPath+'/'+filename
						fs.writeFile('./config/index.js', 'require (\"'+file +'\")\n', {
								flag: 'a'
						}, function(err){
								 if(err) throw err
						})
				}
		})
}

getFileName(files,jsPath)
</pre>

配置我们的入口和输入
<pre>
        entry: [
				'./config/dependencies.js',
				'./config/index.js',
				'./config/cssImport.js'
		],
		output: {
				path: path.join(__dirname, '../build'),
				filename: '[name].js',
		},
</pre>

然后是对我们的gulp task 进行迁移，我们需要配置 js 语法编译，css 预处理，图片压缩的功能。js 编译只需要使用 `babel-loader` ,通过 `npm install babel-loader` 安装加载器，指定匹配的文件及需要忽略的文件，指定转化语法，设定转码规则，就配置成功了

<pre>
        {
			test: /\.js?$/,
			exclude: /(node_modules|bower_components)/,
			loader: 'babel-loader', // 'babel-loader' is also a legal name to reference
			query: {
				presets: ['es2015']
			}
		}
</pre>

css 预编译，我们使用了 less 做为 css 开发工具，编译的时候需要用到的 loader 比较多，通过`less-loader`,`css-loader`,`style-loader`的链式调用，将样式作用于DOM

<pre>
        rules: [{
			test: /\.less$/,
			use: [{
				loader: "style-loader" // creates style nodes from JS strings
			}, {
				loader: "css-loader" // translates CSS into CommonJS
			}, {
				loader: "less-loader" // compiles Less to CSS
			}]
		}]
</pre>

或则

<pre>
loaders: [
	    	{
				test: /\.less$/,
				use: ['style-loader',
					{
						loader: 'css-loader',
						options: {
						    //支持@important引入css
						    importLoaders: 1
						}
					},
					{
						loader: 'postcss-loader',
						options: {
							plugins: function() {
								return [
				                //一定要写在require("autoprefixer")前面，否则require("autoprefixer")无效
								require('postcss-import')(),
								require("autoprefixer")({
								"browsers": ["Android >= 4.1", "iOS >= 7.0", "ie >= 8"]})]
				            }
			            }
			        },
				    'less-loader']
				}]
</pre>

图片压缩我们选择了两个插件分别是`imagemin-webpack-plugin`和`copy-webpack-plugin`，imagemin 实现图片压缩，copy 实现图片资源拷贝，具体配置如下

<pre>
//插件引用
const ImageminPlugin = require("imagemin-webpack-plugin").default
const CopyWebpackPlugin = require('copy-webpack-plugin')

//插件使用
plugins: [
				new CopyWebpackPlugin([
						{ from: 'imgSrc' ,to:'img'}
				]),
				new ImageminPlugin(
						{ test: /\.(jpe?g|png|gif|svg)$/i }
						)
		]
</pre>

了解完细节我们看下一个整体的配置

<pre>
/**
 * Created by bailinlin on 2018/1/4.
 */
const fs = require ("fs")
const path = require ("path")

const ImageminPlugin = require("imagemin-webpack-plugin").default
const CopyWebpackPlugin = require('copy-webpack-plugin')


var files = fs.readdirSync('js')
var jsPath = 'js'

fs.unlink('config/index.js')

var getFileName = function (files,dirPath) {

		files.forEach(function (filename) {
				var fullname = path.join(dirPath,filename)
				var stats = fs.statSync(fullname)

				if (stats.isDirectory()){
						var subFiles = fs.readdirSync(fullname)
						getFileName(subFiles,fullname)
				} else {
						let file = './../'+dirPath+'/'+filename
						fs.writeFile('./config/index.js', 'require (\"'+file +'\")\n', {
								flag: 'a'
						}, function(err){
								 if(err) throw err
						})
				}
		})
}

getFileName(files,jsPath)

module.exports = {
		entry: [
				'./config/dependencies.js',
				'./config/index.js',
				'./config/cssImport.js'
		],
		output: {
				path: path.join(__dirname, '../build'),
				filename: 'bundle.js',
		},
		module: {
				loaders: [
					{
						test: /\.js?$/,
	                exclude: /(node_modules|bower_components)/,
			loader: 'babel-loader', // 'babel-loader' is also a legal name to reference
			query: {
				presets: ['es2015']
			}
	            },{
		        test: /\.less$/,
		        use: ['style-loader',
			    {
				loader: 'css-loader',
				options: {
				 //支持@important引入css
					importLoaders: 1
				}
			    },{
				loader: 'postcss-loader',
				options: {
				plugins: function() {
					return [
				    //一定要写在require("autoprefixer")前面，否则require("autoprefixer")无效
					require('postcss-import')(),
					require("autoprefixer")({
					"browsers": ["Android >= 4.1", "iOS >= 7.0", "ie >= 8"]})]
				     }
			         }
			        },
				    'less-loader']
				},{
				    test: /\.(jpe?g|png|gif|svg)$/i,
					use: [
					{
					    loader: 'url-loader',
						options: {
							limit: 8192
						}
					}]
				}]
		},
		plugins: [
				new CopyWebpackPlugin([
						{ from: 'imgSrc' ,to:'img'}
				]),
				new ImageminPlugin(
						{ test: /\.(jpe?g|png|gif|svg)$/i }
						)
		]
}

</pre>

参考文章列表

>如果你还想更深入理解，你可以继续阅读这些扩展文章
[理解概念]( https://doc.webpack-china.org/concepts/)
[配置讲解]( https://doc.webpack-china.org/configuration/#-)
[常会用到的loaders]( https://doc.webpack-china.org/loaders/)
[webpack打包原理]( http://blog.csdn.net/lancewu0907/article/details/76513231)
[webpack全局变量](https://www.zhihu.com/question/46661735)
[less 图片解析问题](https://github.com/webpack-contrib/css-loader/issues/74)
[其他优秀的配置文章](https://juejin.im/entry/5767a975df0eea0062ffe193)
[《使用webpack-dev-server实现热更新》]( https://github.com/kingvid-chan/webpack2-lessons/tree/master/lesson2)
[imagemin-webpack-plugin]( https://github.com/Klathmon/imagemin-webpack-plugin)
[copy-webpack-plugin](https://github.com/webpack-contrib/copy-webpack-plugin)