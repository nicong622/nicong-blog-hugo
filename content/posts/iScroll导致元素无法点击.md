---
title: iScroll导致元素无法点击
date: 2015-12-16 19:15:03
tags:
---
最近接手了一个游戏，在接入某一个渠道的时候，需要加一个分享的功能。

这个需求很简单，一个`<textarea>`，一个确定按钮，一个取消按钮，加一点点逻辑，理论上就可以了。

但是却遇到了一个问题：点击`<textarea>`的时候无法聚焦到它上面，就是说无法在`<textarea>`里面输入内容，百思不解。

<!-- more -->

于是Google了一下，在其他人的一些文章中发现了问题的所在。

我的游戏中为了在移动端获得比较好的滚动体验，使用了一个工具：[iScroll](http://iscrolljs.com/)。这个工具为了要达到模拟滚动的最佳效果，默认禁用了所有元素（除了`<a>`）的默认事件，所以造成了`<textarea>`等的一些元素无法点击，也就无法聚焦。

解决办法也有一些，有的是直接修改源码。但为了防止其他bug的出现，我选择了用js来对特定元素进行聚焦，代码如下：
```javascript
_getFocus(){
        document.getElementsByClassName('inputArea')[0].focus();
},
render(){
    return (
        ...
        <textarea className="inputArea" ref="share" placeholder={this.state.defaultMsg} >
        ...
    )
}
```

这里用了jsx的语法，但大意还是很清楚，就是点击了这个`<textarea>`的时候，就focus到这个元素上，很简单。