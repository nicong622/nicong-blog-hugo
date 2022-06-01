---
title: "页面拥有ID的元素会创建全局变量"
date: 2015-09-14 21:16:40
tags: 前端杂谈
---

前段时间在网上看到一篇文章介绍了一些前端冷知识[文章在此](http://web.jobbole.com/83473/)，其中里面有说到了一点：页面拥有ID的元素会创建全局变量。<!--more-->例如：
```html
<div id='hello'>i'm a div</div>
```
然后我在控制台中输入id：
```javasccript
hello
```
得到如下结果：
![](/images/id1.png)

### 扩展1
假设页面中有两个或者多个元素，它们拥有相同的id（实际上是不允许的），那结果会是怎样？
```html
<div id="hello">i'm a div</div>
<div id="hello">i'm div number2</div>
<div id="hello">i'm div number3</div>
<div id="hello">i'm div number4</div>
<div id="hello">i'm div number5</div>
```
在控制台输入id`hello`之后：
![](/images/id2.png)

也就是说，所有拥有相同id的元素都会被选中，并且存在一个数组里面。情况有点像用`document.getElementsByClassName()`。同时要注意到如果用`document.getElementById()`只会获取到第一个匹配的元素，也就是`<div id="hello">i'm a div</div>`。

### 扩展2
如果在其中一个`div`里面再嵌套一个id为`hello`的元素，情况会是如何呢？
```html
<div id="hello">i'm a div</div>
<div id="hello">i'm div number2
    <div id="hello">i'm a child</div>
</div>
```
输出结果：
![](/images/id3.png)

可以看到数组中有三个元素，也就是说页面中拥有相同id的元素有多少个，数组中就有多少个元素，即使某些元素之间存在嵌套关系。