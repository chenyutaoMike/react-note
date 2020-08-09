# React-Router

### 安装

```
$ npm install --save react-router-dom
或者
$ yarn add react-router-dom
```

### 使用

> BrowserRouter路由
>
> 它是基于H5中history API(pushState,replaceState,popstate)来保持UI和URL的同步，真实项目中应用的不多，一般只有当前项目是基于服务器端渲染的，我们才会使用浏览器路由

````react
import React from 'react';
import Home from './pages/Home'
import Mine from './pages/Mine'
import {
    BrowserRouter as Router,
    Route,
  } from "react-router-dom";

function App() {
    return (
        <div>
            <Router>
                <Route path="/home" component={Home}></Route>
                <Route path="/Mine" component={Mine}></Route>

            </Router>
           
        </div>
    );
}

export default App;
````

>  HashRouter模式
>
>  客户端渲染，经常使用的是哈希路由来完成，它依据相同的页面地址，不同的哈希值，来规划当前页面中的哪一个组件呈现渲染，它基于原生JS构造了一套类似于history API的机制，每一次路由切换都基于 history stack 完成的！

1.当前项目一旦使用HashRouter，则默认在页面的地址后面加上“#/”，也就是Hash默认值是一个斜杠，一般让其显示首页组件信息内容

2.HashRouter中只能出现一个子元素

3.HashRouter机制中，我们需要根据哈希地址不同，展示不同的组件内容，此时需要使用Route

#### Route

path: 设置匹配地址，但是默认不是严格匹配，当前页面哈希地址只要包含完整的它（内容是不变的），都能被匹配上

````js
path='/' : 和它匹配的地址只有斜杆即可（都能和它匹配）
path='/user':'#/user/ligin' 也可以匹配， 但是'#/user2'这个无法匹配
````

component：一旦哈希值和当前Route的path相同了，则渲染component指定的组件

````jsx
 import {
    HashRouter,   //把这里改为HashRouter就变为哈希模式了
    Route,
  } from "react-router-dom";

<HashRouter>
         <div>
            <Route path="/" component={A}/>
            <Route path="/b" component={B}/>
            <Route path="/C" component={C}/>
         </div>
   </HashRouter>
````



> link
>
> to[string]：跳转到指定的路由地址
>
> to[object]：可以提供一些参数配置项（和redirect类似）
>
> {
>
> ​	pathname：跳转地址
>
> ​	search：问号传参
>
> ​	state：基于这个方式传递信息
>
> }
>
> replace:flase 是替换history stack中当前的地址(true),还是追加一个新的地址（false）
>
> 原理：基于Link组件渲染，渲染后的结果就是一个A标签，to对应的信息最后变为href中的内容
>
> react-router中提供的组件都要在任何一个router（hash-router）包裹的范围内使用
>
> NavLink：和Link类似，都是为了实现路由切换跳转的，不同在于，NavLink组件在当前页面Hash和组件对应地址相吻合的时候，会默认给组件加一个active样式，让其有选中状态

````react
import React from 'react';
import Home from './pages/Home'
import Mine from './pages/Mine'
import {
    BrowserRouter as Router,
    Route,
    Link
  } from "react-router-dom";

function App() {
    return (
        <div>
            <Router>
                <ul>
                    <li>
                        <Link to="/home">Home页面</Link>
                    </li>
                    <li>
                        <Link to="/mine">mine</Link>
                    </li>
                </ul>
                <Route path="/home" component={Home}></Route>
                <Route path="/mine" component={Mine}></Route>
            </Router>
           
        </div>
    );
}

export default App;
````

> exact 匹配规则
>
> - 加上exact代表进行精准匹配，只有精准匹配到的才会显示

````react
import React from 'react';
import Home from './pages/Home'
import Mine from './pages/Mine'
import Ucenter from './pages/Ucenter'
import {
    BrowserRouter as Router,
    Route,
    Link
} from "react-router-dom";

function App() {
    return (
        <div>
            <Router>
                <ul>
                    <li>
                        <Link to="/">Home页面</Link>
                    </li>
                    <li>
                        <Link to="/mine">mine</Link>
                    </li>
                    <li>
                        <Link to="/mine/ucenter">ucenter</Link>
                    </li>
                </ul>
                <Route exact path="/" component={Home}></Route>  
                <Route exact path="/mine" component={Mine}></Route>
                <Route path="/mine/ucenter" component={Ucenter}></Route>
            </Router>

        </div>
    );
}

export default App;
````

> strict 
>
> 更精准匹配
>
> `http://localhost:3000/mine/`这个路径后面多了一个斜杠,路由也是会匹配上mine这个组件的
>
> 如果想多出/不进行匹配，那么可以使用strict 

````react
  <Route exact path="/" component={Home}></Route>
  <Route exact strict path="/mine" component={Mine}></Route>
  <Route strict path="/mine/ucenter"  component={Ucenter}></Route>
````

> Switch :只显示一个
>
> 404页面可以通过这个标签来实现

````react
import React from 'react';
import Home from './pages/Home'
import Mine from './pages/Mine'
import Ucenter from './pages/Ucenter'
import NotFound from './pages/NotFound'
import {
    BrowserRouter as Router,
    Route,
    Link,
    Switch  //引入
} from "react-router-dom";

function App() {
    return (
        <div>
            <Router>
                <ul>
                    <li>
                        <Link to="/">Home页面</Link>
                    </li>
                    <li>
                        <Link to="/mine">mine</Link>
                    </li>
                    <li>
                        <Link to="/mine/ucenter">ucenter</Link>
                    </li>
                </ul>
                <Switch>  //包裹住，每次只会显示一个路由
                    <Route exact path="/" component={Home}></Route>
                    <Route exact strict path="/mine" component={Mine}></Route>
                    <Route strict path="/mine/ucenter" component={Ucenter}></Route>
                    <Route component={NotFound}></Route>
                </Switch>
            </Router>

        </div>
    );
}

export default App;
````

> render 
>
> render:当页面的哈希地址和path匹配，会把render规划的方法执行，在方法中一般做“权限校验”（渲染组件之前验证是否存在权限，不存在做一些特殊处理）
>
> 这个参数可以用来显示页面，也可以用来传递数据

````react
import React from 'react';
import Home from './pages/Home'
import Mine from './pages/Mine'
import Ucenter from './pages/Ucenter'
import NotFound from './pages/NotFound'
import RenderDemo from './pages/RenderDemo'
import {
    BrowserRouter as Router,
    Route,
    Link,
    Switch
} from "react-router-dom";

function App() {
    return (
        <div>
            <Router>
				<Route path="/demo" 
                    //使用render这个属性把props这个属性传递过去
                    render={(props)=> <RenderDemo {...props}/>} //组件用{...props}展开接收
                 >
                </Route>
                 
            </Router>

        </div>
    );
}

export default App;
````

组件中：

> 组件中就可以接收到props这个属性
>
> 控制台：
>
> 1. history: {length: 44, action: "POP", location: {…}, createHref: *ƒ*, push: *ƒ*, …}
> 2. location: {pathname: "/demo", search: "", hash: "", state: undefined}
> 3. match: {path: "/demo", url: "/demo", isExact: true, params: {…}}
> 4. staticContext: undefined
> 5. __proto__: Object

````react
import React from 'react';

const RenderDemo = (props) =>{
  console.log(props)
  return (
    <div>
        demo页面
    </div>
  )
}

export default RenderDemo
````

> NavLike
>
> - 选中的时候高亮显示 ，当你点击的时候会自动添加一个 .active类名
> - 使用activeClassName属性，可以更改想要的高亮类名
> - 使用activeStyle={{color:'red'}} 这个属性，可以随意的更改颜色
> - exact & strice 控制匹配的时候是否是严格匹配
> - isActive：匹配后执行对应的函数
>
> NavLink不是点击谁，谁有选中的样式（但是可以路由切换），而是当前页面哈希后的地址和NavLink中的to进行比较，哪个匹配了，哪个才有选中的样式

```react
import React from 'react';
import Home from './pages/Home'
import Mine from './pages/Mine'
import Ucenter from './pages/Ucenter'
import NotFound from './pages/NotFound'
import RenderDemo from './pages/RenderDemo'
import './pages/style.css'
import {
    BrowserRouter as Router,
    Route,
    Link,
    Switch,
    NavLink
} from "react-router-dom";

function App() {
    return (
        <div>
            <Router>
                <ul>
                    <li>    //activeClassName
                        <NavLink exact activeClassName="selete" to="/">Home页面</NavLink>
                    </li>
                    <li>         //activeStyle
                        <NavLink activeStyle={{color:'skyblue'}} exact to="/mine">mine</NavLink>
                    </li>
                    <li>
                        <NavLink exact to="/mine/ucenter">ucenter</NavLink>
                    </li>
                </ul>
                <Switch>
                    <Route exact path="/" component={Home}></Route>
                    <Route exact strict path="/mine" component={Mine}></Route>
                    <Route strict path="/mine/ucenter" component={Ucenter}></Route>
                    <Route path="/demo" render={(props)=> <RenderDemo {...props}/>}></Route>
                    <Route component={NotFound}></Route>
                </Switch>
            </Router>

        </div>
    );
}

export default App;
```

> 路由传参
>
> 第一种:

````react
<Route strict path="/mine/ucenter/:id" component={Ucenter}></Route>
````

跳转的时候：

````react
  <li>
  <NavLink exact to="/mine/ucenter/1001">ucenter</NavLink>
 </li>
````

组件中接收：

````react
import React from 'react';


export default function Ucenter(props) {
  
  return (
    <div>Ucenter
          <h1>{props.match.params.id}</h1>
    </div>
  )
}


````

> 第二种
>
> 通过 new URLSearchParams(props.location.search) 获取参数
>
> 例如页面的路径是这样的：`http://localhost:3000/mine?name=zhangsan&age=20`
>
> props.location.search =>这个参数会返回 `?name=zhangsan&age=20`问号后面的字符串

可以这样实现

````react
import React, { Component } from 'react';

class Mine extends Component{
  constructor(props){
    super(props)
    const params = new URLSearchParams(props.location.search)
    console.log(params)
    console.log(params.get("name")) //这里就可以读到页面传递过来的内容
    console.log(params.get("age"))
  }
  render(){
   
    return (
      <div>Mine</div>
    )
  }
}

export default Mine
````

> NavLink、Link的 to
>
> to属性可以传递一个Object过去

````react
 						 <NavLink
                            exact
                            activeClassName="selete"
                            to={{
                                pathname: "/",  // 跳转的路径
                                search: "?name=xiaohong&age=22", //路径参数
                                state: { referrer: "传递属性" } //这个属性可以传递一些参数过去，URL地址栏看不到，但是可以接收到
                            }}
                        >Home页面</NavLink>
````

组件中接收：



````react
import React from 'react';

const Home =  (props) =>{
 
  console.log(props.location)   pathname: "/"
								search: "?name=xiaohong&age=22"
								hash: ""
								state:
								referrer: "传递属性" //这里就是state中传递过来的属性
								__proto__: Object
								key: "p4xcib"
    return (
      <div>Home</div>
    )
  
}

export default Home
````

> Redirect 重定向
>
> 用于登陆重定向等等业务
>
> {
>
> ​	pathname:定向的地址，
>
> ​	search:给定向地址问号传参（结合当前案例，真实项目中，我们有时候会根据是否存在问号参数值来统计是正常进入首页还是非正常跳转过来的，也有可能根据问号传参值做不同的事情）
>
> ​	state:给定向后的组件传递一些信息
>
> ​	push：如果设置了这个属性，当前跳转的地址会加入到history stack中一条记录
>
> ​	from：设定当前来源的页面地址
>
> ​		`<Redirect from='/custom' to=' /curstom/list' />`
>
> }

````react
import React from 'react';
import {Redirect} from 'react-router-dom'
class Shop extends React.Component {
  state = {
    flag : true
  }
  render(){
    return (
      <div>
         {
           this.state.flag ? <div>商品页面</div>   //如果为true就显示商品页面
           : <Redirect to="/" />   //如果为false就跳回首页
         }
      </div>
    )
  }
}


export default Shop
````

````jsx
<Redirect to={{pathname:'/'.search:'?lx=404'}} />
````



> 使用js跳转

````js
handleClick = () =>{
  //  this.props.history.push("/")   //普通跳转
   this.props.history.replace("/")
  }
````

> withRouter 
>
> 高阶组件
>
> 如果你的组件没有被Router标签包裹住。那么它的props是没有location这个属性的
>
> 所以也不能跳转，如果想要进行跳转或者是其他的，就要使用withRouter 这个高阶组件进行包裹

````react
import React, { Component } from 'react';
import {withRouter} from 'react-router-dom'
class MineDemo extends Component{

  handlClick = () =>{
    this.props.history.replace("/")
  }
  render(){
    return (
      <div>
        MineMemo
        <button onClick={this.handlClick}></button>
      </div>
    )
  }
}

export default withRouter(MineDemo)
````

> 路由嵌套

````react
<Switch>
    <Route exact path="/" component={Home}></Route>
    <Route exact strict path="/mine" component={Mine}></Route>
    <Route strict path="/mine/ucenter/:id" component={Ucenter}></Route>
    <Route path="/demo" render={(props) => <RenderDemo {...props} />}></Route>
    <Route exact  path="/shop" component={Shop}></Route>
    <Book>
       <Switch>  //二级路由
             <Route  path="/book/javabook" component={JavaBook}></Route>
              <Route  path="/book/webbook" component={WebBook}></Route>
       </Switch>
     </Book>
      <Route component={NotFound}></Route>
 </Switch>
````

要显示二级路由，就要在Book这个组件里面写上props.children属性

Book.js:

````react
import React, { Component } from 'react';

class Book extends Component {
  render(){
    return (
      <div>
        book页面
        {this.props.children}  //这里面是二级路由
      </div>
    )
  }
}
export default Book
````

### 受路由管控组件的一些特点

1.只有当前页面的哈希地址(/#/...)和路由指定的地址(path)匹配，才会把对应的组件渲染(withRouter是没有地址匹配，都被模拟成受路由管控的)

2.路由切换的原理，凡是匹配的路由，都会把对应的组件内容，重新添加到页面中，相反，不匹配的都会在页面中移除掉，下一次重新匹配上，组件需要重新渲染到页面中；每一次路由切换的时候(页面的哈希路由地址改变)，都会从一级路由开始重新校验一遍

3.所有受路由管控的组件，在组件的属性props上，都默认添加了三个属性

- history
  - push ：向池子追加一条新的信息，达到切换到指定路由地址的目的
    - this.props.history.push('/plan‘) js中实现路由切换
  - go： 跳转到指定的地址（传的是数字 0当前 -1上一个 -2上两个...）
  - goBack：等价于go(-1) 回退到上一个地址
  - goForward：等价于go(1) 向前走一步

-  location：获取当前哈希路由渲染组件的一些信息
  - pathname：当前哈希路由地址    /custom/list
  - search：当前页面的问号传参值   ?/lx=unsafe
  - state：基于redirect/Link/NavLink中的to,传递是一个对象，对象编写的state,就可以在location.state中获取到
  - 

- match：获取的是当前路由匹配的一些结果
  - params：如果当前路由匹配的是地址路径传参，则这里可以获取传递参数的值