---
title: node.js里的forEach()也是异步的吗？
tags:
  - node.js
  - 同步
  - 异步
id: 6
categories:
  - Javascript
date: 2015-01-28 14:59:00
---

node里几乎所有用到回调函数的地方，都是异步的，回调函数后面的代码很可能比回调函数中的代码后先执行，特别是数据库操作。当然，node也提供了同步版本的函数，例如文件操作，fs.readfilesync()是fs.readfile()的同步版本。

那么问题来了，foreach()是不是异步的呢？按理说，没有加sync，应该是异步的呀。

[js]
var arr = ['a', 'b', 'c'];
var str = '123';
arr.foreach(function(item) {
 str += item;
 while (true) {}; //用一个死循环，卡死它~~
});
console.log(str);
[/js]

运行上面的代码，结果它就这么卡死了，没有任何输出。。

所以说，node里的**foreach()是同步的**！！

第一次用node的时候，没有考虑过这个问题，按同步的写了，写突然想到，测试后虚惊一场，以为以前的代码都写错了。

如果在某些情况下，需要异步处理foreach，谷歌了一下，有个node-array，可以试试看~~ 传送门：[https://github.com/cfsghost/node-array](https://github.com/cfsghost/node-array)