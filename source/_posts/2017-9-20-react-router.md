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

---

### Route

之后用React做主页跳转的详情页时，碰到一个问题：

```
<BrowserRouter>
  <Switch>
      <Route exact path="/" component={PCIndex}></Route>
      <Route path="/details/:uniquekey" component={PCNewsDetail} />
  </Switch>
</BrowserRouter>
```

就这样的一个页面，在首页点击详情连接一直无法跳转。

发现是如果不加 exact 这个参数，会把已 `/`这个开头的url全部匹配上，所有必须加上 `exact` 进行精确匹配。

#### exact

> 只会匹配和 location.pathname 完全相同的路径

| path | location.pathname | exact | matches? |
| --- | --- | --- | --- |
| /one | /one/two | true | No |
| /one | /one/two | false | Yes |

#### strict

> 如果path末尾有一个斜线，则会匹配到尾部的斜线为止，如果后面还有其他的url段，对后面的url段就不起作用了。

| path | Location.pathname | mathces? |
| --- | --- | --- |
| /one/ | /one | No  |
| /one/ | /one/ | Yes |
| /one/ | /one/two | Yes  |

---

### 问题

1. 用 `webpack-dev-server` 刷新子页面的时候 `http://localhost:8081/details/161028202106247`，页面会有404错误,

可以在运行 `webpack-dev-server` 的时候后面加上参数 `--history-api-fallback`

```
// package.json

"scripts": {
    "build": "webpack",
    "dev": "webpack-dev-server --devtool eval --progress --colors --content-base build --history-api-fallback",
    "test": "echo \"Error: no test specified\" && exit 1"
},
```

> 直接在命令行在根目录运行 `npm run dev` 就可以运行 **package.json中预设的指令**





