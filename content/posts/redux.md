---
title: Redux 初探
date: 2015-12-31 10:13:02
tags: redux react
---
最近发现了一个东西叫做 redux ，感觉很厉害的样子，于是就抽时间学习了一下。

<!--more-->

## 文档

- [中文文档](http://camsong.github.io/redux-in-chinese/index.html)

- [英文文档](http://rackt.org/redux/index.html)

  ​

## Redux 的三个原则

### 1. 只有一个store

​	在 redux 中，你只有一个 store ，这个 store 中有一棵 object tree ，它维护着整个引用的 state 。这样就保证了数据的单一来源。在 flux 中，由于你可以有多个 store ，导致你有时候分不清你的 state 是在哪里发生变化的，这样就导致了一些莫名其妙的 bug ，特别是对于初学者来说。

### 2. state 是只读的

​	在你的应用中，你只能通过 action 来改变 state 。无论是视图还是网络请求，都不能直接改变 state ，只能通过 dispatch 一个 action 来表达想要改变 state 的意图。其实这也是保证了单一的数据源。

### 3. 使用纯函数来修改数据

​	在 redux 中，我们通过 reducer 来接受 action 和旧的 state ，然后返回新的 state ，从而改变整个应用的 state tree。reducer 只是一些纯函数（不产生副作用，不发出网络请求）。你可以有多个 reducer ，并按照需要把 reducers 写在不同的文件中，最后通过 redux 的一个函数 combineReducers 把他们合并在一起。



## 一个非常简单的例子

​	我们来做一个非常简单的例子，有一个按钮，点击一下分成一个随机数，并显示在屏幕上。真的非常简单！但是为了学习 redux 我还是稍微做的复杂了一点。

首先安装一下 redux ，由于我们是和 react 一起用，所以还有装一个 react 绑定库。

``` javascript
npm install --save redux react-redux
```

然后添加一个 action：

``` javascript
//actions.js

export const CHANGE = 'change';


/*
 这里我们返回了一个事件(action)，每次我们想要改变 state tree
 的时候就调用这个函数。在这里，我们只返回一个 action ，不做其他
 的事情，不产生任何的副作用。这样的函数我们叫做 action creater
*/
export function newRandom(){
	return {
		type: CHANGE
	}
}
```

接着我们有一个 reducer：

``` javascript
//reduder.js

import { CHANGE } from '../actions/actions'

/*
 reducer 从 store 中接收两个参数：state 和 action ，你也可以为 state 加上默认值。
 根据 aciton.type 来改变应用的 state tree (返回新的 state)。
 建议默认返回原来的 state ，避免出现问题。
*/
export default function reducer(state = 'hello', action){
	switch (action.type) {
		case CHANGE:
			return {
				text: Math.random()
			};
			break;
		default:
			return state;
			break;
	}
}
```

接下来是我们的顶层组件：

``` javascript
// app.js

import React from 'react';
import { newRandom } from './actions/actions';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';

var App = React.createClass({
    render() {
        const { dispatch } = this.props;
        return (
            <div>
                <h1>{this.props.text}</h1>
                <button onClick={()=>dispatch(newRandom())}>click me!</button>
            </div>
        );
    }
});

/*
 这个函数能接收到应用的 state tree ，并且返回我们需要的数据，也就是 props
*/
function mapStateToProps(state) {
    return {
        text: state.text
    }
}

//使用 connect 函数之后，App 组件就会被注入全局的 state 和一个 dispatch 函数
export default connect(mapStateToProps)(App)
```

最后是我们的 index 文件：

``` javascript
// index.js

import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import reducer from './reducers/reducer';
import App from './app';

//实例化一个 store
let store = createStore(reducer);

/*
 用 Provider 把我们的顶级组件 App 包裹其中，并传入 store 实例，
 这样在 App 中我们就能访问到 store 了。
*/
ReactDOM.render(
	<Provider store={store}>
		<App />
	</Provider>,
	document.getElementById('root')
);
```

最后，用 webpack 之类的打包工具打包一下，并开启本地服务器，就能在浏览器中访问我们的应用了：

  ![redux01](/images/redux01.png)



现在，每次点击按钮，都会产生一个随机数：

 ![redux02](/images/redux02.png)



great！我们已经用 react ＋ redux 做出了一个应用！现在来解释一下数据流。

1. 调用`store.dispatch(action)`

   用 dispatch 函数来派发一个 action ，一个 action 就是一个简单的对象。一个 action 至少要有一个表示事件类型的属性，例如 `type` 。然后，你可以添加任何你需要的属性，就像这样：

   ``` javascript
   {
     type: 'click',
     name: 'banana'
   }
   ```

   你可以在任何你需要的地方调用 dispatch ，可以在一个 ajax 的回调函数中，或者一个 setInterval 中。

2. store 调用 reducer

   还记得我们在实例化 store 的时候传进来了一个 reducer 了吗？我们的 store 会在这个时候调用这个 reducer ，同时传入两个参数：当前的 state tree 和 上一步的 action 。

   **强调一下** ：我们的 reducer 只是一个纯函数，它只会计算新的 state 。同时它的行为是可以预测的：同样的输入，每次都会得到同样的输出。它不应该执行任何会有副作用的动作，例如调用API ，路由，网络请求等，这些行为都应该在 dispatch( action ) 之前执行。

3. root reducer 把多个 reducer 的返回值合并成一棵单一的 state tree

   也许你的应用中只有一个 reducer ，但当应用的规模变大的时候，就会需要多个 reducer 来处理不同的逻辑，这时候，你可以用 `combineReducers( )` 来把多个 `reducer` 合并成一个 `rootReducer` 。就像下面这样：

   ``` javascript
   function todos(state = [], action) {
      // Somehow calculate it...
      return nextState
    }

    function visibleTodoFilter(state = 'SHOW_ALL', action) {
      // Somehow calculate it...
      return nextState
    }

    let todoApp = combineReducers({
      todos,
      visibleTodoFilter
    })
   ```

   当你派发来一个事件的时候，`todoApp` 就会调用传进来的所有 `reducers` ，在这里就是 `todos()` 和 `visibleTodoFilter()` ，然后把各个 `reducer` 返回的新的 state 组合成 state tree 。

4. store 把 root reducer 返回的整棵 state tree 保存起来

   来到这里，你的 app 中就有了一棵新的 state tree 了。这时候，所有用 `store.subscribe( listener )` 注册的 `listener` 函数都会被调用，在 `listener` 函数中，可以调用 `store.getState()` 来获取到当前的 state ，也就是最新的 state 了。最后，我们的 UI 就会更新，在 react 中，就相当于自动调用了 `component.setState()`。

