---
title: 关于CSS一些细节问题
tags:
  - css
id: '88'
categories:
  - - css
date: 2017-01-03 18:59:13
---
本文是2016年的最后一篇文章，主要是将一些CSS细节问题整理一下。

* `background-position`取值为百分比的计算方式
* `margin`相邻折叠问题
* `position`的 `absolute`和 `relative`定位 `top、left、right、bottom`问题
* 覆盖样式问题

**1、background-position取值为百分比的计算方式**

background-position属性相信你用的不少，可是当取值为百分比时，你是否认为它是相对于背景图片的尺寸来计算（我之前也是这种想法，但看了大漠老师的《[你真的了解background-position](http://www.w3cplus.com/css/background-position-with-percent.html)》后，才知大错特错）

来看看下面的代码：

```
.box {   
  width:400px;   
  height:400px;   
  background-color:black;   
  background-image:url(mm.jpg);   /* 图片原尺寸150 * 225 */
  background-repeat:no-repeat;   
  background-position:50% 50%;  
}
```

相信 `background-position: 50% 50%`你用的不少，这是让背景图片居中，相当于 `center center`。

效果如下：

![](http://7s1r1c.com1.z0.glb.clouddn.com/t_sdfsdf2343124.png)

如果 `50%`是按照图片的尺寸（`150 * 225`）来计算的，那么其值转换为像素值应该是 `75px 112.5px`，你觉得背景图片能在 `400 * 400`的块里居中吗？答案很明显，是否定的，那是如何计算的呢？

其实官方说法是这样的：

当背景图片尺寸（`background-size`）不做任何的重置（也就是100% 100%）时，水平百分比的值等于容器宽度百分比值减去背景图片宽度百分比值。垂直百分比的值等于容器高度百分比值减去背景图片高度百分比值。

水平位置： (400 - 150) \* 50% = 125px 

垂直位置： (400 - 225) \* 50% = 87.5px

**2、margin相邻折叠问题**

在开发中，我们偶尔会遇到明明两个div都设置了 `margin`，可是它们之间的距离就是不等于两个div的 `margin`之间的和，这是为什么呢？其实是因为在某些情况下，两个或多个 `块元素`的相邻边界（`其间没有任何非空内容、padding、边框`）会发生合并成单一边界，也就是标题说的折叠。

先来看看兄弟块级元素的折叠，如下图所示：

![](http://7s1r1c.com1.z0.glb.clouddn.com/t_dfasdfsadfwer.jpg)

还要注意的是，父元素与其子元素之间也会发生折叠：

![](http://7s1r1c.com1.z0.glb.clouddn.com/t_margins2.jpg)

2个或多个块级相邻元素的外边距（`margin`）的折叠规则：

* 外边距都为正值时，取最大值
* 不全是正值时，则用正值减去（所有值的绝对值中）最大值
* 全为负值时，则取最小值

不发生折叠情况：

* 水平（左右）外边距不会折叠
* 浮动元素的外边距不会折叠，并且浮动元素与它的子元素之间也不会发生折叠
* 设置了 `overflow`且值不为 `visible`的块级元素与它的子元素之间的外边距也不会被折叠
* 绝对定位（`position:absolute;`）元素的 `margin`不与任何 `margin`发生折叠，并且与它的子元素之间的 `margin`也不会发生折叠

解决折叠的方法：

* 外层元素用 `padding`替代 `margin`
* 外层元素设置 `overflow:hidden`
* 内层元素加 `padding:1`或者 `border`
* 内层元素加浮动（`float`）或设为（`display:inline-block`）
* 内层元素使用绝对定位（`position:absolute;`）

**3、position的absolute和relative定位top、left、right、bottom问题**

**绝对定位**（`position:absolute`）的 `top、left、right、bottom`是相对于其具有相对定位属性（`position`不为 `static`）的父元素（不一定是其直接父元素，有可能是祖先元素）。

如果两者（`top、bottom`）同时存在时，只有 `top`起作用；如果两者（`left、right`）同时存在时，只有 `left`起作用。

**相对定位**（`position:relative`）元素会保留原来占有的位置，其后面的元素按原来文档流仍然保持原来的位置

* `top`的值表示对象相对原位置向下偏移的距离
* `bottom`的值表示对象相对原位置向上偏移的距离
* `left`的值表示对象相对原位置向右偏移的距离
* `right`的值表示对象相对原位置向左偏移的距离

如果两者（`top、bottom`）同时存在时，只有 `top`起作用；如果两者（`left、right`）同时存在时，只有 `left`起作用。

看一个例子：

![](http://7s1r1c.com1.z0.glb.clouddn.com/t_rresdfsd13.jpg)

```
<style>
.prev{   
  width:100px;  
  height:100px;   
  position:relative;   
  background:blue;   
  top:20px;   
}   
.next{   
  width:100px;   
  height:100px;   
  background:red;   
}   
.fl{   
  float:left;   
  margin:20px;   
}
</style>

<div class="fl">   
  <div class="prev" style="position:static"></div>   
  <div class="next"></div>   
</div>   
<div class="fl">   
  <div class="prev"></div>   
  <div class="next"></div>   
</div>
```

 

从上面的代码和效果图，你可以看出，设置了 `position:relative`的元素设置了 `top:20px`，虽然（相对其原位置）向下移动了 `20px`，可是并不会影响其后面的元素。

**4、覆盖样式问题**

比如我们有一个公共文件，其下有一个公共样式：

```
.box {
color: red;
}
```

有些时候，我们需要覆盖这个样式，比如将红色改为绿色。由于是公共元素，我们不能直接修改，但方法还是有多种：

 

```
/*第一种*/
.parent .box {
color: green;
}

/*第二种*/
.box {
color: green !important;
}
```

上面这些是我们常用的，但还有一个小技巧，是我们平常不太注意的，可以如下设置：

```
.box.box {
color: green;
}
```

今天就介绍这么多，如有错误，欢迎指正！ 原文链接：[http://ghmagical.com/article/page/id/tQOOxx1N1K1a](http://ghmagical.com/article/page/id/tQOOxx1N1K1a)
