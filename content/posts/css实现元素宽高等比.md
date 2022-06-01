---
title: css实现元素宽高等比
date: 2016-05-03 14:04:44
tags:
---

### 实现

如何用纯 css 实现元素的宽高等比？方法很简单，代码如下：

html :
```
<div class="out">
  <div class="in"></div>
</div>
```
css :
```
.out{
  position: relative;
  width: 50%;
  padding-bottom: 50%;
}
.in{
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  background-color: #4AE3B5;
}
```

### 原理解析

这里的关键点就是 `padding-bottom` 这个属性。当它的值为百分比的时候，它参照的是这个元素的包含块的宽度。

在这里，我们把元素的 `width`，`padding-bottom` 的值设置成相等的**百分比**，那么最终计算出来的`width` 和 `padding-bottom` 的值也是一样的。同时，由于没有显式的设置 `height`，所以元素的高度就是 `padding-bottom` 的值。这样就可以在不同宽度下实现元素的宽高等比了。

把 `padding-bottom` 换成 `padding-top` 也有同样的效果。