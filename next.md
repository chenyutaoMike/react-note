## Next.js简介

> Next.js 是一个轻量级的 React 服务端渲染应用框架。

用一个框架，就要知道它的优点（或者是解决了我们什么问题）:

- 完善的React项目架构，搭建轻松。比如：Webpack配置，服务器启动，路由配置，缓存能力，这些在它内部已经完善的为我们搭建完成了。
- 自带数据同步策略，解决服务端渲染最大难点。把服务端渲染好的数据，拿到客户端重用，这个在没有框架的时候，是非常复杂和困难的。有了Next.js，它为我们提供了非常好的解决方法，让我们轻松的就可以实现这些步骤。
- 丰富的插件帮开发人员增加各种功能。每个项目的需求都是不一样的，包罗万象。无所不有，它为我们提供了插件机制，让我们可以在使用的时候按需使用。你也可以自己写一个插件，让别人来使用。
- 灵活的配置，让开发变的更简单。它提供很多灵活的配置项，可以根据项目要求的不同快速灵活的进行配置。

目前Next.js是React服务端渲染的最佳解决方案，所以如果你想使用React来开发需要SEO的应用，基本上就要使用Next.js。

#### 手动创建

1.初始化项目

```
npm init -y
```

2.安装依赖

```
 yarn add react react-dom next
```

这里安装了react react-dom 和nextjs这三个依赖

3.配置package.json文件

```json
{
  "name": "nextjs",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "next",                 //第一步配置
    "build": "next build",         //第二步配置
    "start": "next start"          //第三步配置
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "next": "^9.1.3",
    "react": "^16.12.0",
    "react-dom": "^16.12.0"
  }
}
```

4.运行项目

```
yarn dev
```

现在项目已经跑起来了，可以打开浏览器查看3000端口，然后会看到404页面，现在就差最后一步就可以看到页面了

5.新建pages文件夹，然后在pages文件下下面新建index.js

在根项目下新建pages文件夹，里面的index.js：

```
export default () => <span>index</span>
```

这样就可以在页面上看到内容了

#### 使用脚手架创建

1.要使用脚手架安装，就要现在全局安装这个包

```
npm i -g create-next-app
```

2.使用命令安装项目

```
npx create-next-app next-create
```

第二步其实还有另外一种创建方式，都是一样的

```
yarn create next-app next-create
```

3.进入项目文件夹，运行项目

```
yarn dev
```

#### 脚手架生成文件目录介绍

- .next  -> next编译代码生成的文件(不要更改)
- components文件夹:这里是专门放置自己写的组件的，这里的组件不包括页面，指公用的或者有专门用途的组件。
- node_modules文件夹：Next项目的所有依赖包都在这里，一般我们不会修改和编辑这里的内容。
- pages文件夹：这里是放置页面的，这里边的内容会自动生成路由，并在服务器端渲染，渲染好后进行数据同步。
- static文件夹： 这个是静态文件夹，比如项目需要的图片、图标和静态资源都可以放到这里。
- .gitignore文件： 这个主要是控制git提交和上传文件的，简称就是忽略提交。
- package.json文件：定义了项目所需要的文件和项目的配置信息（名称、版本和许可证），最主要的是使用`npm install` 就可以下载项目所需要的所有包。

### Next.js的Page和Component的使用

#### 新建页面和访问路径

直接在根目录下的`pages`文件夹下，新建一个`hong.js`页面。然后写入下面的代码：

```js
function Hong() {
    return (<button>按我加加</button>)
}


export default Hong
```

只要写完上面的代码，`Next`框架就自动作好了路由，这个也算是Next的一个重要优点，给我们节省了大量的时间。

现在要作一个更深的页面，比如把有关博客的界面都放在这样的路径下`http://localhost:3000/blog/nextBlog`,其实只要在`pages`文件夹下再建立一个新的文件夹`blog`，然后进入`blog`文件夹，新建一个`nextBlog.js`文件，就可以实现了。

nextBlog.js文件内容,我们这里就用最简单的写法了

```js
export default ()=><div>nextBlog page</div>
```

写完后，就可以直接在浏览器中访问了

#### Component组件的制作

制作组件也同样方便，比如要建立一个Hong组件，直接在`components`目录下建立一个文件`hong.js`,然后写入下面代码:

```js
export default ({children})=><button>{children}</button>
```

组件写完后需要先引入，比如我们在Index页面里进行引入：

```js
import Hong from '../components/hong'
```

使用就非常简单了，直接写入标签就可以。

```js
<Hong>按钮</Hong>
```

### 路由-基础和基本跳转

页面跳转一般有两种形式，第一种是利用标签`<Link>`,第二种是用js编程的方式进行跳转，也就是利用`Router`组件。先来看一下标签的形式如何跳转。

#### 标签式导航`<Link>`

在编写代码之前，先删除`index.js`中的代码，保证代码的最小化。使用标签式导航需要先进行引入，代码如下:

```js
import Link from 'next/link'
```

然后新建两个页面`hong.js`和`hongB.js`，新建后写个最简单的页面，能标识出来A、B两个页面就好。

```js
//hong.js
import Link from 'next/link'

function Hong() {
    return (
        <>
            <h1>小红的A页面</h1>
            <Link href='/'><a>返回首页</a></Link>
        </>

    )
}

export default Hong
```

写完A页面后，可以直接复制A页面的内容，然后修改一下就是B页面。

```js
//hongB.js

import Link from 'next/link'

function HongB() {
    return (
        <>
            <h1>小红的B页面</h1>
            <Link href='/'><a>返回首页</a></Link>
        </>

    )
}

export default HongB

```

有了两个页面后，可以编写首页的代码，实现跳转了。

```js
//index.js
import React from 'react'
import Link from 'next/link'


const Home = () => (
  <>
    <div>我是首页</div>
    <p><Link href="/Hong"><a>去小红A页面</a></Link></p>
    <p><Link href="/HongB"><a>去小红B页面</a></Link></p>
  </>
)


export default Home

```

用`<Link>`标签进行跳转是非常容易的，但是又一个小坑需要你注意一下，就是他不支持兄弟标签并列的情况。

```js
 <div>
  <Link href="/hong">
    <span>去hongA页面</span>
    <span>前端博客</span>
  </Link>
</div>
```

如果这样写会直接报错，报错信息如下

```text
client pings, but there's no entry for page: /_error
Warning: You're using a string directly inside <Link>. This usage has been deprecated. Please add an <a> tag as child of <Link>
```

但是你可以把这两个标签外边套一个父标签，就可以了，比如下面的代码就没有错误。

```js
<Link href="/hong">
  <a>
    <span>去hongA页面</span>
    <span>前端博客</span>
  </a>
</Link>
```

通过标签跳转非常的简单，跟使用`<a>`标签几乎一样。

#### Router模块进行跳转(编程式跳转)

在`Next`框架中还可以使用Router模块进行编程式的跳转，使用前也需要我们引入`Router`，代码如下：

```js
import Router from 'next/router'
```

然后在`Index.js`页面中加入，直接使用Router进行跳转就可以了。

```js
 <div>
    <button onClick={()=>{Router.push('/hong')}}>去小红A页面</button>
  </div>
```

这样写只是简单，但是还是耦合性太高，跟Link标签没什么区别，你可以修改一下代码，把跳转放到一个方法里，然后调用方法。

```js
import React from 'react'
import Link from 'next/link'
import Router from 'next/router'

const Home = () => {

  function gotoA(){  //跳转到小红A页面
    Router.push('/hong')
  }

  return (
    <>
      <div>我是首页</div>
      <p><Link href="/Hong"><a>去小红A页面</a></Link></p>
      <p><Link href="/HongB"><a>去小红B页面</a></Link></p>
      <div>
        <button onClick={gotoA}>去小红A页面</button>
        
      </div>
    </>
  )
}


export default Home

```

这样也是可以实现跳转的，而且耦合性也降低了

#### 路由-跳转时用query传递和接受参数

项目开发中一般都不是简单的静态跳转，而是需要动态跳转的。动态跳转就是跳转时需要带一个参数或几个参数过去，然后在到达的页面接受这个传递的参数，并根据参数不同显示不同的内容。比如新闻列表，然后点击一个要看的新闻就会跳转到具体内容。这些类似这样的需求都都是通过传递参数实现的。

#### 只能用query传递参数

##### 用Link标签传递参数

```jsx
import React from 'react'
import Link from 'next/link'

const Home = () => {
  return (
    <>
      <div>我是首页</div>
      <div>
        <Link href="/girl?name=小红"><a>选择小红</a></Link>
        <br />
        <Link href="/girl?name=小燕"><a>选择小燕</a></Link>
      </div>
     

    </>
  )
}


export default Home

```

这样编写query参数就可以进行传递过去了，接下来就是要接受参数了。

##### 接收传递过来的参数

接收参数页面：

```jsx
import { withRouter } from 'next/router'
import Link from 'next/link'

const Girl = ({ router }) => {
    return (
        <>
            <div>你选择了：{router.query.name}</div>
            <Link href="/"><a>返回首页</a></Link>
        </>
    )
}


export default withRouter(Girl)
```

`withRouter`是Next.js框架的高级组件,通过withRouter包装过我们编写的组件，就可以使用router.query获取传递过来的参数

#### 编程式跳转传递参数

回了`<Link>`这种标签式跳转传递参数的形式，那编程式跳转如何传递那，其实也可以简单使用`?加参数`的形式，代码如下：

```jsx
 <div>
  <button onClick={gotoGirl}>选择小红</button>
</div>

function gotoGirl(){
    Router.push('/girl?name=小红')
  }
```

这种形式跳转和传递参数是完全没有问题的，但是不太优雅，可以再push里面写成对象的形式

```jsx
import React from 'react'
import Link from 'next/link'
import Router from 'next/router'

const Home = () => {

  function gotoGirl() {
    Router.push({
      pathname: '/girl',
      query: { name: '小红' }
    })
  }

  return (
    <>
      <div>我是首页</div>
      <div>
        <Link href="/girl?name=小红"><a>选择小红</a></Link>
        <br />
        <Link href="/girl?name=小燕"><a>选择小燕</a></Link>
        <Link href={{pathname:'/girl',query:{name:'小燕'}}}><a>选择小燕</a></Link>
      </div>
      <div>
        <button onClick={gotoGirl}>选择小红</button>
      </div>

    </>
  )
}


export default Home

```

Link标签也可以写成对象的形式

```jsx
 <Link href={{pathname:'/girl',query:{name:'小燕'}}}><a>选择小燕</a></Link>
```

### 路由-六个钩子事件的讲解

路由的钩子事件，也就是当路由发生变化时，可以监听到这些变化事件，执行对应的函数。它一共有六个钩子事件

##### ● routeChangeStart   

##### ● routeChangeComplete 

##### ● beforeHistoryChange  

##### ● routeChangeError

##### ● hashChangeStart

##### ● hashChangeComplete

#### `routerChangeStart`路由发生变化时

在监听路由发生变化时，我们需要用Router组件，然后用`on`方法来进行监听,在`pages`文件夹下的`index.js`，然后写入下面的监听事件，

```js
Router.events.on('routeChangeStart', (...args) => {
    console.log('1.routeChangeStart -> 路由开始变化，参数为：', ...args)
    //1.routeChangeStart -> 路由开始变化，参数为： /girl?name=小红
  })
```

这个时路由发生变化时，时间第一时间被监听到，并执行了里边的方法。

#### `routerChangeComplete`路由结束变化时

路由变化开始时可以监听到，那结束时也时可以监听到的，这时候监听的事件是`routerChangeComplete`。

```js
  Router.events.on('routeChangeComplete',(...args)=>{
    console.log('routeChangeComplete->路由结束变化,参数为:',...args)
  })
```

#### `beforeHistoryChange`浏览器history触发前

history就是HTML中的API，如果这个不了解可以百度了解一下，`Next.js`路由变化默认都是通过history进行的，所以每次都会调用。 不适用history的话，也可以通过hash

```js
  Router.events.on('beforeHistoryChange',(...args)=>{
    console.log('3,beforeHistoryChange->在改变浏览器 history之前触发,参数为:',...args)
  })
```

#### `routeChangeError`路由跳转发生错误时

```js
 Router.events.on('routeChangeError',(...args)=>{
    console.log('4,routeChangeError->跳转发生错误,参数为:',...args)
  })
```

需要注意的是404找不到路由页面不算错误

### 转变成hash路由模式

还有两种事件，都是针对hash的，所以现在要转变成hash模式。hash模式下的两个事件`hashChangeStart`和`hashChangeComplete`

```js
  Router.events.on('hashChangeStart',(...args)=>{
    console.log('5,hashChangeStart->hash跳转开始时执行,参数为:',...args)
  })

  Router.events.on('hashChangeComplete',(...args)=>{
    console.log('6,hashChangeComplete->hash跳转完成时,参数为:',...args)
  })
 
```

在下面的jsx语法部分，再增加一个链接,使用hash来进行跳转，代码如下：

```jsx
<div>
 <Link href="#xiaohong"><a>哈希连接</a></Link>
</div>
```

利用钩子事件是可以作很多事情的，比如转换时的加载动画，关掉页面的一些资源计数器.....。

### 在`getInitialProps`中用Axios获取远端数据

在`Next.js`框架中提供了`getInitialProps`静态方法用来获取远端数据，这个是框架的约定，所以你也只能在这个方法里获取远端数据。

#### `getInitialProps`中获取数据

在`girl.js`页面中使用`getInitialProps`，因为是远程获取数据，所以我们采用异步请求的方式。

```jsx
import { withRouter } from 'next/router'
import Link from 'next/link'
import axios from 'axios'


const Girl = ({ router, list }) => {  //使用了withRouter之后，异步加载的数据可以在此获取
    return (
        <>
            <div>你选择了：{router.query.name}</div>
            <div>
                {list}
            </div>
            <Link href="/"><a>返回首页</a></Link>
        </>
    )
}

Girl.getInitialProps = async () => { //所有的数据请求都要写在getInitialProps里面
    const promise = new Promise((resolve) => {
        axios('https://www.easy-mock.com/mock/5cfcce489dc7c36bd6da2c99/xiaojiejie/getList').then((res) => {
            console.log('远程数据结果', res)
            resolve(res.data.data)
        })
    })
    return await promise
}

export default withRouter(Girl)
```

### 使用`Style JSX`编写页面的CSS样式

在`Next.js`中引入一个CSS样式是不可以用的，如果想用，需要作额外的配置。因为框架为我们提供了一个`style jsx`特性，也就是把CSS用JSX的语法写出来。

##### 初识`Style JSX`语法 把字体设成蓝色

把页面中字的颜色变成蓝色，就可以使用`Style JSX`语法。直接在`<></>`之间写下如下的代码:

```jsx
function Hong() {
    return (
        <>
            <div>小红的A页面</div>
            <style jsx>
                {`
                    div{  //选中div
                        color:blue;
                    }
                    `}
            </style>
        </>

    )
}

export default Hong
```

##### 自动加随机类名 不会污染全局CSS

加入了`Style jsx`代码后，`Next.js`会自动加入一个随机类名，这样就防止了CSS的全局污染。比如我们把代码写成下面这样，然后在浏览器的控制台中进行查看，你会发现自动给我们加入了类名，类似`jsx-xxxxxxxx`

#### 动态显示样式

`Next.js`使用了`Style jsx`,所以定义动态的CSS样式就非常简单，比如现在要作一个按钮，点击一下，字体颜色就由蓝色变成了红色。下面是实现代码。

```jsx
import React, { useState } from 'react'


function Hong() {

    const [color, setColor] = useState('blue');
    const changeColor = () => {
        setColor(color === 'blue' ? 'red' : 'blue')
    }

    return (
        <>
            <div>小红的A页面</div>
            <div><button onClick={changeColor}>按我</button></div>
            <style jsx>
                {`
                    div{
                        color:${color};
                    }
                    `}
            </style>
        </>

    )
}

export default Hong
```

### `Lazy Loading`实现模块懒加载

当项目越来越大的时候，模块的加载是需要管理的，如果不管理会出现首次打开过慢，页面长时间没有反应一系列问题。这时候可用`Next.js`提供的`LazyLoading`来解决这类问题。让模块和组件只有在用到的时候在进行加载，一般我把这种东西叫做“懒加载”.它一般分为两种情况，一种是懒加载（或者说是异步加载）模块，另一种是异步加载组件。他们使用的方法也稍有不同

#### 懒加载模块

这里使用一个在开发中常用的模块`Moment.js`，它是一个JavaScript日期处理类库，使用前需要先进行安装，这里使用`yarn`来进行安装。

```js
yarn add momnet
```

然后在`pages`文件夹下，新建立一个`time.js`文件，并使用刚才的`moment`库来格式化时间，代码如下:

```js
import React, {useState} from 'react'
import moment from 'moment'

function Time(){
    
    const [nowTime,setTime] = useState(Date.now())

    const changeTime=()=>{
        setTime(moment(Date.now()).format())
    }
    return (
        <>
            <div>显示时间为:{nowTime}</div>
            <div><button onClick={changeTime}>改变时间格式</button></div>
        </>
    )
}
export default Time
```

这个看起来很简单和清晰的案例，缺存在着一个潜在的风险，就是如何有半数以上页面使用了这个`momnet`的库，那它就会以公共库的形式进行打包发布，就算项目第一个页面不使用`moment`也会进行加载，这就是资源浪费，

下面我们就通过`Lazy Loading`来进行改造代码。

```js
import React, {useState} from 'react'
//删除import moment
function Time(){
    
    const [nowTime,setTime] = useState(Date.now())

    const changeTime= async ()=>{ //把方法变成异步模式
        const moment = await import('moment') //等待moment加载完成
        setTime(moment.default(Date.now()).format()) //注意使用defalut
    }
    return (
        <>
            <div>显示时间为:{nowTime}</div>
            <div><button onClick={changeTime}>改变时间格式</button></div>
        </>
    )
}
export default Time
```

这时候就就是懒加载了。

#### 懒加载自定义组件

懒加载组件也是非常容易的，我们先来写一个最简单的组件，在`components`文件夹下建立一个`one.js`文件，然后编写如下代码：

```js
export default ()=><div>Lazy Loading Component</div>
```

有了自定义组件后，先要在懒加载这个组件的文件中引入`dynamic`,我们这个就在上边新建的`time.js`文件中编写了。

```js
import dynamic from 'next/dynamic'
```

引入后就可以懒加载自定义模块了，代码如下：

```js
import React, {useState} from 'react'
import dynamic from 'next/dynamic'

const One = dynamic(import('../components/one'))  //引入组件 

function Time(){
    
    const [nowTime,setTime] = useState(Date.now())

    const changeTime= async ()=>{
        const moment = await import('moment')
        
        setTime(moment.default(Date.now()).format())
    }
    return (
        <>
            <div>显示时间为:{nowTime}</div>
            <One/>  //只有执行到这一行的时候才会去加载组件
            <div><button onClick={changeTime}>改变时间格式</button></div>
        </>
    )
}
export default Time
```

### 自定义`<Head>` 更加友好的SEO操作

既然用了`Next.js`框架，你就是希望服务端渲染，进行SEO操作。那为了更好的进行SEO优化，可以自己定制`<Head>`标签，定义`<Head>`一般有两种方式

#### 方法1：在各个页面加上`<Head>`标签

先在`/pages`文件夹下面建立一个`header.js`文件，然后写一个最简单的`Hooks`页面，代码如下:

```js
function Header(){ 
    return (<div>xiaohong.com</div>)
}
export default Header
```

写完后到浏览器中预览一下，可以发现title部分并没有任何内容，显示的是`localhost:3000/header`,接下来就自定义下`<Head>`。自定义需要先进行引入`next/head`。

```js
import Head from 'next/head'
```

引入后你就可以写一些列的头部标签了，全部代码如下:

```jsx
import Head from 'next/head'
function Header(){ 
    return (
        <>
            <Head>
                <title>小红网</title>
                <meta charSet='utf-8' />
            </Head>
            <div>xiaohong.com</div>
    
        </> 
    )
}
export default Header
```

这时候再打开浏览器预览，你发现已经有了`title`。

#### 定义全局的`<Head>`

这种方法相当于自定义了一个组件，然后把`<Head>`在组件里定义好，以后每个页面都使用这个组件,其实这种方法用处不大，也不灵活。因为`Next.js`已经把`<Head>`封装好了，本身就是一个组件，我们再次封装的意义不大。

比如在`components`文件夹下面新建立一个`myheader.js`,然后写入下面的代码:

```jsx
import Head from 'next/head'


const MyHeader = () => {
    return (
        <>
        <Head>
            <title>小红</title>
        </Head>
        </>
    )
}

export default MyHeader
```

这时候把刚才编写的`header.js`页面改写一下，引入自定义的`myheader`，在页面里进行使用，最后在浏览器中预览，也是可以得到`title`的。

```jsx
import MyHeader from '../components/myheader'

function Header() {
    return (
        <>
           <MyHeader />
           <div>小红网</div>
        </>
    )
}


export default Header
```

### Next.js框架下使用Ant Design UI

#### 1.首先让Next.js支持CSS文件

`Next.js`默认是不支持CSS文件的，它用的是`style jsx`，也就是说它是不支持直接用`import`进行引入`css`的。

比如在根目录下新建一个文件夹`static`（其实正常情况下你应该已经有这个文件了），然后在文件夹下建立一个`test.css`文件，写入一些`CSS Style`。

```css
body{
    color:green;
}
```

然后用`import`在`header.js`里引入。

```js
import '../static/test.css'
```

写完这些后到浏览器中进行预览，没有任何输出结果而且报错了。这说明`Next.js`默认是不支持CSS样式引入的，要进行一些必要的设置，才可以完成。

##### 开始进行配置，让Next.js支持CSS文件

先用`yarn`命令来安装`@zeit/next-css`包，它的主要功能就是让`Next.js`可以加载CSS文件，有了这个包才可以进行配置。

```text
yarn add @zeit/next-css
```

包安装好以后就可以进行配置文件的编写了，建立一个`next.config.js`.这个就是`Next.js`的总配置文件

```js
const withCss = require('@zeit/next-css')

if(typeof require !== 'undefined'){
    require.extensions['.css']=file=>{}
}

module.exports = withCss({})
```

这时候CSS就配置好了，样式也能生效了

#### 按需加载`Ant Design`

加载`Ant Design`在我们打包的时候会把`Ant Design`的所有包都打包进来，这样就会产生性能问题，让项目加载变的非常慢。这肯定是不行的，现在的目的是只加载项目中用到的模块，这就需要我们用到一个`babel-plugin-import`文件。

先来安装`Ant Design`库 

直接使用yarn来安装就可以。

```text
yarn add antd
```

 安装和配置`babel-plugin-import` 插件

```text
yarn add babel-plugin-import
```

安装完成后，在项目根目录建立`.babelrc`文件，然后写入如下配置文件。

```js
{
    "presets":["next/babel"],  //Next.js的总配置文件，相当于继承了它本身的所有配置
    "plugins":[     //增加新的插件，这个插件就是让antd可以按需引入，包括CSS
        [
            "import",
            {
                "libraryName":"antd",
                "style":"css"
            }
        ]
    ]
}
```

这样配置好了以后，`webpack`就不会默认把整个`Ant Design`的包都进行打包到生产环境了，而是我们使用那个组件就打包那个组件,同样CSS也是按需打包的。

通过上面的配置，就可以愉快的在`Next.js`中使用`Ant Desgin`，让页面变的好看起来。

可以在`header.js`里，引入`<Button>`组件，并进行使用，代码如下。

```js
import Myheader from '../components/myheader'
import {Button} from 'antd'


import '../static/test.css'
function Header(){ 
    return (
        <>
            <Myheader />
            <div>JSPang.com</div>
            <div><Button>我是按钮</Button></div>
    
        </> 
    )
}
export default Header
```

### Next.js生产环境打包

其实Next.js大打包时非常简单的，只要一个命令就可以打包成功。但是当你使用了`Ant Desgin`后，在打包的时候会遇到一些坑。

> 打包 ：next build

> 运行：next start -p 80

先把这两个命令配置到`package.json`文件里，比如配置成下面的样子。

```js
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start -p 80"
},
```

然后在终端里运行一下`yarn build`，如果这时候报错，其实是我们在加入`Ant Design`的样式时产生的，这个已经在`Ant Design`的Github上被提出了，但目前还没有被修改，你可以改完全局引入CSS解决问题。

在page目录下，新建一个`_app.js`文件，然后写入下面的代码。

```js
import App from 'next/app'

import 'antd/dist/antd.css'

export default App
```

这样配置一下，就可以打包成功了，然后再运行`yarn start`来运行服务器，看一下我们的`header`页面，也是有样式的。说明打包已经成功了。