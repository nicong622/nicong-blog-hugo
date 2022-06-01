---
title: React开发环境配置
date: 2016-06-08 16:15:44
tags:
---

自从2015年开始接触ReactJS以后，就爱上了React。不过这篇文章不是要介绍React，而是介绍React开发环境的配置。

最初用React的时候，是用[browserify](http://browserify.org/)来打包，后来听说[webpack](https://webpack.github.io/)之后，就果断转向webpack。以下是我配置react开发环境的过程。

## 目录结构

我的目录结构如下。

![目录结构](/images/react01.jpg)

## Babel

babel是一个优秀JavaScript编译工具。能够把多种JavaScript衍生语言（coffeeScript，TypeScript等）和下一代的JavaScript（ES6）编译成能够直接运行的原生js。在React的项目中，我们一般需要以下几个babel presets。

```
$ npm install --save-dev babel-core babel-preset-es2015 babel-preset-react babel-preset-react-hmre babel-preset-stage-0
```

有了所需要的包以后，还要有相应的配置，以启用上面安装的包。babel的配置可以直接写在`package.json`的`presets`参数中，但我比较喜欢用`.babelrc`配置文件。

在项目的根目录下新建一个`.babelrc`文件，然后添加以下配置。

```javascript
{
  "presets": ["es2015", "stage-0", "react"]
}
```

在开发的过程中，我们需要用到react的模块热加载功能来提高开发效率，也就是要启用babel的`react-hmre`这个preset。但如果在开发完成，进行部署的时候启用了这个preset，就会报错。于是我们就要根据当前所处的环境（开发或者部署）来觉得是否启用这个preset。babel中的`env`参数可以帮我们完成以上工作。

```JavaScript
{
  "presets": ["es2015", "stage-0", "react"],
  "env": {
    "development": {
      "presets": ["react-hmre"]
    }
  }
}
```

## webpack

webpack是一个很好用的工具，但不得不承认，它的配置是一件很痛苦的事情。但是有了[hjs-webpack](https://github.com/HenrikJoreteg/hjs-webpack)这个工具之后，一切都变得简单很多了。

`hjs-webpack`是一个配置webpack的工具。它对外提供了很多简单的接口，这些接口调用的结果是生成一份对应的webpack配置文件。最强大的地方是，`hjs-webpack`已经帮我们设置好了很多loader的配置，当我们需要用到的时候，只需要`npm install`一下对应的包就可以了，免去自己再写loader的相关配置这一步骤。具体的用法下面会介绍到。`hjs-webpack`所支持的loader可以在[这里](https://github.com/HenrikJoreteg/hjs-webpack)查看。

接下来，在项目的根目录下新建一个`webpack.config.js`来开始webpack的配置。

在`hjs-webpack`中，提供了两个参数：`in`和`out`分别对应webpack中的`entry`和`output`。因此，最简单的配置就能像下面这样。

```javascript
var getConfig = require('hjs-webpack');
var webpack = require('webpack');
var path = require('path');

var config = getConfig({
  in: path.join(__dirname, 'src/app.js'),
  out: path.join(__dirname, 'dist'),
  clearBeforeBuild: true
})

```

其中`clearBeforeBuild`参数可以在构建新文件之前把旧文件都清除掉。

有时候，有些开发者喜欢把常用的路径都存储在一些变量中，方便使用。这里就不多说了。

在上面我们提到，在development和production两个环境中会分别应用不同的配置。在这里，我们有一个参数`isDev`来告诉webpack我们是否处于development环境中。另外，我们还需要根据`NODE_ENV`这个参数来判断我们所处的环境。

```javascript
var NODE_ENV = process.env.NODE_ENV;
var isDev = NODE_ENV === 'development';
var config = getConfig({
  isDev: isDev,
  in: path.join(__dirname, 'src/app.js'),
  out: path.join(__dirname, 'dist'),
  clearBeforeBuild: true
})
```

最后，我们还需要export一下这个配置。

```javascript
var config = getConfig({
  // ...
})

module.exports = config;
```

## React

在`src`文件夹下新建一个`app.js`文件，然后写一个React组件。这部分不是这篇文章的重点，就不多说了。

## 构建

以前，我们会使用webpack提供的`webpack-dev-server`来开启一个本地服务器。在`hjs-webpack`中，提供了一个`hjs-dev-server`命令。其实最终结果跟`webpack-dev-server`差不多，只是过程简单了很多。

在命令行中输入命令`hjs-dev-server`就能开始构建过程了。也可以把这个命令写在`package.json`中。

```javascript
"scripts": {
  "start": "NODE_ENV=development hjs-dev-server",
  "build": "webpack"
}
```

运行命令`npm start`。

如果没有报错的话，在浏览器中访问`localhost:3000`就能看到我们的应用了。当然，我们可以把端口改成自己喜欢的。详细配置看[这里](https://github.com/HenrikJoreteg/hjs-webpack)。

## CSS

在webpack中，如果要应用CSS，就要用loader。而我们刚刚也提到，在`hjs-webpack`中已经配置了`css-loader`了，我们只需要`npm install`一下，然后在`app.js`中加载我们的css文件就可以了。

```
npm install --save-dev css-loader style-loader
```

然后在`app.js`中导入css文件。

```javascript
import React, { Component } from 'react';
import ReactDOM from 'react-dom';

import './style/app.css'

class App extends Component {
  	render() {
    	return (
				<h1>Hello, world.....</h1>
    	);
  	}
}

ReactDOM.render(<App />, document.getElementById('root'));
```

这样我们就能应用样式了！

在`hjs-webpack`中，包含了对`autoprefixer`这样的`postcss`插件的支持，但如果你想要使用其他的插件，如`cssnano`，就要自己去扩展或者说修改webpack配置。在上面的配置中，`hjs-webpack`会返回一个webpack的配置对象：`config`。我们就可以在这个对象的基础上进行修改配置。

例如我们需要使用`cssnano`和`precss`这两个postcss的插件，可以这么配置：

```javascript
var config = getConfig({
  // ...
})

config.postcss = [].concat([
  require('precss')({}),
  require('cssnano')({})
])
```

到此，我们的开发环境的配置已经基本完成了。如果想了解更多用法，可以参考[这里](https://www.fullstackreact.com/articles/react-tutorial-cloning-yelp/)。

## 总结

本文主要是介绍了React + webpack的开发环境的配置方法，过程中使用了`hjs-webpack`这个工具来简化了webpack的配置。

## 参考

[React Tutorial: Cloning Yelp](https://www.fullstackreact.com/articles/react-tutorial-cloning-yelp/)

[hjs-webpack](https://github.com/HenrikJoreteg/hjs-webpack)

