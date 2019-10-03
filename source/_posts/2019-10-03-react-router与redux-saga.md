---
title: react-router与redux-saga
date: 2019-10-03 09:59:44
tags: React
---

### react-router

> [react-router](超链接地址 "https://reacttraining.com/react-router/web/guides/quick-start")

```js
import React, { Component } from 'react'
import { BrowserRouter, Link, Route, Switch, Redirect } from 'react-router-dom'

export default class MyRouterTest extends Component {
    render() {
        return (<BrowserRouter>
            <Link to="/foo">foo</Link>
            <Link to="/bar">bar</Link>
            <Link to="/mua/abc">mua</Link>
            <Route path="/foo" component={() => <div>foo</div>} />
            <Route path="/bar" component={() => <div>bar</div>} />
            <Route path="/mua/:ns" render={({match}) => match.params.ns} />
        </BrowserRouter>)
    }
}

```

render内部是一个回调函数 render={ (props) => {}} 

props {match, history, location} 三个对象

案例==路由守卫==的实现：

```js
import React, { Component } from 'react'
import { BrowserRouter, Link, Route, Switch, Redirect } from 'react-router-dom'

function ProductList(props) {
    return (
        <div>
            <h3>ProductList</h3>
            <Link to="/detail/web">web</Link>
        </div>
    )
}

function Detail({match, history, location}) {
    console.log(match, history, location)

    return (<div>
        <h3>Detail</h3>
        {match.params.name}
        <button onClick={history.goBack}>后退</button>
    </div>)
}

function ProductMgt() {
    return (<div>
        <h3>ProductMgt</h3>
        <Link to="add">新增</Link>
        <Link to="search">搜索</Link>
        <Route path="/management/add" component={() => <div>add</div>} />
        <Route path="/management/search" component={() => <div>search</div>} />
        <Redirect to="/management/add" />
    </div>)
}
// 路由守卫：定义一个PrivateRoute组件
// 为其扩展一个用户状态检查功能
function PrivateRoute({ component: Component, isLogin, ...rest }) {
    return (
        <Route
        {...rest}
        render={
            // props === ({ match, history, location })
            (props) => 
                isLogin ? (
                    <Component />
                ) : (<Redirect to={{
                    pathname: "/login",
                    state: { redirect: props.location.pathname }
                }} />)
            }
        />
    )
}

export default class MyRouterTest extends Component {
    render() {
        return (<BrowserRouter>
            <nav>
                <Link to="/">商品列表</Link>
                <Link to="/management">商品管理</Link>
            </nav>

            {/* 路由配置 */}
            {/* react-router匹配不是独占的 */}
            <Switch>
                <Route exact path="/" component={ProductList} />
                <Route path="/detail/:name" component={Detail} />
                <PrivateRoute path="/management" component={ProductMgt} isLogin={false} />
                <Route path="/login" component={() => <div>login page</div>} />
                <Route component={() => <h3>页面不存在</h3>} />
            </Switch>
        </BrowserRouter>)
    }
}
```

手动实现react-router
```js
import React, { Component } from "react";
import { createBrowserHistory } from "history";
import pathToRegexp from "path-to-regexp";

const cache = {};
const cacheLimit = 10000;
let cacheCount = 0;

// /detail/web <==> /detail/:name
function compilePath(path, options) {
  const cacheKey = `${options.end}${options.strict}${options.sensitive}`;
  const pathCache = cache[cacheKey] || (cache[cacheKey] = {});

  if (pathCache[path]) return pathCache[path];

  const keys = [];
  const regexp = pathToRegexp(path, keys, options);
  const result = { regexp, keys };

  if (cacheCount < cacheLimit) {
    pathCache[path] = result;
    cacheCount++;
  }

  return result;
}

/**
 * Public API for matching a URL pathname to a path.
 */
function matchPath(pathname, options = {}) {
  if (typeof options === "string") options = { path: options };

  // 用户在Route上配置的path
  const { path, exact = false, strict = false, sensitive = false } = options;

  const paths = [].concat(path);

  return paths.reduce((matched, path) => {
    if (!path) return null;
    if (matched) return matched;

    // detail/web/1
    const { regexp, keys } = compilePath(path, {
      end: exact,
      strict,
      sensitive
    });
    const match = regexp.exec(pathname);

    if (!match) return null;

    const [url, ...values] = match;
    const isExact = pathname === url;

    if (exact && !isExact) return null;

    return {
      path, // the path used to match
      url: path === "/" && url === "" ? "/" : url, // the matched portion of the URL
      isExact, // whether or not we matched exactly
      params: keys.reduce((memo, key, index) => {
        memo[key.name] = values[index];
        return memo;
      }, {})
    };
  }, null);
}

//创建一个上下文保存history、location等
const RouterContext = React.createContext();

// Router：管理历史记录变更，location变更等等，并传递给后代
class BrowserRouter extends Component {
  constructor(props) {
    super(props);

    // 创建浏览器history对象
    this.history = createBrowserHistory(this.props);

    // 创建状态管理location
    this.state = {
      location: this.history.location
    };

    // 开启监听
    this.unlisten = this.history.listen(location => {
      this.setState({ location });
    });
  }

  componentWillUnmount() {
    if (this.unlisten) {
      this.unlisten();
    }
  }

  render() {
    return (
      <RouterContext.Provider
        value={{
          history: this.history,
          location: this.state.location
        }}
        children={this.props.children}
      />
    );
  }
}

class Route extends Component {
  render() {
    return (
      <RouterContext.Consumer>
        {context => {
          const location = context.location;

          // 根据pathname和用户传递props获得match对象
          const match = matchPath(location.pathname, this.props);

          // 要传递一些参数
          const props = { ...context, match };

          // children component render
          let { children, component, render } = this.props;

          if (children && typeof children === "function") {
            children = children(props);
          }

          return (
            <RouterContext.Provider value={props}>
              {children // children优先级最高，不论匹配与否存在就执行
                ? children
                : (props.match // 后面的component和render必须匹配
                ? (component // 若匹配首先查找component
                  ? React.createElement(component) // 若它存在渲染之
                  : (render // 若render选项存在
                  ? render(props) // 按render渲染结果
                  : null))
                : null)}
            </RouterContext.Provider>
          );
        }}
      </RouterContext.Consumer>
    );
  }
}

class Link extends React.Component {
  handleClick(event, history) {
    event.preventDefault();
    history.push(this.props.to);
  }

  render() {
    const { to, ...rest } = this.props;

    return (
      <RouterContext.Consumer>
        {context => {
          return (
            <a
              {...rest}
              onClick={event => this.handleClick(event, context.history)}
              href={to}
            >
              {this.props.children}
            </a>
          );
        }}
      </RouterContext.Consumer>
    );
  }
}

export default class MyRouterTest extends Component {
  render() {
    return (
      <BrowserRouter>
        <Link to="/foo">foo</Link>
        <Link to="/bar">bar</Link>
        <Link to="/mua/abc">mua</Link>
        <Route path="/foo" component={() => <div>foo</div>} />
        <Route path="/bar" component={() => <div>bar</div>} />
        <Route path="/mua/:ns" render={({ match }) => match.params.ns} />
        <Route children={({location}) => "xxx"} />
      </BrowserRouter>
    );
  }
}

```

### redux-saga

> [redux-saga](超链接地址 "https://redux-saga-in-chinese.js.org/docs/api/index.html")
 是一个用于管理应用程序 Side Effect（副作用，例如异步获取数据，访问浏览器缓存等）的 library，它的目标是让副作用管理更容易，执行更高效，测试更简单，在处理故障时更容易。
 
 redux-saga的使用有四个步骤：
 * 1.写一个 worker saga。 处理具体的异步操作
 ```js
 /*
 * call为调用函数，
 * put为派发一个dispatch
 * takeEvery为监听
 */
 
 import { call, put, takeEvery, takeLatest } from 'redux-saga/effects'
import Api from '...'

// worker Saga : 将在 USER_FETCH_REQUESTED action 被 dispatch 时调用
function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}
 ```
 
 * 2.写一个 watcher saga。去监听有没有diapatch这个action
 ```js
 /*
  在每个 `USER_FETCH_REQUESTED` action 被 dispatch 时调用 fetchUser
  允许并发（译注：即同时处理多个相同的 action）
*/
function* mySaga() {
  yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

/*
  也可以使用 takeLatest

  不允许并发，dispatch 一个 `USER_FETCH_REQUESTED` action 时，
  如果在这之前已经有一个 `USER_FETCH_REQUESTED` action 在处理中，
  那么处理中的 action 会被取消，只会执行当前的
*/
function* mySaga() {
  yield takeLatest("USER_FETCH_REQUESTED", fetchUser);
}

export default mySaga;
 ```
 
 * 3.为了能跑起 Saga，我们需要使用 redux-saga 中间件将 Saga 与 Redux Store 建立连接。
 
 ```js
// main.js
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import reducer from './reducers'
import mySaga from './sagas'

// create the saga middleware
const sagaMiddleware = createSagaMiddleware()
// mount it on the Store
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)

// then run the saga
sagaMiddleware.run(mySaga)

// render the application
 ```
 
 * 4.在代码中dispatch 在监听的action 就可以触发redux-saga
 ```js
 class UserComponent extends React.Component {
  ...
  onSomeButtonClicked() {
    const { userId, dispatch } = this.props
    dispatch({type: 'USER_FETCH_REQUESTED', payload: {userId}})
  }
  ...
}
 ```
 
 案例
 ```js
// mySaga.jsx
import React from 'react'
import { createStore, combineReducers, applyMiddleware } from 'redux'
import { Provider, connect } from 'react-redux'
import createSagaMiddleware from 'redux-saga'
import mySaga from './saga.js'

const fetchReaducer = (state = {}, action) => {
    switch (action.type) {
        case 'USER_FETCH_SUCCEEDED':
            return {
                ...state,
                user: action.user
            }
        case 'USER_FETCH_FAILED':
            return {
                ...state,
                message: action.message
            }
        default:
            return state;
    }
}

const sagaMiddleware = createSagaMiddleware()



const store = createStore(
    combineReducers({user: fetchReaducer}),
    applyMiddleware(sagaMiddleware)
)


sagaMiddleware.run(mySaga)


@connect(
    (state) => ({user: state.user})
)
class SagaTest extends React.Component {
    constructor(props) {
        super(props);
    }

    handleClick = () => {
        this.props.dispatch({ type: 'USER_FETCH_REQUESTED', payload: {userId: 'pcm'} })
    }

    render() {
        return (<div>
            <p>{this.props.user.user}</p>
            <button onClick={this.handleClick}>请求</button>
        </div>)
    }
}

export default function () {
    return <Provider store={store}><SagaTest /></Provider>
}
 ```
 
 ```js
 // saga.js
 
 import { call, put, takeEvery } from 'redux-saga/effects'

const Api = {
    fetchUser(usr) {
        return new Promise((resolve, reject) => {
            let result;
            setTimeout(() => {
                resolve(usr + 'hello')
            }, 2000)
        })
    }
}
// worker Saga : 将在 USER_FETCH_REQUESTED action 被 dispatch 时调用
function* fetchUser(action) {
    try {
        const user = yield call(Api.fetchUser, action.payload.userId);
        yield put({type: "USER_FETCH_SUCCEEDED", user: user})
    } catch (e) {
        yield put({type: "USER_FETCH_FAILED", message: e.message});
     }
}

// 监听的saga
function* mySaga() {
    yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

export default mySaga
 ```