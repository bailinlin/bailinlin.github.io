---
title: 埋点的实现原理了解一下
date: 2018-05-15 16:08:20
tags: [埋点, javascript ]
---
### 前言
埋点分析，是网站分析的一种常用的数据采集方法。我们主要用来采集用户行为数据（例如页面访问路径，点击了什么元素）进行数据分析，从而让运营同学更加合理的安排运营计划。现在市面上有很多第三方埋点服务商，百度统计，友盟，growingIO 等大家应该都不太陌生，大多情况下大家都只是使用，没有去研究埋点是怎么实现的，对埋点的实现也比较摸不到头脑，刚好我最近研究了下 web 埋点，你要不要了解下。

### 现有埋点三大类型
>用户行为分析是一个大系统，一个典型的数据平台。由用户数据采集，用户行为建模分析，可视化报表展示几个模块构成。现有的埋点采集方案可以大致被分为三种，手动埋点，可视化埋点，无埋点

1. 手动埋点
    手动代码埋点比较常见，需要调用埋点的业务方在需要采集数据的地方调用埋点的方法。优点是流量可控，业务方可以根据需要在任意地点任意场景进行数据采集，采集信息也完全由业务方来控制。这样的有点也带来了一些弊端，需要业务方来写死方法，如果采集方案变了，业务方也需要重新修改代码，重新发布。
2. 可视化埋点 //TODO 阿里埋点细节
    可是化埋点是近今年的埋点趋势，很多大厂自己的数据埋点部门也都开始做这块。优点：前端没啥工作量，只要引用埋点脚本，甚至啥都不用做，缺点：技术上推广和实现起来有点难（大厂另当别论）。阿里的活动页很多都是运营通过可视化的界面拖拽配置实现，这些活动控件元素都带有唯一标识。通过埋点配置后台，将元素与要采集事件关联起来，可以自动生成埋点代码嵌入到页面中。
3. 无埋点
    无埋点则是前端自动采集全部事件，上报埋点数据，由后端来过滤和计算出有用的数据，优点：前端没有什么工作量，只要加载埋点脚本。缺点：流量和采集的数据过于庞大，服务器性能压力山大，GrowingIO 就是这种实现方案。


因为我们现有的业务上没有可以通过拖拽配置的控件，不同部门的同学也不会专门为了我们的埋点项目为每个元素上加上唯一标识，我们选择暂时放弃可视化埋点的实现。第一期我们在 `手动埋点` 和 `无埋点` 上进行了尝试，为了便于描述，下文我会称采集脚本为 SDK


### 思考几个问题

>埋点开发需要考虑很多内容，贯穿着不轻易动手写代码的原则，我们在开发前先思考下面这几个问题

1. 我们要采集什么内容，进行哪些采集接口的约定
2. 业务方通过什么方式来调用我们的采集脚本
3. 手动埋点：SDK 需要封装一个方法给业务方进行调用，传参方式业务方可控
4. 无埋点：考虑到数据量对于服务器的压力，我们需要对无埋点进行开关配置，可以配置进行哪些元素进行无埋点采集
5. 用户标识：游客用户和登录用户的采集数据的区分及关联
6. 设备Id：用户通过浏览器来访问 web 页面，设备Id需要存储在浏览器上，同一个用户访问不同的业务方网站，设备Id要保持一样
7. 单页面应用：现在流行的单页面应用和普通 web 页面的数据采集是否有差异
8. 混合应用：app 与 h5 的混合应用我们要怎么进行通讯



#### 我们要采集什么内容，进行哪些采集接口的约定

第一期我们先实现对 PV（即页面浏览量或点击量） 、UV（一天内同个访客多次访问） 、点击量、用户的访问路径的基础指标的采集，精细化分析的流量转化需要和业务相关，我们不做重点关注，但要预留扩展。所以我们的采集接口需要进行以下的约定

<pre>

{
    "header":{ // HTTP 头部
        "X-Device-Id":" 550e8400-e29b-41d4-a716-446655440000", //设备ID，用来区分用户设备
		"X-Source-Url":"https://www.baidu.com/", //源地址，关联用户的整个操作流程，用于用户行为路径分析，例如登录，到首页，进入商品详情，退出这一整个完整的路径
		"X-Current-Url":"", //当前地址，用户行为发生的页面
		"X-User-Id":"",//用户ID，统计登录用户行为
    },
    "body":[{ // HTTP Body体
        "PageSessionID":"", //页面标识ID，用来区分页面事件，例如加载和离开我们会发两个事件，这个标识可以让我们知道这个事件是发生在一个页面上
        "Event":"loaded", //事件类型，区分用户行为事件
        "PageTitle":  "埋点测试页",  //页面标题，直观看到用户访问页面
        "CurrentTime":  “1517798922201”,  //事件发生的时间
      	"ExtraInfo":  {
     	 }    //扩展字段，对具体业务分析的传参
	}]
}

</pre>

以上就是我们现在约定好了的通用的事件采集的接口，所传的参数基本上会根据采集事件的不同而发生变化。但是在用户的整一个访问行为中，用户的设备是不会变化的，如果你想采集设备信息可以重新约定一个接口，在整个采集开始之前发送设备信息，这样可以避免在事件采集接口上重复采集固定数据。

<pre>

{
    "header":{ // HTTP 头部
          "X-Device-Id"  ："550e8400-e29b-41d4-a716-446655440000"  ,      //  设备id
    },
    "body":{ // HTTP Body体
              "DeviceType":  "web" ,   //设备类型
             "ScreenWide"  :  768 , //  屏幕宽
             "ScreenHigh":  1366 , //  屏幕高
             "Language":    "zh-cn"  //语言
	}
}
</pre>

#### 业务方通过什么方式来调用我们的采集脚本

埋点应该让调用的业务方，尽可能少有工作量，最好是什么都不用做，😁，但是实现起来有点难额。我们采用的方案是让业务方在代码里通过 script 脚本来引用我们的 SDK ，业务方只要配置一些需要的参数进行埋点定制（👆我们讲到过的无埋点的流量控制），然后什么都不做就可以进行基础数据的采集。

<pre>

    (function() {
				var collect = document.createElement('script');
				collect.type = 'text/javascript';
				collect.async = true;
				collect.src =  'http://collect.trc.com/index.js';
				var s = document.getElementsByTagName('script')[0];
				s.parentNode.insertBefore(collect, s);
		})();


    //用户自定义要进行无埋点采集的元素，如果不进行无埋点采集，可以不配置
     var _XT = [];
  	_XT.push(['Target','div']);

</pre>

#### 手动埋点：SDK

如果业务方需要采集更多业务定制的数据，可以调用我们暴露出的方法进行采集
<pre>

    //自定义事件
  	sdk.dispatch('customEvent',{extraInfo:'自定义事件的额外信息'})

</pre>

#### 游客与用户关联

我们使用 userId 来做用户标识，同一个设备的用户，从游客用户切换到登录用户，如果我们要把他们关联起来，需要有一个设备Id 做关联

#### web 设备Id

用户通过浏览器来访问 web 页面，设备Id需要存储在浏览器上，同一个用户访问不同的业务方网站，设备Id要保持一样。web 变量存储，我们第一时间想到的就是 cookie，sessionStorage，localStorage，但是这3种存储方式都和访问资源的域名相关。我们总不能每次访问一个网站就新建一个设备指纹吧，所以我们需要通过一个方法来跨域共享设备指纹

我们想到的方案是，通过嵌套 iframe 加载一个静态页面，在 iframe 上加载的域名上存储设备id，通过跨域共享变量获取设备id，共享变量的原理是采用了iframe 的 contentWindow通讯，通过 postMessage 获取事件状态，调用封装好的回调函数进行数据处理具体的实现方式

<pre>

    //web 应用，通过嵌入 iframe 进行跨域 cookie 通讯，设置设备id,
        collect.setIframe = function () {
            var that = this
            var iframe = document.createElement('iframe')
            iframe.id = "frame",
            iframe.src = 'http://collectiframe.trc.com' // 配置域名代理，目的是让开发测试生产环境代码一致
            iframe.style.display='none' //iframe 设置的目的是用来生成固定的设备id，不展示
            document.body.appendChild(iframe)

            iframe.onload = function () {
                    iframe.contentWindow.postMessage('loaded','*');
            }

            //监听message事件，iframe 加载完成，获取设备id ，进行相关的数据采集
            helper.on(window,"message",function(event){
                that.deviceId = event.data.deviceId

                if(event.data && event.data.type == 'loaded'){
                    that.sendDevice(that.getDevice(), that.deviceUrl);
                    setTimeout(function () {
                        that.send(that.beforeload)
                        that.send(that.loaded)
                    },1000)
                }
            })
        }

</pre>

iframe 与 SDK 通讯

<pre>

    function receiveMessageFromIndex ( event ) {
        getDeviceInfo() // 获取设备信息
        var data =  {
                deviceId: _deviceId,
                type:event.data
        }

        event.source.postMessage(data, '*'); // 将设备信息发送给 SDK
    }

    //监听message事件
    if(window.addEventListener){
            window.addEventListener("message", receiveMessageFromIndex, false);
    }else{
            window.attachEvent("onmessage", receiveMessageFromIndex, false)

</pre>

如果你想知道可以看我的另一篇博客 [web 浏览器指纹跨域共享](https://bailinlin.github.io/2018/03/05/cookie-share/)

#### 单页面应用：现在流行的单页面应用和普通 web 页面的数据采集是否有差异

>我们知道单页面应用都是无刷新的页面加载，所以我们在页面`跳转`的处理和我们的普通的页面会有所不同。单页面应用的路由插件运用了 window 自带的无刷新修改用户浏览记录的方法，pushState 和 replaceState。

window 的 history 对象 提供了两个方法，能够无刷新的修改用户的浏览记录，pushSate，和 replaceState，区别的 pushState 在用户访问页面后面添加一个访问记录， replaceState 则是直接替换了当前访问记录，所以我们只要改写 history 的方法，在方法执行前执行我们的采集方法就能实现对单页面应用的页面跳转事件的采集了

<pre>

     // 改写思路：拷贝 window 默认的 replaceState 函数，重写 history.replaceState 在方法里插入我们的采集行为，在重写的 replaceState 方法最后调用，window 默认的 replaceState 方法

        collect = {}

        collect.onPushStateCallback : function(){}  // 自定义的采集方法

        (function(history){

            var replaceState = history.replaceState;   // 存储原生 replaceState

            history.replaceState = function(state, param) {     // 改写 replaceState
               var url = arguments[2];

               if (typeof collect.onPushStateCallback == "function") {
                     collect.onPushStateCallback({state: state, param: param, url: url});   //自定义的采集行为方法
               }

               return replaceState.apply(history, arguments);    // 调用原生的 replaceState
            };
         })(window.history);

</pre>


这块介绍起来也比较的复杂，如果你想了解更多，可以看我的另一篇博客[你需要知道的单页面路由实现原理](https://bailinlin.github.io/2018/04/28/history/)

#### 混合应用：app 与 h5 的混合应用我们要怎么进行通讯

>现在大部分的应用都不是纯原生的应用， app 与 h5 的混合的应用是现在的一种主流。

纯 web 数据采集我们考虑到前端存储数据容易丢失，我们在每一次事件触发的时候都用采集接口传输采集到的数据。考虑到现在很多用户的手机会有流量管家的软件监控，如果在 App 中 h5 还是采集到数据就传输给服务端，很有可能会让流量管家检测到，给用户报警，从而使得用户不再信任你的 App , 所以我们在用户操作的时候将数据传给 app 端，存储到 app。用户切换应用到后台的时候，通过 app 端的 SDK 打包传输到服务器，我们给 app 提供的方法封装了一个适配器

<pre>

    // app 与 h5 混合应用，直接将数信息发给 app
    collect.saveEvent = function (jsonString) {

        collect.dcpDeviceType && setTimeout(function () {
            if(collect.dcpDeviceType=='android'){
                android.saveEvent(jsonString)
            } else {
                window.webkit && window.webkit.messageHandlers ? window.webkit.messageHandlers.nativeBridge.postMessage(jsonString) : window.postBridgeMessage(jsonString)
            }

        },1000)
		}

</pre>

### 实现思路

>通过上面几个问题的思考，我们对埋点的实现大致已经有了一些想法，我们使用思维导图来还原下我们即将要做的事情，图片记得放大看哦，太小了可能看不清。

1. 我们需要暴露给业务方调用的方法
![nginx作用域](/images/sdk/method.jpeg)
2. 我们需要处理的事件类型
![nginx作用域](/images/sdk/event.jpeg)
3. SDK 的基本实现思路
![nginx作用域](/images/sdk/logic.jpeg)


### 去 github 上看代码吗

> 代码的篇幅比较长，就不放在博客里了，感兴趣的同学可以在 github 上看完整的项目。如果你有什么不懂的或者想和我交流的，欢迎在文章下面留言联系我

[web 浏览器指纹跨域共享](https://bailinlin.github.io/2018/03/05/cookie-share/)
[你需要知道的单页面路由实现原理](https://bailinlin.github.io/2018/04/28/history/)

[数据埋点是什么？设置埋点的意义是什么？](https://www.zhihu.com/question/36411025)
[数据采集与埋点](https://sensorsdata.cn/blog/shu-ju-jie-ru-yu-mai-dian/)
[美团点评前端无痕埋点实践](https://tech.meituan.com/mt-mobile-analytics-practice.html)
[如何清楚易懂的解释“UV和PV＂的定义](https://www.zhihu.com/question/20448467)