---
title: if (!floor) 小明.跳楼(); 请问小明会在哪些楼层跳楼？
tags:
  - c++
id: 9
categories:
  - others
date: 2014-06-21 16:03:00
---

看到标题请先思考一下这个奇葩的问题。。答案在文章最后揭晓。。

会出现这个问题的起源是这样的，一个同学问我：

```cpp
int main()
{
    int i = -1;
    cout &lt;&lt; !i &lt;&lt; endl;
}
```

为什么输出是0！！！

我查了一些资料，给出的解答是：
> 操作符的对象是bool类型，所以运行时先会把int转换成bool，-1转换成bool是true，所以输出就是0了！
很多人都会忘记了这一点，**只要表达式的值为非0，即为“真”。**

但是为什么会出现0而不是false呢？在cout中，有std::boolalpha和std::noboolalpha这两个选项，分别表示以字母（true, false输出和以数字（1, 0）输出。

[![](http://cdn.imyzf.com/img/blog/2014/a-funny-thing-about-cpp-boolean-variable/1.jpg)](http://cdn.imyzf.com/img/blog/2014/a-funny-thing-about-cpp-boolean-variable/1.jpg)

用codeblocks测试后发现，**默认的是std::noboolalpha**，这一点是不是又有很多人不知道呢？

然后同学说，他可以去跳楼了，学了这么久都不知道。。。

但是考虑到自己的生命安全，他选择0楼，于是我联想到刚刚讲过的问题，编出了这个题目，强烈建议出卷老师采用啊！

所以最终的答案是：<span style="color: #808080; background-color: #808080;">0楼</span>（鼠标选中文字查看）
