---
title: react-module 组件化
date: 2017-09-22 08:23:38
category: Learn
tags:
    - react
---

### 组件化的使用React组件

```js
<PCNewsImageBlock count={10} type="guoji" width="400px"
     cartTitle="国际新闻" imageWidth="112px"></PCNewsImageBlock>
```

<!-- more -->

* count: 图片的数量
* type: 申请新闻接口的类别
* width: 标签的长度
* cartTitle: 标签的名称
* imageWidth: 组件里每张图片的宽度

![react 组件化](react-module.png)

> 每一个附加在组件上的属性均可以通过 `this.props.count` 来获取，我们需要定制什么属性，都可以把接口暴露出来，在调用组件的时候就可以实现组件的高度定制。

```js
import React from 'react';
import {Card} from 'antd';
import {BrowserRouter, Route, Link} from 'react-router-dom'; // 引入路由 

import PCNewsDetail from './pc_news_detail';
var pcCss = require('../../sass/pc.scss'); // 引入CSS

export default class PCNewsImageBlock extends React.Component {
  // 构造函数
  constructor(props) {
    super(props); // 调用父类的构造函数
    this.state = {
      news: ''
    };
  }
    
  // 组件加载完毕之后立即执行
  componentWillMount() {
    fetch('http://newsapi.gugujiankong.com/Handler.ashx?action=getnews'+
        '&type=' + this.props.type + '&count=' + this.props.count, {
      method: 'GET',
      mode: 'cors'
    }).then(response => response.json()).then(json => {
      this.setState({news: json})
    });
  }
  
  // 继承自React.component 每个组件必须实现的方法
  render() {
    // inline style 采用驼峰(camelCased)写法
    // 通过表达式写入到标签里 style={styleImage}
    const styleImage = {
      display: 'block',
      width: this.props.imageWidth, // 图片的宽度
      height: '90px',
    }
    const styleH3 = {
      width: this.props.imageWidth,
      whiteSpace: 'nowrap',
      overflow: 'hidden',
      textOverflow: 'ellipsis',
    }
    
    const {news} = this.state;
    const newsList = news.length
      ? news.map((newsItem, index) => (
        <div key={index} class={pcCss.image_block}>
          <Link to={`details/${newsItem.uniquekey}`} target="_blank">
            <div className="custom-image">
              <img src={newsItem.thumbnail_pic_s} style={styleImage} alt={newsItem.title}/>
            </div>
            <div class="custom-card">
              <h3 style={styleH3}>{newsItem.title}</h3>
              <p>{newsItem.author_name}</p>
            </div>
          </Link>
        </div>
      ))
      : "没有加载到任何新闻";
      
    // 返回JSX的元素
    return (
      <div className="topNewsList">
        <BrowserRouter>
          <Card title={this.props.cartTitle} bordered={true} style={{
            width: this.props.width
          }}>
            {newsList}
          </Card>
        </BrowserRouter>
      </div>
    )
  }
}
```

### React 官方对 Inline Style 的 介绍

1. 采用驼峰(camelCased)写法，对于 backgroundImage 渲染后成为 ` background-image`；
2. 可以不用写 px 直接采用数字 1 取代 1px，React 会自动加上 px，有些 CSS 样式属性不会自动加上 px，请参考不会自动添加 px 的样式属性； WebkitTransition 和 msTransition 中，Webkit 和 ms
3. 是 JS前缀(JavaScript prefix)，解决跨平台，更多参考Modernizr Prefixed;

### 组件化的优势

1. 可扩展
2. 可复用
3. 高内聚/低耦合 - 我们无需关心该组件内部的实现细节

> 部分借鉴自👉👉👉[使用 JS 写 CSS 与 React 内联样式](http://robinchen.me/tech/2016/08/09/tech-Refactor-CSS-into-JS.html)






