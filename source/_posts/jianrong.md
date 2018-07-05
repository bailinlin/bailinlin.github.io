---
title: 如何机智地回答浏览器兼容性问题
date: 2018-07-05 10:31:52
tags: [浏览器兼容性, 思维导图 ]
---
### 前言

有过面试经验的同学应该都被问过浏览器兼容性的问题，对于面试官的问题，常常猝不及防，因为通常他们都是这么问的。"来谈谈浏览器兼容的问题吧"，"你对浏览器的兼容性有了解过吗"，那么如何才是我们正确回答这个问题的姿势呢。

>虽然面试官的问题十分的笼统，浏览器的兼容性无非还是样式兼容性（css），交互兼容性（javascript），浏览器 hack 三个方面。

### 样式兼容性（css）方面

![](https://user-gold-cdn.xitu.io/2018/7/5/16468bfe8343415c?w=1152&h=412&f=jpeg&s=257125)
1. 因为历史原因，不同的浏览器样式存在差异，可以通过 Normalize.css 抹平差异，也可以定制自己的 reset.css，例如通过通配符选择器，全局重置样式

        * { margin: 0; padding: 0; }

2. 在CSS3还没有成为真正的标准时，浏览器厂商就开始支持这些属性的使用了。CSS3样式语法还存在波动时，浏览器厂商提供了针对浏览器的前缀，直到现在还是有部分的属性需要加上浏览器前缀。在开发过程中我们一般通过IDE开发插件、css 预处理器以及前端自动化构建工程帮我们处理。

    浏览器内核与前缀的对应关系如下

    内核 | 主要代表的浏览器 | 前缀
    :-: | :-: | :-:
    Trident | IE浏览器| -ms
    Gecko| Firefox| -moz
    Presto| Opera | -o
    Webkit|Chrome和Safari|-webkit

3. 在还原设计稿的时候我们常常会需要用到透明属性，所以解决 IE9 以下浏览器不能使用 opacit。


        opacity: 0.5;
        filter: alpha(opacity = 50); //IE6-IE8我们习惯使用filter滤镜属性来进行实现
        filter: progid:DXImageTransform.Microsoft.Alpha(style = 0, opacity = 50); //IE4-IE9都支持滤镜写法progid:DXImageTransform.Microsoft.Alpha(Opacity=xx)



### 交互兼容性（javascript）

![](https://user-gold-cdn.xitu.io/2018/7/5/16468c023f296879?w=1272&h=678&f=jpeg&s=332606)
1. 事件兼容的问题，我们通常需要会封装一个适配器的方法，过滤事件句柄绑定、移除、冒泡阻止以及默认事件行为处理


        var  helper = {}

		//绑定事件
		helper.on = function(target, type, handler) {
			if(target.addEventListener) {
				target.addEventListener(type, handler, false);
			} else {
				target.attachEvent("on" + type,
					function(event) {
						return handler.call(target, event);
				    }, false);
			}
		};

		//取消事件监听
        helper.remove = function(target, type, handler) {
        	if(target.removeEventListener) {
        		target.removeEventListener(type, handler);
        	} else {
        		target.detachEvent("on" + type,
        	    function(event) {
        			return handler.call(target, event);
        		}, true);
            }
        };



2. new Date()构造函数使用，'2018-07-05'是无法被各个浏览器中，使用new Date(str)来正确生成日期对象的。 正确的用法是'2018/07/05'.

3. 获取 scrollTop 通过 document.documentElement.scrollTop 兼容非chrome浏览器

        var scrollTop = document.documentElement.scrollTop||document.body.scrollTop;


### 浏览器 hack

![](https://user-gold-cdn.xitu.io/2018/7/5/16468c060484968d?w=985&h=393&f=jpeg&s=167570)
1. 快速判断 IE 浏览器版本


        <!--[if IE 8]> ie8 <![endif]-->

        <!--[if IE 9]> 骚气的 ie9 浏览器 <![endif]-->


2. 判断是否是 Safari 浏览器

        /* Safari */
        var isSafari = /a/.__proto__=='//';

3. 判断是否是 Chrome 浏览器

        /* Chrome */
        var isChrome = Boolean(window.chrome);


### 身段不能掉，我们是个有逼格的前端

“什么？你们公司要兼容IE6，我们今天的面试就到这里为止吧，再见”。现在如果还有哪个公司要兼容IE6的话就不要去了，开发起来得多不幸福。

### 扩展阅读

[如何处理CSS3属性前缀_Autoprefixer](https://www.w3cplus.com/css3/autoprefixer-css-vender-prefixes.html)

[CSS透明opacity和IE各版本透明度滤镜filter的最准确用法](https://blog.csdn.net/freshlover/article/details/17143341)


### 往期文章
[精读《你不知道的 javascript（上卷）》](https://juejin.im/post/5b0cafad51882515624dc6d2)

[精读《你不知道的javascript》中卷](https://juejin.im/post/5b2a07c16fb9a00e36425ef0)

[精读《深入浅出Node.js》](https://juejin.im/post/5b1a18de6fb9a01e312828dd)

[javascript 垃圾回收算法](https://juejin.im/post/5b1f7e62e51d45068a6cb98f)

[精读《图解HTTP》](https://juejin.im/post/5b32f82a518825749e4a218b)

[思维导图下载地址](https://github.com/bailinlin/Awsome-Front-End-Xmind)