---
title: HTTP与WebSocket
tags:
  - node.js
  - WebSocket
id: 47
categories:
  - frontend
date: 2015-03-18 22:25:55
---

HTTP与WebSocket最大的不同是，HTTP每次请求都得重新连接，服务器是被动的，不能主动向客户端发送消息，WebSocket可以一直保持连接，并且服务器能主动向客户端发送消息。

还是来一个生动的例子对比一下吧，假设这是一个网页版聊天室，客户端（浏览器）想要知道有没有新消息，

HTTP与服务器通信就像发邮件：

收件人：服务器
发件人：客户端
内容：我有收到新消息吗？

收件人：客户端
发件人：服务器
内容：抱歉，没有收到新消息

收件人：服务器
发件人：客户端
内容：我有收到新消息吗？

收件人：客户端
发件人：服务器
内容：抱歉，没有收到新消息

收件人：服务器
发件人：客户端
内容：我有收到新消息吗？

收件人：客户端
发件人：服务器
内容：有了！小明给你发送了新消息：Hello, world！

……

这样子的过程一直会重复下去，每次客户端都得重新向服务器发起请求（就像新的邮件），然后服务器响应（就像回复邮件），而且服务器是不能自动给客户端发邮件的，只能回复。（你有见过在打开浏览器前，一个网站主动出现吗？）

HTTP的缺点：每次都重新请求，而且要带上HTTP头（就像邮件里的收件人、发件人），导致传输数据量增大；每次请求都要结果3次握手，所需时间更多。虽然现在有long pull等技术，可以在一定时间内不需要重新3次握手，但是也不是很理想。

然后，WebSocket是这样，就像QQ聊天一样：

客户端：你好！
服务器：你好，有新消息我会告诉你的。
……（一段时间后）
服务器：新消息来了！小明给你发送了新消息：Hello, world!

是的，WebSocket就这么简单，就像QQ聊天的窗口一直开着一样，有消息了，服务器可以马上把数据发给客户端。

WebSocket的优点：服务器能主动发送数据给客户端，实现实时通信；建立连接以后，不会断开，不需要每次都发送HTTP头，减少数据传输量。

&nbsp;

下面看一个 websocket.org 给出的测试数据：

[![HTTP轮询与WebSocket对比](http://cdn.imyzf.com/img/blog/2015/http-and-websocket/poll-ws-compare.gif)](http://cdn.imyzf.com/img/blog/2015/http-and-websocket/poll-ws-compare.gif)

&nbsp;

蓝色表示用HTTP进行1秒/次的轮询，红色表示用WebSocket进行连接。

Use Case A: 1,000个客户端，HTTP: 6.6 Mbps，WebSocket: 0.015 Mbps

Use Case B: 10,000个客户端，HTTP: 66 Mbps，WebSocket: 0.153 Mbps

Use Case C: 100,000个客户端，HTTP: 665 Mbps，WebSocket: 1.526 Mbps

可以看出，随着客户端数量的增长，HTTP每秒传输量呈指数增长，WebSocket每秒传输量呈线性增长。

所以，如果高并发的实时应用需求，使用WebSocket是一个很好的解决方案，大大的提高了性能。目前Node.js对WebSocket的支持非常好，使用socket.io即可完成。
