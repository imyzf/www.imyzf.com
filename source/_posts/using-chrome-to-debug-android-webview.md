---
title: 使用chrome调试安卓WebView里的网页
tags:
  - chrome
id: 70
categories:
  - frontend
date: 2015-04-01 23:08:47
---

一直在想，如果手机里的WebView也能和chrome一样审查元素就好了，结果今天，真的找到了！

该功能支持安卓4.4及以上，进入手机设置 - 开发者选项，打开USB调试，连接电脑。然后进入chrome://inspect/#devices，就会看到下图的内容（此时我已经在手机上打开自带的浏览器）。

[![chrome审查元素](https://cdn.imyzf.com/img/blog/2015/using-chrome-to-debug-android-webview/1.png)](https://cdn.imyzf.com/img/blog/2015/using-chrome-to-debug-android-webview/1.png)
点击inspect，就可以进入审查元素界面。

[![审查元素](https://cdn.imyzf.com/img/blog/2015/using-chrome-to-debug-android-webview/2.png)](https://cdn.imyzf.com/img/blog/2015/using-chrome-to-debug-android-webview/2.png)

更神奇的是，不仅仅是自带浏览器，其他应用里的WebView也可以，例如这是某个应用里的游戏，用的也是网页：

[![应用内网页审查元素](https://cdn.imyzf.com/img/blog/2015/using-chrome-to-debug-android-webview/3.png)](https://cdn.imyzf.com/img/blog/2015/using-chrome-to-debug-android-webview/3.png)

然后你就可以研究研究网页，想干什么就干什么了。。

但是该功能不支持UC和微信这类用了自己的浏览器内核的应用。

最后说一点安全提醒：
开了USB调试以后，电脑不仅可以给你安装应用，连WebView的内容都可以获取到，所以一般情况下还是别开了，特别是公共电脑，不然你手机里的东西别人一览无余了。

关于远程调试，还有更多内容，例如远程端口、代理之类的，详见：https://developer.chrome.com/devtools/docs/remote-debugging
