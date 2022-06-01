---
title: express+webpack for react
date: 2016-05-09 15:02:19
tags: express webpack react
---

最近在做毕业设计，用到了 reactJs， express，mongoDB，然后用 webpack 作为构建工具。每次开始工作的时候都要先开启三个命令行窗口或标签页，分别用来启动 mongoDB，express 和 webpack构建命令。但是觉得每次都要开三个窗口太麻烦了，于是就想着能不能把 express 和 webpack 的启动放在一起。

Google 了一下之后果然找到了办法。

## webpack middleware

这里主要用到了 webpack 的一个中间件：`webpack-dev-middleware`。

**提醒一下**，如果你原来是用 `webpack-dev-server` 的话，最好先 uninstall 一下，不然可能会出错。其实，`webpack-dev-server` 在内部也是用到了 `webpack-dev-middleware` ，我们这里只是把它独立出来使用。

可能用人会问，`webpack-dev-server` 不也是内置了 express 的功能吗？为什么不直接用？我开始的时候也是这样想，但后来发现，这个内置的「express」好像只具备「server」的功能，没有处理网络请求的功能，所以当我在前端就不能正确的访问数据库。（当然也有可能是我的配置不对）

另外，还用到了另一个中间件：`webpack-hot-middleware`。由于没有`webpack-dev-server` ，所以用这个中间件来提供「hot reload」的功能。

## webpack 配置文件

主要是 entry 的配置：

```javascript
entry: [
  'webpack-hot-middleware/client',
  'webpack/hot/dev-server'
  './src/index.js'
],
```

## app.js

```javascript
var express = require('express');
var app = express();
var webpack = require('webpack');
var WebpackDevMiddleware = require('webpack-dev-middleware');
var WebpackHotMiddleware = require('webpack-hot-middleware');
var config = require('./webpack.config');

var compiler = webpack(config);

app.use(WebpackDevMiddleware(compiler, {
  publicPath: config.output.publicPath,
  historyApiFallback: true,
  stats: {colors: true}
}));

app.use(WebpackHotMiddleware(compiler));

app.listen(3000)；
```



