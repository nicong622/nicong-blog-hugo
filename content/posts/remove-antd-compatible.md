---
title: "移除项目中的 @ant-design/compatible 依赖"
date: 2022-06-21T07:33:46
draft: false
tags: 
  - ast
  - jscodeshift
---

Ant Design 第4版发布之后不久，我就按照 antd 官方的指引利用迁移工具帮项目做了升级。由于迁移工具实际上是通过引入 `@ant-design/compatible` 来使那些不兼容的组件能继续运行，所以这样升级完之后实际上项目里就会同时存在 v3 和 v4 两个版本的 antd，不仅代码体积有冗余，还会有两套 Form 组件的写法，不利用后期维护。于是最近花时间移除了对 `@ant-design/compatible` 的依赖，实现了“完全”的升级。

这篇文章主要讲述我如何使用 AST 转换工具来帮助我完成升级。

## 修改范围与整体思路

需要做的迁移工作可以参考 [antd 官方的说明](https://ant.design/components/form/v3-cn/)。

对于我的项目，由于使用到 `@ant-design/compatible` 的基本上只有 v3 版本的 Form 组件， 所以要修改的范围主要有这些：

1. 移除 `Form.create({})(Component)` 写法；
2. 把 `getFieldDecorator` 的写法改成用 `<Form.Item>`；
3. 移除 `@ant-design/compatible` 相关的 `import` 语句；
4. 修改表单实体方法的调用方式。

第 1 点基本上可以通过编辑器的字符串替换功能加上正则表达式完成。

第 2、3 点可以我用一个 AST 转换工具来辅助修改。这是本文的重点，后面会展开说。

第 4 点理论上也可以用刚刚提到的工具来修改，但是项目里调用的方法不统一，例如有 `this.props.form.setFieldsValue` 也有 `const { setFieldsValue } from this.props.form` 等等，工具里需要的判断比较多。而且项目里使用到的实体方法不算很多，修改起来工作量比 `getFieldDecorator` 少很多。所以我打算手动修改，刚好可以边自测边修改。

确认范围之后就可以开干了。

## 代码修改工具

下面主要说说上面提到的 “代码修改工具”。

### 准备知识

- babel 插件开发：可以看看 [babel-handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md)；
- AST 相关：上面的 babel-handbook 也有说明。但是感觉 [jscodeshift](https://github.com/facebook/jscodeshift#core-concepts) 这里说得简单直白一点；

### AST & babel

提到 “代码修改”，我第一时间想到的就是利用 AST。把原始代码解析成 AST，然后修改与 `getFieldDecorator` 调用相关的节点，最后生成代码。一说这个流程，估计大多数人都会首先想到 babel。写一个 babel 插件，然后对代码做一次编译，就能完成 `getFieldDecorator` 的迁移。

关于 AST 节点操作的思路，下面会展开说。

开发之前有一点要注意，为了能解析 jsx 语法，通常我们会直接使用 `@babel/preset-react`，但是这样编译完之后得到的是 jsx 的编译产物，即 `React.createElement()` 这样的代码。而实际上我们想要的依然是 jsx 代码，仅仅是把 `getFieldDecorator` 去掉！这时候我们可以使用 [`@babel/plugin-syntax-jsx`](https://babeljs.io/docs/en/babel-plugin-syntax-jsx) 这个插件。这其实是包含在 `@babel/preset-react` 中的一个插件，它的作用仅仅是使得 babel 可以解析 jsx 语法，最后生成的代码依然是 jsx 代码。

代码不放了，具体开发思路可以参考[这里](https://github.com/nicong622/antd-upgrade-tool/blob/babel/plugins/index.js)。

下面是实际效果，左边是原始代码，右边是编译后的代码：

![Xnip2022-06-20_15-21-45](https://i.imgur.com/EdgtTd3.jpg)

看起来没什么问题。但是如果你细心的话就会发现注释的位置变了！第9行用来描述 `onChange` 函数的注释现在跑到上面的 `const` 语句后面了，这是我不能容忍的！由于我们的代码里没有动过注释相关的节点，所以这应该就是 babel 在生成代码时候的问题。

不过回想一下，为什么 antd 官方提供的代码迁移工具就没出现这样的问题？原来 antd 的迁移工具 [@ant-design/codemod-v4](https://github.com/ant-design/codemod-v4) 是基于另一个叫做 [jscodeshift](https://github.com/facebook/jscodeshift) 的工具开发的。`jscodeshift` 的其中一个功能就是在做 AST 转换的时候尽可能保持原来的代码样式！这似乎就能解决上面遇到的注释移位的问题了。

###  jscodeshift

虽然换了一个开发的工具，但本质上思路是一样的，都是对 AST 节点的增删改，只不过使用的 API 或者说是开发风格不一样了。jscodeshift 可以说就是用来操作 AST 节点的‘jQuery’，因为它提供了一系列的链式 API。例如：

```js
/**
 * This replaces every occurrence of variable "foo".
 */
module.exports = function(fileInfo, api, options) {
  return api.jscodeshift(fileInfo.source)
    .findVariableDeclarators('foo')
    .renameTo('bar')
    .toSource();
}
```

由于已经用 babel 实现了想要的功能，理论上只要把相关的逻辑用 jscodeshift 的 API 再实现一次就可以了。具体代码实现参考[这里](https://github.com/nicong622/antd-upgrade-tool/blob/jscodeshift/transform.js)。

## AST 转换的流程

上面只说了两个可以实现 AST 转换的工具，现在我们来讲一下 AST 转换的基本思路，以 `getFieldDecorator` 的迁移为例。首先明确一下我们最终的目标是：

1. 把 `getFieldDecorator()` 的参数转换成 `<Form.Item>` 元素上的属性；
2. 把 `<Cascader onChange={onChange} placeholder='Please select' />` 这个组件改成直接用 `<Form.Item>` 元素包裹；
3. 最后移除 `getFieldDecorator` 的引入。

以这段代码为例：

```jsx
export default function LinkType(props) {
  const { getFieldDecorator } = props;

  return (
    <>
      <Form.Item label='跳转页类型'>
        {getFieldDecorator('linkType', {
          rules: [{ required: true }],
          initialValue: '1',
        })(<Cascader onChange={onChange} placeholder='Please select' />)}
      </Form.Item>
    </>
  );
}
```

### 找到需要修改的节点

然后我们要找到需要修改的 AST 节点。

我们可以把原始代码复制到 [AST Explorer](https://astexplorer.net/) ，然后看看解析出来的 AST 是什么结构。在开启 `autofocus` 功能之后，就可以通过点击原始代码中的关键字，自动定位到 AST 中对应的节点。

对于示例代码中，第 7 到第 10 行对应的 AST 节点是：

![image-20220621112848136](https://i.imgur.com/7p6Nigy.png)

由于这几行代码外面有一对大括号包裹，所以是一个 JSX 表达式，对应的 AST 节点类型就是 `JSXExpressionContainer` 。`JSXExpressionContainer` 的 `expression` 属性是一个函数调用表达式 `CallExpression`，即原始代码中的这一块

```js
getFieldDecorator('linkType', {
	rules: [{ required: true }],
	initialValue: '1',
})(<Cascader onChange={onChange} placeholder='Please select' />)
```

可以看出来这里其实是两次函数调用，调用 `getFieldDecorator()` 之后会返回另一个函数，返回的函数用来修饰传入的组件。所以，这一个 `CallExpression` 的 `arguments` 就是我们传进去的 `JSXElement` 组件

```jsx
<Cascader onChange={onChange} placeholder='Please select' />
```

而 `callee` 就是原始代码中的：

```js
getFieldDecorator('linkType', {
	rules: [{ required: true }],
	initialValue: '1',
})
```

这一块代码虽然是另一个函数调用的 `callee` ，但它本身也是一次函数调用，所以它也是一个 `CallExpression` 类型的节点。这个节点的 `callee` 就是我们要找到 `getFieldDecorator` 关键词了。再看看这个`CallExpression` 的参数，即 `arguments`，里面有两个元素，第一个是 `StringLiteral` 类型，值是 `linkType` ；第二个是 `ObjectExpression` 类型，对应的是 `{ rules: [{ required: true }], initialValue: '1' }` 。

用同样的方式，可以找到引入 `getFieldDecorator` 的语句对应的 AST 节点。

节点都找到了，接下来就是做转换。

### 节点转换

节点的转换说简单点其实就是对节点进行增删改的操作。所以对于某个节点，我们首先要知道需要转换成什么样的节点。

例如我们需要把上面例子中 `getFieldDecorator()`  的第一个参数 `linkType` 转成 `<Form.Item>` 元素中的一个属性，属性名是 `name` ，即 `<Form.Item name='linkType'>`。所以可以先观察一下 `<Form.Item name='linkType'>` 这一行代码的 AST 结构。如下：

![image-20220621143743887](https://i.imgur.com/0X73n4U.png)

可以看到，某个元素的属性是放在一个类型是 `JSXOpeningElement` 的节点的 `attributes` 属性中，且 `attributes` 属性是一个数组。数组中的元素是类型为 `JSXAttribute` 的节点，代表元素中的一个属性。 `JSXAttribute`  节点中的 `name` 和 `value` 分别对应这个属性的 key 和 value，并且分别是不同类型的节点。

接下来我们要做的就是为 `name='linkType'` 这个属性创建一个 `JSXAttribute` 节点，然后添加到 `attributes` 数组中。

虽然 AST 的节点看起来就是一个个的对象，但是创建的时候需要用到 AST 中的 Builder 这个概念，你可以简单理解成就是 AST 节点的构造函数。对于每一种 AST 节点，都会有一个与之对应的 Builder，一般是与节点的 type 同名，但是以小写字母开头。例如，`JSXAttribute` 节点的 builder 就是 `jsxAttribute` 。在 [jscodeshift](https://github.com/facebook/jscodeshift#local-documentation-server) 的文档可以找到用法。顺便吐槽一下，它竟然没有在线的文档，需要自己在本地启动一个服务或者 fork 一下，然后配置一个 github pages 。

调用 `JSXAttribute()` 创建节点的时候需要注意，它的参数也是 AST 节点，所以要先调用对应的 builder ，然后把返回值传给  `JSXAttribute()` 。即：

```js
// j 就是 jscodeshift 的一个引用
j.jsxAttribute(
  j.jsxIdentifier('name'), 
  'linkType' // 由于 'linkType' 对应的节点是类型是 StringLiteral，所以可以直接把字符串 'linkType' 作为参数。如果是其他类型的节点，需要用对应的 builder 创建一个节点
)
```

以此类推，很容易就能完成其他节点的转换。

## 结尾

至此，代码修改工具的核心逻辑就完成了， 后面的工作就是手动修改一些特殊情况以及测试了。理论上用 AST 转换的方式能处理绝大部分的代码迁移工作，主要就是看是“自己手动改”还是“用工具来辅助”比较省事。