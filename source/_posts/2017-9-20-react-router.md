---
title: react-router 路由
date: 2017-09-20 21:40:21
category: Learn
tags:
    - react
    - router
---

#### 下载react-router-dom

> react-router和react-router-dom只需要一个就行了，后者比前者多出了<Link> <BrowserRouter>这样的 DOM 类组件，所以选择引入 react-router-dom

```
npm install --save react-router-dom
```

#### 第一种写法

下面引入的Router是 BrowserRouter，之前老的react-router把history分为三类（history是用来兼容不同浏览器或者环境下的历史记录管理的，当我跳转或者点击浏览器的后退按钮时，history就必须记录这些变化）

* hashHistory 老版本浏览器的history
* browserHistory h5的history
* memoryHistory node环境下的history，存储在memory中

4.0之前版本的react-router针对三者分别实现了createHashHistory、createBrowserHistory和create MemoryHistory三个方法来创建三种情况下的history

到了4.0版本，在react-router-dom中直接将这三种history作了内置，于是我们看到了BrowserRouter、HashRouter、MemoryRouter这三种Router

```js
var React = require('react');
var ReactDOM = require('react-dom');
import { BrowserRouter, Link, Route } from 'react-router-dom';
require('./sass/style.scss');
import List from './component/list';
import Detail from './component/detail';

export default class Index extends React.Component {
  render(){
    return (
      <BrowserRouter>
        <div>
          <ul>
            <li><Link to="/list/111">list</Link></li>
            <li><Link to="/detail">detail</Link></li>
          </ul>
          <Route path='/list/:id' component={List}/>
          <Route path='/detail' component={Detail}/>
        </div>
      </BrowserRouter>
    )
  }
}
```

> Router的path里冒号":" 后的部分是依次匹配Link的to后面的值，在组件里可以通过 `this.props.match.params.id` 来获取传递的值。

**BrowserRouter只允许有一个子元素，存在多个会报错**

#### 第二种写法

```js
var React = require('react');
var ReactDOM = require('react-dom');
import { Link, Switch, Route, BrowserRouter } from 'react-router-dom';
require('./sass/style.scss');
import List from './component/list';
import Detail from './component/detail';

export default class Index extends React.Component {
  render(){
    return (
      <div>
        <ul>
          <li><Link to="/list/111">list</Link></li>
          <li><Link to="/detail">detail</Link></li>
        </ul>
        <Switch>
          <Route path='/list/:id' component={List}/>
          <Route path='/detail' component={Detail}/>
        </Switch>
      </div>
    )
  }
}
```


