---
nav:
  title: 生态
  order: 3
group:
  title: 数据流
  order: 3
title: Redux
order: 2
---

# Redux

> Redux is a predictable state container for JavaScript apps
>
> Redux 是针对 JavaScript 应用的可预测状态容器

Redux 是一个 **数据管理中心**，提供 **可预测化** 的状态管理，可以让你构建一致化的应用，运行于不同的环境（客户端、服务端、原生应用），并且易于测试。

Redux 参考了 Flux 的架构思想，对 Flux 中冗余部分（如 Dispatcher）进行简化，同时将 Elm 语言中 **函数式编程** 的思想融入其中。

```jsx | inline
import React from 'react';
import img from '../../assets/redux-model.jpg';

export default () => <img alt="redux-model" src={img} width={520} />;
```

<br/>

**Redux 的特点：**

- **可预测的（Predictable）**：因为 Redux 用了纯函数（Pure Function）的概念，每个新的 `state` 都会由旧的 `state` 创建一个全新的 `state`，这样可以作所谓的时光旅行调试
- **状态容器（State Container）**：`state` 是集中在单一个对象树状结构下的单一 `store`，`store` 即是应用程序领域（App Domain）的状态集合
- **JavaScript 应用**：这说明 Redux 并非单指设计给 React 使用的，它是独立的一个函数库，可通用于各种功能 JavaScript 应用

通过一张图看 Redux 如何简化状态管理。

```jsx | inline
import React from 'react';
import img from '../../assets/redux_1.jpg';

export default () => <img alt="Redux State" src={img} width={640} />;
```

## 产生背景

随着 JavaScript 应用越来越大，越来越复杂，我们需要管理的 `state` 变得越来越多。这些 `state` 可能包括服务器响应、缓存数据、本地生成尚未持久化到服务器的数据，也包括 UI 状态，如激活的路由，被选中的标签，是否显示加载动效或者分页器等。

管理不断变化的 `state` 非常困难。如果一个 `model` 的变化会引起另一个 `model` 的变化，那么当 `view` 变化时，就可能引起对应 `model` 以及另一个 `model` 的变化，依次地，可能会引起另一个 `view` 的变化。直至你搞不清楚到底发生了什么。`state` 在什么时候，由于什么原因，如何变化已然不受控制。当系统变得错综复杂时，想重现问题或者添加新功能就会变得非常复杂。

虽然 React 试图在视图层禁止异步和直接操作 DOM 来解决这个问题。美中不足的是，React 依旧把处理 `state` 中数据的问题留给了你。Redux 就是为了帮你解决这个问题的。

Redux 应用中所有的 `state` 都以一个对象树的形式储存在一个单一的 `store` 中，唯一改变 `state` 的办法是触发 `action`，`action` 是一个描述发生了什么的对象。为了描述 `action` 如何改变 `state` 树，你需要编写 `reducers`。

## 三大原则

- **单一数据源**：整个应用只有唯一的状态树，也就是所有状态 `state` 最终维护在一个根级 `store` 中
- **状态只读**：为了保证状态的 **可控性**，最好的方式就是监控状态的变化
  - Redux 中 `store` 的数据无法被直接修改
  - 严格控制修改的执行。
- **纯函数**：规定只能通过一个纯函数（Reducer）来描述修改

### 单一数据源

**整个应用的状态数据被储存在一棵 Object Tree 中，并且只存在于唯一一个 Store 中。**

在传统的 MVC 架构中，我们可以根据需要创建无数个 Model，而 Model 之间可以互相监听、 触发事件甚至循环或嵌套触发事件，这些在 Redux 中都是不允许的。

因为在 Redux 的思想里，一个应用永远只有唯一的数据源。我们的第一反应可能是：如果有一个复杂应用，强制要求唯一的数据源岂不是会产生一个特别庞大的 JavaScript 对象。

实际上，使用单一数据源的好处在于整个应用状态都保存在一个对象中，这样我们随时可以提取出整个应用的状态进行持久化（比如实现一个针对整个应用的即时保存功能）。此外，这样的设计也为服务端渲染提供了可能。

至于我们担心的数据源对象过于庞大的问题，可通过了解 Redux 提供的工具函数 `combineReducers` 是如何化解的。

> 📌 Flux 可能有多个 Store（区别于 Flux 的 Store，Redux 的 State 不会存放逻辑，操作数据在 Reducer 中处理）

### 状态只读

**只能通过触发事件（也就是触发 Action）来产生新的状态数据，Action 是一个用于描述已发生事件的普通对象。**

这一点和 Flux 的思想不谋而合，不同的是在 Flux 中，因为 Store 没有 `setter` 而限制了我们直接修改应用状态的能力，而在 Redux 中，这样的限制被执行得更加彻底，因为我们压根没有 Store。

在 Redux 中，我们并不会自己用代码来定义一个 Store。取而代之的是，我们定义一个 Reducer， 它的功能是根据当前触发的 Action 对当前应用的 State 进行迭代，这里我们并没有直接修改应用的 State，而是返回了一份全新的状态。

Redux 提供的 `createStore` 方法会根据 Reducer 生成 Store。最后，我们可以利用 `store.dispatch` 方法来达到修改状态的目的。

```js
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1,
});

store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPELETED',
});
```

### 纯函数

**在 `reducer` 中指定状态数据转换的逻辑。**

**纯函数的特点：**

1. 相同的输入，结果始终相同
2. 不对外部环境进行操作

在 Redux 里，我们通过定义 `reducer` 来确定状态的修改，而每一个 `reducer` 都是纯函数，这意味着它没有副作用，即接受一定的输入，必定会得到一定的输出。

这样设计的好处不仅在于 `reducer` 里对状态的修改变得简单、纯粹、可测试，更有意思的是， Redux 利用每次新返回的状态生成酷炫的时间旅行（time travel）调试方式，让跟踪每一次因为触发 Action 而改变状态的结果成为了可能。

```js
function visibilityFilter(state = 'SHOW_ALL', action) {
  switch(action.type) {
    case 'SET_VISIBILITY_FILTER'
        return action.filter
    default
        return state
  }
}

function todos(state = [], action) {
  switch(action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
            text: action.text,
            completed: false
        }
      ]
    case 'COMPLTE_TODO':
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: true
          })
        }
        return todo
      })
    default:
      return state
  }
}

import { combineReducers, createStore } from 'redux'
let reducer = combineReducers({ visibilityFilter: todos })
let store = createStore(reducer)
```

> 为什么需要使用纯函数？

因为 Redux 的 `store` 设计，并不是原本 Flux 架构的 `store`，而是 `reducer store`，这个 `reducer store` 是一个在 Flux 中的 `store` 的进化版，在说明中它有一个叫做 `reduce` 的方法。

## 组成部分

### Store

**Store** 是整个应用程序的数据集合，并包含了所有对数据的变动方法。

通过 `redux` 的 `createStore` API 生成 Store，该方法可传三个参数：

- `reducer`：
- `preloadedstate`：State 的初始值
- `enhancer`：

返回值为对象类型，具有以下属性：

- `getState`：获取状态（`state`）
- `dispatch`：触发动作（`action`），更新状态（`state`）
- `subscribe`：订阅数据变更，注册监听器（数据变更时触发）
- `replaceReducer`：
- `[$$observable]`：

```js
import { createStore } from 'redux';

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([action.text]);
    default:
      return state;
  }
}

// const store = createStore(Reducer, initStore);
const store = createStore(todos, ['Coding']);

store.dispatch({
  type: 'ADD_TODO',
  text: 'Drinking coffe',
});

console.log(store.getState());
// [ 'Coding', 'Drinking coffee' ]
```

### Action

Action 是把数据从应用传到 Store 的有效载荷（Payload）。它是 Store 数据的唯一来源，也就是说要改变 Store 中的 State 就需要触发一个 Action。

Action 本质上是一个普通的 JavaScript 对象，`action` 内必须使用一个字符串类型 `type` 字段来表示将要执行的动作，除了 `type` 字段外，`action` 对象的结构完全由你来决定。多数情况下，`type` 会被定义成字符串常量。当应用规模越来越大时，建议用单独的模块或文件来存放 Action Type。

```js
import { createStore } from 'redux';
const store = createStore(todos, ['Use Redux']);

function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text,
  };
}

store.dispatch(addTodo('Read the docs'));
store.dispatch(addTodo('Read about the middleware'));
```

### Action Creator

```js
function addToDo(text) {
  return {
    type: 'ADD_TODO',
    text,
  };
}
```

这样将使 `action` 创建函数更容易被移动和测试。

### Reducer

**Reducer** 是根据 `action` 修改 `store` 将其转变成下一个 `state`，记住 `actions` 只描述了有事情发生了这一事实，并没有描述应用如何更新 `state`。

```js
(prevState, action) => nextState;
```

保持 Reducer 纯净非常重要。永远不要在 Reducer 里做这些操作：

- 修改传入参数
- 执行有副作用、如 API 请求和路由跳转
- 调用非纯函数，如 `Date.now()` 和 `Math.random()`

```js
// reducer
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Obejct.assign({}, state, {
        visibilityFilter: action.filter,
      });
    default:
      return state;
  }
}
```

- 不要直接修改 State
  - 使用 `Object.assign` 新建一个副本。不能这样使用 `Object.assign(state, { visibilityFilter: action.filter })`，因为它会改变第一参数的值。你必须把第一个参数设置为空对象。
  - 你也可以开启对 ES7 提案对象展开运算符的支持，从而使用 `{...state, visibilityFilter: action.filter}` 达到相同的目的
- 在 `default` 情况下返回旧的 `state`。遇到未知的 `action` 时，一定要返回旧的 state。

```js
// A normal Reducer
const initList = [];
function ListReducer(state = initList, action) {
  switch (action.type) {
    case 'ADD_LIST':
      return state.concat([action.item]);
      break;
    default:
      return state;
  }
}
```

⚠️ **注意事项：**

- 遵守数据不可变原则，不能直接修改 State，而是返回一个新对象
- 默认情况下需要返回原数据，避免数据被清空
- 最好设置初始值，便于应用的初始化及数据稳定

## 工作流程

1. 用户（操作 View）发出 Action，发出方式就用到了 `store.dispatch` 方法
2. 然后，Store 自动调用 Reducer，并且传入两个参数（当前 State 和收到 Action），Reducer 根据 Action 返回新的 State，如果有 Middleware，Store 会将当前 State 和收到的 Action 传递给 Middleware，Middleware 会调用 Reducer 然后返回新的 State
3. State 一旦有变化，Store 就会调用监听函数，来更新 View

## 对比测评

### 自我评价

优点：

- 可预测：始终有一个唯一准确的数据源（single source of truth）即 `store`，因此不存在如何将当前状态 `store` 与动作和应用的其他部分同步的问题
- 易维护：具备可预测的结果和严格的组织结构让代码更容易维护
- 易测试：编写可测试代码的首要准则是编写可以仅做一件事并且独立的小函数（single responsibility principle），Redux 的代码几乎全部都是这样的函数：短小、纯粹、分离

缺陷：

- 一个组件所需要的数据，必须由父组件传过来，而不能像 Flux 中直接 从 Store 取。
- 当一个组件相关数据更新时，即使父组件不需要用到这个组件，父组件还是会 `re-render`，可能会有效率影响，或者需要写复杂的 `shouldComponentUpdate` 进行判断。

### 与 Flux 对比

Redux 是 Flux 架构思想的实践方案，同时又在其基础上进行改进。Redux 承载了 Flux 单向数据流、Store 是唯一的数据源的思想。

- Redux 中没有 Dispatcher：它使用 Store 的 `Store.dispatch` 方法来把 action 传给 Store，由于所有的 action 处理都会经过这个 `Store.dispatch` 方法，所以在 Redux 中很容易实现 Middleware 机制。Middleware 可以让你在 `reducer` 执行与执行后进行拦截并插入代码，来达到操作 `action` 和 `store` 的目的，这样一来很容易实现灵活的日志打印、错误收集、API 请求、路由等操作。
- Redux 仅有一个 `store`：Flux 中允许有多个 `store`，但是 Redux 只允许有一个，相较于多个 `store` 的 Flux，一个 `store` 更加清晰，并易于管理

<br />

| Flux                            | Redux                         |
| :------------------------------ | :---------------------------- |
| Store 包含状态和更改逻辑        | Store 和更改逻辑是分开的      |
| 有多个 Store                    | 只有一个 Store                |
| 所有 Store 都互不影响且是平级的 | 带有分层 reducer 的单一 Store |
| 有单一调度器                    | 没有调度器的概念              |
| React 组件订阅 Store            | 容器组件是有联系的            |
| 状态是可变的                    | 状态是不可改变的              |

🌰 **标准示例：**

```js
import { createStore } from 'redux';
/**
 * 这是一个 reducer，形式为 (state, action) => state 的纯函数
 * 描述了 action 如何把 state 转变成下一个 state
 *
 * state 的形式取决于你，可以是基本类型、数组、对象
 * 当 state 修改时需要返回全新的对象，而不是修改传入的参数
 *
 * 下面例子使用 `switch` 语句和字符串来做判断，但你可以写帮助类（helper）
 * 根据不同的约定（如方法映射）来判断，只要适用于你的项目就可以了
 */
function counter(state = 0, action) {
  switch ((action, type)) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
}

// 创建 Redux Store 来存放应用状态
// API 是 { subscribe, dispatch, getState }
let store = createStore(counter);

// 可以手动订阅更新，也可以事件绑定到视图层
store.subscribe(() => {
  console.log(store.getState());
});

// 改变内部 state 唯一方法是 dispatch 一个 action
// action 可以被序列化，用日记记录和储存下来，后期还可以回放的方式执行
store.dispatch({ type: 'INCREMENT' });
// 1
store.dispatch({ type: 'INCREMENT' });
// 2
store.dispatch({ type: 'DECREMENT' });
// 1
```

## 相关第三方库

异步数据流

- redux-thunk
- redux-promise
- redux-saga
- redux-persist：支持 Store 本地持久化
- redux-observable：实现可取消的 action

## 参考资料

- [📖 Redux 中文文档](https://github.com/camsong/redux-in-chinese)
- [📝 Redux 源码分析与设计思路分析](https://github.com/WisestCoder/blog/issues/1)
- [📝 单页应用的数据流方案探索](https://zhuanlan.zhihu.com/p/26426054)
- [📝 Redux 概念之一：Redux 简介](https://www.imooc.com/article/16061)
- [📝 Redux 概念之二：Redux 的三大原则](https://www.imooc.com/article/16062)
- [📝 Redux 概念之三：Action 与 ActionCreator](https://www.imooc.com/article/16063)
- [📝 Redux 概念之四：Reducer 与纯函数](https://www.imooc.com/article/16064)
- [📝 Redux 概念之五：Redux 套用七步骤](https://www.imooc.com/article/16065)
- [如何在 React+Redux 的项目中更优雅的实现前端自动化测试](https://testerhome.com/topics/8032)
- [Redux 从设计到源码](https://tech.meituan.com/2017/07/14/redux-design-code.html)
- [Vuex、Flux、Redux、Redux-saga、Dva、MobX](https://zhuanlan.zhihu.com/p/53599723)
- [Flux、Redux、Vuex、MobX 总结（概念篇）](https://zhuanlan.zhihu.com/p/75696114)
