---
title: "Object.hasOwn vs Object.hasOwnProperty"
date: 2022-06-28T15:00:11+08:00
draft: false
toc: false
images:
tags: 
  - js
---

[`Object.hasOwn()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwn) 是 ES2022 中的新特性，他它是一个**静态**方法，作用是提供一种安全的方法来检查某个对象是否自主拥有（不是继承而来的）某个属性。看起来作用跟 `Object.hasOwnProperty()` 一样，那为什么还要提出 `Object.hasOwn()` 呢？

原因有以下几个：

1. 由于 `Object.hasOwnProperty()` 方法是从对象的原型 `Object.prototype` 上继承而来的，如果用户在对象上定义了一个同名的方法，就会出现意料之外的结果，即所谓的“原型污染”。
2. 对于用 `Object.create(null)` 创建出来的对象，由于对象的原型就是 `null` ，所以对象本身没有 `hasOwnProperty()` 方法。虽然可以用 `Object.prototype.hasOwnProperty.call()` ，但是这样看起来就比较笨重了，并且 Eslint 的内置规则是禁止直接使用 `Object.prototype` 上的方法的。

`Object.hasOwn()` 就是为了解决上面的问题而提出的，所以应该用这个方法来代替 `Object.hasOwnProperty()`。

**注意**：不过需要注意的是，你依然可以重新定义 `Object.hasOwn()` 方法。
