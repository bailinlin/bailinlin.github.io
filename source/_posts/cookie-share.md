---
title: web 浏览器指纹跨域共享
date: 2018-03-05 17:41:49
tags: cookie,跨域
---

> 概念：设备id 即设备指纹，用来表示用户设备的唯一性

### 问题

web 变量存储，我们第一时间想到的就是 cookie，sessionStorage，localStorage，但是这3种存储方式都和访问资源的域名相关。我们总不能每次访问一个网站就新建一个设备指纹吧，所以我们需要通过一个方法来跨域共享设备指纹

### 方法

我们想到的方案是，通过嵌套 iframe 加载一个静态页面，在 iframe 上加载的域名上存储设备id，通过跨域共享变量获取设备id，共享变量的原理是采用了iframe 的 contentWindow通讯，通过 postMessage 获取事件状态，调用封装好的回调函数进行数据处理

### 实现

SDK 采集端，调用方初始化的时候调用方法


    collect.setIframe = function () {
          var that = this
          var iframe = document.createElement('iframe')
          iframe.src = "http://localhost:82/"
          iframe.style = 'display:none'
          document.body.appendChild(iframe)
          iframe.onload = function () {
            iframe.contentWindow.postMessage('loaded','*');
          }

          //监听message事件
          window.addEventListener("message", function(){
                that.deviceId = event.data.deviceId

                console.log('获取设备id',that.deviceId)

                sessionStorage.setItem('PageSessionID',helper.upid())
                helper.send(that.getParames(), that.eventUrl);
                helper.sendDevice(that.getDevice(), that.deviceUrl);
          }, false);
    }

嵌套在 iframe 静态页面里的脚本

       <script>

            var getDeviceId = function() {
                    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
                            var r = Math.random() * 16 | 0, v = c == 'x' ? r : (r & 0x3 | 0x8);
                            return v.toString(16);
                    });
            };

            var hasCreated = false
            var _deviceId = ''
            if(document.cookie){
                    document.cookie.split(';').forEach(function(i){
                            if(i.indexOf('deviceId')>-1){
                                    hasCreated = true
                                    _deviceId = i.split('=')[1]
                            }
                    })
            }

            if(!_deviceId && (sessionStorage.getItem('deviceId')||localStorage.getItem('deviceId'))){
                    hasCreated = true
                    _deviceId = sessionStorage.getItem('deviceId')||localStorage.getItem('deviceId')
            }


            if(!hasCreated) {
                    _deviceId =  getDeviceId()
                    document.cookie = 'deviceId=' + _deviceId
                    sessionStorage.setItem('deviceId',_deviceId)
                    localStorage.setItem('deviceId',_deviceId)
            }

            //回调函数
            function receiveMessageFromIndex ( event ) {
                    console.log( 'receiveMessageFromIndex', event )
                    parent.postMessage( {deviceId: _deviceId}, '*');
            }
            //监听message事件
            window.addEventListener("message", receiveMessageFromIndex, false);

        </script>
