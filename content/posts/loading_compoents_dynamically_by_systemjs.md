---
title: 一个基于 Systemjs 的组件动态加载方案
date: 2022-01-09
draft: false
tags:
  - systemjs
---

## 背景

在低代码平台的设计中，通常需要动态地加载一个组件。

Bigo 的活动配置平台 raptor 使用 [requirejs](https://requirejs.org/) 来实现动态加载。requirejs 要求加载的是 [AMD](https://requirejs.org/docs/whyamd.html#amd) 格式的包。但是很多优秀的第三方组件都没有打包成 AMD 格式，而是只有Commonjs 和 ESM 版本，这就意味着不能直接在平台里使用这些组件。

为了解决上述问题，可以使用 [systemjs](https://github.com/systemjs/systemjs)。利用 systemjs ，可以通过动态加载的方式加载一个远程的 ESM 模块，并且运行在不支持 ESM 格式的浏览器中。

## Systemjs

systemjs 实现了一个叫做 [System.register](https://github.com/systemjs/systemjs/blob/main/docs/system-register.md) 的模块格式，用于在浏览器中实现动态 `import()`、`import.meta` (包括 `import.meta.url` and `import.meta.resolve`) 等功能。更多的功能请参考 [systemjs 的文档](https://github.com/systemjs/systemjs)。

例如以下例子：

<iframe height="300" style="width: 100%;" scrolling="no" title="Untitled" src="https://codepen.io/Nicong622/embed/oNGMByL?default-tab=js%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/Nicong622/pen/oNGMByL">
  Untitled</a> by Nicong (<a href="https://codepen.io/Nicong622">@Nicong622</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

### 使用

使用的时候，需要提前添加以下 script 标签：

```html
<script src="https://cdn.jsdelivr.net/npm/systemjs/dist/system.min.js"></script>
<script type="systemjs-importmap">
// 定义 importmap
{
  "imports": {
    "lodash": "https://unpkg.com/lodash@4.17.10/lodash.js"
    // 或者其他需要动态加载的模块
  }
}
</script>
```

接着，就可以使用以下两种方法来加载并使用一个在上面的 importmap 中配置过的模块。

-  window.System.import

```js
// window.System.import 返回的是一个 Promise 对象
window.System.import('lodash')
	.then(lodash => lodash.partition([1, 2, 3, 4], (n) => n % 2))
```

- `<script />` 标签

```html
<!-- 下面的标签会加载模块并挂载到 window 对象上 -->
<script type="systemjs-module" src="import:lodash"></script>
<script>
	window.lodash.partition([1, 2, 3, 4], (n) => n % 2)
</script>
```

## 实战

下面的例子中，我们通过点击按钮来动态加载一个组件 [qrcode.react](https://github.com/zpao/qrcode.react) 。完整代码如下，后面会有相关解析。

```typescript
import { useState } from "react";

const config = {
  imports: {
    classnames: "https://ga.system.jspm.io/npm:classnames@2.3.1/index.js",
    "qrcode.react":
      "https://ga.system.jspm.io/npm:qrcode.react@1.0.1/lib/index.js"
  },
  scopes: {
    "https://ga.system.jspm.io/": {
      "object-assign":
        "https://ga.system.jspm.io/npm:object-assign@4.1.1/index.js",
      "prop-types": "https://ga.system.jspm.io/npm:prop-types@15.8.0/index.js",
      "qr.js/lib/ErrorCorrectLevel":
        "https://ga.system.jspm.io/npm:qr.js@0.0.0/lib/ErrorCorrectLevel.js",
      "qr.js/lib/QRCode":
        "https://ga.system.jspm.io/npm:qr.js@0.0.0/lib/QRCode.js",
      react: "https://ga.system.jspm.io/npm:react@17.0.2/index.js"
    }
  }
};

export default function App() {
  const [El, setEl] = useState();

  async function loadQrcode() {
    importMap();
    const module = await window.System.import("qrcode.react");

    setEl(() => module.default);
  }

  function importMap() {
    const el = document.createElement("script");
    el.type = "systemjs-importmap";
    el.innerText = JSON.stringify(config);
    document.body.appendChild(el);
  }

  return (
    <div className="App">
      <div>
        <button onClick={loadQrcode}>添加二维码</button>
      </div>

      {El && <El value="https://www.baidu.com" />}
    </div>
  );
}

```


在点击「添加二维码」按钮时，会首先调用 `importMap()`，作用是在 html 里添加一个 script 标签，里面包含了一份 importMap 配置。

接着在调用 `window.System.import("qrcode.react")` 的时候会从刚刚添加的 importMap 里找到 `qrcode.react` 这个模块对应的 url，以及它的依赖的 url，然后下载这些模块。

之后就可以使用 `qrcode.react` 这个组件了。

示例中 importMap 中模块的 url 可以通过 [jspm](https://jspm.org/) 获取。

## 未解决的问题

- 如果动态引入的组件里使用了 react-hook，在使用这个组件的时候会报以下错误：

```js
Unhandled Runtime Error
Error: Invalid hook call. Hooks can only be called inside of the body of a function component. This could happen for one of the following reasons:
1. You might have mismatching versions of React and the renderer (such as React DOM)
2. You might be breaking the Rules of Hooks
3. You might have more than one copy of React in the same app
See https://reactjs.org/link/invalid-hook-call for tips about how to debug and fix this problem.
```
