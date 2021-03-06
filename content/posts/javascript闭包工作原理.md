---
title: javascript闭包工作原理
date: 2015-10-12 20:18:55
tags:
 - javascript
---

闭包是JavaScript的一个重要内容，但我以前一直没搞清楚它的工作原理，直到看到了[一篇文章](http://blog.leapoahead.com/2015/09/15/js-closure/)。

下面就简单的概括一下。

<!-- more -->

## 作用域链（scope chain）

在JavaScript中，每一个function都有一个对应的*作用域对象*，用来保存function中的局部变量以及参数。你可以把这个*作用域对象*想象成一个普通的JavaScript对象，但你不能直接获取这个对象，你只能修改它的属性（也就是修改局部变量）。当你在函数外部定义变量时，这些变量就会添加到全局对象中，这些变量就是我们平常说的全局变量。显然，全局变量的作用域就是全局对象。

当在function中嵌套另一个function的时候，就会出现*父级作用域*。把所有的作用域对象像链条一样一个一个的链接起来就形成了一个*作用域链*。作用域的顶部是一个全局对象。当JavaScript中的function需要查找（或者叫做变量解析）某一个变量x的时候，它首先会在当前函数的*作用域对象*里面查找，如果没有找到，就会在它的父级作用域对象中一级一级的往上查找。如果到最后还是没有找到，就会抛出一个错误：`ReferenceError`。

在这里可以引出一个关于JavaScript性能优化的小tips：当你需要在一个函数中多次的引用一个外部变量时，可以先把它保存在一个局部变量中，再对这个局部变量进行操作。这样就可以省去在父级作用域中多次查找变量的时间。

## 对作用域的引用

有一下一段代码：
```javascript
var foo = 1;
var myfunc = function(){
    var bar = 2;
    var innerfunc(){
        console.log(foo);
    }
    innerfunc();
}
myfunc();
```

当函数被定义时，这个函数的标识符就会被添加到当前作用域中（在上面的代码中就是全局作用域），标识符所引用的是一个函数对象，这个函数对象包含了函数的源代码和一些内部属性，其中一个我们关心的就是[[scope]]。[[scope]]所引用的是这个函数在定义时所能直接访问的作用域（这里是全局作用域）。如果函数是嵌套在另外一个函数中，那么它的[[scope]]就是指向外层函数的作用域。这一点对于理解闭包**很重要！**

我们知道，在`myfunc`被调用的时候会创建一个新的作用域，用来保存局部变量和参数，这个作用域的父级作用域就是定义`myfunc`时所处的作用域，也就是 myfunc 的[[scope]]所指向的作用域。

调用`myfunc`时,会在内部调用`innerfunc`，这时JavaScript需要在`innerfunc`的作用域中查找`foo`,没找到，于是在`innerfunc`的[[scope]]所指向的作用域（也就是`myfunc`的作用域）中查找，还是没找到，继续在`myfunc`的[[scope]]所指向的作用域（全局作用域）中查找，找到了`foo = 1`，查找结束。

当函数执行完并返回的时候，再也没有一个对象会引用它的作用域，这个作用域就可以被垃圾回收器回收，因此也就不能继续访问函数内部的变量了。上面的代码中，当`myfunc`执行完毕并返回后，就不能再访问它内部的`bar`了。

## 返回引用
上面讨论到了，在函数调用结束并返回之后，由于它的作用域会被回收而不能继续访问它的局部变量。假设我们把函数作用域的引用返回，并保存在一个外部的变量里，是不是就意味着可以继续访问函数的局部变量呢？答案是肯定的，这也就是我们平常说的闭包所做的事情了。

```javascript
var counter = function(){
    var a = 1;
    var add = function(){
        a++;
        console.log(a);
    }
    return add;
}
var myCounter = counter();
myCounter()     //2
myCounter()     //3
myCounter()     //4
```

上面是一个简单的闭包。调用`counter`的时候`counter`函数返回了一个`add`函数并赋值给`myCounter`，使得`myCounter`指向了`add`。同时，由于`add`的[[scope]]保存了对`counter`作用域的引用，所以即使在counter调用结束了，`myCounter`依然可以访问到`counter`中的变量`a`。

需要注意的是作用域链是不会被复制的，每次函数调用只会往作用域链下面新增一个作用域对象。

有一个经典的闭包例子：
```javascript
var arr = [];
for(var i=0;i<3;i++){
	arr[i] = function(){return i};
}
```
当依次调用`arr[0](),arr[1](),arr[2]()`的时候会得到什么结果呢？

## 总结

闭包是什么？
闭包是一个对象，它包含了对函数对象的引用以及对作用域对象的引用，所以可以通过闭包在函数外部访问到函数内部的局部变量。实际上，JavaScript所有的对象都是闭包。

闭包什么时候被创建和销毁？
由于所有JavaScript对象都是闭包，所以在定义一个函数的时候就创建了一个闭包；而当这个闭包没有被任何对象引用的时候它就会被销毁。




