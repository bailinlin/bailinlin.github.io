---
title: 跨域问题导致设置 cookie 不生效的问题
date: 2018-03-05 16:18:09
tags: cookie
---
我们看下跨域不生效的问题，首先抛出两个问题：
>1. 我们如何设置 cookie ？
2. 又如何确定 cookie 设置是否生效了 ？

首先，我们实现一个简单的接口,新建一个 test.js 文件，将如下代码复制进去，通过 `node test.js` 启动服务，在本地就可以通过 `http://localhost:3000/rest/collect/event/h5/v1/` 来访问了我们创建的接口了（node 环境安装的教程网上有很多详细的教程，本文不再赘述）
<pre>
var express = require('express');
var app = express();
var URL = require('url')
var path = require('path');


app.post('/rest/collect/event/h5/v1/', function(req, res) {
		res.cookie('token','11111112222222224444444444')
		res.cookie('httpOnly-token','11111112222222224444444444',{ httpOnly: true })

		function User() {
				this.name;
				this.city;
				this.age;
		}

		var user = new User();

		if(params.id == '1') {

				user.name = "ligh";
				user.age = "1";
				user.city = "北京市";

		}else{
				user.name = "SPTING";
				user.age = "1";
				user.city = "杭州市";
		}

		var response = {status:1,data:user};
		res.send(JSON.stringify(response));
});

app.listen(3000);
console.log('Listening on port 3000...');
</pre>

访问效果如下

![接口访问效果如下](/images/cookie/3.jpeg)

在前端代码中访问我们的接口
![cookie设置](/images/cookie/4.jpeg)
![cookie查看](/images/cookie/5.jpeg)
在浏览器中我们可以看到请求的 Resopnse Headers 里，有两个 `set-cookie`头部，区别在于一个带有 `HttpOnly`的标识，我们打开浏览器的调试窗口`Application`我们可以看到，两个数值都被设置到浏览器里了，`httpOnly`的值在浏览器调试窗口的`http`一栏，打了个小勾，说明这个变量是只能通过 http 请求来获取到这个cookie ，前端无法通过 js 的 `document.cookie`来获取到
![就是无法操作的cookie](/images/cookie/7.jpeg)
讲到这块内容，我们顺便讲下 cookie 设置的其他参数的作用

![其他参数](/images/cookie/6.jpeg)
cookie 和域名相关的哟，`Domain` 变量表示 cookie 生效的域名，`expries`和`max-age`表示 cookie 的有效时间

#### 问题描述及解决
在开发阶段我自己用node 简单的写了一个接口，便于联调前端传参问题，希望通过 http 的set-cookie 存储变量， 但是却始终没有把 cookie 成功设置到浏览器里，经过排查发现是跨域导致的 cookie 设置不生效
![cookie设置](/images/cookie/1.jpeg)
![cookie查看](/images/cookie/2.jpeg)

不生效的原因是我本地项目启动在 `http://localhost:70`,但是调用的接口在 `http://localhost:3000`上，端口不一样，存在跨域的问题，所以虽然在 Response Header 里看到了`set-cookie`的操作，但是在浏览器的 `application`里看到，并没有被设置进来，解决办法，通过nginx 代理（最长用的跨域解决办法）

#### 扩展

跨域的问题在开发过程中比较常见，我们经常会碰到，简单来说`只要请求资源的协议，域名，端口不一致，都会导致跨域`，网上的解决方法也比较多，比较成熟，本文不做扩展，附带几个链接供大家参考

[跨域中的预检测请求](http://harttle.land/2016/12/30/cors-preflight.html)
[CORS 跨域中的 Cookie](https://www.jianshu.com/p/13d53acc124f)
[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
[Web开发中跨域的几种解决方案](http://harttle.land/2015/10/10/cross-origin.html)

