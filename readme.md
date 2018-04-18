<img width="112" alt="screen shot 2016-10-25 at 2 37 27 pm" src="https://cloud.githubusercontent.com/assets/13041/19686250/971bf7f8-9ac0-11e6-975c-188defd82df1.png">

[![NPM version](https://img.shields.io/npm/v/next.svg)](https://www.npmjs.com/package/next)
[![Build Status](https://travis-ci.org/zeit/next.js.svg?branch=master)](https://travis-ci.org/zeit/next.js)
[![Build status](https://ci.appveyor.com/api/projects/status/gqp5hs71l3ebtx1r/branch/master?svg=true)](https://ci.appveyor.com/project/arunoda/next-js/branch/master)
[![Coverage Status](https://coveralls.io/repos/zeit/next.js/badge.svg?branch=master)](https://coveralls.io/r/zeit/next.js?branch=master)
[![Join the community on Spectrum](https://withspectrum.github.io/badge/badge.svg)](https://spectrum.chat/next-js)

NextJS是一个极简的、为React应用提供服务端渲染的框架
**访问 [learnnextjs.com](https://learnnextjs.com) 来开始使用NextJS.**

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
<!-- https://github.com/thlorenz/doctoc -->

- [如何使用](#如何使用)
  - [安装](#安装)
  - [代码自动分割](#代码自动分割)
  - [CSS](#css)
    - [内置 CSS 支持](#内置 CSS 支持)
    - [CSS-in-JS方案](#CSS-in-JS方案)
  - [静态文件服务 (如：图片)](#静态文件服务 (如：图片))
  - [自定义`<head>`](#自定义`<head>`)
  - [数据获取与组件生命周期](#数据获取与组件生命周期)
  - [路由](#路由)
    - [使用`<Link>`组件](#使用`<Link>`组件)
    - [命令式调用](#命令式调用)
      - [路由事件](#路由事件)
      - [浅路由](#浅路由)
    - [高阶组件用法](#高阶组件用法)
  - [页面预加载](#页面预加载)
    - [使用`<Link>`组件](#with-link-1)
    - [命令式调用](#imperatively-1)
  - [自定义服务器与路由](#custom-server-and-routing)
  - [动态导入](#dynamic-import)
  - [自定义 `<Document>`](#custom-document)
  - [自定义错误处理 error handling](#custom-error-handling)
  - [自定义 配置](#custom-configuration)
  - [自定义webpack 配置](#customizing-webpack-config)
  - [自定义babel配置](#customizing-babel-config)
  - [为资源前缀添加CDN支持](#cdn-support-with-asset-prefix)
- [生产部署](#production-deployment)
- [静态 HTML导出](#static-html-export)
- [多分区](#multi-zones)
- [方案](#recipes)
- [FAQ](#faq)
- [贡献](#contributing)
- [作者](#authors)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 如何使用

### 安装

开始安装:

```bash
npm install --save next react react-dom
```

> NextJS 4 目前只支持 [React 16](https://reactjs.org/blog/2017/09/26/react-v16.0.html).<br/>
> 由于React16的工作与使用方式和React15有所差异，我们不得不放弃了对React15的支持。

在你的package.json中添加script：

```json
{
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
```

接下来,主要工作都由文件系统处理. 每个 `.js` 文件被当作自动处理和渲染的一个路由.

在项目下创建 `./pages/index.js`:

```jsx
export default () => <div>Welcome to next.js!</div>
```

之后只需运行 `npm run dev`命令，并访问 `http://localhost:3000`即可。 如果需要指定端口号, 你可以运行`npm run dev -- -p <your port here>`。

目前为止, 我们实现了:

- 代码自动化转换与绑定 (基于webpack和babel)
- 代码热更新
- 基于`./pages`目录的服务端渲染
- 静态文件服务。`./static/`会被映射到`/static/`

想了解这一切是多么简单, 请查看[范例应用 - nextgram](https://github.com/zeit/nextgram)

### 代码自动分割

每个 `import` 声明会被绑定并服务于每个页面。这意味着页面永远不会加载多余的代码!

```jsx
import cowsay from 'cowsay-browser'

export default () =>
  <pre>
    {cowsay.say({ text: 'hi there!' })}
  </pre>
```

### CSS

#### 内置CSS支持

<p><details>
  <summary><b>Examples</b></summary>
  <ul><li><a href="./examples/basic-css">Basic css</a></li></ul>
</details></p>

NextJS内置了[styled-jsx](https://github.com/zeit/styled-jsx)来提供对隔离作用域CSS的支持。 目的是提供类似web组件的`shadow CSS`, 遗憾的是[`shadow CSS`不支持服务器渲染，它只支持js](https://github.com/w3c/webcomponents/issues/71).

```jsx
export default () =>
  <div>
    Hello world
    <p>scoped!</p>
    <style jsx>{`
      p {
        color: blue;
      }
      div {
        background: red;
      }
      @media (max-width: 600px) {
        div {
          background: blue;
        }
      }
    `}</style>
    <style global jsx>{`
      body {
        background: black;
      }
    `}</style>
  </div>
```

更多示例请参见[styled-jsx文档](https://www.npmjs.com/package/styled-jsx) 

#### CSS-in-JS方案

<p><details>
  <summary>
    <b>Examples</b>
    </summary>
  <ul><li><a href="./examples/with-styled-components">Styled components</a></li><li><a href="./examples/with-styletron">Styletron</a></li><li><a href="./examples/with-glamor">Glamor</a></li><li><a href="./examples/with-glamorous">Glamorous</a></li><li><a href="./examples/with-cxs">Cxs</a></li><li><a href="./examples/with-aphrodite">Aphrodite</a></li><li><a href="./examples/with-fela">Fela</a></li></ul>
</details></p>

可以使用任何CSS-in-JS解决方案。最简单的是行内样式:

```jsx
export default () => <p style={{ color: 'red' }}>hi there</p>
```

为了使用更复杂的CSS-in-JS方案,你通常需要实现强制刷新服务端渲染内容的样式。我们允许你使用[自定义的`<Document>`组件](#user-content-custom-document)来包装每个页面 。

#### 导入 CSS / Sass / Less 文件

你可以使用下面的模块来支持 `.css` `.scss` or `.less` 文件的导入, 它们针对服务端渲染内置了常规的默认配置.

- [@zeit/next-css](https://github.com/zeit/next-plugins/tree/master/packages/next-css)
- [@zeit/next-sass](https://github.com/zeit/next-plugins/tree/master/packages/next-sass)
- [@zeit/next-less](https://github.com/zeit/next-plugins/tree/master/packages/next-less)

### 静态文件服务 (如：图片)

在你项目的根节点下创建`static`文件夹。你就可以在你的代码中，以`/static/`开头的url来引用其下的文件：

```jsx
export default () => <img src="/static/my-image.png" />
```

### 自定义`<head>`

<p><details>
  <summary><b>Examples</b></summary>
  <ul>
    <li><a href="./examples/head-elements">Head elements</a></li>
    <li><a href="./examples/layout-component">Layout component</a></li>
  </ul>
</details></p>

Next.js暴露了一个内置组件（built-in component），该组件下的内容最终会被放置在页面的`<head>`标签下.

```jsx
import Head from 'next/head'

export default () =>
  <div>
    <Head>
      <title>My page title</title>
      <meta name="viewport" content="initial-scale=1.0, width=device-width" />
    </Head>
    <p>Hello world!</p>
  </div>
```

你可以使用`key`属性来避免`<head>`中重复的标签，这会确保标签只被渲染一次:

```jsx
import Head from 'next/head'
export default () => (
  <div>
    <Head>
      <title>My page title</title>
      <meta name="viewport" content="initial-scale=1.0, width=device-width" key="viewport" />
    </Head>
    <Head>
      <meta name="viewport" content="initial-scale=1.2, width=device-width" key="viewport" />
    </Head>
    <p>Hello world!</p>
  </div>
)
```

在这个例子中，只有第2个`<meta name="viewport" />`标签生效.

_注意: 在组件被卸载时，会清除`<head>`的内容，因此要确保每个页面`<head>`中的定义的内容是完整的，而不要假定其它页面已经定义过了_

###  数据获取与组件生命周期

<p><details>
  <summary><b>Examples</b></summary>
  <ul><li><a href="./examples/data-fetch">Data fetch</a></li></ul>
</details></p>

你可以导出一个继承了`React.Component`的组件 (而不是象上面这种无状态组件)，来完成状态管理，处理生命周期钩子或者**数据初始化(initial data population)**等操作：

```jsx
import React from 'react'

export default class extends React.Component {
  static async getInitialProps({ req }) {
    const userAgent = req ? req.headers['user-agent'] : navigator.userAgent
    return { userAgent }
  }

  render() {
    return (
      <div>
        Hello World {this.props.userAgent}
      </div>
    )
  }
}
```

请注意，当页面初始化时加载数据, 我们使用`getInitialProps`，这是一个[`异步(async)`](https://zeit.co/blog/async-and-await)的静态方法。它可以异步地获取任何可以被解析为JavaScript对象的数据，并将数据填充进`props`对象中.

在服务端渲染的场景下，`getInitialProps`返回的数据会被序列化，序列化过程类似于`JSON.stringify`。因此要确保`getInitialProps`中返回的对象是Object类型的，不要使用`Date`, `Map`或`Set`类型。

初始页面加载时, 将在服务端执行`getInitialProps`。只有在通过`Link`组件，或者使用命令式API来导航到另外的路由时，才会在客户端上执行“getInitialProps”。

_注意： 子级组件**不能**使用`getInitialProps`。`getInitialProps`只会作用于`pages`目录中的顶层页面组件。_

<br/>

> 如果你在`getInitialProps`里面引入了一些服务器模块时，请确保 [正确地导入它们](https://arunoda.me/blog/ssr-and-server-only-modules).
> 否则,它会拖慢你的应用。

<br/>

您还可以为无状态组件定义`getInitialProps`生命周期方法:

```jsx
const Page = ({ stars }) =>
  <div>
    Next stars: {stars}
  </div>

Page.getInitialProps = async ({ req }) => {
  const res = await fetch('https://api.github.com/repos/zeit/next.js')
  const json = await res.json()
  return { stars: json.stargazers_count }
}

export default Page
```

`getInitialProps`接收一个具有以下属性的上下文对象：

- `pathname` - URL的路径(path)部分
- `query` - URL中的查询字符串经转换后的对象
- `asPath` - 浏览器中显示的实际路径（包括查询）的`字符串`
- `req` - HTTP请求对象(服务器环境)
- `res` - HTTP响应对象(服务器环境)
- `jsonPageRes` - [Fetch的响应对象](https://developer.mozilla.org/en-US/docs/Web/API/Response)(客户端环境)
- `err` - 呈现过程报错抛出的错误对象

### 路由

#### `<Link>`

<p><details>
  <summary><b>Examples</b></summary>
  <ul>
    <li><a href="./examples/hello-world">Hello World</a></li>
  </ul>
</details></p>

客户端可以使用`<Link>`组件实现路由之间的跳转。 例如下面的2个页面：

```jsx
// pages/index.js
import Link from 'next/link'

export default () =>
  <div>
    Click{' '}
    <Link href="/about">
      <a>here</a>
    </Link>{' '}
    to read more
  </div>
```

```jsx
// pages/about.js
export default () => <p>Welcome to About!</p>
```

__注意:  [`<Link prefetch>`](#prefetching-pages)可以在后台同时预加载页面和数据，从而提高应用的加载性能__

客户端路由行为与浏览器完全一样：

1. 获取组件
2. 如果定义了`getInitialProps`, 就会获取数据。如果报错，则呈现`_error.js`
3. 完成1和2之后, 将执行`pushState`并渲染新组件

每个顶层组件都会接收到`url`属性，它包含以下API：

- `pathname` - `String`类型，不包含查询字符串的路径
- `query` - `Object`类型，解析后的查询字符串。默认值为`{}`
- `asPath` - `String`类型 浏览器中展示的路径(包含查询字符串)
- `push(url, as=url)` - 以指定url调用`pushState`
- `replace(url, as=url)` - 以指定url调用`replaceState`

`push`和`replace`的第2个`as`参数是URL_可选_的。它作用于你在服务器上配置的自定义路由.

##### URL对象

<p><details>
  <summary><b>Examples</b></summary>
  <ul>
    <li><a href="./examples/with-url-object-routing">With URL Object Routing</a></li>
  </ul>
</details></p>

`<Link>`组件可以接收一个URL对象，它会自动格式化为URL字符串。

```jsx
// pages/index.js
import Link from 'next/link'

export default () =>
  <div>
    Click{' '}
    <Link href={{ pathname: '/about', query: { name: 'Zeit' } }}>
      <a>here</a>
    </Link>{' '}
    to read more
  </div>
```

上面的例子生成了URL字符串`/about?name=Zeit`, 你可以参考[Node.js URL模块的文档](https://nodejs.org/api/url.html#url_url_strings_and_url_objects)中支持的所有属性。

##### 如何`替换(replace)`而非`追加(push)`url

`<Link>`组件默认会把新的url`追加`到路由栈中。您可以使用`replace`方法来执行`替换`而不是`追加`动作。

```jsx
// pages/index.js
import Link from 'next/link'

export default () =>
  <div>
    Click{' '}
    <Link href="/about" replace>
      <a>here</a>
    </Link>{' '}
    to read more
  </div>
```

##### 让组件支持`onClick`事件

`<Link>`支持所有支持`onClick`事件的组件。如果你没有为`<Link>`提供`<a>`标签，它只响应`onclick`事件，而会忽略`href`属性，这样会导致页面间导航失效。

```jsx
// pages/index.js
import Link from 'next/link'

export default () =>
  <div>
    Click{' '}
    <Link href="/about">
      <img src="/static/image.png" />
    </Link>
  </div>
```

##### 将Link的`href`暴露给它的子级

 当子级是一个`<a>`标签，并且没有定义href属性时，我们会自动为子级定义href(与Link的href一致)来省去用户重复定义的工作。 然而在特殊情况下，当你需要在`<a>`标签外包裹一层容器时，因为这层容器并不会被`Link`识别为是一个*超链接*，所以不会将它的`href`传递给子级。 此时，你应该在`Link`上定义一个布尔型`passHref`属性，该属性的作用是强制将`href`属性暴露给子级。

**请注意**：使用除`a`之外的标签，并且没有传递`passHref`，可能会让链接看起来可以成功导航，但是当被搜索引擎抓取时，不会被识别为链接（由于缺乏`href`属性）。 这可能会影响网站的搜索引擎优化(SEO)。


```jsx
import Link from 'next/link'
import Unexpected_A from 'third-library'

export default ({ href, name }) =>
  <Link href={href} passHref>
    <Unexpected_A>
      {name}
    </Unexpected_A>
  </Link>
```

##### 禁止滚动到页面顶部

默认情况下，点击`<Link>`组件会使页面滚动到顶部。并且，当`<Link>`组件定义了的hash属性时，它会像普通的`<a>`标签一样滚动到特定的ID。为了避免滚动到顶部或特定hash的动作发生，可以在`<Link>`上添加`scroll = {false}`：

```jsx
<Link scroll={false} href="/?counter=10"><a>Disables scrolling</a></Link>
<Link href="/?counter=10"><a>Changes with scrolling to top</a></Link>
```

#### 命令式调用

<p><details>
  <summary><b>Examples</b></summary>
  <ul>
    <li><a href="./examples/using-router">Basic routing</a></li>
    <li><a href="./examples/with-loading">With a page loading indicator</a></li>
  </ul>
</details></p>

在客户端你也可以使用`next/router`来实现页面跳转

```jsx
import Router from 'next/router'

export default () =>
  <div>
    Click <span onClick={() => Router.push('/about')}>here</span> to read more
  </div>
```

####拦截 popstate
In some cases (for example, if using a custom router), you may wish to listen to popstate and react before the router acts on it. For example, you could use this to manipulate the request, or force an SSR refresh.
在某些情况下(例如，使用[自定义的路由](#custom-server-and-routing))，您可能希望侦听popstate，用以在路由跳转之前做一些处理。例如，您可能需要操作请求，或强制SSR刷新。


```jsx
import Router from 'next/router'

Router.beforePopState(({ url, as, options }) => {
  // I only want to allow these two routes!
  if (as !== "/" || as !== "/other") {
    // Have SSR render bad routes as a 404.
    window.location.href = as
    return false
  }

  return true
});
```
If you return a falsy value from beforePopState, Router will not handle popstate; you'll be responsible for handling it, in that case. See Disabling File-System Routing.
当你在beforePopState函数中返回的是非true的值时，路由将不触发popstate；这种情形下，你需要手动处理。参考[Disabling File-System Routing](#disabling-file-system-routing)


上面的`Router`对象拥有如下API:

- `route` - `String`类型 当前路由节点
- `pathname` - `String`类型 不包含查询字符串的当前路径
- `query` - `Object`类型 由查询字符串转译后的对象。默认值是`{}`
- `asPath` - `String`类型 浏览器中展示的路径(包含查询字符串)
- `push(url, as=url)` - 以指定url调用`pushState`
- `replace(url, as=url)` - 以指定url调用`replaceState`

`push`和`replace`的第2个`as`参数提供了额外的配置项。它作用于你在服务器端配置的自定义路由.

_注意：在组件内使用`props.url.push` 和 `props.url.replace`，可以以编程的方式，在不触发导航及组件获取动作的前提下更改路由。_

##### URL对象
Router的URL对象与`<Link>`组件的URL对象一致，都可以用于`push`和`replace`函数。

```jsx
import Router from 'next/router'

const handler = () =>
  Router.push({
    pathname: '/about',
    query: { name: 'Zeit' }
  })

export default () =>
  <div>
    Click <span onClick={handler}>here</span> to read more
  </div>
```

具体的参数与`<Link>`组件也是一样的。

##### 路由事件

Router还提供了相关的监听事件。
以下是支持的事件列表：

- `onRouteChangeStart(url)` - 当路由开始发生改变时触发
- `onRouteChangeComplete(url)` - 当路由更改完成时触发
- `onRouteChangeError(err, url)` - 当路由在改变过程中报错时触发
- `onBeforeHistoryChange(url)` - 在浏览器的历史记录发生改变之前触发

> 这里的`url`参数指的是浏览器的链接地址。当你使用`Router.push(url, as)` (或类似方法)时，`as`的值此时就是`url`。

下面是如何正确监听路由的`onRouteChangeStart`事件的示例：

```js
Router.onRouteChangeStart = url => {
  console.log('App is changing to: ', url)
}
```

当不再需要监听该事件时，你可以象这样很方便地取消监听：

```js
Router.onRouteChangeStart = null
```

如果路由的加载中断了(比如快速地连续点击了2个链接), `routeChangeError`会被触发。该函数会传递一个`err`对象，这个对象上的`cancelled`属性值为`true`。

```js
Router.onRouteChangeError = (err, url) => {
  if (err.cancelled) {
    console.log(`Route to ${url} was cancelled!`)
  }
}
```

##### 浅路由

<p><details>
  <summary><b>Examples</b></summary>
  <ul>
    <li><a href="./examples/with-shallow-routing">Shallow Routing</a></li>
  </ul>
</details></p>

浅路由允许你在不触发`getInitialProps`的情况下改变URL。你可以在不丢失状态的情况下，在同一页面重新加载后，在`url`属性上取得新的`pathname`和`query`的值。

在调用`Router.push` 或 `Router.replace`时，你可以在option参数中添加`shallow: true`来使浅路由生效。下面是一个示例：

```js
// Current URL is "/"
const href = '/?counter=10'
const as = href
Router.push(href, as, { shallow: true })
```

Now, the URL is updated to `/?counter=10`. You can see the updated URL with `this.props.url` inside the `Component`.
现在，URL被更新为`/?counter=10`。你可以在组件内使用`this.props.url`来查看更新后的URL的值。

你可以使用[`componentWillReceiveProps`](https://facebook.github.io/react/docs/react-component.html#componentwillreceiveprops)钩子函数来监听URL的更改:

```js
componentWillReceiveProps(nextProps) {
  const { pathname, query } = nextProps.url
  // fetch data based on the new query
}
```

> 小结:
>
> Shallow routing works 浅路由**只能**处理同一页面下的URL更改。举例来说, 假定我们有一个`about`页面,然后执行下面的代码：
> ```js
> Router.push('/?counter=10', '/about?counter=10', { shallow: true })
> ```
> 由于上面操作导航到了一个新的页面，所以即使我们声明了浅路由，当前依然页面会被卸载，之后重新加载并调用`getInitialProps`函数。

#### 使用高阶组件(HOC Higher Order Component)

<p><details>
  <summary><b>Examples</b></summary>
  <ul>
    <li><a href="./examples/using-with-router">Using the `withRouter` utility</a></li>
  </ul>
</details></p>

If you want to access the `router` object inside any component in your app, you can use the `withRouter` Higher-Order Component. Here's how to use it:
你可以使用`withRouter`高阶组件，来让应用当中的任意组件内可以访问到`router`对象。下面是使用示例:

```jsx
import { withRouter } from 'next/router'

const ActiveLink = ({ children, router, href }) => {
  const style = {
    marginRight: 10,
    color: router.pathname === href? 'red' : 'black'
  }

  const handleClick = (e) => {
    e.preventDefault()
    router.push(href)
  }

  return (
    <a href={href} onClick={handleClick} style={style}>
      {children}
    </a>
  )
}

export default withRouter(ActiveLink)
```

上面的`router`对象有一个类似于[`next/router`](#imperatively)的API。

### 页面预加载

⚠️ 该特性只面向生产环境 ⚠️

<p><details>
  <summary><b>Examples</b></summary>
  <ul><li><a href="./examples/with-prefetching">Prefetching</a></li></ul>
</details></p>

Next.js提供了一个API来允许你实现页面的预加载。

由于NextJS在服务端渲染了页面，所以应用会提前准备好涉及到的所有关联路径，以实现即时交互的目的。事实上NextJS已经完成了一部分初始化工作，来帮助你的_web站点(website)_达到最佳的下载性能，并提供了_应用(app)_提前下载的功能。[阅读更多](https://zeit.co/blog/next#anticipation-is-the-key-to-performance)


> 由于NextJS只会预加载JS代码。所以当页面返回之后，你可能需要等待数据部分的加载。

<span id="with-link-1">here</span>
#### 使用`<Link>`组件

你可以在任意`<Link>`中添加`prefetch`属性，来让NextJS在后台预先下载对应的页面。

```jsx
import Link from 'next/link'

// example header component
export default () =>
  <nav>
    <ul>
      <li>
        <Link prefetch href="/">
          <a>Home</a>
        </Link>
      </li>
      <li>
        <Link prefetch href="/about">
          <a>About</a>
        </Link>
      </li>
      <li>
        <Link prefetch href="/contact">
          <a>Contact</a>
        </Link>
      </li>
    </ul>
  </nav>
```

#### 命令式调用

通过`<Link />`可以满足大多数预加载需求，同时我们也为高级用法开放了一个命令式调用的API：

```jsx
import Router from 'next/router'

export default ({ url }) =>
  <div>
    <a onClick={() => setTimeout(() => url.pushTo('/dynamic'), 100)}>
      A route transition will happen after 100ms
    </a>
    {// but we can prefetch it!
    Router.prefetch('/dynamic')}
  </div>
```

### 自定义服务器与路由

<p><details>
  <summary><b>Examples</b></summary>
  <ul>
    <li><a href="./examples/custom-server">Basic custom server</a></li>
    <li><a href="./examples/custom-server-express">Express integration</a></li>
    <li><a href="./examples/custom-server-hapi">Hapi integration</a></li>
    <li><a href="./examples/custom-server-koa">Koa integration</a></li>
    <li><a href="./examples/parameterized-routing">Parameterized routing</a></li>
    <li><a href="./examples/ssr-caching">SSR caching</a></li>
  </ul>
</details></p>

正常情况下你可以使用`next start`来启动next服务。但是，为了自定义路由，使用路由匹配表达式等，你也完全可以以编辑方式启动服务。

当使用服务器文件（如：`server.js`）来自定义服务的配置时，记得更新`package.json`中的脚本：

```json
{
  "scripts": {
    "dev": "node server.js",
    "build": "next build",
    "start": "NODE_ENV=production node server.js"
  }
}
```

下面的示例将`/a`的请求解析为`./pages/b`的内容，将`/b`的请求解析为`./pages/a`的内容：

```js
// This file doesn't go through babel or webpack transformation.
// Make sure the syntax and sources this file requires are compatible with the current node version you are running
// See https://github.com/zeit/next.js/issues/1245 for discussions on Universal Webpack or universal Babel
const { createServer } = require('http')
const { parse } = require('url')
const next = require('next')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  createServer((req, res) => {
    // Be sure to pass `true` as the second argument to `url.parse`.
    // This tells it to parse the query portion of the URL.
    const parsedUrl = parse(req.url, true)
    const { pathname, query } = parsedUrl

    if (pathname === '/a') {
      app.render(req, res, '/b', query)
    } else if (pathname === '/b') {
      app.render(req, res, '/a', query)
    } else {
      handle(req, res, parsedUrl)
    }
  }).listen(3000, err => {
    if (err) throw err
    console.log('> Ready on http://localhost:3000')
  })
})
```


`next` 的API如下:
- `next(opts: object)`

options支持的参数有:
- `dev` (`bool`) 是否在dev模式下运行NextJS - 默认是 `false`
- `dir` (`string`) Next 项目的路径 - 默认是`'.'`
- `quiet` (`bool`) 是否隐藏包含服务端信息在内的报错信息 - 默认是 `false`
- `conf` (`object`) 与`next.config.js`结构一致的配置对象 - 默认是 `{}`

然后，更改你的`start` 脚本为`NODE_ENV=production node server.js`。

#### 如何禁用文件系统路由 Disabling file-system routing

默认情况下，`Next`会根据`/pages`文件夹下的每个文件，将文件名映射为访问路径来部署服务（例如，`/pages/some-file.js`对应的访问路径就是`site.com/some-file`）。

当你的项目实现了自定义路由时，可能会导致多个访问路径返回了相同的内容，从而对搜索引擎优化（SEO）和用户体验（UX）带来了一些隐患。

为了禁止这种行为，并防止基于`/pages`的路由，只需要在你的`next.config.js`中配置如下选项：

```js
// next.config.js
module.exports = {
  useFileSystemPublicRoutes: false
}
```

#### 动态资源前缀

有些时候，我们需要动态地指定`站点资源的前缀`(assetPrefix)。根据传入的请求来更改`资源的前缀`特别有用。我们可以使用`app.setAssetPrefix`来实现。

下面的示例演示了如何使用：

```js
const next = require('next')
const micro = require('micro')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  const server = micro((req, res) => {
    // Add assetPrefix support based on the hostname
    if (req.headers.host === 'my-app.com') {
      app.setAssetPrefix('http://cdn.com/myapp')
    } else {
      app.setAssetPrefix('')
    }

    handleNextRequests(req, res)
  })

  server.listen(port, (err) => {
    if (err) {
      throw err
    }

    console.log(`> Ready on http://localhost:${port}`)
  })
})

```

### 动态导入

<p><details>
  <summary><b>Examples</b></summary>
  <ul>
    <li><a href="./examples/with-dynamic-import">With Dynamic Import</a></li>
  </ul>
</details></p>


NextJS支持JavasScript TC39 的[动态引用提案（dynamic import proposal）](https://github.com/tc39/proposal-dynamic-import)。
这样，你就可以动态地导入JavaScript模块(例如：React Components)并使用它们。

你可以将动态导入视作实现代码分割为可管理的块的另一种实现方式。
由于 NextJS 支持服务端渲染的动态导入，你可以利用这一点做一些特别的事情。

以下是使用动态导入的几种方法：

#### 1. 基础用法(同样作用于SSR)

```jsx
import dynamic from 'next/dynamic'

const DynamicComponent = dynamic(import('../components/hello'))

export default () =>
  <div>
    <Header />
    <DynamicComponent />
    <p>HOME PAGE is here!</p>
  </div>
```

#### 2. 自定义加载组件

```jsx
import dynamic from 'next/dynamic'

const DynamicComponentWithCustomLoading = dynamic(
  import('../components/hello2'),
  {
    loading: () => <p>...</p>
  }
)

export default () =>
  <div>
    <Header />
    <DynamicComponentWithCustomLoading />
    <p>HOME PAGE is here!</p>
  </div>
```

#### 3. 禁止SSR

```jsx
import dynamic from 'next/dynamic'

const DynamicComponentWithNoSSR = dynamic(import('../components/hello3'), {
  ssr: false
})

export default () =>
  <div>
    <Header />
    <DynamicComponentWithNoSSR />
    <p>HOME PAGE is here!</p>
  </div>
```

#### 4. 一次性加载多个模块

```jsx
import dynamic from 'next/dynamic'

const HelloBundle = dynamic({
  modules: props => {
    const components = {
      Hello1: import('../components/hello1'),
      Hello2: import('../components/hello2')
    }

    // Add remove components based on props

    return components
  },
  render: (props, { Hello1, Hello2 }) =>
    <div>
      <h1>
        {props.title}
      </h1>
      <Hello1 />
      <Hello2 />
    </div>
})

export default () => <HelloBundle title="Dynamic Bundle" />
```

### 自定义`<Document>`

<p><details>
  <summary><b>Examples</b></summary>
  <ul><li><a href="./examples/with-styled-components">Styled components custom document</a></li></ul>
  <ul><li><a href="./examples/with-amp">Google AMP</a></li></ul>
</details></p>

`NextJS`跳过了最外围文档标记的定义。比如，你不需要包含类似`<html>`, `<body>`这类标签。如果要覆盖默认行为，你需要创建一个`./pages/_document.js`文件，你可以在该文件中继承 `Document` 类并实现它：

```jsx
// ./pages/_document.js
import Document, { Head, Main, NextScript } from 'next/document'
import flush from 'styled-jsx/server'

export default class MyDocument extends Document {
  static getInitialProps({ renderPage }) {
    const { html, head, errorHtml, chunks } = renderPage()
    const styles = flush()
    return { html, head, errorHtml, chunks, styles }
  }

  render() {
    return (
      <html>
        <Head>
          <style>{`body { margin: 0 } /* custom! */`}</style>
        </Head>
        <body className="custom_class">
          {this.props.customValue}
          <Main />
          <NextScript />
        </body>
      </html>
    )
  }
}
```

`ctx`对象相当于[`getInitialProps`](#fetching-data-and-component-lifecycle)中接收到的参数，有一个额外的补充

- `renderPage` (`Function`) 是真正执行React渲染逻辑的回调函数 (同步地)。它用来修饰服务端渲染，来支持象 Aphrodite 的 [`renderStatic`](https://github.com/Khan/aphrodite#server-side-rendering)。

__注意：React组件`<Main />`以外部分不会被浏览器初始化。所以如果你需要在所有页面共用同一个组件(象菜单或工具条组件)，_不要_在这里添加应用的逻辑代码，然后可以看一下[这个示例](https://github.com/zeit/next.js/tree/master/examples/layout-component).__

### 自定义错误处理

404 or 500 errors are handled both client and server side by a default component `error.js`. If you wish to override it, define a `_error.js` in the pages folder:
客户端和服务器端的404或500错误默认由`error.js`中的组件处理。你可以在pages文件夹下定义`_error.js`来重写处理逻辑：

```jsx
import React from 'react'

export default class Error extends React.Component {
  static getInitialProps({ res, err }) {
    const statusCode = res ? res.statusCode : err ? err.statusCode : null;
    return { statusCode }
  }

  render() {
    return (
      <p>
        {this.props.statusCode
          ? `An error ${this.props.statusCode} occurred on server`
          : 'An error occurred on client'}
      </p>
    )
  }
}
```

### 复用内置的错误页面

你可以使用`next/error`来渲染一个内置的错误页面：

```jsx
import React from 'react'
import Error from 'next/error'
import fetch from 'isomorphic-unfetch'

export default class Page extends React.Component {
  static async getInitialProps() {
    const res = await fetch('https://api.github.com/repos/zeit/next.js')
    const statusCode = res.statusCode > 200 ? res.statusCode : false
    const json = await res.json()

    return { statusCode, stars: json.stargazers_count }
  }

  render() {
    if (this.props.statusCode) {
      return <Error statusCode={this.props.statusCode} />
    }

    return (
      <div>
        Next stars: {this.props.stars}
      </div>
    )
  }
}
```

>  如果你已经创建了一个自定义的错误页面，你必须导入自己的`_error`组件，而不是 `next/error`。

### 自定义配置

实现NextJS更深层次的自定义，你只需要在项目的根目录创建一个`next.config.js`文件即可(与`pages/` 和 `package.json`同级)

注意：`next.config.js` 是一个标准的Node.js模块,而不是一个JSON文件。该文件中用于NextJS服务端及项目构建阶段的配置，在浏览器构建阶段并不生效。

```js
// next.config.js
module.exports = {
  /* config options here */
}
```

或者以函数的形式：

```js
module.exports = (phase, {defaultConfig}){
  //
  // https://github.com/zeit/
  return {
    /* config options here */
  }
}
```

上面函数中的`phase`参数指向配置加载后的上下文对象。你可以在这查阅所有的phase： [Phases常量](./lib/constants.js) 可以从 `next/constants` 中导入：

```js
const {PHASE_DEVELOPMENT_SERVER} = require('next/constants')
module.exports = (phase, {defaultConfig}){
  if(phase === PHASE_DEVELOPMENT_SERVER) {
    return {
      /* development only config options here */
    }
  }

  return {
    /* config options for all phases except development here */
  }
}
```

#### 自定义构建目录

你可以指定用于构建的目录。例如：下面的配置将创建`build`文件夹以替代`.next`文件夹。如果没有指定配置，NextJS默认会创建`.next`文件夹。

```js
// next.config.js
module.exports = {
  distDir: 'build'
}
```

#### 禁用ETag生成

您可以根据缓存策略禁用HTML页面的ETag生成。 如果没有指定配置，则Next默认会为每个页面生成etags。

```js
// next.config.js
module.exports = {
  generateEtags: false
}
```

#### 配置onDemandEntries

Next 开放了一些配置项，可以让你控制服务器的部署，以及构建页面的缓存：

```js
module.exports = {
  onDemandEntries: {
    // period (in ms) where the server will keep pages in the buffer
    maxInactiveAge: 25 * 1000,
    // number of pages that should be kept simultaneously without being disposed
    pagesBufferLength: 2,
  }
}
```

上面的配置只适用于开发环境。如果你希望在生产环境下缓存SSR页面，可以查看[SSR-caching](https://github.com/zeit/next.js/tree/canary/examples/ssr-caching) 示例。

#### 配置从`pages`下查找并解析页面时，可支持的扩展名

对于像 [`@zeit/next-typescript`](https://github.com/zeit/next-plugins/tree/master/packages/next-typescript) 这样的模块, 它添加了针对`.ts`后缀的支持。 `pageExtensions` 的作用是，当在`pages`目录寻找并解析页面时，配置可识别的文件扩展名。 

```js
// next.config.js
module.exports = {
  pageExtensions: ['jsx', 'js']
}
```

### 自定义Webpack配置

<p><details>
  <summary><b>Examples</b></summary>
  <ul><li><a href="./examples/with-webpack-bundle-analyzer">Custom webpack bundle analyzer</a></li></ul>
</details></p>

你可以通过在 `next.config.js` 中定义一个函数来扩展`webpack`

```js
// This file is not going through babel transformation.
// So, we write it in vanilla JS
// (But you could use ES2015 features supported by your Node.js version)

module.exports = {
  webpack: (config, { buildId, dev, isServer, defaultLoaders }) => {
    // Perform customizations to webpack config

    // Important: return the modified config
    return config
  },
  webpackDevMiddleware: config => {
    // Perform customizations to webpack dev middleware config

    // Important: return the modified config
    return config
  }
}
```

一些常见的功能，是作为模块提供的：

- [@zeit/next-css](https://github.com/zeit/next-plugins/tree/master/packages/next-css)
- [@zeit/next-sass](https://github.com/zeit/next-plugins/tree/master/packages/next-sass)
- [@zeit/next-less](https://github.com/zeit/next-plugins/tree/master/packages/next-less)
- [@zeit/next-preact](https://github.com/zeit/next-plugins/tree/master/packages/next-preact)
- [@zeit/next-typescript](https://github.com/zeit/next-plugins/tree/master/packages/next-typescript)

*警告：`webpack` 函数会被执行两次，服务端与客户端各一次。你可以使用`isServer`属性来区分服务端还是客户端。*

多种配置可以用函数结构组合在一起。例如：

```js
const withTypescript = require('@zeit/next-typescript')
const withSass = require('@zeit/next-sass')

module.exports = withTypescript(withSass({
  webpack(config, options) {
    // Further custom configuration here
    return config
  }
}))
```


### 自定义Babel配置

<p><details>
  <summary><b>Examples</b></summary>
  <ul><li><a href="./examples/with-custom-babel-config">Custom babel configuration</a></li></ul>
</details></p>

你可以方便地在应用的根目录中定义一个`.babelrc`文件，来扩展`babel`的配置。这个文件是可选的。

如果文件存在，我们会将该文件作为Babel配置的解析*来源*，因此Next的相关依赖也需要定义进去，也就是需要将`next/babel`设定进preset中去。

这样设计是为了让您对我们对babel配置所做的修改不感到惊讶。

下面是`.babelrc`文件的示例:

```json
{
  "presets": ["next/babel"],
  "plugins": []
}
```

#### 向服务端与客户端开放配置

`config`键的作用是暴露应用程序的运行时配置。默认情况下，所有的键只在服务端可见。要将配置公开给服务器和客户端，您可以使用`public`键。
The `config` key allows for exposing runtime configuration in your app. All keys are server only by default. To expose a configuration to both the server and client side you can use the `public` key.

```js
// next.config.js
module.exports = {
  serverRuntimeConfig: { // Will only be available on the server side
    mySecret: 'secret'
  },
  publicRuntimeConfig: { // Will be available on both server and client
    staticFolder: '/static'
  }
}
```

```js
// pages/index.js
import getConfig from 'next/config'
const {serverRuntimeConfig, publicRuntimeConfig} = getConfig()

console.log(serverRuntimeConfig.mySecret) // Will only be available on the server side
console.log(publicRuntimeConfig.staticFolder) // Will be available on both server and client

export default () => <div>
  <img src={`${publicRuntimeConfig.staticFolder}/logo.png`} />
</div>
```

### 使用CDN支持资源前缀

你可以设置`assetPrefix`来配置你的CDN源，从而使域名与NextJS托管的保持一致。

```js
const isProd = process.env.NODE_ENV === 'production'
module.exports = {
  // You may only need to add assetPrefix in the production.
  assetPrefix: isProd ? 'https://cdn.mydomain.com' : ''
}
```

注意： NextJS 会在script脚本加载时自动添加前缀，Nextjs将在其加载的脚本中自动使用该前缀，但对`/static`中的文件无效。如果你想通过CDN提供这些资源，你必须自己指定前缀。[在该示例中](https://github.com/zeit/next.js/tree/master/examples/with-universal-configuration)介绍了如何根据不同环境在组件内部引入前缀的方法。

## 生产与部署

在部署环节，需要提前为生产环境执行构建，而不是运行`next`命令。因此，构建与启动是两条单独的命令：
To deploy, instead of running `next`, you want to build for production usage ahead of time. Therefore, building and starting are separate commands:

```bash
next build
next start
```

例如, 象[`now`](https://zeit.co/now)这样部署一个package.json文件：

```json
{
  "name": "my-app",
  "dependencies": {
    "next": "latest"
  },
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
```

然后运行`now`项目查看执行结果。

NextJS也可以部署到其他托管解决方案。 请查看维基的['Deployment'](https://github.com/zeit/next.js/wiki/Deployment)部分。

注意：`NODE_ENV`需要在`next`命令执行时正确地配置，不配置的话，默认以最化大性能来部署。 如果你[以编程方式](#custom-server-and-routing)使用NextJS ，那么您需要手动设置`NODE_ENV = production`！

注意：我们建议你把`.next`或你的[自定义dist文件夹](https://github.com/zeit/next.js#custom-configuration)配置在`.npmignore`或`.gitignore`中。
或者，使用`files`或`now.files`来指定希望部署的文件的白名单，并从名单中排除掉`.next`或自定义dist目录。

## 导出静态HTML

<p><details>
  <summary><b>Examples</b></summary>
  <ul><li><a href="./examples/with-static-export">Static export</a></li></ul>
</details></p>

这是一种不使用Node.JS服务器，将你的NextJS应用作为独立的静态应用运行的方法。导出的应用几乎支持NextJS的所有特性 包括动态Urls，预获取，预加载和动态导入。

### 使用方法

象你通常使用NextJS一样简单地开发你的应用。之后创建一个如下的自定义NextJS[配置](https://github.com/zeit/next.js#custom-configuration)：

```js
// next.config.js
module.exports = {
  exportPathMap: function() {
    return {
      '/': { page: '/' },
      '/about': { page: '/about' },
      '/readme.md': { page: '/readme' },
      '/p/hello-nextjs': { page: '/post', query: { title: 'hello-nextjs' } },
      '/p/learn-nextjs': { page: '/post', query: { title: 'learn-nextjs' } },
      '/p/deploy-nextjs': { page: '/post', query: { title: 'deploy-nextjs' } }
    }
  }
}
```

> 注意如果路径以文件夹结尾，它将被导出为`/dir-name/index.html`，但是如果以扩展名结尾，它会被导出为指定的文件, 比如： 上面的`/readme.md`。如果你使用的文件扩展名不是`.html`，你需要将服务器返回内容的头`Content-Type`设定为`text/html`。


在这里，您需要指定需要导出为静态HTML的页面。

然后简单运行下面的命令：

```sh
next build
next export
```

为此，您可能需要像这样将一个NPM脚本添加到`package.json`中：

```json
{
  "scripts": {
    "build": "next build && next export"
  }
}
```

然后运行一次：

```sh
npm run build
```

之后在"out"文件夹中就会有一个应用的静态版本。

> 你也可以指定输出文件夹。运行 `next export -h` 来查看帮助文档。

现在你可以使用任何静态托管服务来部署该目录了。请注意，如果要将页面部署到GitHub，还需要多做一步，[详细文档](https://github.com/zeit/next.js/wiki/Deploying-a-NextJS-app-into-GitHub-Pages)。

举个例子，如果要部署你的应用到[ZEIT now](https://zeit.co/now)，只需要访问“out”目录并运行以下命令。

```sh
now
```

### 局限

通过nextJS导出，当您运行命令`next export`时，我们会构建HTML版本的应用程序。 此时候，我们将运行你的页面的`getInitialProps`函数。

因此，您只能使用传递给`getInitialProps`的`context`对象的`pathname`，`query`和`asPath`字段。 而不能使用`req`或`res`字段。

> 基本上，当我们预先构建HTML文件时，您将无法动态呈现HTML内容。 如果需要，你需要用'next start'来运行你的应用程序。

## 支持多空间

<p><details>
  <summary><b>Examples</b></summary>
  <ul><li><a href="./examples/with-zones">With Zones</a></li></ul>
</details></p>

每个空间对应NextJS应用的一个单独部署。你可以像这样得到多个空间。然后将它们合并为一个应用。

举个例子，你有下面这样两个不同的空间：
For an example, you can have two zones like this:

* https://docs.my-app.com 为`/docs/**`提供服务
* https://ui.my-app.com 为其它所有页面提供服务

在多空间的支持下，你可以将它们合并为一个。用户使用一个URL就可以浏览它。而你可以独立开发和部署两个应用。

> 这与微服务的概念完全相同，相当于前端应用程序的微服务。

### 如何定义一个空间

There are no special zones related APIs. You only need to do following things:
空间有关的API并没有什么特别之处。你只需要作如下处理：

* Make sure to keep only the pages you need in your app. (For an example, https://ui.my-app.com should not contain pages for `/docs/**`)
* Make sure your app has an [assetPrefix](https://github.com/zeit/next.js#cdn-support-with-asset-prefix). (You can also define the assetPrefix [dynamically](https://github.com/zeit/next.js#dynamic-assetprefix).)

* 确保应用中只包含需要的页面。(举例来说, https://ui.my-app.com 不能包含`/docs/**`的页面)
* 确保应用中设置了[资源前缀](https://github.com/zeit/next.js#cdn-support-with-asset-prefix). (你也可以[动态地](https://github.com/zeit/next.js#dynamic-assetprefix)定义资源的前缀。)

### 如何将多个空间合并

你可以使用任何HTTP代理来合并多个空间。

你可以使用[微代理](https://github.com/zeit/micro-proxy)作为你本地的代理服务。它可以允许你像下面这样轻松地定义路由规则：
You can use  as your local proxy server. It allows you to easily define routing rules like below:

```json
{
  "rules": [
    {"pathname": "/docs**", "method":["GET", "POST", "OPTIONS"], "dest": "https://docs.my-app.com"},
    {"pathname": "/**", "dest": "https://ui.my-app.com"}
  ]
}
```

针对生产环境，如果你使用的是[ZEIT now](https://zeit.co/now),则可以使用[path alias](https://zeit.co/docs/features/path-aliases)支持的特性。 否则，您可以为现有代理服务器 配置一组像上面一样的规则来路由HTML页面。

## 相关技巧

- [Setting up 301 redirects](https://www.raygesualdo.com/posts/301-redirects-with-nextjs/)
- [Dealing with SSR and server only modules](https://arunoda.me/blog/ssr-and-server-only-modules)
- [Building with React-Material-UI-Next-Express-Mongoose-Mongodb](https://github.com/builderbook/builderbook)

## FAQ

<details>
  <summary>Is this production ready?</summary>
  NextJS has been powering https://zeit.co since its inception.

  We’re ecstatic about both the developer experience and end-user performance, so we decided to share it with the community.
</details>

<details>
  <summary>How big is it?</summary>

The client side bundle size should be measured in a per-app basis.
A small Next main bundle is around 65kb gzipped.

</details>

<details>
  <summary>Is this like `create-react-app`?</summary>

Yes and No.

Yes in that both make your life easier.

No in that it enforces a _structure_ so that we can do more advanced things like:
- Server side rendering
- Automatic code splitting

In addition, NextJS provides two built-in features that are critical for every single website:
- Routing with lazy component loading: `<Link>` (by importing `next/link`)
- A way for components to alter `<head>`: `<Head>` (by importing `next/head`)

If you want to create re-usable React components that you can embed in your NextJS app or other React applications, using `create-react-app` is a great idea. You can later `import` it and keep your codebase clean!

</details>

<details>
  <summary>How do I use CSS-in-JS solutions?</summary>

NextJS bundles [styled-jsx](https://github.com/zeit/styled-jsx) supporting scoped css. However you can use any CSS-in-JS solution in your Next app by just including your favorite library [as mentioned before](#css-in-js) in the document.
</details>

<details>
  <summary>What syntactic features are transpiled? How do I change them?</summary>

We track V8. Since V8 has wide support for ES6 and `async` and `await`, we transpile those. Since V8 doesn’t support class decorators, we don’t transpile those.

See [this](https://github.com/zeit/next.js/blob/master/server/build/webpack.js#L79) and [this](https://github.com/zeit/next.js/issues/26)

</details>

<details>
  <summary>Why a new Router?</summary>

NextJS is special in that:

- Routes don’t need to be known ahead of time
- Routes are always lazy-loadable
- Top-level components can define `getInitialProps` that should _block_ the loading of the route (either when server-rendering or lazy-loading)

As a result, we were able to introduce a very simple approach to routing that consists of two pieces:

- Every top level component receives a `url` object to inspect the url or perform modifications to the history
- A `<Link />` component is used to wrap elements like anchors (`<a/>`) to perform client-side transitions

We tested the flexibility of the routing with some interesting scenarios. For an example, check out [nextgram](https://github.com/zeit/nextgram).

</details>

<details>
<summary>How do I define a custom fancy route?</summary>

We [added](#custom-server-and-routing) the ability to map between an arbitrary URL and any component by supplying a request handler.

On the client side, we have a parameter call `as` on `<Link>` that _decorates_ the URL differently from the URL it _fetches_.
</details>

<details>
<summary>How do I fetch data?</summary>

It’s up to you. `getInitialProps` is an `async` function (or a regular function that returns a `Promise`). It can retrieve data from anywhere.
</details>

<details>
  <summary>Can I use it with GraphQL?</summary>

Yes! Here's an example with [Apollo](./examples/with-apollo).

</details>

<details>
<summary>Can I use it with Redux?</summary>

Yes! Here's an [example](./examples/with-redux)
</details>

<details>
<summary>Why aren't routes I have for my static export accessible in the development server?</summary>

This is a known issue with the architecture of NextJS. Until a solution is built into the framework, take a look at [this example solution](https://github.com/zeit/next.js/wiki/Centralizing-Routing) to centralize your routing.
</details>

<details>
<summary>Can I use Next with my favorite Javascript library or toolkit?</summary>

Since our first release we've had **many** example contributions, you can check them out in the [examples](./examples) directory
</details>

<details>
<summary>What is this inspired by?</summary>

Many of the goals we set out to accomplish were the ones listed in [The 7 principles of Rich Web Applications](http://rauchg.com/2014/7-principles-of-rich-web-applications/) by Guillermo Rauch.

The ease-of-use of PHP is a great inspiration. We feel NextJS is a suitable replacement for many scenarios where you otherwise would use PHP to output HTML.

Unlike PHP, we benefit from the ES6 module system and every file exports a **component or function** that can be easily imported for lazy evaluation or testing.

As we were researching options for server-rendering React that didn’t involve a large number of steps, we came across [react-page](https://github.com/facebookarchive/react-page) (now deprecated), a similar approach to NextJS by the creator of React Jordan Walke.

</details>

## Contributing

Please see our [contributing.md](./contributing.md)

## Authors

- Arunoda Susiripala ([@arunoda](https://twitter.com/arunoda)) – [ZEIT](https://zeit.co)
- Tim Neutkens ([@timneutkens](https://github.com/timneutkens)) – [ZEIT](https://zeit.co)
- Naoyuki Kanezawa ([@nkzawa](https://twitter.com/nkzawa)) – [ZEIT](https://zeit.co)
- Tony Kovanen ([@tonykovanen](https://twitter.com/tonykovanen)) – [ZEIT](https://zeit.co)
- Guillermo Rauch ([@rauchg](https://twitter.com/rauchg)) – [ZEIT](https://zeit.co)
- Dan Zajdband ([@impronunciable](https://twitter.com/impronunciable)) – Knight-Mozilla / Coral Project
