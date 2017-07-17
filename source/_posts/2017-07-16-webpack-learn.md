---
title: webpack 3.3.0 学习小记
date: 2017-07-16 20:43:33
category: Learn
toc: true
tags:
    - webpack 
---

暑假打算借着留校的机会 学下react，在学react之前，想学下webpack，之后有空可以再细看gulp这类打包工具。初学，记一篇博文，也算巩固下所学。

-------

<!--more-->

先附上[webpack官网](https://webpack.js.org/)

进入项目目录，我们用npm包管理器

```
npm init

npm install webpack --save-dev 
```

然后在根路径下新建`webpack.config.js` （webpack配置文件）

```js
// webpack.config.js
var path = require('path');

module.exports = {
  entry: './src/index.js',                  // 入口文件
  output: {                                 // 打包输出配置
    filename: 'bundle.js',                  // 输出文件名
    path: path.resolve(__dirname, 'dist')   // 必须用绝对路径
  }
};
```

这里要提下， `output->path` 的值，老版本的webpack里是可以用相对路径的，但在新版本中要官方要求用绝对路径，不然打包会报错，但是绝对路径也不能写死，我们就在文件头部引入 path，如下
```js
var path = require('path');
```

path是node的内置模块，直接引入就能使用了，然后在通过

```
path.resolve(__dirname, 'dist')
```

获取到根路径下的`/dist`文件路径。

`output->filename`有很多变量可以加入，例如`[name]-[chunkhash:5].js`


| template | Desciption |
| --- | --- |
| [hash] | 模块标识符的hash（每次构建都不同） |
| [chunkhash] | chunk内容的hash（只有当文件打包的内容发生改变，才会不同） |
| [name] | 模块名称 |
| [id] | 模块标识符 |
| [file] | 模块文件名称 |
| [filebase] | 模块的basename |
| [query] | 模块的query |


-------

webpack插件 plugin

```
var HtmlWebpackPlugin = require('html-webpack-plugin');
var CleanPlugin = require('clean-webpack-plugin');

module.exports = {
    entry: {
        index: './src/index.js',
        join: './src/join.js',
        vendor: ['jquery', 'mustache'],
    },
    plugins: [
        // 可以将确定的第三方文件都打包到主入口文件中，
        // 这样就不需要在别的模块引入公用的文件。例如jquery
        // 在entry（入口） vendor中加入公用的第三方文件的名称
        new webpack.optimize.CommonsChunkPlugin({
            name: ["vendor"],
            minChunks: 2,
        }),
        // 可以使用我们自己的模板，并自动把打包好的js文件引入模板。
        new HtmlWebpackPlugin({
            filename: 'index.html',     // 导出的文件名
            template: './index.html',   // 模板位置
            inject: 'body',     // 插入js的位置 header|body
            minify: {
                removeComments: true,   // 删除注释
                collapseWhitespace: true,   // 删除空格
            }
        }),
        new HtmlWebpackPlugin({ //根据模板插入css/js等生成最终HTML
            filename: 'join.html', //生成的html存放路径，相对于path
            template: './join.html', //html模板路径
            inject: 'body', //js插入的位置，true/'head'/'body'/false
            minify: { //压缩HTML文件  
                removeComments: true, //移除HTML中的注释
                collapseWhitespace: false //删除空白符与换行符
            }
        }),
        // 每次打包之前，先清空导出的文件夹
        new CleanPlugin(['dist']),
    ],
}
```



-------

### loader

因为webpack只能处理js，我们需要引入很多loader来处理诸如.css、.scss、.less、.png、.jsx.....

具体需要如何配置也可以去[npm官网](https://www.npmjs.com/)搜索

#### [babel-loader](https://www.npmjs.com/package/babel-loader)

第一个我们需要`babel-loader`来转化我们的js代码，让我们能够用上最新的javascript的语法，而不用等待浏览器支持。

我们需要先在项目根目录下运行命令行，

```
npm install --save-dev babel-loader babel-core babel-preset-env webpack
```

在webpack.config.js中配置`module`

这是最新的loader配置的写法，有些博客写的webpack版本老，写法不太一样。但只要参照[官网的配置方式](https://webpack.js.org/loaders/babel-loader/)肯定没问题。所以学会一门技术，最好的方式就是直接去查看官方文档，而且现在技术迭代很快，半年可能就会有个大版本更新，一般的博文都会有滞后的问题。

```
module.exports = {
    entry: {
        index: './src/index.js',
        join: './src/join.js',
        vendor: ['jquery', 'mustache'],
    },
    output: {
        filename: production ? 'js/[name]-[hash].js' : 'js/[name]-bundle.js',
        path: path.resolve(__dirname, 'dist'),
        publicPath: path.resolve(__dirname, 'dist/builds'),
        chunkFilename: 'js/[name].[chunkhash:5].chunk.js',
    },
    module: {
       rules: [{
           test: /\.js$/,   
           exclude: /(node_modules|bower_components)/, 
           include: path.resolve(__dirname, 'src/Components/js'),
           use: {
               loader: 'babel-loader',
               options: {
                   presets: ['env']
               }
           }
       }]
    }
}
```

在`module->rules`里可以配置多个loader。解释下里面的一些主要配置信息：

* `text` 正则匹配需要loader处理的文件
* `exclude` 排除需要处理的文件夹或文件，因为每次loader都会查找项目下所有匹配的文件，很耗时，有些文件我们不需要loader处理，就在这里进行排除。
* `include` 对应的 这是只处理我们需要loader处理的文件或文件夹
* `use`具体使用的loader信息及配置，可同时配置多个loader来处理同一种文件类型。

* [style-loader](https://www.npmjs.com/package/style-loader) 将css以style标签引入到网页中。
* [css-loader](https://www.npmjs.com/package/css-loader) 转换css到js文件里。
* [postcss-loader](https://www.npmjs.com/package/postcss-loader) 可以自动为一些属性加上浏览器前缀。

`postcss-loader`的配置需要提下，

```js
{
   test: /\.css$/,
   use: [
     {
       loader: 'style-loader',
     },
     {
       loader: 'css-loader',
       options: {
           importLoaders: 1,
       }
     },
     {
       loader: 'postcss-loader',
       options: {
         config: {
           path: './postcss.config.js' // 和webpack.config.js同目录的配置文件
         }
       }
     }
   ]
},
```
```js
// postcss.config.js
module.exports = {
    plugins: [
        require('autoprefixer') // 自动添加浏览器前缀的插件
    ]
}
```

* [sass-loader](https://www.npmjs.com/package/sass-loader)
* [file-loader](https://www.npmjs.com/package/file-loader) 可以处理照片类的文件。
* [url-loader](https://www.npmjs.com/package/url-loader) 和file-loader功能类似，还能设置`limit`，可以将少于这个大小的文件转换成Base64编码直接嵌入网页中，可以减少http请求，但是会增加网页的大小。

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192  // 单位是B，字节
            }
          }
        ]
      }
    ]
  }
}
```
* [html-loader](https://www.npmjs.com/package/html-loader)


-------

### require.ensure() 按需加载

`require.ensure(dependencies: String[], callback: function(require), errorCallback: function(error), chunkName: String)`

有些组件不是一开始就需要的，而且一开始就加载会影响加载速度，影响用户体验，因此我们可以使用按需加载的形式，等需要了再通过ajax将模块加载进来。

> 之前很多博客提到第三个参数是设定块的名称，但实际上在新版本使用时，第四个参数才是设定块(chunk)的名称。

```js
require.ensure([], () => {
    // 引入Button组件
    const Button = require('./Components/Button');
    const button = new Button('google.com');

    button.render('a');
}, () => {}, 'button');
```











