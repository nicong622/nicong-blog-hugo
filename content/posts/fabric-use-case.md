---
title: "基于Fabric.js实现一个可以在图片上创建多个点击热区的组件"
date: 2023-11-24T11:30:50+08:00
draft: false
toc: false
images:
tags:
  - js
---

## 需求描述

前段时间接到一个需求，需要在管理后台的页面装修模块里增加一个组件。组件的主体就是一张图片，通过在图片中任意位置拖动鼠标可以创建一个矩形选区，该选区就是一个可点击的区域，所以还需要配置点击该区域后的页面跳转路径。可以创建多个选区，但选区不能重叠，也不能超出图片的边界。这个组件在用户端渲染出来后，用户可以通过点击该组件的不同区域跳转到对应的页面。

## 需求分析
实现这个组件的思路比较简单。先创建一个 canvas，同时用自定义的图片填充这个 canvas，然后创建选区。创建完选区后需要保存到服务端的主要数据包括图片的链接、每个选区的尺寸、选区左上角的坐标、选区的跳转链接。之后在用户端再根据这些数据在图片上创建透明的点击区域即可。

这里的重点在于如何通过拖动鼠标的方式创建可点击区域。

其实这个交互在一般的画板应用中都会看到，就是创建矩形的功能。原理也比较简单，就是监听鼠标事件。在 `mouse:down` 的时候记录下当前鼠标位置的坐标，然后在 `mouse:up` 的时候再获取一次当前坐标。有了这两组坐标就可以创建一个矩形了。当然，为了用户体验，一般还需要通过监听 `mousemove` 事件在拖动鼠标的时候实时渲染一个过渡状态的选区，让用户感知到当前选区的实际位置与大小。

市面上有不少库实现了类似功能，我选用了 [Fabric.js](http://fabricjs.com/)。

## 最终结果

<iframe src="https://codesandbox.io/embed/tapareacreator-8zr3s6?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="TapAreaCreator"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## 需求实现

### 创建 canvas

既然是要在图片上创建点击区域，自然就要先创建一个与图片宽高“一样”的 canvas。但是由于运营实际上传的图片尺寸可能非常大，不可能在创建选区的时候直接按图片原尺寸创建 canvas，所以需要给定一个最大宽度，然后按图片的宽高比例进行缩放得到实际的 canvas 宽高。

一种实现方案是在页面里创建一个看不见的 `img` 元素并设置它的最大宽度和高度，加载完成后就可以通过查询这个 `img` 元素的宽高从而得到图片缩放后的实际宽高。也可以监听 `img` 元素的 `load` 事件，获取图片原始宽高然后根据设定的最大宽高计算得到缩放后的实际宽高。

```js
const BORDER_WIDTH = 3;

const bgImg = new fabric.Image(document.getElementById("bgImg"));
bgImg.scale(this.canvasWidth / this.imgWidth);

const canvas = new fabric.Canvas("canvas", {
  backgroundImage: bgImg,
  selectionColor: "rgba(255, 255, 255, 0)", // 鼠标框选区域背景颜色
  selectionLineWidth: BORDER_WIDTH, // 鼠标框选区域边框宽度
  selectionBorderColor: ACTIVE_COLOR, // 鼠标框选区域边框颜色
});
```

### 创建选区

创建选区主要是监听 canvas 对象上的 `mouse:down` 和 `mouse:up` 事件。

`mouse:down` 的时候记录下当前坐标：

```js
canvas.on("mouse:down", (options) => {
  // options.e.offsetX -> 鼠标相对于 canvas 元素左上角的 x 轴距离
  // options.e.offsetY -> 鼠标相对于 canvas 元素左上角的 y 轴距离
  this.currentSelection.startX = options.e.offsetX;
  this.currentSelection.startY = options.e.offsetY;
});
```

`mouse:up` 的时候再获取当前坐标，结合 `mouse:down` 时候的坐标得出矩形的宽高，然后调用 `fabric.Rect` 构造函数创建矩形：

```js
// 获取矩形结束点的坐标
function getEndPoint(options) {
  const { x, y } = document
    .getElementById("canvas")
    ?.getBoundingClientRect() ?? { x: 0, y: 0 };
  const { pageX, pageY } = options;

  let endX = Math.min(this.canvasWidth, pageX - x) - BORDER_WIDTH; // 防止超出右边界
  endX = Math.max(endX, 0); // 防止超出左边界

  let endY = Math.min(this.canvasHeight, pageY - y) - BORDER_WIDTH; // 防止超出下边界
  endY = Math.max(endY, 0); // 防止超出上边界

  return {
    endX,
    endY,
  };
}

canvas.on("mouse:up", (options) => {
  const { endX, endY } = getEndPoint(options.e)

  // 创建矩形
  const rect = new fabric.Rect({
    top: this.currentSelection.startX,
    left: this.currentSelection.startY,
    width: endX - this.currentSelection.startX,
    height: endY - this.currentSelection.startY,
    fill: "rgba(255, 255, 255, 0)",
    stroke: ACTIVE_COLOR, // 边框颜色
    strokeWidth: BORDER_WIDTH, // 边框大小
    transparentCorners: false,
    lockRotation: true, // 不允许旋转
  })
});
```

注意上面在获取矩选区结束点（矩形右下角）的时候，没有直接用 `offsetX` 和 `offsetY` 属性值，而是用 `pageX` 和 `pageY` 属性值减去 `canvas` 元素自身的 `x` 和 `y` 属性值。主要是为了处理拖动鼠标时候指针超出了 `canvas` 元素范围的边界情况，避免创建的选区超出 `canvas` 元素的可视范围。

另外需要注意选区的边框宽度对选区实际宽高的影响。我在创建 `rect` 的时候已经在 `width` 和 `height` 里减去了边框宽度，所以通过 `rect.width` 和 `rect.height` 获取到的矩形的宽高不会包含边框宽度。但是如果通过 `rect.getBoundingRect()` 获取的 `width` 和 `height` 是会包含边框宽度的。其实就跟 DOM 的 API 差不多。各位可以根据自己的需要选用 `rect.width`/`rect.height` 或者 `rect.getBoundingRect()`。 由于在后面计算两个选区是否有重叠以及选区是否有超出 `canvas` 可视范围的时候要把边框宽度也算在了选区的宽高中，所以我用 `rect.getBoundingRect()`。不过大家也可以不用边框来圈出选区，而是用背景颜色或者背景图，就不用考虑边框宽度对选区宽高的影响了。

### 获取当前选中的选区

如果想在点选了某个选区后获取对应选区的 `Rect` 实例，可以监听 `canvas` 对象的这几个事件：

- `selection:cleared`：在已经选中了一个选区的情况下点击 `canvas` 元素的空白位置时触发，即不选中任何选区；
- `selection:updated`：在已经选中了一个选区的情况下点击选中另一个选区时触发；
- `selection:created`：本来没有选中选区，在选中一个选区时候触发；

在事件的回调函数中就能获取到当前选中的 `Rect` 实例。

### 处理一些边界情况

#### 防止选区重叠

主要有三种情况可能会出现选区重叠：创建、移动、缩放选区。

这几种情况都可以在 `mouse:up` 事件的回调函数里处理。处理的思路就是获取所有 `Rect` 实例然后两两检测是否重叠，检测的方法可以参考 2D 图形的碰撞检测：

```js
function isTwoRectOverlap(rect1, rect2) {
  return !(
    rect1.left > rect2.left + rect2.width ||
    rect1.left + rect1.width < rect2.left ||
    rect1.top > rect2.top + rect2.height ||
    rect1.top + rect1.height < rect2.top
  );
}
```

#### 防止选区超出 canvas 可视范围

上面在创建选区的时候就已经检查过新创建的选区是否会超出 `canvas` 元素的可视范围。在创建选区之后也需要防止移动、缩放选区的时候超出 `canvas` 元素可视范围的情况。核心思路与上面提到的差不多，这里不再赘述。详情代码可以看文章开头的 codesandbox 链接。

### 总结

本文主要介绍了如何利用 Fabric.js 实现一个可以在图片上创建点击区域的组件，本质上都是对 canvas 的操作。整体实现思路不难，主要卡点在于处理边界情况和 Fabric.js 的文档不够完善，需要自己慢慢调试或者看源码。