---
title: React + ES6 实践中遇到的问题
tags:
  - es6
  - react
id: 118
categories:
  - Javascript
date: 2015-07-25 17:56:39
---

**问题一：Cannot read property 'setState' of undefined**

在ES6的class中React是不会自动绑定this的，所以需要自己绑定：

将onClick={this.props.onBtnClick}改成onClick={this.props.onBtnClose.bind(this)}

或者在class的构造函数中绑定this：this.handleClose = this.handleClose.bind(this);

或者直接使用箭头函数()=&gt;

当然，用creatClass这种写法，就不会有这个问题了。

详见：https://facebook.github.io/react/docs/reusable-components.html#es6-classes

&nbsp;

**问题二：Uncaught TypeError: Cannot read property 'xxx' of undefined**

如果你调用不了某个prop，请检查是否在constructor里声明，例如这样子：

[js]
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
  }
}
[/js]

因为你没有在构造函数里传入，深入当然找不到咯。另外不要忘记调用super，因为es6中规定子类构造函数必须调用父类构造函数。

&nbsp;

**问题三：Adjacent JSX elements must be wrapped in an enclosing tag**

遇到这个问题，你可能会去不停的招，我哪个标签没有闭合？明明全都闭合了呀！！

你可以看看你的代码是不是这个样子的：

[js]
class MyComponent extends React.Component {
  render() {
    return (
      &lt;h1&gt;Welcome to&lt;/h2&gt;
      &lt;h2&gt;www.imyzf.com&lt;/h2&gt;
    )
  }
}
[/js]

是的，这里的每一个标签都闭合，可它还是会报错。因为上面的代码，按照JSX编译器的规则，会被翻译成以下内容：（实际上并不能编译，会报错的）

[js]
class MyComponent extends React.Component {
  render() {
    return (
      React.createElement(&quot;h1&quot;, null, &quot;Welcome to&quot;),
      React.createElement(&quot;h2&quot;, null, &quot;www.imyzf.com&quot;)
    )
  }
}
[/js]

发现问题了吧，**一个函数不能同时返回两个值，所以render中只能返回一个DOM结点**，这里你可以用div把内容包围起来，当然你要注意一下包围了以后会不会影响到css。