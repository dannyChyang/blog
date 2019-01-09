
> 原文链接：[https://css-tricks.com/server-side-react-rendering/](https://css-tricks.com/server-side-react-rendering/)

React是最有名的客户端javascript框架之一，但你是否了解React服务端渲染的相关知识？

假设你已经建立了一个灵活的新闻列表React客户端应用，该应用调用了你最喜欢的API服务。几周后，用户告诉你，他们的页面无法被Google收录，在Facebook上看起来也不太好。似乎不是什么大问题，对吗？

你意识到解决问题的关键在于**React页面需要由服务器端执行初始化加载，这样搜索引擎的爬虫以及社交媒体才可以解析它们**。

有证据显示，由Javascript生成的内容有时可以正确地被Google爬虫索引到，但并不保证总是这样。因此，如果你想确保良好的SEO，及其它服务如Facebook,Twitter的兼容性，推荐使用服务器端渲染。

在本教程中，我们将一步步完成一个服务器端渲染的示例。解决一个从API服务获取数据的React应用中的常见问题。

## 服务器端渲染的优点
SEO可能是让你的团队开始讨论服务器端渲染的原因，但并不仅仅只是这一点。

一个大的好处是：**服务器端渲染可以更快地显示页面。** 通过服务器端渲染，服务器返回给浏览器的HTML是已经准备好的，因此浏览器可以直接显示，而不必等待javascript下载并执行。在浏览器下载执行JavaScript和其他资源时，页面不会展示一片空白，这种情况会发生在在客户端React站点。

## 准备工作
让我们使用 `Babel` 和 `Webpack` 为一个基本的React应用添加服务端渲染。该应用包含一个从第3方获取数据的接口。你可以在GitHub上查看[完整的例子](https://github.com/ButterCMS/react-ssr-example/releases/tag/starter-code) 及启动代码。

应用的入口是 `hello.js` 中的React组件，它发送一个异步请求到theButterCMS的API服务器，将返回的JSON格式的博客列表数据解析并渲染。ButterCMS是一个面向个人免费的博客API引擎，它对模拟真实开发环境非常有用。启动代码中包含了API授权部分代码，你可以使用Github帐号 [登录ButterCMS获取自己的API token授权码](https://buttercms.com/github/oauth) ，`将代码中的token改为自己的。`

``` JS
import React from 'react';
import Butter from 'buttercms'

const butter = Butter('b60a008584313ed21803780bc9208557b3b49fbb');

var Hello = React.createClass({
  getInitialState: function() {
    return {loaded: false};
  },
  componentWillMount: function() {
    butter.post.list().then((resp) => {
      this.setState({
        loaded: true,
        resp: resp.data
      })
    });
  },
  render: function() {
    if (this.state.loaded) {
      return (
        <div>
          {this.state.resp.data.map((post) => {
            return (
              <div key={post.slug}>{post.title}</div>
            )
          })}
        </div>
      );
    } else {
      return <div>Loading...</div>;
    }
  }
});

export default Hello;
```
启动代码中其它的说明：
* package.json - 定义依赖关系
* Webpack和Babel的配置
* index.html - 应用的HTML文件
* index.js - 加载并渲染Hello组件

要让应用运行起来，首先克隆仓库：
``` Command Line
git clone ...
cd ..
```
安装依赖:

``` Command Line
npm install
```
然后启动开发服务：

```Command Line
npm run start
```
打开 [http://localhost:3000](http://localhost:3000) 查看应用:

![](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_1000,f_auto,q_auto/v1497358286/localhost_r84tot.png)

如果你查看渲染后页面的源代码，会看到发送到浏览器的标签只是一个指向javascript文件的链接。这意味着页面内容不能保证被搜索引擎及社交媒体平台抓取：

![](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_1000,f_auto,q_auto/v1497358332/some-html_mrmpfj.png)

## 添加服务器端渲染
接下来我们将实现服务端泻染，从而将生成的HTML发送到浏览器。如果你想一次查看所有更改，看一下github上的差异部分。
开始之前，我们安装一下Express，一个nodeJS的服务端应用框架：

```Command Line
npm install express --save
```

创建一个渲染React组件的服务器

```javascript
import express from 'express';
import fs from 'fs';
import path from 'path';
import React from 'react';
import ReactDOMServer from 'react-dom/server';
import Hello from './Hello.js';

function handleRender(req, res) {
  // Renders our Hello component into an HTML string
  const html = ReactDOMServer.renderToString(<Hello />);

  // Load contents of index.html
  fs.readFile('./index.html', 'utf8', function (err, data) {
    if (err) throw err;

    // Inserts the rendered React HTML into our main div
    const document = data.replace(/<div id="app"><\/div>/, `<div id="app">${html}</div>`);

    // Sends the response back to the client
    res.send(document);
  });
}

const app = express();

// Serve built files with static files middleware
app.use('/build', express.static(path.join(__dirname, 'build')));

// Serve requests with our handleRender function
app.get('*', handleRender);

// Start server
app.listen(3000);
```

回顾一下发生了什么...
`handleRender` 方法处理了所有接收的请求。文件顶部导入的 [ReactDOMServer类](https://reactjs.org/docs/react-dom-server.html) 提供了 `renderToString` 方法用来将react组件渲染进html完成初始化步骤。

```javascript
ReactDOMServer.renderToString(<Hello />);
```
我们将Hello组件标签传入 `ReactDOMServer.renderToString()`中，并将返回的HTML注入到index.html中，从而在服务器端生成完整的的HTML。

```javascript
const document = data.replace(/<div id="app"><\/div>/, `<div id="app">${html}</div>`);
```
要启动服务，需要更新package.json的script配置项，然后运行'npm run start’命令：

```JSON
"scripts": {
  "start": "webpack && babel-node server.js"
},
```
在浏览器中打开 [http://localhost:3000](http://localhost:3000) 查看应用。看！你的页面现在由服务器端渲染。但还有个问题，如果你查看页面源代码。你会注意到，博客API接口返回的内容依然没有包含进来。
![](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_1000,f_auto,q_auto/v1497358447/devtools_qx5y1o.png)

发生了什么事？如果你打开chrome控制台的network面版，就会看到浏览器发出的博客API的请求。

虽然我们在服务器端渲染了React组件，但博客api请求是在 `componentWillMount` 时发出的异步请求，它在浏览器端展示页面的请求完成之后才得到响应。
因此尽管我们使用了服务器端渲染，也只做了其中的一部分。
事实上，这里有一个[关于这个问题的探讨](https://github.com/facebook/react/issues/1739)，并有超过100条关于讨论及解决方案的留言。

## 如何在渲染前获取数据
为了解决这个问题，我们要保证API请求在Hello组件渲染前完成数据获取。

这意味着需要把API的声明放在在React组件的生命周期外面，并在组件渲染前得到返回数据。

我们将逐步解决这个问题，你可以在Github上查看修改后完整的代码
要将数据获取移到组件渲染之前，我们需要用到 `react-transmit`库

```Command Line
npm install react-transmit --save
```

React Transmit优雅地包装组件(通常称之后“高阶组件”)，让我们在客户端与服务器端获取数据。

这里是我们的组件被React Transmit改造后的代码：

```JS
import React from 'react';
import Butter from 'buttercms'
import Transmit from 'react-transmit';

const butter = Butter('b60a008584313ed21803780bc9208557b3b49fbb');

var Hello = React.createClass({
  render: function() {
    if (this.props.posts) {
      return (
        <div>
          {this.props.posts.data.map((post) => {
            return (
              <div key={post.slug}>{post.title}</div>
            )
          })}
        </div>
      );
    } else {
      return <div>Loading...</div>;
    }
  }
});

export default Transmit.createContainer(Hello, {
  // These must be set or else it would fail to render
  initialVariables: {},
  // Each fragment will be resolved into a prop
  fragments: {
    posts() {
      return butter.post.list().then((resp) => resp.data);
    }
  }
});
```
我们使用 `Transmit.createContainer` 将组件封装在获取数据的高阶组件中，并从组件周期方法中删除了获取数据的代码，因为不需要获取两次数据。

由于React Transmit将API获取到的数据通过props传给了组件，需要对render方法进行一些修改，使用props而不是state来拿到数据。
为了确保组件渲染完成之前获取到数据，我们导入Transmit,使用 `Transmit.renderToString` 方法代替 `ReactDOM.renderToString` 方法。

```JS
import express from 'express';
import fs from 'fs';
import path from 'path';
import React from 'react';
import ReactDOMServer from 'react-dom/server';
import Hello from './Hello.js';
import Transmit from 'react-transmit';

function handleRender(req, res) {
  Transmit.renderToString(Hello).then(({reactString, reactData}) => {
    fs.readFile('./index.html', 'utf8', function (err, data) {
      if (err) throw err;

      const document = data.replace(/<div id="app"><\/div>/, `<div id="app">${reactString}</div>`);
      const output = Transmit.injectIntoMarkup(document, reactData, ['/build/client.js']);

      res.send(document);
    });
  });
}

const app = express();

// Serve built files with static files middleware
app.use('/build', express.static(path.join(__dirname, 'build')));

// Serve requests with our handleRender function
app.get('*', handleRender);

// Start server
app.listen(3000);
```
重启服务并浏览[http://localhost:3000](http://localhost:3000)。查看页面源代码，你将看到服务器端返回了包含数据的代码！
![](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_1000,f_auto,q_auto/v1497358548/rendered-react_t5neam.png)

## 更进一步
完成了！在服务器端使用React可能有些复杂，特别是数据需要从一些API接口中获得时尤其如此。幸运的是，React社区欣欣向荣，创造了许多有用的工具。如果您对构建大型React应用程序的框架感兴趣，想在客户机和服务器上呈现，可以看看Walmart实验室的[Electrode](https://github.com/electrode-io/electrode)或者[Next.js](https://github.com/zeit/next.js)。如果你想用Ruby渲染React,可以考虑一下Airbnb的[Hypernova](https://github.com/airbnb/hypernova)。