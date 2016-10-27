---
title: Vue.js 2.0源码测试分析
tags:
  - vue
  - test
id: 132
categories:
  - frontend
date: 2016-08-23 20:21:57
description: 在vue.js 2.0的源码中，可以看到有很多测试用例，并且编写了配置进行自动化测试。本文介绍了nightwatch、karma、jasmine等测试工具，以及vue.js如何使用它们进行e2e测试、单元测试和服务端渲染测试，并对相关源码结构进行了分析，阐述了该部分代码文件之间的关系。
---

测试是保证框架稳定性的重要方法，如今已经有很多前端测试工具可以使用，能够很方便地进行自动化测试。在vue.js 2.0的源码中，可以看到有很多测试用例，并且编写了配置进行自动化测试。如果需要对vue.js框架进行自定义修改，这些测试可以帮助你检查问题和保障稳定性。这里就将最近研究vue.js 2.0源码学习到的测试方法分享给大家。本文基于vue.js 2.0 rc3源码，一般来说正式版与rc版源码结构不会有多少变化。

### 测试工具

在研究vue的测试代码前，需要了解以下工具：

* `Nightwatch`: 基于node.js的e2e测试解决方案，通过发送HTTP请求到Selenium WebDriver来控制浏览器进行测试，其原理可以通过下图来表示。[![nightwatch](http://cdn.imyzf.com/img/blog/2016/vuejs-2-test-analysis/nightwatch.png)](http://cdn.imyzf.com/img/blog/2016/vuejs-2-test-analysis/nightwatch.png)
* `Selenium`: WebDriver: Nightwatch调用的后端程序，在vue测试的过程中不需要手动去控制，只要控制Nightwatch就行了。Selenium是基于java的，你会发现Nightwatch调用了一个jav文件，所以需要安装java来提供运行环境，支持的最低版本是java 7。
* `karma`: 前端测试运行工具，可以用他来自动调用不同的浏览器，并启动后端服务输出页面到浏览器。但是他只是一个test runner，要完成测试，还需要test framework，例如接下来介绍的jasmine。
* `jasmine`: 测试框架，支持在浏览器和node.js环境中运行，自带断言库，适合用来编写单元测试。所以在测试中，对每个测试用例的执行和判断，是由jasmine完成的。

### 源码分析

在vue的源码中，以下文件和测试有关

```
├── build
│   ├── karma.base.config.js  // karma公共配置，下面几个karma开头的配置文件都会调用
│   ├── karma.cover.config.js // 代码覆盖率测试配置文件
│   ├── karma.dev.config.js   // 开发阶段用的单元测试，只在chrome中运行
│   ├── karma.sauce.config.js // 使用SauceLabs进行云端单元测试
│   ├── karma.unit.config.js  // 完整的浏览器端单元测试，会在chrome、firefox和safari中运行
│   └── nightwatch.config.js  // nightwatch配置文件，进行e2e测试
├── coverage            // 运行覆盖率测试后出现的目录，保存了代码覆盖率报告
│   ├── lcov-report     // web版代码覆盖率报告，打开里面的index.html可以查看
│   └── lcov.info       // 保存了代码覆盖率信息，用相关工具可以查看，atom编辑器也有相关插件
├── examples            // vue的demo，在e2e测试中针对这些demo进行测试
├── package.json        // 定义了npm run test等命令和测试需要的npm包
└── test
    ├── e2e             // e2e测试，采用nightwatch进行
    │   ├── reports     // 运行测试后出现的目录，保存了xml格式的测试报告
    │   ├── screenshots // 运行测试后出现的目录，保存了测试失败时的浏览器截图
    │   ├── specs       // 测试用例，与examples目录对应
    │   └── runner.js   // 测试入口，执行该文件开始e2e测试
    ├── helpers         // 辅助测试的helper方法，例如自定义的jasmine matcher
    ├── ssr             // 服务端渲染测试，采用jasmine进行
    └── unit            // 单元测试，采用karma和jasmine进行
        ├── features    // vue功能特征测试用例，例如component、directive、filter等
        ├── modules     // vue功能模块测试用例，例如compiler、vdom等
        └── index.js    // 测试入口，引入所有helper和测试用例
```

### e2e测试

e2e，是end to end的缩写。e2e测试的方式是模拟用户行为，控制浏览器执行一系列的页面操作，例如点击按钮、输入文字等等，最后判断结果是否符合预期。在vue源码的test/e2e/specs目录下，可以看到全部e2e测试用例，其实这些用例和examples目录里的内容是一一对应的。Vue的e2e测试就是逐个运行了examples目录下的例子，然后判断他们是否符合预期。

在终端中执行下面的命令就可以进行e2e测试，然后会自动打开一堆浏览器：

```bash
npm run test:e2e
```

该命令会首先编译整个工程到dist/vue.js，然后通过node执行test/e2e/runner.js进行测试。在runner.js中，会配置好参数，然后调用./node_modules/.bin/nightwatch去执行build/nightwatch.config.js，所以不需要全局安装nightwatch。nightwatch.config.js中配置的selenium端口是4444，如果测试前4444端口被占用，会出现问题。

[![vue-e2e-test](http://cdn.imyzf.com/img/blog/2016/vuejs-2-test-analysis/vue-e2e-test.jpg)
](http://cdn.imyzf.com/img/blog/2016/vuejs-2-test-analysis/vue-e2e-test.jpg)运行测试后，可以看到图中的浏览器在自动操作界面。

受国内的网络限制，在进行e2e测试时会遇到两个问题：

一是执行npm install后会去安装chromedriver包，这是e2e测试中用来调用chrome的，但是要从谷歌的服务器上下载文件，而不是从npm服务器上下载，所以会遇到问题。比较麻烦的是npm不会使用系统的代理配置，需要添加代理变量：

```bash
# http代理
http_proxy="http://127.0.0.1:8888" npm install
# socket代理
http_proxy="socks5://127.0.0.1:7070" npm install
```

还有一种方法是使用淘宝的镜像安装chromedriver：

```bash
npm install --chromedriver_cdnurl=http://npm.taobao.org/mirrors/chromedriver
```

二是e2e测试中的commits用例，会去github拉取vue仓库的最新几次提交信息，然后会检测是否成功地以li列表的形式添加到页面中。这一步测试默认的超时配置是1000ms，但是github服务器在国外，1000ms内很有可能没有收到返回数据，会报错`Timed out while waiting for element <li> to be present for 1000 milliseconds.`，所以需要修改test/e2e/specs/commits.js：

```javascript
module.exports = {
  'commits': function (browser) {
    browser
    .url('http://localhost:8080/examples/commits/')
      .waitForElementVisible('li', 3000)
      // 将1000改成3000或者更高，等待数据拉取成功后插入
&amp;amp;amp;lt;li&amp;amp;amp;gt;标签
      ...
      .end()
 }
}
```

### 单元测试

单元测试也是vue测试中很重要的一部分，所有的接口、组件和功能特征都会进行测试。test/unit/features目录下为所有功能特征相关的测试用例，test/unit/modules目录下为所有功能模块相关的测试用例。Vue对这些用例的测试在build目录下有4种不同的配置，采用不同的命令执行：

```bash
# 只在chrome浏览器中快速运行测试，适合开发阶段使用，对应karma.dev.config.js
npm run dev:test

# 运行单元测试，对应karma.unit.config.js，会在多个浏览器中运行
npm run test:unit

# 运行代码覆盖率测试，对应karma.cover.config.js
npm run test:cover

# 运行云端测试，对应karma.sauce.config.js，这个基本上用不到，具体参见SauceLabs网站
npm run test:sauce
```

[![vue-coverage-report](http://cdn.imyzf.com/img/blog/2016/vuejs-2-test-analysis/vue-coverage-report.jpg)
](http://cdn.imyzf.com/img/blog/2016/vuejs-2-test-analysis/vue-coverage-report.jpg)图为代码覆盖率报告，vue.js 2.0 rc3覆盖率已经达到99.93%。

### 服务端渲染测试

服务端渲染(server side render, ssr)测试用例在test/ssr目录下，采用jasmine运行，与单元测试类似，所以这里就不再阐述了。使用下面的命令可以进行测试，直接调用jasmine并载入test/ssr/jasmine.json配置文件运行：

```bash
npm run test:ssr
```

最后需要说的是，vue提供了一条命令跑完所有测试，包括lint（语法检查）、flow（类型检查）、e2e（仅phantomjs环境）、代码覆盖率、服务端渲染5个测试：

```bash
npm run test
```
