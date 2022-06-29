---
title: "如何选择用 Map 还是 Object"
date: 2022-06-29T15:26:39+08:00
draft: false
toc: false
images:
tags: 
  - js
---

在很多场景里，我们似乎都可以用 Map 来代替 Object 来完成同样的功能，那么这两者到底要如何选择呢？[这篇文章](https://www.zhenghao.io/posts/object-vs-map)做了详细的描述和对比。

结论是：

- 如果在开发阶段就有固定且属性个数有限，可以使用 `Object` ，例如配置列表；
- 相反，如果 keys 的数量和类型不能提前知道，并且需要频繁更新的，就应该使用 `Map` ，例如 `event emitter`；
- 在插入、删除、遍历的时候，`Map` 的性能都比 `Object` 好，除非 keys 是比较小的整数。

