---
title: "获取图片尺寸"
date: 2015-03-30 11:51:02
tags:
 - 获取图片尺寸
---

最近在一次笔试中遇到了一道题目，其中要求在未显式设置图片大小的情况下获取图片的大小，这里收集了几种方法。

<!--more-->

1. ##通过读取图片的头文件信息

		var img = document.getElementById( 'imag' );
		var imgheader = new Image();//关键
		imgheader.src = img.src;
		console.log( imgheader.width,imgheader.height );

2. ##getBoundingClientRect()方法

	调用元素的`getBoundingClientRect()`方法会返回一个对象，这个对象的属性包括`top,bottom,left,right`。其中，`top,left`分别表示元素的左上角的X和Y坐标，而`bottom,right`分别代表元素右下角的X和Y坐标。

    在非IE浏览器中这个对象还会有两个属性：`width,height`，分别代表当前元素的宽度和高度。但IE不支持这两个属性。要在IE中获取元素的宽度和高度可以用一下方法：

      	w = obj.right - obj.left;
      	h = obj.bottom - obj.top;


[返回顶部](#header)

