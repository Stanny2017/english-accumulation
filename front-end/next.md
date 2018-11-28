# next.js

本文尝试翻译 [next.js](https://github.com/zeit/next.js) 官方文档，希望可以便于大家的理解。
- 原文地址：https://nextjs.org/docs

官方推出的循序渐进入门 next 的教程 [nextjs.org/learn](https://nextjs.org/learn) , 读完基础部分我觉得非常棒！

---

## 如何使用

### 安装


使用 npm 进行安装相关依赖（在此之前可新建空文件夹并且 npm int）

```
npm install --save next react react-dom
```

像如下示例一样添加一些脚本在项目的 `package.json` 文件中：

```json
{
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
```

添加好之后，文件系统就成了主要的 API ，每个 `pages` 目录下的 `.js` 文件名都会自动形成一个路由，这些都在 next 内部被自动处理和渲染。

在项目根目录中新建 pages 文件夹并新建  `./pages/index.js` ，内容如下：

```javascript
export default () => <div>Welcome to next.js!</div>
```

然后执行 `npm run dev` 并且访问 `http://localhost:3000`. 如果想要自定义端口号，执行 `npm run dev -- -p <端口号>`  

按照如上步骤实施完，该项目已经具备了如下特性：
- 自动编译和打包（利用了 webpack 和 babel）
- 代码的热加载
- 服务端渲染，基于 `./pages` 寻址
- 静态文件服务， `./static/` 会映射到 路径`/static/` 下

看一下 [sample app - nextgram](https://github.com/now-examples/nextgram) 项目，你就知道这一切有多么简单了！

## 代码拆分自动化

每一个你 `import` 进来的代码块都会被打包并且服务于每个页面，这意味着所有的页面从不会加载多余的代码。

```js
import cowsay from 'cowsay-browser'

export default () => (
  <pre>
    {cowsay.say({ text: 'hi there!' })}
  </pre>
)
```

## CSS

### 支持内建 css

例子： [Basic css](https://github.com/zeit/next.js/tree/master/examples/basic-css)

我们使用 [style-jsx](https://github.com/zeit/styled-jsx) 的方式来支持独立作用域的 css 。这样做的目的时支持 `shadow CSS` (写法和原生 css 完全一致)，但可惜的是他不支持在服务端渲染并且是`JS-only`

```javascript
export default () => (
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
)
```

更多例子请翻阅  [styled-jsx documentation](https://www.npmjs.com/package/styled-jsx)

#### CSS-in-JS

例子：// TODO

我们很有可能会去使用目前存在的 CSS-in-JS 解决方案，最简单的就是定义行内样式：
```js
export default () => <p style={{ color: 'red' }}>hi there</p>
```
如果想使用更为复杂的 CSS-in-JS 方案，一般来说，为了服务端渲染你需要实施`style flushing`. 我们允许用户自定义 `custom <Document>` 组件(包裹了每个页面) 来实现它。

#### Importing CSS / Sass / Less / Stylus files

为了支持可以引入 `.css`,`.scss`,`.less` 和 `.styl` 文件，你可以使用以下模块方案，他们都默认被配置为服务端渲染应用。
- [@zeit/next-css](https://github.com/zeit/next-plugins/tree/master/packages/next-css)
- [@zeit/next-sass](https://github.com/zeit/next-plugins/tree/master/packages/next-sass)
- [@zeit/next-less](https://github.com/zeit/next-plugins/tree/master/packages/next-less)
- [@zeit/next-stylus](https://github.com/zeit/next-plugins/tree/master/packages/next-stylus)


## 静态文件服务(例如图片)

在项目根目录下创建一个文件夹 `static`用于存放一些资源 。然后在你的代码里可以通过 `/static/` 的路径就可以引用这些资源。

```javascript
export default () => <img src="/static/my-image.png" alt="my image" />
```

注意：不要将 `static` 修改成任何其他名字，只能用这个名字并且这个目录是 next.js 唯一用来服务静态资源的目录。

## 填充 `<head>`

例子 // TODO

我们提供了一个内建的组件便于在页面的 `<head>` 标签里面填充元素。

```javascript
import Head from 'next/head'

export default () => (
  <div>
    <Head>
      <title>My page title</title>
      <meta name="viewport" content="initial-scale=1.0, width=device-width" />
    </Head>
    <p>Hello world!</p>
  </div>
)
```
为了避免重复的标签在你的 `head` 里面，你可以使用 `key` 属性，`key` 属性可以标签只被渲染一次。

```javascript
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
在这个例子中，只有第二个 meta 标签会被渲染。

注意: 一旦当前组件被 unmounting `<head>` 里面的内容就会被清空，所以确保每一个页面都完全定义了自己需要的 `head`，不要假设其他页面添加了什么。

## 获取数据和组件生命周期

例子： [Data fectch](https://github.com/zeit/next.js/tree/master/examples/data-fetch)

当你要使用 state 生命周期函数或者 初始数据填充时，你需要 export 一个 react 组件（而不是无状态的纯函数）如下所示：

```js
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
注意在页面加载的时候如果需要获取数据，我们使用 `getInitialProps` ,这是一个异步静态方法，可以异步地获取到任何数据并解析为纯 js 对象，返回的对象内容会填充到当前组件的 props 属性上。

`getInitialPorps` 返回的数据在服务端渲染的时候被序列化，类似于`JSON.stringify` . 需要确保 `getInitialProps` 返回的对象是一个纯对象，不要使用 `Date`,`Map` 或者 `Set`.

对于初始页面的加载，`getInitialProps` 将只会在服务端执行。在通过 `Link` 组件或者其他路由 api 在客户端进行切换不同路由的时候，`getInitialProps` 才会在客户端执行。

并且注意： `getInitialProps` 不可以在子组件中使用，只能用在 `pages` 下的组件。


如果你在 `getInitialProps` 里使用了某些仅支持服务端的模块，请确保他们被正确的导入。否则，你的 app 会变慢哦。

你也可以在无状态的组件中定义 `getInitialProps` 这个生命周期函数：

```javascript
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

`getInitialProps`  接收一个对象作为参数，这个对象包含以下属性：

- `pathname` - URL 的路径部分
- `query` - 被解析成对象的 URL 查询参数
- `asPath` - String of the actual path (including the query) shows in the browser
- `req` - HTTP request object (server only)
- `res` - HTTP response object (server only)
- `jsonPageRes` - Fetch Response object (client only)
- `err` - Error object if any error is encountered during the rendering

## 路由

### With `<Link>`

例子：[Hello World](https://github.com/zeit/next.js/tree/master/examples/hello-world)

客户端的路由切换通过 `<Link>` 组件实现。
如下是两个页面：

```js
// pages/index.js
import Link from 'next/link'

export default () => (
  <div>
    Click{' '}
    <Link href="/about">
      <a>here</a>
    </Link>{' '}
    to read more
  </div>
)
```
```js
// pages/about.js
export default () => <p>Welcome to About!</p>
```

提示：使用 `<Link prefetch>` 来达到性能最优，可以链接另一个页面的同时在后台预抓取数据。

浏览器端的路由 行为与浏览器完全相同
1. The component is fetched
2. 如果定义了 `getInitialProps`，数据将会被抓取.如果发生了错误，`_error.js` 将会被渲染。
3. 前面两个步骤都完成后，`pushState` 就会被执行并且新组件会被渲染。

你可以使用 `withRouter` 来为你的组件注入 `pathname`、`query` 和 `asPath` .

**With URL object**

例子： [With URL Object Routing](https://github.com/zeit/next.js/tree/master/examples/with-url-object-routing)

`<Link>` 组件也可以接收一个 URL 对象并且它会自动将其格式化来的创建 URL 字符串。

```js
// pages/index.js
import Link from 'next/link'

export default () => (
  <div>
    Click{' '}
    <Link href={{ pathname: '/about', query: { name: 'Zeit' } }}>
      <a>here</a>
    </Link>{' '}
    to read more
  </div>
)
```

这种方式就会生成 URL 字符串 `/about?name=Zeit`，你也可以使用每一个 [node.js URL 模块文档](https://nodejs.org/api/url.html#url_url_strings_and_url_objects) 中定义的属性

**Replace instead of push url**

`<Link>` 组件默认的行为是向栈内推入一个新的 url，你可以使用 `replace` 属性来进行替换而不是推入一个 url。

```js
// pages/index.js
import Link from 'next/link'

export default () => (
  <div>
    Click{' '}
    <Link href="/about" replace>
      <a>here</a>
    </Link>{' '}
    to read more
  </div>
)
```

**使用支持 `onClick`的组件**

`<Link>` 支持任何可以带有 `onClick` 事件的组件。如果不使用 `<a>`（使用了不支持 onClick） 标签，它会试图添加 onClick 事件，但是无济于事，例如下面这种情况就不会进行跳转。

```js
// pages/index.js
import Link from 'next/link'

export default () => (
  <div>
    Click{' '}
    <Link href="/about">
      <img src="/static/image.png" alt="image" />
    </Link>
  </div>
)
```

**强制让 Link 把 `href` 暴露给子组件**

如果子组件是 `<a>` 并且没有 `href` 属性，we specify it so that the repetition is not needed by the user. 
然而，有时你想要去往里面传递一个 a 标签，但是 Link 并不能识别它是一个超链接，所以就不会把它的 `href` 标签传递给子组件，这种情况下，你应该定义一个布尔类型的 `passHref` 属性值给 `Link`，这个属性可以强制其把 `href` 属性传递给子组件。

请注意：如果使用了非 a 标签的元素并且传递 `passHref` 时失败了，原因可能是当被搜索引擎爬取时识别不出它是一个链接（由于缺乏 href 属性）。这点在你想要对站点进行搜索引擎优化时可能会导致负面的影响。

```js
import Link from 'next/link'
import Unexpected_A from 'third-library'

export default ({ href, name }) => (
  <Link href={href} passHref>
    <Unexpected_A>
      {name}
    </Unexpected_A>
  </Link>
)
```

**Disabling the scroll changes to top on page**

`<Link>` 的默认行为是滚动到页面的顶部（切换页面时默认从最顶部开始显示）。如果想要切换的时候滑动到固定的 id 值的位置如同常用的 a 标签一样，可以添加 `scroll={false}` 到 `<Link>` 上

```js
<Link scroll={false} href="/?counter=10"><a>Disables scrolling</a></Link>
<Link href="/?counter=10"><a>Changes with scrolling to top</a></Link>
```

## Imperatively(命令式的)

例子：
- [Basic routing](https://github.com/zeit/next.js/tree/master/examples/using-router)
- [With a page loading indicator](https://github.com/zeit/next.js/tree/master/examples/with-loading)

你也可以使用 `next/router` 来进行客户端路由的跳转。

```js
import Router from 'next/router'

export default () => (
  <div>
    Click <span onClick={() => Router.push('/about')}>here</span> to read more
  </div>
)
```

### 拦截 `popstate`

某些情况下（举个例子，如果使用了 custom router）您可能希望在路由器对 popstate 进行操作之前监听 popstate 并作出响应。相应地，你可以使用这个特性去操纵请求，或者强制服务端刷新。

```js
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

如果 `beforePopState` 返回了 false,`Router` 就不会处理 `popstate`;你应该自己手动去处理，在这种情况下，看一下 `禁用 file-system routing`

上述的 `Router` 对象有如下 api
- route - String of the current route
- pathname - String of the current path excluding the query string
- query - Object with the parsed query string. Defaults to {}
- asPath - String of the actual path (including the query) shows in the browser
- push(url, as=url) - performs a pushState call with the given url
- replace(url, as=url) - performs a replaceState call with the given url
- beforePopState(cb=function) - intercept popstate before router processes the event.

`push` 和 `replace` 第二个参数 `as` 是可选项，在你需要在服务器上配置 custom routes 的时候有用。
 
 **With URL object**

 你可以使用一个 URL 对象来 push 和替换 URL， 用法和在 `<Link>` 组件中使用一样。
```js
import Router from 'next/router'

const handler = () => {
  Router.push({
    pathname: '/about',
    query: { name: 'Zeit' }
  })
}

export default () => (
  <div>
    Click <span onClick={handler}>here</span> to read more
  </div>
)
```
这种方式使用了同样的参数，和在 `<Link>` 组件中的用法一样。

**Router Events**

你可以监听发生在 Router 内部的不同事件，以下列出了一些支持的事件。

- routeChangeStart(url) - Fires when a route starts to change 路由改变开始时触发
- routeChangeComplete(url) - Fires when a route changed completely 
- routeChangeError(err, url) - Fires when there's an error when changing routes
- beforeHistoryChange(url) - Fires just before changing the browser's history
- hashChangeStart(url) - Fires when the hash will change but not the page
- hashChangeComplete(url) - Fires when the hash has changed but not the page

```
这里的 `url` 指的就是显示在浏览器中的 URL, 如果你调用了 `Router.push(url, as)` （或者类似的函数），然后 `url` 值和 `as` 的值保持一致。
```

一个正确监听 路由事件 `routeChangeStart` 的示例:

```js
const handleRouteChange = url => {
  console.log('App is changing to: ', url)
}

Router.events.on('routeChangeStart', handleRouteChange)
```

如果不想继续监听这个事件了，就可以使用 off 方法取消监听：

```js
Router.events.off('routeChangeStart', handleRouteChange)
```

如果某一个路径加载被取消了（例如 快速连续点击了两个链接） 就会触发 `routeChangeError` , 参数 err 会包含一个被设置为 true 的 `cancelled` 属性。

```js
Router.events.on('routeChangeError', (err, url) => {
  if (err.cancelled) {
    console.log(`Route to ${url} was cancelled!`)
  }
})
```

**Shallow Routing**

例子：  [Shallow Routing](https://github.com/zeit/next.js/tree/master/examples/with-shallow-routing)

Shallow routing 允许在改变 URL 之后不需要每次都执行 `getInitialProps`，你将从相同加载过的页面获得更新后的 `pathname` 和 `query` ，并且不会丢失 状态值。

你可以通过调用 `Router.push` 或者 `Router.replace` 带一个额外的参数来达到这种效果，如下示例：

```js
// Current URL is "/"
const href = '/?counter=10'
const as = href
Router.push(href, as, { shallow: true })
```

现在，你的 URL 更新到了`/?counter=10` 你可以看到更新过的 URL 在组件中带有 `this.props.router.query` （确保你使用了 withRouter 来包裹了你的组件）

你可以通过 `componentDidUpdate` 来观察 URL 的变化，如下代码所示：

```js
componentDidUpdate(prevProps) {
  const { pathname, query } = this.props.router
  // verify props have changed to avoid an infinite loop
  if (query.id !== prevProps.router.query.id) {
    // fetch data based on the new query
  }
}
```

```
提示：
shallow routing 仅在同页面的 URL 改变的情况下奏效
假设现在当前有一个页面 `/about` 并且你在当前页面执行下面的代码
```

```js
Router.push('/?counter=10', '/about?counter=10', { shallow: true })
```
因为这是一个新页面，即使我们加了参数来做 shallow routing ，它也会卸载当前页面加载新页面并且调用 `getInitialProps` 


### 使用高阶组件

如果你要在组件中使用 `router`，你要使用高阶组件 `withRouter`,看一下如何使用：
```js
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

上述的 `router` 对象的用法 和 `next/router` 用法一致，有如下属性：
- route - String of the current route
- pathname - String of the current path excluding the query string
- query - Object with the parsed query string. Defaults to {}
- asPath - String of the actual path (including the query) shows in the browser
- push(url, as=url) - performs a pushState call with the given url
- replace(url, as=url) - performs a replaceState call with the given url
- beforePopState(cb=function) - intercept popstate before router processes the event

## 预抓取页面

**这只是在生产环境下的特性**

例子： [prefetching](https://github.com/zeit/next.js/tree/master/examples/with-prefetching)

Next.js 有 api 支持预抓取页面。

由于 next 采用服务端渲染页面，这使得原本需要下一时刻才能进行的交互可以瞬间执行（ this allows all the future interaction paths of your app to be instant）, next 可以有效地进行提前下载，

```
next 只是进行预加载 js 代码，当页面正在渲染时，可能需要等待数据的获取
```

### With `<Link>`

你可以在任何 `<Link>` 标签上添加 `prefetch` 属性，next 会在后台进行预抓取这些页面.

```js
import Link from 'next/link'

// example header component
export default () => (
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
)
```

### Imperatively（命令式地）

大多数的 预抓取 都是通过 `<Link />`来寻址，但是我们也提供了一种高级用法：

```js
import { withRouter } from 'next/router'

export default withRouter(({ router }) => (
  <div>
    <a onClick={() => setTimeout(() => router.push('/dynamic'), 100)}>
      A route transition will happen after 100ms
    </a>
    {// but we can prefetch it!
    router.prefetch('/dynamic')}
  </div>
)
```

可是这个用法仅限于在客户端使用。当在服务端渲染路由的时候，为了避免相关错误，在 `componentDidMount()` 使用声明式的预抓取方法

```js
import React from 'react'
import { withRouter } from 'next/router'

class MyLink extends React.Component {
  componentDidMount() {
    const { router } = this.props
    router.prefetch('/dynamic')
  }

  render() {
    const { router } = this.props

    return (
       <div>
        <a onClick={() => setTimeout(() => router.push('/dynamic'), 100)}>
          A route transition will happen after 100ms
        </a>
      </div>
    )
  }
}

export default withRouter(MyLink)
```

## 自定义服务器和路由

例子：
- [Basic custom server](https://github.com/zeit/next.js/tree/master/examples/custom-server)
- Express integration
- Hapi integration
- [Koa integration](https://github.com/zeit/next.js/tree/master/examples/custom-server-koa)
- Parameterized routing
- SSR caching

通常你执行 `next start` 来开启 next 服务。但也可以完全自己写一段代码来自定义路由。

当你写了一个文件（比如文件名为 `server.js`）来自定义服务器，确保更新 `package.json`:

```json
{
  "scripts": {
    "dev": "node server.js",
    "build": "next build",
    "start": "NODE_ENV=production node server.js"
  }
}
```

以下这个例子将路由 `/a` 指向 `./pages/b` ,并且把 `/b` 指向 `./pages/a`:

```js
// 这个文件不会被 babel 和 webpack 转换。
// 确保该文件中的语法和依赖项都与当前 node 版本兼容。
// See https://github.com/zeit/next.js/issues/1245 for discussions on Universal Webpack or universal Babel
const { createServer } = require('http')
const { parse } = require('url')
const next = require('next')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  createServer((req, res) => {
    // 确保给 url.parse 传的第二个参数是 true 
    // 这样传参意味着需要对 URL 的查询参数进行 parse 解析
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

`next` API  如下所示：
- `next(opts: object)`
支持的可选参数：
  - `dev` 布尔类型，表示是否在 开发者模式下 运行 Next.js - 默认为 `false`
  - `dir` 字符类型，表示 next 项目的路径，默认 `'.'`
  - `quiet` 布尔， 表示是否隐藏错误信息包括服务端信息， 默认 `false`
  - `conf` object 类型， 和在 `next.config.js` 里使用的一致， 默认是空对象 `{}`

然后，改变 `start` 脚本为 `NODE_ENV=production node server.js`.


### 禁止文件系统式的路由

默认情况下，`Next` 将会以 `/pages` 目录下的文件名作为默认的路由，（例如 文件`/pages/some-file.js` 是访问`site.com/some-file` 的结果）
如果你的项目中使用了自定义路由，这个默认行为将会导致 多个路由匹配同一个内容。这会给 SEO 和 用户体验带来一些问题。

如果需要禁用这种默认行为（不让路由基于路径 `/pages` 目录），可以简单的在 `next.config.js` 进行设置：

```js
// next.config.js
module.exports = {
  useFileSystemPublicRoutes: false
}
```

注意 `useFileSystemPublicRoutes` 只是简单地在服务端渲染的时候禁用了 文件名路由。客户端路由还是可以触及到那些路径。如果使用了这个选项，你应该防止切换到那些你不想跳转的路由。

如果你还想要禁用默认的客户端路由，参考 `Intercepting`

### Dynamic assetPrefix

有时我们想要动态的设置 `assetPrefix` ，这在我们基于传入的请求改变`assetPrefix` 的时候很有用，为了达到这样的效果，我们使用 `app.setAssetPrefix` 

如下是一个例子：

```js
const next = require('next')
const micro = require('micro')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handleNextRequests = app.getRequestHandler()

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

## 引入 dynamic

// TODO

例子： [with dynamic import](https://github.com/zeit/next.js/tree/master/examples/with-dynamic-import)


## 自定义 `<App>`

