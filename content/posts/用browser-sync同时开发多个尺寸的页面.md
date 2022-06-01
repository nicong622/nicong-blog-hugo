---
title: browser-sync + gulp + postcss配置文件
date: 2015-10-24 17:37:34
tags:
  - 前端工具
---

最近发现了一个工具[browser-sync](http://www.browsersync.io/)。使用这个工具可以同时在多个尺寸的浏览器中打开同一个页面，当在A浏览器中点击了一个按钮，在B浏览器中也会响应这个事件。另外，当你修改了本地的 HTML 文件或者 css 文件的时候，浏览器也会自动刷新网页，就像是liveEdit一样。非常cool的功能！
而且，它也可以结合[gulp](http://gulpjs.com/)或[grunt](http://gruntjs.com/)来使用，自动编译文件（例如.sacc）后实时显示效果。这样，你就可以同时打开几个浏览器窗口，分别显示不同的尺寸。

<!-- more -->

安装什么的就不啰嗦了，官网上有。使用也很简单，一行命令就可以了。

如果用自动化工具（gulp/grunt）来运行 browser-sync 的话，可以同时结合其他gulp插件来使用。例如自动编译scss然后同步在浏览器中显示效果。

由于最近在学习[postcss](https://github.com/postcss/postcss)，所以就想结合gulp、postcss、browser-sync来玩一下。以下是我的gulpfile：
```javascript
var gulp = require('gulp');
var browserSync = require('browser-sync').create();
var postcss = require('gulp-postcss');
var autoprefixer = require('autoprefixer');

gulp.task('browser-sync',['css'],function(){
	browserSync.init({
		proxy: 'localhost/postcss/index.html',
		files: "build/css/base.css"
	});

	gulp.watch('src/css/base.css',['css']);
});

gulp.task('css',function(){
	return gulp.src('./src/css/*.css')
		.pipe(
			postcss([
				require('precss')({})
			])
		)
		.pipe(gulp.dest('./build/css'));
});
```

修改了对应的css文件之后就会自动用postcss来处理，然后显示在浏览器中。现在这里mark一下，已备不时之需。