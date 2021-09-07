---
title: "nuxt-link预加载的原理"
date: 2021-03-02
draft: false
tags:
  - Vue
  - Nuxt
---

`nuxt-link` 是 [nuxtjs](https://nuxtjs.org/) 提供的一个组件，基于 [vue-router](https://router.vuejs.org/) 的 `router-link` 做了一层封装，实现了[预加载](https://nuxtjs.org/blog/introducing-smart-prefetching#introducing-smart-prefetching-%EF%B8%8F)的功能。本文来简单聊聊这个“预加载”的原理。

## 一点准备工作

如果直接打开 `nuxt` 项目中 [nuxt-link的源码](https://github.com/nuxt/nuxt.js/blob/dev/packages/vue-app/template/components/nuxt-link.client.js)，会发现里面使用了很多类似于 `<%...%>` 这样的模板语法。这其实是 [lodash.template](https://lodash.com/docs/4.17.15#template) 提供的模板语法。如果觉得影响阅读，可以按照 `nuxt` 的文档，新建一个 nuxt 应用，然后构建一次，再找到 `.nuxt/components/nuxt-link.client.js`，这个就是编译过后的 `nuxt-link` 代码。

## 前置条件

要想触发 `nuxt-link` 的预加载，需要同时满足以下几个条件：

1. `nuxt-link` 的 `prefetch` 属性值是 `true` 。

   默认值是 nuxt 配置里的 `router.prefetchLinks`。`prefetchLinks` 的默认值是 `true` ，所以 `prefetch` 的默认值也是 `true`。

2. `nuxt-link` 的 `noPrefetch` 属性值是 `false`。默认值是 `false`。

3. 当 `nuxt-link` 组件出现在可见区域内。

4. `nuxt-link` 的 `to`  属性指向的页面是同一个应用的页面。

5. 有网络链接并且不是 `2g` 网络。

1、2 两点控制是否启动预加载功能（默认是开启预加载），3、4、5是决定什么时候执行预加载的逻辑。

## 原理

接下来梳理一下预加载的大致流程。为了说明原理，`nuxt-link` 源码里有些条件分支会省略，但不影响理解。下面会以 `/cart` 这个链接作为 `nuxt-link` 的跳转地址来做说明。`/cart` 页面对应的文件路径是 `/pages/cart/index.vue`。

```html
<nuxt-link to="/cart">购物车</nuxt-link>
```

### 事件监听

在 `nuxt-link` 的 `mounted` 阶段，如果设置了开启预加载，即 `this.prefetch && !this.noPrefetch`，就会开始监听 `nuxt-link` 元素是否进入了可视区域。

当元素进入了可视区域，就会调用 `prefetchLink` 方法进行预加载。

监听过程中使用了 [window.requestIdleCallback](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback) 和 [window.IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) 这两个 API。

### 预加载

核心的逻辑在 `prefetchLink()` 函数里：

```js
prefetchLink () {
  if (!this.canPrefetch()) {
    return
  }
  // Stop observing this link (in case of internet connection changes)
  observer.unobserve(this.$el)
  const Components = this.getPrefetchComponents()

  for (const Component of Components) {
    const componentOrPromise = Component()
    if (componentOrPromise instanceof Promise) {
      componentOrPromise.catch(() => {})
    }
    Component.__prefetched = true
  }
}
```

首先要获取需要预加载的组件。

在 `getPrefetchComponents()` 方法里，使用了 `vue-router` 的 `router.resolve` 方法，找到将要跳转的页面对应的页面入口，即 `Components`。

```js
getPrefetchComponents () {
  const ref = this.$router.resolve(this.to, this.$route, this.append)
  const Components = ref.resolved.matched.map(r => r.components.default)

  return Components.filter(Component => typeof Component === 'function' && !Component.options && !Component.__prefetched)
},
```

在浏览器开发工具里打印一下 `Components`，看到以下信息：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecc70c999a4f45e98e4f5b30e4dca864~tplv-k3u1fbpfcp-watermark.image)

可以看到 `Components` 里的每个元素都是一个函数，点击 `[[FunctionLocation]]` 的值，`router.js`，可以跳到函数定义的位置。然后你就会发现打开的这个 `router.js` 其实就是我们的项目构建完之后得到的 `.nuxt/router.js` 。

通过 `_30f4c240()`  这个函数的定义可以发现它的核心功能其实就是引入了 `/cart` 对应的页面文件。如果调用这个方法，就会在浏览器开发者工具的 network 面板看到浏览器请求了 `/_nuxt/pages/cart/index.js` 这个文件，即加载了 `/cart`  这个页面对应的代码。具体的加载过程属于 `webpack` 的功能，这里不做说明。

```js
// .nuxt/router.js
const _30f4c240 = () => interopDefault(import('../src/pages/cart/index.vue' /* webpackChunkName: "pages/cart/index" */))
```

在获取到上面的加载函数之后，`prefetchLink` 就会调用这些函数，达到预加载的效果。之后，还会设置一个 `__prefetch` 的标识，防止重复加载。

以上就是 `nuxt-link` 预加载的大致过程。

## 延伸

在 `nuxt-link` 的源码里，还能找到一些有趣的点：

### 自定义预加载

有些时候我们没有使用 `nuxt-link` 来做页面跳转，但又想预加载这个页面，就可以这样做：

```js
myPrefetch(path) {
  const ref = this.$router.resolve(path, this.$route, false);
  const Components = ref.resolved.matched.map((r) => r.components.default);

  Components
    .filter((Component) => typeof Component === 'function' && !Component.options && !Component.__prefetched)
    .forEach((Component) => {
      const componentOrPromise = Component();
      if (componentOrPromise instanceof Promise) {
        componentOrPromise.catch(() => {});
      }
      Component.__prefetched = true;
    });
},
```

可以使用以下方法简化一下 `Components` 的获取：

```js
const Components = this.$router.getMatchedComponents(path)
```

### navigator.connection

可以使用 [navigator.connection](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/connection) 来判断网络状态，但是要注意兼容性。

### quicklink

在 `react` 项目里，可以使用 [quicklink](https://github.com/GoogleChromeLabs/quicklink) 实现预加载的功能。
