---
title: "innerHTML, outerHTML, innerText, outerText之间的区别"
date: 2015-03-24 23:34:43
tags:
  - javascript
---

之前一直迷惑`innerHTML `，`outerHTML`以及`outerText`，`innerText`之间的关系。今天自己写了一些代码然后总结一下。


<!--more-->

html:

    <div id="outerDiv">
		i'm outer div
		<div id="innerDiv">
			i'm inner div
			<p id="innerP">
				i'm inner p
			</p>
		</div>
	</div>

执行以下语句:

    var outerDiv = document.getElementById('outerDiv');
    var innerDiv = document.getElementById('innerDiv');
    var innerP = document.getElementById('innerP');

	outerDiv.innerHTML;

得到：

	"
		i'm outer div
		<div id="innerDiv">
			i'm inner div
			<p id="innerP">
				i'm inner p
			</p>
		</div>
	"

 执行语句：

 	outerDiv.outerHTML;

输出结果：

	"<div id="outerDiv">
		i'm outer div
		<div id="innerDiv">
			i'm inner div
			<p id="innerP">
				i'm inner p
			</p>
		</div>
	</div>"

而执行以下两条语句

	outerDiv.innerText;
    outerDiv.outerText;

将会得到相同的输出：

	"i'm outer div
	i'm inner div
	i'm inner p"

而事实上

	outerDiv.innerText === outerDiv.outerText

结果为*true*。

因此可以总结出以下几点：

- `outerText`和`innerText`包含的是当前对象中的所有文本，包括嵌套的文本；
- `innerHTML`的范围是当前对象中包含的所有标签和文本，包括嵌套的，但不包括当前对象自身的标签；
- `outerHTML`是在`innerHTML`的基础上加上自身标签；
- 还应该注意到这几个属性的值都是字符串。

[返回顶部](#header)