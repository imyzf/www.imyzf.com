---
title: 在NW.js中调用Node.js的方式及孤儿进程的解决
tags:
  - node.js
  - nw.js
categories:
  - frontend
keywords: nw.js, nwjs, nw, node.js, node, javascript
description: 在NW.js中有多种方式调用Node.js，除了官方文档中说明的直接在前端代码调用、通过node-main调用，本文还介绍了如何植入Node.js可执行文件，进行更高级的调用以解决一些疑难杂症。
---

NW.js是一种非常不错的跨平台桌面客户端开发方案，通过他可以直接调用Node.js API，结合了浏览器端和Node.js端开发的优势。根据官方文档的说明，提供了两种方式调用Node.js。针对一般场景，这两种方式已经足够，但是对于复杂的应用，可能会出现问题，所以这里我还会介绍第三种特别的调用方式。

## 在前端代码中直接调用
在前端代码中调用是最简单的方式，只要把模块导入进来即可，包括自己用NPM安装的模块也可以导入。

```html
<script>
  const fs = require('fs')

  if (fs.existsSync('/Users/me/www.imyzf.com')) {
    alert('exist')
  } else {
    alert('not exist')
  }
</script>
```

## 通过node-main调用
在`package.json`中定义`node-main`配置，就可以在启动应用时，后台运行js脚本，而且不会随着页面改变而重新加载，适合跑一些后端服务。

```javascript
{
  ...
  "name": "imyzf-demo",
  "node-main": "server/index.js",
  ...
}
```
我们可以直接在`server/index.js`中使用`module.exports`导出方法，然后在前端使用`process.mainModule.exports.xxx()`调用，不需要HTTP等协议进行前后端通信。

## 调用Node.js可执行文件
如果需要通过子进程的方式来启动NPM、Webpack之类的工具库，以上两种方式就有可能出问题，所以我们需要添加Node.js可执行文件，然后在NW.js中调用，实现Node.js支持的所有功能。

### 环境准备

由于我们的用户不一定会安装Node.js，所以带上自己的Node.js可执行文件是很有必要的。我们可以从Node.js官网下载程序包，然后解压，建议放到`node_modules`下，并在`node_modules/.bin`中建立软链接。建立软链接的好处是，我们在代码中只要调用`node_modules/.bin/node`就可以了，更新Node.js版本以后，不需要更改代码。如果是Windows环境下，软连接比较麻烦，只能直接调用可执行文件了。

目录结构如下：

```
├── node_modules
│   ├── .bin
│   │   └── node -> ../.node-v7.5.0-darwin-x64/node
│   └── .node-v7.5.0-darwin-x64
│       └── node
└── server
    ├── index.js
    └── runner.js
```

在上面的目录中，我们把Node.js放到`node_modules`下可以防止其被提交到git，而且不会影响代码目录结构。在`node-v7.5.0-darwin-x64`的目录名前面加上点，是为了防止NPM把他当做NPM包解析后，给出找不到package.json的警告。

如果要与其他小伙伴合作开发，可以写一个shell脚本自动下载Node.js可执行文件，添加到package.json的`scripts.postinstall`配置中，实现自动化。

最后打包应用的时候，一定要记得把`node_modules`下和Node.js相关的文件打进去。

### 具体调用

做好了以上环境准备，接下来讲讲如何调用。主要思路是利用`child_process`模块的`spawn`方法去调用node可执行文件。

我们将runner.js添加到package.json的`node-main`中，使其在我们的应用启动时加载。


```javascript
const spawn = require('child_process').spawn

const childProcess = spawn('./node_modules/.bin/node', ['server/index.js'])

// 在node-main退出的时候，杀死子进程
process.on('exit', function () {
  childProcess.kill()
})
```
这样子之后，index.js中代码就在我们自己添加的node.js中运行了。

如果需要输出子进程的命令行日志，监听子进程`stdout`和`stderr`的`data`事件即可。

### 孤儿进程

使用了子进程后，可能会出现一个问题，就是主程序退出以后，子进程可能还残留，变成孤儿进程。我在Mac版的NW.js v0.19.5 中测试发现，前端窗口调用`gui.App.quit()`并不会触发`node-main`进程的`exit`事件，导致留下孤儿进程。于是我们需要在前端的`close`方法中手动触发。

```javascript
nw.Window.get().on('close', () => {
  gui.App.quit()
  process.exit()
})
``` 
