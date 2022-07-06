---
title: "【How Vue Diff】之简单 Diff"
date: 2022-07-06T14:57:34+08:00
draft: false
toc: true
images:
tags: 
  - Vue
  - diff
---

简单分析一下 Vuejs 中的“简单 Diff”算法，[参考代码](https://github.com/HcySunYang/code-for-vue-3-book/blob/master/course5-%E6%B8%B2%E6%9F%93%E5%99%A8/code-9.6.html)，主要关注 `patchChildren()` 函数的实现过程。

首先假设新旧 vnode 的子节点都是一组节点。因为如果其中一个 vnode 的子节点不是数组的话，Diff 的意义就不大了。

## 算法的整体流程

Diff 算法的主要目标是通过新旧两组节点的对比，找到可以复用的节点，然后移动节点到新的位置。所以，算法整体的流程总结一下就是：

1. 找到可以复用的元素（即分别在新旧 children 中 key 相同的两个元素）；
2. 对这两个节点进行 `patch` 操作；
3. 移动元素到合适的位置；
4. 处理新增或者不存在的元素；

## 找到可复用的元素

通过参考代码可以看到，算法的主要逻辑就是循环里嵌套循环：遍历新的子节点数组 `newChildren`，对于里面的每一个子节点 `newVNode`，都尝试在 `oldChildren` 中找到一个 `oldVNode`，使得 `newVNode.key === oldVNode.key` 即**元素可复用**。然后做一下更新，即 `patch()` 。

主体代码如下：

```js
// 遍历新的 children
for (let i = 0; i < newChildren.length; i++) {
  const newVNode = newChildren[i]
  let j = 0
  // 遍历旧的 children
  for (j; j < oldChildren.length; j++) {
    const oldVNode = oldChildren[j]
    // 如果找到了具有相同 key 值的两个节点，则调用 `patch` 函数更新之
    if (newVNode.key === oldVNode.key) {
      patch(oldVNode, newVNode, container)
      break // 这里需要 break
    }
  }
}
```

## 如何找到需要移动的元素

接下来需要判断一个节点是否需要移动，这里可以用逆向思维，即什么时候节点不需要移动？答案是如果在新旧两组子节点中，如果节点的相对顺序没有改变，就不需要移动。

例如，对于下面这种情况，就不需要移动元素：

```js
// 旧 children 
p0 // 索引：0
p1 // 索引：1
p2 // 索引：2

// 新 children 
p0 // 索引：0
p1 // 索引：1
p2 // 索引：2
```

节点的索引组成的序列在更新前后都是：0、1、2，是一个递增的序列。如果在更新后，序列不再是递增，就表示有元素需要移动。而打破这个递增关系的元素就是需要移动的元素。

例如下面这个情况，更新后的序列是：2、0、1，所以需要移动的元素是 p0 和 p1 的元素：
```js
// 旧 children 
p0 // 索引：0
p1 // 索引：1
p2 // 索引：2

// 新 children 
p2 // 索引：2
p0 // 索引：0
p1 // 索引：1
```

对于 p0 和 p1，它们的索引都比 p2 的索引小，说明在旧 children 中，p0 和 p1 都排在 p2 前面，但是在新 children 中，p0 和 p1 都排在 p2 后面，所以 p0 和 p1 需要移动。

可以发现，这里用 p2 在旧 children 中的索引作为后续比较的基准，实际上可以这样定义这个索引：**在旧 children 中寻找具有相同 key 值节点的过程中，遇到的最大索引值**。如果在后续寻找过程中，存在索引值比当前遇到的最大索引值小的节点，则意味着该节点需要移动。

反映到代码中就是：

```js
// 利用 lastIndex 存储当前遇到的最大索引值；
let lastIndex = 0
// 遍历新的 children
for (let i = 0; i < newChildren.length; i++) {
  const newVNode = newChildren[i]
  let j = 0
  // 遍历旧的 children
  for (j; j < oldChildren.length; j++) {
    const oldVNode = oldChildren[j]
    if (newVNode.key === oldVNode.key) {
      // ...
      if (j < lastIndex) {
        // 需要移动
        const prevVNode = newChildren[i - 1]
        if (prevVNode) {
          const anchor = prevVNode.el.nextSibling
          insert(newVNode.el, container, anchor)
        }
      } else {
        // 更新 lastIndex
        lastIndex = j
      }
      break // 这里需要 break
    }
  }
}
```

### 如何移动元素

移动元素时候主要关注两点：

1. **获取真实的 DOM 元素**：通过 `vnode.el` 来获取节点对应的真实 DOM 元素；
2. **插入的位置**：由于新 children 的顺序就是更新后真实 DOM 节点的顺序，所以插入的位置就在 `prevNode` 后面，所以用 `prevVNode.el.nextSibling` 作为插入的锚点；

## 处理新增或者不存在的元素

### 添加新元素

如何知道某个元素是否是新增？

如果新 children 中的某个元素在旧 children 中没有找到 key 值相同的节点，这个元素就是新增的元素。对应的代码如下：

```js
for (let i = 0; i < newChildren.length; i++) {
  const newVNode = newChildren[i]
  let j = 0
  let find = false
  // 遍历旧的 children
  for (j; j < oldChildren.length; j++) {
    const oldVNode = oldChildren[j]
    if (newVNode.key === oldVNode.key) {
      find = true
      // ...
      break // 这里需要 break
    }
  }
  if (!find) {
    const prevVNode = newChildren[i - 1]
    let anchor = null
    if (prevVNode) {
      anchor = prevVNode.el.nextSibling
    } else {
      anchor = container.firstChild
    }
    patch(null, newVNode, container, anchor)
  }
}
```

这里需要注意一下插入的锚点 `anchor`。

### 移除不存在的元素

```js
// 遍历旧的节点
for (let i = 0; i < oldChildren.length; i++) {
  const oldVNode = oldChildren[i]
  // 拿着旧 VNode 去新 children 中寻找相同的节点
  const has = newChildren.find(
    vnode => vnode.key === oldVNode.key
  )
  if (!has) {
    // 如果没有找到相同的节点，则移除
    unmount(oldVNode)
  }
}
```

## 总结

简单 Diff 算法的核心流程就是拿新的一组子节点中的每个节点去旧的一组子节点中寻找可复用的节点。如果能找到，还要判断该节点是否需要移动；如果没找到，表示是新节点，需要做添加操作。最后再遍历旧 children，移除在新 children 中不存在的元素。

## 参考

- 《Vue.js 设计与实现》第 9 章  简单 Diff 算法

