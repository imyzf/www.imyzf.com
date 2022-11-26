---
title: 官方揭秘！你的颜色是这样算出来的……
tags:
  - frontend
  - javascript
categories:
  - frontend
date: 2021-05-31 14:55:00
description:
---

![主题图](https://cdn.imyzf.com/img/blog/2021/color-quiz-theme.jpg)

上周三，你的朋友圈是不是这样子的？

![](https://cdn.imyzf.com/img/blog/2021/color-quiz-girl.jpg)

在我们的颜色测试广泛传播的同时，有网友开始分析起了本次活动的计算逻辑，甚至反编译出了所有可能的颜色情况。作为本次活动的核心开发人员，接下来将为大家介绍颜色测试活动的技术细节。

> 小剧透：本文将在最后重点介绍大家最想了解的结果计算逻辑

## 整体结构

本次活动的 H5 其实是一个单页应用（SPA），通过 react-router 进行路由控制，内部包含了 13 个页面，包括首页、问题页、结果页等部分，其中每个问题都是一个页面。页面之间采用了 react-transition-group 实现淡入淡出的切换效果，问题页之间用 canvas 实现了类似幕布拉动的切换动画。

答题类的页面与一般的 H5 页面的不同之处在于，用户的操作路径是确定的，即每个页面的下一页路由是固定的，所以我们在 router 层面做了优化，提前预加载了下一个页面，这样做的目的有两点：

* 优化体验，点击立即出现下一页，无加载过程
* 很多页面内有视频，需要提前加载 DOM 节点，才能通过 `click` 事件触发 `video` 标签的播放，同时也实现了视频的预加载

<img src="https://p5.music.126.net/obj/wo3DlcOGw6DClTvDisK1/9194415962/895d/5a05/c224/3cc74b02676816e4a25583975016446c.png" width="300" />

如图所示，下一页会提前加载，隐藏在当前页面底下。

## 动画效果

本次活动中的页面运用了大量动效，提供了良好的用户体验。我们使用了很多方式来实现动效，主要分为两类：

* 预渲染：对于复杂的、没有交互逻辑的动画，我们采用动效师预先渲染好的视频，以取得最佳的性能和兼容性，例如大部分问题的背景动画。唯一的缺点就是需要加载更大的资源，这一点通过最大程度压缩视频体积和上面提到的预加载得以解决。
* 实时渲染：对于有交互要求的动效，我们采用 canvas、WebGL、物理引擎等方式实时渲染，提供更高的可玩性。

接下来将介绍一下几个比较酷炫的动效。

### 翻页动效

每个问题页面之间的切换会有带弹性的幕布拉动效果，我们采用了 canvas 进行实现，基于贝赛尔曲线进行绘制。

<img src="https://p5.music.126.net/obj/wo3DlcOGw6DClTvDisK1/9196604980/89b1/c404/08a2/987e38e6dc6c7e96c71effeb74c2b0e3.png" width="300" />

如图所示，用户触发跳转下一页的点击操作后，我们使用 P1-P5 五个点构成的灰色遮罩闭合区域遮挡当前页面。其中 P4 和 P5 是固定的点，用于确定右边界。我们通过不断向左移动 P1、P2 和 P3 的 x 轴坐标，并且修改贝塞尔曲线控制点值，实现拉动效果。这里用到的最核心的 canvas API 是 `bezierCurveTo` 方法。

### 云层动效

第 5 个问题中，背景中出现了云层动效，这一部分我们基于 three.js 实现，采用了 WebGL 进行渲染。这里的云朵采用了着色器材质（ShaderMaterial）载入顶点着色器和片元着色器，贴图进行渲染，然后移动相机位置，模拟穿梭效果。这里每朵云出现的位置都是随机的，不同人看到的都不一样。

### 掉落动效

第 7 个问题中，进入页面会有按键和物品掉落的效果，这里采用了物理引擎 Matter.js 模拟了自由落体运动和碰撞效果。这里掉落后的位置也是随机的，千人千面，更加真实。

## 结果计算

接下来将为大家揭秘测试结果是如何计算出来的。

1、每个选项都有对应的数个颜色，例如第一题：

* 选项 A：金、绿
* 选项 B：紫、银、橙
* 选项 C：粉、蓝

2、我们会记录每个题目的选择，在最后计算的时候，将对应的颜色进行累加计数。例如第一题选了 A，我们会在计算的时候将`金`和`蓝`各加 1。

3、至于单色还是双色，是根据第 8 题的选择来判断的，如果选了“悲伤”，结果就是单色，选了“浪漫”，结果就是双色。

4、如果是单色，就取单色计数最高的为颜色作为结果。

5、如果是双色，就取组合两色之和最高的颜色作为结果，例如，假设`橙+金`计数之和是最高的，结果就是`橙+金`。当然，这里的组合是有限制的，只有 9 种预设的组合，所以我们计算的时候将结果限定在了这 9 种之内。

6、如果在排序时遇到了求和结果相同的颜色或组合，我们会按照策划同学给出的优先级取结果。例如，假设`橙`和`金`的计数都是 5，我们会按照`橙>金`的优先级，取`橙`为结果。

举个例子，某位小伙伴的选择是：

<!--  [2,3,1,3,3,3,3,1] { '紫': 3, '银': 6, '橙': 3, '金': 3, '绿': 2, '粉': 2, '蓝': 3 } 金橙 -->

```js
[B, C, A, C, C, C, C, A]
```

那么他的颜色计数是

```js
{
     '紫': 3, '银': 6, '橙': 3, '金': 3, '绿': 2, '粉': 2, '蓝': 3
}
```

由于最后一题选择了“浪漫”，所以结果是双色，按优先级求和，“金+橙”最大（排除不存在的结果组合），所以结果是“金+橙”。

<img src="https://p5.music.126.net/obj/wo3DlcOGw6DClTvDisK1/9204378352/6d10/d2d1/4dcb/fd6961fbbf7697ba0f3b0e8396d213d9.png" width="300" />

本次测试总共有 `2^7*2=4374` 种选择路径，有 7 种单色结果和 9 种双色结果，总共 16 种结果。

单色结果：

```js
['绿', '橙', '银', '紫', '蓝', '金', '粉']
```

双色结果：

```js
['粉金', '金橙', '粉紫', '金蓝', '金紫', '橙粉', '蓝粉', '金绿', '橙绿']
```

由于结果总数相对可控，所以计算逻辑都在前端完成，并且读取内置的配置展示文案，本次活动并没有后端同学参与开发。

### 小插曲

另外值得一提的是，本次的结果计算逻辑中，并没有将颜色和对应的英文进行映射，而是全程使用了中文。例如我们的结果配置文件中，直接使用了中文 key：

```js
export default {
    蓝: {
        attracted: ['橙粉', '粉金'],
        keepAway: ['金', '银'],
        ......
    }
}
```

目前 JS 对 Unicode 的支持已经足够好，甚至支持 [Unicode 变量名](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Grammar_and_types#%E5%8F%98%E9%87%8F)，在开发完成后，我们进行了最低版本为安卓 5.0 的兼容性测试，并没有发现任何问题，实际上线后也没有遇到这方面的问题。

甚至我们在 less 代码中使用了中文类名：

```less
.金 {
    background: rgb(228, 198, 114);
}
```

据[相关资料](https://stackoverflow.com/questions/19123336/can-i-safely-use-unicode-characters-e-g-accents-in-css-class-names-or-ids)显示，从 HTML 4.01 开始，就支持了 Unicode 字符作为 class 属性名。当然我们的工程由于启用了 CSS Module，这里的中文类名会被转换为纯英文的 hash 字符串，不用考虑兼容性问题。

虽然我们不推荐在编程过程中大范围使用中文，但是在该场景下，结果的颜色枚举数量较多，使用中文作为每个颜色的唯一标识，更加直观，可以增加代码可读性，减少将颜色翻译成英文并进行映射的工作量，也是非常值得的做法。

## 结语

本次活动开发的技术细节就介绍到这里，可能有小伙伴还会问，为什么相同的回答路径会产生不同的结果？这里就不细说了，保留一点神秘感，等着大家去挖掘吧！