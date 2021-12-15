---
title: "在 web 环境中获取 uuid 的简单方法"
date: 2021-12-15T10:10:51+08:00
draft: false
toc: false
images:
tags:
  - uuid
  - javascript
  - web
---

偶然在掘金看到了一个在 web 环境中生成 uuid 的方法，只需要一行代码。来自[这篇文章](https://juejin.cn/post/7039960318897815565)的评论区。

```js
URL.createObjectURL(new Blob()).substr(-36);
```

## 解析
下面稍微解析一下这行代码。

`URL.createObjectURL()` 接收一个类型是 `File`, `Blob`, 或者 `MediaSource` 的对象作为参数，然后返回一个 `DOMString`。这个 `DOMString` 指向作为参数传入的对象。然后就可以通过这个 `DOMString` 访问到对应的对象。

即使传入的是同一个对象，每次调用 `URL.createObjectURL()` 都会返回一个新的 `DOMString` 。留意返回的 `DOMString`，会发现最后的 36 个字符是一串唯一的字符串，用来区分每个 `DOMString` 。例如：

```js
'blob:https://www.google.com/82482eae-c186-4246-b02b-40d43d9f4d90'
```

所以截取最后的 36 个字符后就能获得一个 uuid。对于一般的 web 应用来说，这个方法比引入一个 UUID 或 NanoId 库更简单。

另外，在 MDN 的文档里也提到了，虽然浏览器会在页面 unloaded 的时候自动释放由 `URL.createObjectURL()` 创建的 `object URL`，但如果你对性能和内存使用有要求的话，最好还是手动调用一下 `URL.revokeObjectURL()` 来释放 `object URL`。


## 参考
- [createObjectURL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL)
- [DOMString](https://developer.mozilla.org/en-US/docs/Web/API/DOMString)
