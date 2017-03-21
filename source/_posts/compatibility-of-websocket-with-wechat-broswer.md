---
title: 微信内置浏览器对WebSocket的兼容性
tags:
  - WebSocket
  - 微信
id: 55
categories:
  - frontend
date: 2015-03-22 23:12:39
---

WebSocket是一个非常好的东西，用于开发实时应用非常好。它在微信内置浏览器上的兼容性怎么样呢？我做了一个简单测试，虽然不是很准确，但也值得参考。

测试方法很简单，我把socket.io官网的聊天室Demo发到朋友圈，看看大家能否正常使用。

[![微信websocket测试](https://cdn.imyzf.com/img/blog/2015/compatibility-of-websocket-with-wechat-broswer/1.jpg)](https://cdn.imyzf.com/img/blog/2015/compatibility-of-websocket-with-wechat-broswer/1.jpg)

总结了一下大家的反馈如下：

*   Android版微信没问题，因为使用的是QQ浏览器内核，不受本身系统浏览器影响。
*   iOS版微信没问题，使用的是Safari浏览器
*   WindowsPhone版微信没问题，但是只测试了WP8
所以，如果网页是为微信设计的，可以放心使用WebSocket了。

最后附上一张caniuse.com公布的WebSocket浏览器端兼容性对比（点击查看大图）。虽然Android4.3及以下系统浏览器都不支持WebSocket，但好在微信内置QQ浏览器内核，解决了这一问题。

[![Websocket浏览器兼容性对比](https://cdn.imyzf.com/img/blog/2015/compatibility-of-websocket-with-wechat-broswer/2.png)](https://cdn.imyzf.com/img/blog/2015/compatibility-of-websocket-with-wechat-broswer/2.png)
