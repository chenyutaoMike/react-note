# next.js

### 创建next.js项目

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

### koa

#### 服务器

nextjs自身带有服务器，只处理ssr渲染

处理HTTP请求，并根据请求数据返回相应的内容

根据域名之类的HOST来定位服务器

#### nextjs文法处理服务端

数据接口

数据库连接

session状态

#### 所有使用koa作为服务端使用，nextjs作为中间件

koa是一个非常流行的轻量级nodejs服务端框架

#### 安装

```
yarn add koa
```

使用：

小型服务器

```js
const Koa = require('koa')
const next = require('next')

const dev = process.env.NODE_ENV !== 'PRODUCTION'
const app = next({ dev })
const handle = app.getRequestHandler()


const server = new Koa()
server.use(async (ctx, next) => {  //匹配中间件
    const path = ctx.path      //可以获取到请求的路径
    const method = ctx.method  //可以获取到请求的类型
    ctx.body = `<span>Koa server ${path} ${method}</span>`  //返回的html内容
    await next()  //要调用一下next()，如果不调用，所有的请求会被这个中间件捕获
})

server.listen(3000, () => {  //开启服务器
    console.log('koa server linstening on 3000')
})

```

#### koa**-**router

安装

```
yarn add koa-router
```

简单使用方式：

```js
const Koa = require('koa')
const Router = require('koa-router')  //引入Router
const next = require('next')

const dev = process.env.NODE_ENV !== 'PRODUCTION'
const app = next({ dev })
const handle = app.getRequestHandler()

const server = new Koa()
const router = new Router()  //实例化Router
 
router.get('/test/:id', (ctx) => {   //配置Router
    const id = ctx.params.id
    ctx.body = `<p>requset /test${id}</p>`
})
server.use(async (ctx, next) => {
    await next()
})

server.use(router.routes())  //运行



server.listen(3000, () => {
    console.log('koa server linstening on 3000')
})

```

如果不想返回一个html页面，而是一个json对象，可以这么做：

```js
router.get('/test/:id', (ctx) => {
    ctx.body = { success: true } //要返回的JSON对象
    ctx.set('Content-Type', 'application/json') //设置请求头告诉浏览器这是JSON对象
})
```

### redis数据库

1、下载安装包

2、将zip文件解压到指定文件夹下面，如：D:\MyDatabase\redis

3、启动服务

启动cmd，进入redis文件夹：cd d:/MyDatabase/redis
运行程序： .\redis-server.exe redis.windows.conf

因为不是用mis包安装了，没有配置全局变量，所有运行这些命令一定要到安装的文件夹中去才能正常与运行

#### 使用ioredis将nodejs连接redis

ioredis

安装：

```
yarn add ioredis
```

使用方式：

新建一个Test\test-redis.js：

```js
async function test() {
    const Redis = require('ioredis') //引入redis

    const redis = new Redis({    //实例化redis并传入配置，port是端口号，password是密码
        port: 6379,
        password: 123456
    })

    await redis.set('c', 123)  // set => 设置key
    const keys = await redis.keys('*')  //读取所有的keys，因为是异步的，所以要加await 
	await redis.setex('b',10, 456) //设置一个10秒之后过期的key
    console.log(keys) //[ 'c' ]
    console.log(await redis.get('c')) //123   get => 获取key的value
}

test()

//执行 ： node Test\test-redis.js
```

### 集成Antd

nextJS默认不支持CSS文件的

所以要用zeit/next-css来配置

安装

```
yarn add @zeit/next-css
```

在根目录下创建next.config.js文件

```js
const withCss = require('@zeit/next-css')

if (typeof require !== undefined) {
    require.extensions['.css'] = file => { }
}

module.exports = withCss({})
```

然后在根目录下创建一个test.css文件

```css
body{
    background-color: pink;
}
```

最后在pages/index.js文件下引入

```js
import '../test.css'

export default () => <span>index</span>
```

这样就可以实现把CSS加入到nextjs中运行了

#### 配置Antd按需加载

安装

```
yarn add antd
yarn add babel-plugin-import
```

配置：

1. 在根目录下创建一个 .babelrc文件

   ```js
   {
       "presets": [
           "next/babel"
       ],
       "plugins": [
           [
               "import",
               {
                   "libraryName": "antd"
               }
           ]
       ]
   }
   ```

2. 在pages文件下创建一个 _app.js文件

   ```js
   import App from 'next/app'
   
   import 'antd/dist/antd.css'
   
   export default App
   ```

3. 在pages文件下的index.js中引入使用即可

   ```js
   import { Button } from 'antd'
   
   export default () => <Button type="primary">index</Button>
   ```

   

### nextJS目录

pages文件下的文件就是next的路由，在例如在页面上输入/test/index.js ，next就会帮我们找到pages/test/index.js，并且渲染到页面上

### 路由

Link

```jsx
import { Button } from 'antd'
import Link from 'next/link' //引入

export default () => (
    <Link href="a">  //使用，这样就可以跳转到a页面了
        <Button type="primary">index</Button>
    </Link>
)
========================================================================
     <Link href="a"> //错误示范，是不能传两个节点进去的
        <Button type="primary">index</Button> 
		<span>a</span>  
    </Link>
      <Link href="a">  //这样处理就可以
        <div> 
        	<Button type="primary">index</Button> 
			<span>a</span> 
         </div>
    </Link>

```

Router

```jsx
import { Button } from 'antd'
import Link from 'next/link'
import Router from 'next/router' //引入Router

export default () => {

    function gotoTestB(){
        Router.push('/test/b')  //push('里面传入要跳转的路径')
    }

    return (
        <>
            <Link href="a">
                <Button type="primary">index</Button>
            </Link>
            <Button onClick={gotoTestB}>test b</Button> //绑定点击事件
        </>
    )
}


```

#### 动态路由

query

动态传递路由参数

```jsx
import { Button } from 'antd'
import Link from 'next/link'
import Router from 'next/router'

export default () => {

    function gotoTestB(){ 
        Router.push({
            pathname:'/test/b',  //跳转的页面
            query:{   //传递的参数
                id:2
            }
        })
    }

    return (
        <>
            <Link href="a?id=1"> //跳转的页面带个 ?号 ,后面是传递的参数
                <Button type="primary">index</Button>
            </Link>
            <Button onClick={gotoTestB}>test b</Button>
        </>
    )
}


```

接收

```jsx
import { withRouter } from 'next/router' //引入withRouter


const A = ({router}) =>{  //可以在组件中获取到传递过来的参数了
    return (
        <span>{router.query.id}</span>
    )
}

export default withRouter(A) //包裹组件
```

#### 路由映射

as

跳转后的路由在页面显示：127.0.0.1:3000/a?id=1 

但是有时候并不想这样显示，而是以比较友好的方式显示出来，例如: 127.0.0.1:3000/a/1

这样就用到路由映射

```jsx
import { Button } from 'antd'
import Link from 'next/link'
import Router from 'next/router'

export default () => {

    function gotoTestB(){
        Router.push({
            pathname:'/test/b',
            query:{
                id:2
            }
        },'/test/2') //push的第二个参数可以加上 显示的路径
    }

    return (
        <>
            <Link href="a?id=1" as="a/1"> //加上 as属性。as就是页面显示的路径
                <Button type="primary">index</Button>
            </Link>
            <Button onClick={gotoTestB}>test b</Button>
        </>
    )
}


```

##### 注意：使用这个方法是有一个问题的，就是在跳转后的页面刷新之后就会变成404,因为服务端是没有这个路由的，只不过是通过一种手段来让客户端的路径变成这样子而已

### 数据获取方式

getInitialProps

```jsx
import { withRouter } from 'next/router'


const A = ({ router, name }) => {
    return (
        <span>{router.query.id}{name}</span>
    )
}

A.getInitialProps = () => {   //在这里返回的数据可以在组件的porps属性获取到
    return {
        name: '小红'
    }
}



export default withRouter(A)
```

注意，这个方法只能在pages文件下面的组件使用，因为pages下面的每个文件都相当于一个页面，components文件夹下的组件是不可以使用的

这个方法不单可以在服务器上调用，还可以在客户端调用，这样就实现了客户端和服务器同步

#### ajax获取数据

```jsx
import { withRouter } from 'next/router'
import axios from 'axios'

const A = ({ router, name }) => {
    return (
        <span>{router.query.id}{name}</span>
    )
}

A.getInitialProps = async () => {
    
    const promise = new Promise((resolve)=>{
        axios('http.....').then((res)=>{
            resolve(res)
        })
    })
    return await promise
}


export default withRouter(A)
```

### 自定义app

#### 作用

- 固定布局
- 保持一些公用状态
- 给页面传入一些自定义数据
- 自定义错误处理