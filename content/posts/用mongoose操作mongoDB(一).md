---
title: 用mongoose操作MongoDB(一)
date: 2016-01-06 09:58:14
tags:
  - mongoose
  - MongoDB
---



最近在做一个课程设计，想做一个单页应用。但我是学前端的，后端知识比较缺乏，所以有点苦恼后端要怎么实现。正好前阵子看到了一本书，《单页 web 应用 javascript 从前端到后端》(写得挺详细的一本书，推荐一下)，受到了启发，于是决定用 nodejs 做后端。

这篇文章主要是讲一下在数据库方面的使用。其实我自己对于数据库是不太懂，这次的设计算是我第一次真正意义上的使用数据库，而且还是我自己一步一步摸索（瞎搞）出来的，有不对的地方请多多指教。

数据库我是用 [MongoDB](https://www.mongodb.org/) 。因为 mongoDB 的存储格式是 JSON ，比较适合单页应用。在 MongoDb 的驱动方面，我选择了一个 ODM( Object Docunemt Mapper) ：[mongoose](http://mongoosejs.com/)。

**注意** 我是用 OSX 的，其他系统的操作可能会有些不同。



## 安装

### MongoDB 安装

在 osx 下，推荐用 homebrew 来安装：

```
brew install mongodb
```

如果 homebrew 安装失败的话，可以直接从官网下载对应的二进制文件。

其他平台的安装可以看[官方文档](https://docs.mongodb.org/manual/installation/)。

### mongoose 安装

mongoose 的安装也非常简单：

```
npm install --save mongoose
```

安装之前先确保你已经正确安装了 nodejs 。



## 启动数据库

### 默认路径下启动

启动 MongoDB 最简单的方法就是在命令行输入：

```
mongod
```

这时候，数据库就会在默认的路径 'data/db' 下启动数据库。

### 特定路径下启动

如果你想在自己定义的项目目录下启动的话可以这样：

```
//假设你在当前目录下有一个叫做 'test' 的目录

mongod --dbpath test
```

如果数据库正确启动了，test 目录里面就会多出一些文件。

### 特定端口启动

MongoDB 默认是启动在 27017 端口，如果想用其他端口的话可以这样：

```
mongod --port 8080
```

到这里，你的数据库应该就能成功启动了。如果不行，可能是因为授权的问题，可以参考官方文档，或者 Google 一下。



## 连接数据库

在项目目录下新建一个文件 app.js :

``` javascript
var mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/db/test');

var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function() {
  console.log('connect success');
});
```

然后，在保持数据库启动的状态下，打开一个新的命令窗口，输入命令：

```
node app.js
```

这个时候如果连接成功的话就会在命令窗口中显示 'connect success' 这样的字样，表示数据库已经连接成功了。现在我们可以对数据进行 CRUD 操作了。这部分我们下次再讲。