---
title: javascript数组降维
date: 2016-02-19 13:08:27
tags:
 - JavaScript
---

今天看到了一篇文章，[《优雅的数组降维——Javascript中apply方法的妙用》](http://www.cnblogs.com/front-end-ralph/p/4871332.html)。里面介绍了3种多维数组降维的方法，个人感觉比较实用，于是记录一下。

<!-- more -->

下面以二维数组转一维数组为例来说明。

### 1、一般的转换

看到数组降维，我们可能首先就想到用循环，这也是最直观最简单的。

``` javascript
function reduceDimension(arr) {
    var reduced = [];
    for (var i = 0; i < arr.length; i++) {
        for (var j = 0; j < arr[i].length; j++) {
            reduced.push(arr[i][j]);
        }
    }
    return reduced;
}
```

这个方法的思路很简单，利用嵌套的循环来遍历二维数组中的每一个元素。



### 2、利用 concat()

js 的 concat() 函数用于拼接数组，它的参数可以是一个元素也可以是一个数组，如果是数组的话，数组中的每一个元素都会按照顺序拼接到新的数组的尾部。对于二维数组来说，arr[i] 就是一个数组。因此 arr[i] 就可以作为 concat() 的参数来，于是我们可以改进一下第一种方法，省去了内层的循环。

``` javascript
function reduceDimension(arr) {
    var reduced = [];
    for (var i = 0; i < arr.length; i++){
        reduced = reduced.concat(arr[i]);
    }
    return reduced;
}
```



### 3、利用 concat() 和 apply()

concat() 的参数可以只有一个，也可以有多个，如果是多个参数的话，就会按顺序进行拼接。我们的二维数组 arr，从另一种角度来看也是一维数，它的每一个元素都是一个数组，我们现在只需要把它的每一个元素（数组）作为参数，一次过传给 concat()，这样就可以不用循环了。

所以现在的问题就是怎么样**不用**循环把 arr 包含的多个数组一次过传给 concat()。幸运的是在 js 里面我们有 apply() 函数。apply() 函数的第二个参数是一个数组，这个数组中的每一个元素会依次作为被调用函数的参数。利用这个特点，我们就可以解决上面的问题了。

``` javascript
function reduceDimension(arr) {
    return Array.prototype.concat.apply([], arr);
}

//对于二维数组，arr = [[1,2], [3,4], [5,6]]
//上面的函数就相当于下面的函数

function reduceDimension(arr) {
    return [].concat(arr[0], arr[1], arr[2]);
}
```



相对于前面两种方法，第三种方法显得更加简单，巧妙。