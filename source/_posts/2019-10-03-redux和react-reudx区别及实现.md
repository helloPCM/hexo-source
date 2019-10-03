---
title: redux和react-reudx区别及实现
date: 2019-10-03 10:02:17
tags: React
---

### Rdeux的基本用法

#### createStore

> createStore 创建一个 Redux store 来以存放应用中所有的 state，项目中仅有一个
> 入参：createStore(reducer, [preloadedState], enhancer)
* ==reducer== (Function): 接收两个参数，分别是当前的 state 树和要处理的 action，返回新的 state 树。

* ==[preloadedState]== (any): 初始时的 state。 在同构应用中，你可以决定是否把服务端传来的 state 水合（hydrate）后传给它，或者从之前保存的用户会话中恢复一个传给它。如果你使用 combineReducers 创建 reducer，它必须是一个普通对象，与传入的 keys 保持同样的结构。否则，你可以自由传入任何 reducer 可理解的内容。
* ==enhancer== (Function): Store enhancer 是一个组合 store creator 的高阶函数，返回一个新的强化过的 store creator。这与 middleware 相似，它也允许你通过复合函数改变 store 接口。

```js
import React from 'react'
import { createStore, combineReducers } from 'redux'

const counterReaducer = (state = 0, action) => {
    switch (action.type) {
        case 'INCREMENT':
            return state + 1;
        case 'DECREMENT':
            return state - 1;
        default:
            return state;
    }
}

// reducer参数形式
const store = createStore(counterReaducer);
store.getState()

// preloadedState形式
const store = createStore(combineReducers({counter: counterReaducer}));
store.getState().counter
```

创建的store有几个重要方法：
* store.getState() 获取数据。 此数据并不是响应式的。需要借助第三方库实现响应式
* store.dispatch(action) 只能通过派发action 来改变数据
* store.subscriber(() => { console.log('状态改变了') }) 状态改变之后的回调

```js
// createStore的实现原理
export function createStore(reducer) {
    let currentState = undefined;
    let subscriberList = [];

    function dispatch (action) {
        currentState = reducer(currentState, action)

        subscriberList.forEach(cb => cb())
        return action
    }

    function getState () {
        return currentState
    }

    function subscriber (cb) {
        subscriberList.push(cb)
    }

    dispatch({type: "@IMO00OC/PCM-REDUX"}) // 通过触发一个不存在的值使它有默认值

    return {
        dispatch,
        getState,
        subscriber
    }
}
```

#### applyMiddleware
> Middleware 最常见的使用场景是无需引用大量代码或依赖类似 Rx 的第三方库实现异步 actions。这种方式可以让你像 dispatch 一般的 actions 那样 dispatch 异步 actions。

==applyMiddleware(...middlewares)==

middleware 的函数签名是 ({ getState, dispatch }) => next => action。

```js
import logger from 'redux-logger';
import thunk from 'redux-thunk';
import { createStore, combineReducers, applyMiddleware } from 'redux'

const store = createStore(
    combineReducers({counter: counterReaducer}),
    applyMiddleware(logger, thunk)
);

function logger() {
    return next => action => {
        console.log(action.type ? action.type : 'acync' + '执行了')
        return next(action)
    }
}

function thunk({getState}) {
    return next => action => {
        if (typeof action === 'function') {
            return action(next, getState)
        }

        return next(action)
    }
}
<button onClick={() => {
    store.dispatch((dispatch, getState) => {
        setTimeout(() => {
            console.log('我是async')
            dispatch({type: 'ADD'})
            this.forceUpdate()
        }, 1000)
    })
}}>scync +</button>
// 如果dipatch的参数为一个函数，则会通过thunk 的异步去处理
```

#### 手写Redux的实现
```js
export function createStore(reducer, enhancer) {
    // 如果存在中间件, 通过enhancer这个高阶函数把createStore这个函数包装更强大，然后返回
    if (enhancer) {
        return enhancer(createStore)(reducer)
    }

    let currentState = undefined;
    const currentListeners = [];

    function getState() {
        return currentState
    }

    function dispatch(action) {
        currentState = reducer(currentState, action)
        currentListeners.forEach(v => v())

        return action
    }

    function subscribe(cb) {
        currentListeners.push(cb)
    }

    // 初始化状态, 通过一个不存在的值 让state有默认值
    dispatch({ type: "@IMOOC/KKB-REDUX" });

    return {
        getState,
        dispatch,
        subscribe
    }
}

export function applyMiddleware(...middlewares) {
    // 传递所有的中间件
    return createStore => (...args) => {
        // args为 reducer
        // 通过普通的createStore函数完成基本功能
        const store = createStore(...args)

        let dispatch = store.dispatch;
        // 此处dispatch中的args参数为，中间件中，调用dispatch的action
        const midApi = {
            getState: store.getState,
            dispath: (...args) => dispatch(...args)
        };
        // 将来中间件函数签名如下： funtion ({}) {}
        //[fn1(dispatch),fn2(dispatch)] => fn(diaptch)
        const chain = middlewares.map(mw => mw(midApi));
        // 强化dispatch,让他可以按顺序执行中间件函数
        dispatch = compose(...chain)(store.dispatch);
        console.log(dispatch)
        // 返回全新store，仅更新强化过的dispatch函数
        return {
        ...store,
        dispatch
        };
    }
}

export function compose(...funcs) {
    if (funcs.length === 0) {
        // 返回最原始的store.dispatch
      return arg => arg;
    }
    if (funcs.length === 1) {
        // 手动调用一下中间件
      return funcs[0];
    }
    // 聚合函数数组为一个函数 [fn1,fn2] => fn2(fn1())
    return funcs.reduce((left, right) => (...args) => right(left(...args)));
}
```


### react-redux的意义

> Provider, connect
* 通过Provider 实现了全局上下文
  
* connect 将state和dispatch 映射到组件的props中去

```js

// counter.js
// 全是返回action的函数
export const add = (num) => ({
    type: 'ADD',
    payload: num
})

export const minus = () => ({type: 'Minus'})

export const asyncAdd = (dispatch, getSate) => dispatch => {
    setTimeout(() => {
        dispatch({type: 'ADD'})
    }, 1000)
}

export const counterReducer = (state = 5, action) => {
    const num = action.payload || 1;
    switch (action.type) {
        case 'ADD':
            return state + num
        case 'Minus':
            return state - num
        default:
            return state
    }
}


// 主页面中

import React from 'react'
import thunk from 'redux-thunk'
import { createStore, combineReducers, applyMiddleware } from 'redux'
import { counterReducer, add, minus, asyncAdd } from '../store/counter'
import { Provider, connect } from 'react-redux'



const store = createStore(
    combineReducers({counter: counterReducer}),
    applyMiddleware(thunk)
)

export default class ReactReduxText extends React.Component {
    render() {
        return <Provider store={store}><ReactContent /></Provider>
    }
}

// 参数1：mapStateToProps = (state) => {return {num: state}}
// 参数2：mapDispatchToProps = dispatch => {return {add:()=>dispatch({type:'add'})}}
// connect两个任务：
// 1.自动渲染
// 2.映射到组件属性
@connect(
    state => ({ num: state.counter }),
    {
        add,
        minus,
        asyncAdd
    }
)
class ReactContent extends React.Component {
    render() {
        return (
            <div>
                <p>{this.props.num}</p>
                <button onClick={() => {this.props.add(2)}}>+</button>
                <button onClick={this.props.minus}>-</button>
                <button onClick={this.props.asyncAdd}>async+</button>
            </div>
        )
    }
}
```
