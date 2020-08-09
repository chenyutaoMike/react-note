# react后台管理项目

### 初始化项目

````bash
npx react-react-app myadmin
````

### 项目描述

> 1) 此项目为一个前后台分离的后台管理的 SPA, 包括前端 PC 应用和后端应用 
>
> 2) 包括用户管理 / 商品分类管理 / 商品管理 / 权限管理等功能模块 
>
> 3) 前端: 使用 React 全家桶 + Antd + Axios + ES6 + Webpack 等技术 
>
> 4) 后端: 使用 Node + Express + Mongodb 等技术 
>
> 5) 采用模块化、组件化、工程化的模式开发 

### 项目基本结构

![image-20200108215612127](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200108215612127.png)

### 引入antd，并实现按需打包

 安装antd

````bash
yarn add antd
````

#### 实现组件的按需打包

##### 第一步:下载依赖模块

````bash
yarn add react-app-rewired customize-cra babel-plugin-import
````

##### 第二步:配置config-overrides.js文件

在根目录下创建一个`config-overrides.js`文件，进行配置:

````js
const { override, fixBabelImports } = require('customize-cra');
module.exports = override( 
  //针对antd实现按需打包: 根据import来打包(使用babel-plugin-import这个插件进行的)
  fixBabelImports('import', { 
    libraryName: 'antd', 
    libraryDirectory: 'es', 
    style: 'css', 
  }), )
;
````

##### 第三步: 修改package.json的配置

> 直接复制覆盖原来的scripts

````json
 "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-scripts eject"
  }
````

按需打包完成

#### 自定义antd主题

> 自定义主题颜色

##### 第一步：下载less 和 less-loader工具包

````bash
yarn add less less-loader
````

##### 第二步：修改 config-overrides.js

````js
const { override, fixBabelImports , addLessLoader} = require('customize-cra');
module.exports = override( 
  //针对antd实现按需打包: 根据import来打包(使用babel-plugin-import这个插件进行的)
  fixBabelImports('import', { 
    libraryName: 'antd', 
    libraryDirectory: 'es', 
    style: true, 
  }), 
    //使用 less-loader 对源码中的less的变量进行覆盖
  addLessLoader({ 
    javascriptEnabled: true,
     modifyVars: {'@primary-color': '#1DA57A'},  //配置主题颜色
     }),
);



````

定义主题官方文档：https://ant.design/docs/react/customize-theme-cn

### 引入路由

安装

````bash
yarn add react-router-dom
````

配置路由

> 在pages文件夹下创建两个子文件夹，分别对应的是login和admin，并且在里面创建组件
>
> │  
> ├─admin
> │      admin.jsx
> │      
> └─login
>         login.jsx
>         

login.jsx:

````react
import React, { Component } from 'react';

/**
 *  登陆的路由组件
 */

export class Login extends Component {
  render() {
    return (
      <div>
        login
      </div>
    );
  }
}

export default Login;

````

admin.jsx:

````react
import React, { Component } from 'react';

/**
 * 后台管理路由组件
 */

class Admin extends Component {
  render() {
    return (
      <div>
        admin
      </div>
    );
  }
}

export default Admin;
````

然后在App.js是引入配置路由

App.js:

````react
import React, { Component } from 'react';
import { BrowserRouter, Route, Switch } from 'react-router-dom'

import Login from './pages/login/login'
import Admin from './pages/admin/admin'

class App extends Component {

  render() {
    return (
      <BrowserRouter>
        <Switch>     {/* 只匹配一个 */}
          <Route path="/login" component={Login}></Route>
          <Route path="/" component={Admin}></Route>
        </Switch>
      </BrowserRouter>
    );
  }

}

export default App;

````

### 引入reset.css

> 在public文件夹下创建一个css文件，里面创建一个reset.css文件，然后在index.html中用link引入即可

### 搭建Login页面

> 可以在login文件夹下面创建一个 login.less的文件来控制样式

login.jsx:

````react
import React, { Component } from 'react';
import './login.less';
import logo from '../../assets/images/logo.png'  //引入图片
/**
 *  登陆的路由组件
 */

export class Login extends Component {
  render() {
    return (
      <div className="login">
        <header className="login-header">
          <img src={logo} alt=""/> //使用图片
          <h1>后台管理系统</h1>
        </header>
        <section className="login-content">
          <h2>用户登陆</h2>
          <div>Form组件标签</div>
        </section>
      </div>
    );
  }
}

export default Login;

````

在Login组件使用antd的Form表单

> 这个页面是个静态页面
>
> `<Inpit />`里面的prefix属性 :这个perfix里面可以传递字符串或者React节点，所以这里传递了一个Icon组件，antd已经定义好的组件，type就是样式，可以去antd官网找相应的样式

````react
import React, { Component } from 'react';
import { Form, Icon, Input, Button } from 'antd'; //引入需要的组件
import './login.less';
import logo from '../../assets/images/logo.png'
/**
 *  登陆的路由组件
 */

export class Login extends Component {
  handleSubmit = e => {  //提交会触发的事件
    e.preventDefault();
    this.props.form.validateFields((err, values) => {
      if (!err) {
        console.log('Received values of form: ', values);
      }
    });
  };
  render() {
    return (
      <div className="login">
        <header className="login-header">
          <img src={logo} alt="" />
          <h1>后台管理系统</h1>
        </header>
        <section className="login-content">
          <h2>用户登陆</h2>
          <Form onSubmit={this.handleSubmit} className="login-form">
            <Form.Item>
              <Input
                prefix={<Icon type="user" style={{ color: 'rgba(0,0,0,.25)' }} />}    //这个perfix里面可以传递字符串或者React节点，所以这里传递了一个Icon组件，antd已经定义好的组件，type就是样式，可以去antd官网找相应的样式
                placeholder="Username"
              />
            </Form.Item>
            <Form.Item>
              <Input
                prefix={<Icon type="lock" style={{ color: 'rgba(0,0,0,.25)' }} />}
                type="password"
                placeholder="Password"
              />
            </Form.Item>
            <Form.Item>
              <Button type="primary" htmlType="submit" className="login-form-button">
                登陆
              </Button>
            </Form.Item>
          </Form>
        </section>
      </div>
    );
  }
}

export default Login;

````

#### antd的Form表单写法

> 要想使用form表单的功能，第一步要先用`Form.create()(Login)`这个高阶函数包裹住组件
>
> 这时候`this.props`就有一个form对象了，里面包含了很多方法
>
>  `getFieldDecorator()`的用法：
>
> `getFieldDecorator`('标识名称',{里面是匹配规则})(标签组件)
>
> `this.props.form.getFieldsValue()`这个函数返回的是一个对象

````react
import React, { Component } from 'react';
import { Form, Icon, Input, Button } from 'antd';
import './login.less';
import logo from '../../assets/images/logo.png'

class Login extends Component {
  handleSubmit = (event) => {
    event.preventDefault(); //阻止事件的默认行为
    const valus = this.props.form.getFieldsValue();
    console.log(valus) //这里拿到的是对象{username: "213123", password: "24124"}
  };
  render() {
    //得到具收集表单验证，验证表单数据的对象form
    // const form =this.props.form;
    const { getFieldDecorator } = this.props.form;

    return (
      <div className="login">
        <header className="login-header">
          <img src={logo} alt="" />
          <h1>后台管理系统</h1>
        </header>
        <section className="login-content">
          <h2>用户登陆</h2>
          <Form onSubmit={this.handleSubmit} className="login-form">
            <Form.Item>
              {/* username:标识名称，以后收集表单数据的时候用到的，username的下一个参数就是表单验证规则*/}
              {getFieldDecorator('username', {
                rules: [{ required: true, message: 'Please input your username!' }],
              })(<Input
                prefix={<Icon type="user" style={{ color: 'rgba(0,0,0,.25)' }} />}
                placeholder="用户名"
              />)}

            </Form.Item>
            <Form.Item>
              {getFieldDecorator('password', {
                rules: [{ required: true, message: 'Please input your username!' }],
              })(<Input
                prefix={<Icon type="lock" style={{ color: 'rgba(0,0,0,.25)' }} />}
                type="password"
                placeholder="密码"
              />)}

            </Form.Item>
            <Form.Item>
              <Button type="primary" htmlType="submit" className="login-form-button">
                登陆
              </Button>
            </Form.Item>
          </Form>
        </section>
      </div>
    );
  }
}
/**
 * 1.高阶函数
 * 2.高阶组件
 */
const WrapLogin = Form.create()(Login);
export default WrapLogin;

````

#### 高阶函数和高阶组件的概念

高阶函数

> 1.高阶函数
>
>  	1). 一类特别的函数
>	
>  		a. 接收函数类型的参数
>
> ​		 b. 返回值是函数
>
> ​	2). 常见的高阶函数
>
>    	 a. 定时器: setTimeout() / setInterval()
>
> ​		b. Promise: Promise(() => {}) then(res => {}, err => {})
>
>  		c. 数组遍历相关的方法:forEach()/filter()/map()/reduce()/find()/findIndex()
>	
>  		d. 函数对象的bind()
>	
>  		e. Form.create()(Login) / getFieldDecorator()()
>
> ​	3). 高阶函数更加动态，更加具有扩展性

高阶组件

> 2.高阶组件
>
> ​	1). 本质就是一个函数
>
> ​	2). 接收一个组件(被包装组件)，返回一个新的组件(包装组件),包装组件会向被包装组件传入特定属性
>
> ​	3). 作用：扩展组件的功能
>
> ​	4). 高阶组件也是高阶函数：接收一个组件函数，返回一个新的组件函数



#### 前台表单验证

> 利用antd提供的方法进行前台表单验证
>
> 用户名/密码的合法性要求: 
>
> 1. 必须输入
>
> 2. 必须大于等于* *4* *位* 
>
> 3. 必须小于等于* *12* *位* 
>
> 4. 必须是英文、数字或下划线组成* 

#### Form表单校验规则

> 第一种：声明式验证
>
> 直接写在了`getFieldDecorator('xxx',{声明式验证写在这里})()`
>
> required:必须填写
>
> whitespace: 不能有空格
>
> min： 最少长度
>
> max：最大长度
>
> pattern：正则验证
>
> 更多内容需要查文档：https://ant.design/components/form-cn/

````react
 <Form onSubmit={this.handleSubmit} className="login-form">
            <Form.Item>
              {getFieldDecorator('username', {
                //声明式验证：直接使用别人定义好的验证规则进行验证
                rules: [
                  { required: true,whitespace:true, message: '用户必须输入' },
                  { min: 4, message: '用户名至少4位' },
                  { max: 12, message: '用户名最多12位' },
                  { pattern: /^[a-zA-Z0-9_]+$/, message:'用户名必须是英文、数字或下划线组成' }
                ],
              })(<Input
                prefix={<Icon type="user" style={{ color: 'rgba(0,0,0,.25)' }} />}
                placeholder="用户名"
              />)}

            </Form.Item>
````

> 第二种：自定义验证
>
> getFieldDecorator('password', { 
>
> ​	rules: [ 
>
> ​	{ validator: this.validatePwd }
>
> ]})(xxx)
>
> ​    自定义验证：
>
>   rules: [ { validator: this.validatePwd }]   => validator是固定写法: this.validatePwd是自定义函数

````react
 validatePwd = (rule, value, callback) => {
    // callback()  //不传递参数，代表验证成功
    // callback('xxx') //传递参数，代表验证失败，传递过去的是提示内容
    if (!value) {
      callback('密码必须输入')
    } else if (value.length < 4) {
      callback('密码长度不能小于4位')
    } else if (value.length > 12) {
      callback('密码长度不能大于12位')
    } else if (!/^[a-zA-Z0-9_]+$/.test(value)) {
      callback('密码必须是英文、数字或下划线组成')
    }
    callback()
  }
 ======================================================
   //组件部分
       <Form.Item>
              {getFieldDecorator('password', {
                rules: [
                  { validator: this.validatePwd }
                ],
              })(<Input
                prefix={<Icon type="lock" style={{ color: 'rgba(0,0,0,.25)' }} />}
                type="password"
                placeholder="密码"
              />)}

            </Form.Item>
````

#### Form表单指定初始值

> 在`rules`后面加上`initialValue`就可以指定初始值

````react
 {getFieldDecorator('username', {
              
                rules: [
                  { required: true, whitespace: true, message: '用户必须输入' },
                  { min: 4, message: '用户名至少4位' },
                  { max: 12, message: '用户名最多12位' },
                  { pattern: /^[a-zA-Z0-9_]+$/, message: '用户名必须是英文、数字或下划线组成' }
                ],
                initialValue:'admin' //指定初始值
              })(xxxx)
````



#### 提交Form表单

> 使用`handleSubmit`函数进行表单提交  => 这个事件是Form组件上面的`onSubmit`事件
>
> ` <Form onSubmit={this.handleSubmit} className="login-form">`
>
> 配合`this.props.form.validateFields`函数进行表单验证和数据收集

````react
handleSubmit = (event) => {
    event.preventDefault(); //阻止事件的默认行为
    //对所有表单字段进行校验
    this.props.form.validateFields((err, values) => {
        //err就代表校验有错误，values就是表单的数据
      if (!err) {
        //没有错误，就代表校验成功
        console.log('校验成功，提交请求', values);
      } else {
        //校验失败
        console.log('校验失败')
      }
    });
  };
````

### 封装axios

在api文件夹下创建两个文件：`axios.js`、`index.js`

`axios.js`:用来封装axios请求

````js
/**
 * 发送异步ajax请求的函数模块
 * 封装axios库
 * 函数的返回值是promise对象
 * 1. 优化：统一处理请求异常
 *    在外层包一个自己创建的Promise对象
 *    在请求出错时，不reject(error)，而是显示错误提示
 * 2. 优化2：异步得到不是reponse,而是response.data
 *    在请求成功resolve时: resolve(response.data)
 */
import axios from 'axios'
import { message } from 'antd'
export default function ajax(url, data = {}, type = 'GET') {
  return new Promise((resolve, reject) => {  //返回一个promise函数
    let promise;
    if (type === 'GET') {   //发送get请求
      promise = axios.get(url, {  //配置对象
        params: data   //指定请求参数 
      })
    } else {  //发送post请求
      promise = axios.post(url, data)
    }
    promise.then(response => {   //成功的回调
      resolve(response.data)
    }).catch(error => {   //请求出错，在这里处理请求错误，而不是用reject返回错误
      message.error('请求出错:' + error.message)
    })

  })
 
}
````

`index.js`: 用来包装请求

````js
/**
 * 包含应用中所有接口请求函数的模块
 * 每个函数的返回值都是promise
 */

import ajax from './ajax'
// export function reqLogin(username,password){
//  return ajax('/login',{username,password},'POST')
// }
//简写
const BASE = '';   //基本请求路径
//登陆请求接口封装
export const reqLogin = (username, password) => ajax(BASE + '/login', { username, password }, 'POST')
//增加新用户请求接口封装
export const reqAddUser = (user) => ajax(BASE + '/manage/user/add', user, 'POST')
````

在组件中使用：

````react
import { reqLogin } from '../../api'  //导入

 handleSubmit = (event) => {   //提交表单触发的事件
    event.preventDefault(); 
     // 使用 async await 实现同步编码 
    this.props.form.validateFields(async(err, values) => { 
      if (!err) {
        const {username,password} = values
        const result = await reqLogin(username,password)
        console.log(result)
      } else {
       
        console.log('校验失败')
      }
    });
  };
````

### async和await

> 1.作用
>
> - 简化promise对象的使用
>   - 不用再使用then()来指定成功/失败的回调函数
>   - 以同步编码(没有回调函数)方式实现异步流程
>
> 2.哪里写await?
>
> - 在返回Promise的表达式左侧写 await: 不想要promise,想要promise异步执行的成功的数据
>
> 3.哪里写async?
>
> - await所在函数(最近的)定义的左侧写async



### 配置代理解决开发中的跨域问题

在`package.json`中配置

````json
"proxy": "http://localhost:5000"
````

### 维持登录的两种方式

##### 第一种:

> 第一种，不使用redux的情况下

第一步：在utils文件夹下创建两个文件：`memoryUtils.js`  和 `storageUtils.js`

 `memoryUtils.js`：用来存储用户信息

`storageUtils.js`：用来设置localStorage信息

第二步: 初始化 `memoryUtils.js`

````js
/**
 * 用来在内存保存一些数据的工具函数
 */

export default {
  user: {} //保存当前登陆的user
}
````

第三步: 写好`storageUtils.js`里面的工具函数

> 这里用到了store这个js库，是用来简化操作localStorage的
>
> 安装：`yarn add store`
>
> `localStorage.setItem(USER_KEY, JSON.stringify(user))` 和` store.set(USER_KEY, user)` 这两个命令是完全相等的，只不过第二条用了store这个库，简化了这个操作而已
>

````js
import store from 'store'
const USER_KEY = 'user_key'
export default {
  /**
   * 保存user
   */
  saveUser(user) {
    // localStorage.setItem(USER_KEY, JSON.stringify(user))
    store.set(USER_KEY, user)
  },
  /**
   * 读取user
   */
  getUser() {
    // return JSON.parse(localStorage.getItem(USER_KEY) || '{}')
    return store.get(USER_KEY)
  },

  /**
   * 删除user
   */
  removeUser() {
    // localStorage.removeItem(USER_KEY)
    store.remove(USER_KEY)
  }
}
````

第四步:  发送登陆请求之后存储登陆信息

````js
handleSubmit = (event) => {
    event.preventDefault(); 
  
    this.props.form.validateFields(async (err, values) => {
      if (!err) {
       
        const { username, password } = values
        const result = await reqLogin(username, password)
        if (result.status === 0) {
          
          message.success('登陆成功')
           //登陆成功之后，拿到需要的user信息
          const user = result.data
          memoryUtils.user = user  //把user信息存到memoryUtils中去
          storageUtils.saveUser(user) //把user信息存到localStorage中
          this.props.history.replace('/')
        } else {
       
          message.error(result.msg)
        }
      } else {
      
        console.log('校验失败')
      }
    });
  };
````

第五步: 为了防止刷新数据丢失，在主入口文件index.js中处理登陆

````js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import storageUtils from './utils/storageUtils'
import memoryUtils from './utils/memoryUtils'

//读取local中保存user,保存在内存中
memoryUtils.user = storageUtils.getUser()

ReactDOM.render(<App />, document.getElementById('root'));

````

第六步：验证登陆

````react
 render() {
    //如果用户已经登陆，自动跳转到管理界面
    const user = memoryUtils.user
    if (user && user._id) {
      return <Redirect to="/" />
    }
 }
````

##### 第二种：

> 第二种：使用redux的情况

第一步：走常规流程，先把登陆后的用户信息保存在store中，这个并不难

第二步：持久化store中的用户信息

​				1.在主入口文件`index.js`中添加以下内容

````js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import storageUtils from './utils/storageUtils'
import { Provider } from 'react-redux'
import store from './store

//把存在local中的user读取出来,然后通过dispatch事件发送消息，并且把user传递过去
const user = storageUtils.getUser()
store.dispatch({type:'upDateUser',user:user})

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>
  , document.getElementById('root'));

````

第三步：在reducer.js中处理这个事件：

````js

import { SET_USER } from './actionTypes'

const defaultVale = {
  user: {}
}
export default (state = defaultVale, action) => {
  switch (action.type) {
    case SET_USER:
      const newState = JSON.parse(JSON.stringify(state))
      newState.user = action.user
      return newState
    case 'upDateUser':
        // 持久化user
      return Object.assign({}, state, {
        user: action.user
      })
    default:
      return state
  }
}
````

完成，这样每次刷新页面都不会丢失登录信息

### 使用antd的Layou布局

> 引入 antd 的 Layou，然后配合自己的组件形成一个页面

````react
import { Layout } from 'antd';
import LeftNav from '../../components/left-nav'
import Header from '../../components/header'
const { Footer, Sider, Content } = Layout;
class Admin extends Component {
  render() {
    const user = memoryUtils.user
    if (!user || !user._id) {
      return <Redirect to="/login" />
    }
    return (
        //layout布局
      <Layout style={{ height: '100%' }}>
        <Sider>
          <LeftNav /> //自定义左侧导航
        </Sider>
        <Layout> 
          <Header>Header</Header>  //这是自定义的header组件
          <Content style={{backgroundColor:'#fff'}}>Content</Content>
          <Footer style={{textAlign:'center',color:'#ccc'}}>推荐使用谷歌浏览器</Footer>  //可以在style里面加上样式，调节页面
        </Layout>
      </Layout>
    );
  }
}

export default Admin;
````

#### 配置二级路由

````react
import React, { Component } from 'react';
import memoryUtils from '../../utils/memoryUtils'
import { Redirect, Switch, Route } from 'react-router-dom'
import { connect } from 'react-redux'
import { Layout } from 'antd';
import LeftNav from '../../components/left-nav'
import Header from '../../components/header'
import Home from '../home/home'
import Category from '../category/category'
import Product from '../product/product'
import User from '../user/user'
import Bar from '../charts/bar'
import Pie from '../charts/pie'
import Line from '../charts/line'


const { Footer, Sider, Content } = Layout;
/**
 * 后台管理路由组件
 */

class Admin extends Component {
  render() {
    const user = memoryUtils.user

    //如果内存没有存储user ===> 证明当前没有登陆
    if (!user || !user._id) {
      return <Redirect to="/login" />
    }
    //store中的登录信息
    // if (!this.props.user || !this.props.user._id) {
    //   return <Redirect to="/login" />
    // }
    return (
      <Layout style={{ height: '100%' }}>
        <Sider>
          <LeftNav />
        </Sider>
        <Layout>
          <Header>Header</Header>
          <Content style={{ backgroundColor: '#fff' }}>
           
              <Switch>
                <Route path="/home" component={Home}/>
                <Route path="/category" component={Category}/>
                <Route path="/product" component={Product}/>
                <Route path="/user" component={User}/>
                <Route path="/charts/bar" component={Bar}/>
                <Route path="/charts/pie" component={Pie}/>
                <Route path="/charts/line" component={Line}/>
                <Redirect to="/home" />
              </Switch>
          
          </Content>
          <Footer style={{ textAlign: 'center', color: '#ccc' }}>推荐使用谷歌浏览器</Footer>
        </Layout>
      </Layout>
    );
  }
}
const mapStateToProps = (state) => {
  return {
    user: state.userReducer.user
  }
}
export default connect(mapStateToProps)(Admin);
````



#### antd的Menu导航菜单

> 用法：

````react
 render() {
    return (
      <div className="left-nav">
        <Link to="/" className="left-nav-header">
          <img src={logo} alt='' />
          <h1>后台管理</h1>
        </Link>
        <Menu
          defaultSelectedKeys={['1']}
          defaultOpenKeys={['sub1']}
          mode="inline"
          theme="dark"
        >
          <Menu.Item key="1">
            <Link>
              <Icon type="pie-chart" />
              <span>首页</span>
            </Link>
          </Menu.Item>
          <SubMenu      //这个是可展开的
            key="sub1" 
            title={
              <span>
                <Icon type="mail" />
                <span>商品</span>
              </span>
            }
          >
            <Menu.Item key="5"> //下面是可展开的项
              <Link>
                <Icon type="mail" />
                品类管理
              </Link>
            </Menu.Item>
            <Menu.Item key="6">
              <Link>
                <Icon type="mail" />
                商品管理
              </Link>
            </Menu.Item>
          </SubMenu>
        </Menu>
      </div>


    );
  }
````

##### 导航菜单配置:

##### ` config/menuConfig.js`

````js
const menuList = [
  {
    title: '首页',   // 菜单标题名称 
    key: '/home', // 对应的 path 
    icon: 'home', // 图标名称 
  },
  {
    title: '商品', 
    key: '/products',
    icon: 'appstore',
    children: [ // 子菜单列表 
      { title: '品类管理', key: '/category', icon: 'bars' },
      { title: '商品管理', key: '/product', icon: 'tool' },
    ]
  },
  {
    title: '用户管理',
    key: '/user',
    icon: 'user'
  },
  {
    title: '角色管理',
    key: '/role',
    icon: 'safety',
  },
  {
    title: '图形图表',
    key: '/charts',
    icon: 'area-chart',
    children: [
      { title: '柱形图', key: '/charts/bar', icon: 'bar-chart' },
      { title: '折线图', key: '/charts/line', icon: 'line-chart' },
      { title: '饼图', key: '/charts/pie', icon: 'pie-chart' },]
  },
]
export default menuList
````

##### 根据这个数组生成一个Menu导航菜单

> 可以用循环遍历生成相应的导航，这里就用到了两种：map,reduce
>
> 这两种都是比较好用的方式

````react
import React, { Component } from 'react';
import { Link } from 'react-router-dom'
import logo from '../../assets/images/logo.png'
import menuList from '../../config/menuConfig'  //导航配置数组
import './index.less'
import { Menu, Icon } from 'antd';

const { SubMenu } = Menu;
class LeftNav extends Component {

  /**
   * 根据menu的数据数组生成对应的标签数组
   * 使用map() + 递归调用
   */
  getMenuNodes_map = (menuList) => {
    return menuList.map(item => {
      if (!item.children) {
        return (
          <Menu.Item key={item.key}>
            <Link to={item.key}>
              <Icon type={item.icon} />
              <span>{item.title}</span>
            </Link>
          </Menu.Item>
        )
      } else {
        return (
          <SubMenu
            key={item.key}
            title={
              <span>
                <Icon type={item.icon} />
                <span>{item.title}</span>
              </span>
            }
          >
            {this.getMenuNodes(item.children)}   //递归调用
          </SubMenu>
        )
      }

    })
  }
  // reduce方式
  getMenuNodes = (menuList) => {
    return menuList.reduce((pre, item) => {
      if (!item.children) {
        pre.push(
          <Menu.Item key={item.key}>
            <Link to={item.key}>
              <Icon type={item.icon} />
              <span>{item.title}</span>
            </Link>
          </Menu.Item>
        )
      } else {
        pre.push((
          <SubMenu
            key={item.key}
            title={
              <span>
                <Icon type={item.icon} />
                <span>{item.title}</span>
              </span>
            }
          >
            {this.getMenuNodes(item.children)}  //递归调用
          </SubMenu>
        ))
      }
      return pre
    }, [])
  }
  render() {
    return (
      <div className="left-nav">
        <Link to="/" className="left-nav-header">
          <img src={logo} alt='' />
          <h1>后台管理</h1>
        </Link>
        <Menu
          defaultSelectedKeys={['1']}
          defaultOpenKeys={['sub1']}
          mode="inline"
          theme="dark"
        >
          {this.getMenuNodes(menuList)} //调用函数并传入数组数组
        </Menu>
      </div>


    );
  }
}

export default LeftNav;
````

#### 配置Menu导航菜单的一些属性

> `defaultSelectedKeys:{[path]}` => 这个属性的意思就是写在这个数组里面的路径会被选中，刷新的时候也不会取消选中

````react
//动态获取路径path的方法
const path = this.props.location.pathname;
使用这个方法的前提需要你这个组件是路由组件
如果不是路由组件，那么就需要引入withRouter(组件)对你的组件进行包装
````

> `selectedKeys;{[path]}` => 这个属性和上面的`defaultSelectedKeys`的功能是一模一样的，但是`selectedKeys`这个属性是可用动态更新的，当你的路由有重定向这些操作的时候，最好使用
>
> `selectedKeys`这个属性

> `defaultOpenKeys={[openKey]}` => 默认打开选择框，页面刷新也不会丢失
>
> 这个属性里面传的也是一个数组，难的就是难在如何确定是哪一个选择框

在遍历函数中可以查找出路径：

````react
getMenuNodes = (menuList) => {
    const path = this.props.location.pathname;
    return menuList.reduce((pre, item) => {
      if (!item.children) {
        pre.push(
          <Menu.Item key={item.key}>
            <Link to={item.key}>
              <Icon type={item.icon} />
              <span>{item.title}</span>
            </Link>
          </Menu.Item>
        )
      } else {
         //关键代码
        //能进入这里的就代表着它有子元素，它可以打开
         //根据它的子元素与路径相匹配，如果找到了，证明要打开的就是这个路径
        //查找一个与当前请求路径匹配的子Item
        const cItem = item.children.find(citem => citem.key === path)
        if (cItem) {
          //如果存在，说明当前item的子列表需要打开
          this.openKey = item.key  //把路径赋值给this.openKey
        }
        console.log(this.openKey)
        pre.push((
          <SubMenu
            key={item.key}
            title={
              <span>
                <Icon type={item.icon} />
                <span>{item.title}</span>
              </span>
            }
          >
            {this.getMenuNodes(item.children)}
          </SubMenu>
        ))
      }
      return pre
    }, [])
  }
````

在render中拿到这个路径

````react
render() {
    //得到当前的路径
    const path = this.props.location.pathname
    const openKey = this.openKey  //取出路径
    console.log(path)
    return (
      <div className="left-nav">
        <Link to="/" className="left-nav-header">
          <img src={logo} alt='' />
          <h1>后台管理</h1>
        </Link>
        <Menu
          selectedKeys={[path]}
          mode="inline"
          theme="dark"
          defaultOpenKeys={[openKey]} //把路径放在这里
        >
          {this.menuNodes}
        </Menu>
      </div>


    );
  }
````

这样选中后刷新也不会丢失了

### 定义jsonp请求的接口请求函数

使用github上的一个插件帮助我们发送jsonp请求

#### 安装

````bash
$ npm install jsonp
或者
$ yarn add jsonp
````

#### 使用

> jsonp(url,{},fn) 的三个参数
>
> 第一个参数是请求地址
>
> 第二个参数是配置对象，写个空对象代表使用默认配置
>
> 第三个参数是回调函数，回调的第一个参数是错误信息，第二个是接口返回的数据

````js
import jsonp from 'jsonp'  //引入jsonp
import ajax from './ajax'
import {message} from 'antd'

/**
 * json请求的接口函数
 */

export const reqWeather = (city) => {   //封装jsonp
  return new Promise((resolve, reject) => {
    const url = `http://api.map.baidu.com/telematics/v3/weather?location=${city}&ou tput=json&ak=3p49MVra6urFRGOT9s8UBWr2`
    jsonp(url, {}, (err, data) => {    //发送请求
     if(!err && data.status === 'success'){
      resolve(data)
     }else{
      message.error('获取天气信息失败')
     }
     
    })
  })
}

````

> `jsonp`解决ajax跨域的原理
>
> 1. jsonp只能解决GET类型的ajax请求跨域问题
>
> 2. jsonp请求不是ajax请求，而是一般的get请求
>
> 3. 基本原理
>
>    浏览器端：
>
>    ​	动态生成`<script>`来请求后台接口(src就是接口的url)
>
>    ​	定义好用于接收响应数据的函数(fn),并将函数名通过请求参数提交给后台(如: callback = fn)
>
>    服务器端：
>
>    接收到请求处理产生结果数据后，返回一个函数调用的js代码，并将结果数据作为实参传入函数调用
>
>    浏览器端：
>
>    收到响应自动执行函数调用的js代码，也就执行了提前定义好的回调函数，并得到了需要的结果数据

#### date时间函数封装

````js
/*格式化日期 */
export function formateDate(time) {
  if (!time) return ''
  let date = new Date(time)
  return date.getFullYear() + '-' + (date.getMonth() + 1) +
    '-' + date.getDate() + ' ' + date.getHours() + ':' +
    date.getMinutes() + ':' + date.getSeconds()
}
````

### 根据当前路径自动切换title

> 根据路径切换title标题

![image-20200112154750094](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200112154750094.png)

````react
 getTitle = () => {
    //得到当前请求路径
    const path = this.props.location.pathname;
    let title;
    menuList.forEach(item => {  //根据路径数组遍历
      if (path === item.key) {  //如果当前item对象的key与path一样，item的title就是需要显示的title
        title = item.title
      } else if (item.children) {
        //在所有的子item中查找匹配的
        const cItem = item.children.find(childrenitem => childrenitem.key === path)
        //如果有cItem，那么说明有匹配的
        if (cItem) {
          title = cItem.title
        }
      }
    })
    return title  //title返回出去
  }
````

在render()中渲染title

````react
render(){
	const title = this.getTitle()
	return (
		<div>{title}</div>
	)
}
````

封装一个通用的 LinkButton 组件

````react
import React from 'react';
import './index.less'
/**
 * 
 * 外形像链接的按钮
 */
function LinkButton(props){
  return (
  <button className="link-button"  {...props}></button> //展开所有传递过来的信息
  )
}
export default LinkButton;

````

````less
.link-button{
  background-color: transparent;
  border: none;
  outline: none;
  color: #1da57a;
  cursor: pointer;
}
````



### antd中的Modal(对话框)的使用

> 第一种，直接引入定义事件即可使用
>
> 

````react
import { Modal } from 'antd';
import LinkButton from '../../components/link-button'
logout = () => {
    //显示确认框
    Modal.confirm({   //调用这个事件，就可以淡出Modal框
      content: '确定退出码',
      onOk: () => {  //点击了确定
        //删除保存的user数据
        storageUtils.removeUser()
        memoryUtils.user = {}
        //跳转到login页面
        this.props.history.replace('/login')
      },
      onCancel() { //点击了取消
        console.log('Cancel');
      },
    });
  }
render(){
    return (
        <div>
          <LinkButton onClick={this.logout}>退出</LinkButton>
        </div>
    )
}
````

> 第二种：组件形式使用
>
> `title`:Modal框的标题
>
> `visible`: 设定一个变量让他隐藏和显示
>
> `onOk`:   点击确定会调用的回调函数
>
> `onCancel` 点击取消会调用的回调函数

````react
  <Modal
          visible={showStatus === 1}
          title="添加分类"
          onOk={this.addCategory}
          onCancel={this.handleCancel}
        >
          <p>添加界面</p>
          
        </Modal>
````



### antd中的cart组件

> cart中的属性：
>
> title: card的左侧，可以单独抽出来写
>
> extra: card的右侧，同样可以单独抽出来写，里面可以写组件标签

````react
import React, { Component } from 'react';
import {
  Card,   //引入Card
  Table,
  Button,
  Icon
} from 'antd';

class Category extends Component {
  render() {
    //card的左侧
    const title = '一级分类列表'
    //card的右侧
    const extra = (
      <Button type="primary"> 
        <Icon type='plue'></Icon>
        添加
      </Button>
    )
    return (
      <Card title={title} extra={extra} style={{ width: '100%' }}>
        <p>Card content</p>
        <p>Card content</p>
        <p>Card content</p>
      </Card>
    );
  }
}

export default Category;
````

### antd中的Table组件

> 表格的配置：
>
> `dataSource` : 数据数组
>
> `columns` : 表格列的配置描述
>
> `bordered` : 加上代表显示边框
>
> `rowKey` : 表格行 key 的取值，可以是字符串或一个函数 
>
> `loading`: true  开启后会有加载图标
>
> data数组里面的配置：
>
>  `title` : 'xxx',              第一列的名称
>
> `dataIndex`: 'xxx',       指定显示数据对应的属性名
>
> `width` :  300，            为这一列设置宽度
>
> `render`：() => (要显示的标签)   render是一个函数，这里可以设置要显示的标签

Table的静态布局演示

````react
import React, { Component } from 'react';
import {
  Card,
  Table,
  Button,
  Icon
} from 'antd';
// 商品分类路由

import LinkButton from '../../components/link-button'

class Category extends Component {



  render() {
    //card的左侧
    const title = '一级分类列表'
    //card的右侧
    const extra = (
      <Button type="primary">
        <Icon type='plue'></Icon>
        添加
      </Button>
    )
    const dataSource = [   //要显示的数据
      {
        "parentId": "0",
        "_id": "5d4ada96cfcc7a4c34bf0480",
        "name": "电脑",
        "__v": 0
      },
      {
        "parentId": "0",
        "_id": "5d4adac2cfcc7a4c34bf0482",
        "name": "家用电器",
        "__v": 0
      },
      {
        "parentId": "0",
        "_id": "5d4adac5cfcc7a4c34bf0483",
        "name": "图书2",
        "__v": 0
      },
      {
        "parentId": "0",
        "_id": "5d4adae4cfcc7a4c34bf0484",
        "name": "服装",
        "__v": 0
      },
      {
        "parentId": "0",
        "_id": "5d4adae8cfcc7a4c34bf0485",
        "name": "食品",
        "__v": 0
      },
      {
        "parentId": "0",
        "_id": "5d4adaf2cfcc7a4c34bf0486",
        "name": "电竞",
        "__v": 0
      },
      {
        "parentId": "0",
        "_id": "5d4adaf8cfcc7a4c34bf0487",
        "name": "小红",
        "__v": 0
      }
    ];

    const columns = [  //要显示的列数，这里是两列
      {
        title: '分类的名称',   //第一列的名称
        dataIndex: 'name',  //指定显示数据对应的属性名
      },
      {
        title: '操作',  //第二列的名称
        width:300,   //指定宽度
        render: () => (   //指定返回需要显示的界面标签
          <span>
            <LinkButton>修改分类</LinkButton>
            <LinkButton>查看子分类</LinkButton>
          </span>

        )
      },

    ];


    return (
      <Card title={title} extra={extra} style={{ width: '100%' }}>
        <Table
          dataSource={dataSource}
          columns={columns}
          bordered
          rowKey='_id'
        />;
      </Card>
    );
  }
}

export default Category;
````

#### 表格分页器配置

> 表格分页器配置
>
> `pagination`  : 分页器 `pagination`里面是一个对象
>
> `pagination` ： 可配置的属性
>
> ​	`defaultCurrent` : 5   => 默认的每页条数
>
> ​	`showQuickJumper`:true  => 开启页数跳转功能
>
> ​	`disabled` : true    => 禁用分页功能
>
> ​	`current`: '动态页数'
>
> 文档：https://ant.design/components/pagination-cn/

````react
 <Table
          dataSource={categorys}
          columns={this.columns}
          bordered
          rowKey='_id'
          loading={loading}
          pagination={{defaultPageSize:5,showQuickJumper:true}}
        />;
````

#### Table中Column的render()属性

> Column的render()属性不仅可以返回标签结构，而且还可以接收一个参数
>
> render(category) => 这个参数就是这个表格这一行相对应的数据
>
> => 例如 这个表格渲染的是 => {
>         "parentId": "0",
>         "_id": "5d4adae8cfcc7a4c34bf0485",
>         "name": "食品",
>         "__v": 0
>       } 这个数据，然后render(category) 会把这个数据接收到，然后可以保存下来
>
> 

````react
this.columns = [
      {
        title: '分类的名称',
        dataIndex: 'name',  //指定显示数据对应的属性名
      },
      {
        title: '操作',
        width: 300,   //指定宽度
        render: (category) => (   //指定返回需要显示的界面标签
          <span>
            <LinkButton>修改分类</LinkButton>
            {/* 向事件回调函数传递参数:先定义一个匿名函数，在函数调用处理的函数传入参数 */}
            <LinkButton onClick={() => this.showSubCategorys(category)}>查看子分类</LinkButton>
          </span>

        )
      },

    ];
````

> render()方法的第二种用法
>
> 注意：本次render方法之前是有`dataIndex`的值存在的，所以这次render()函数接收的参数也是不一样的
>
> 如果没写` dataIndex: 'price',`这个属性，那么render()接收的是本行的所有属性
>
> 但是如果写了` dataIndex: 'price',` 那么本次接收的就是 `price`  的值了，可以修改显示方式

![image-20200114220535563](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200114220535563.png)

````react
 {
        title: '价格',
        dataIndex: 'price',
        render: (price) => '￥' + price //当前指定了对应的属性，传入的是对应的属性值
        
      }
````



#### Select组件的使用和封装form表单到Modal组件中使用

> 

````react
import React, { Component } from 'react';
import {
  Form,
  Select,               //引入组件
  Input
} from 'antd'
const Item = Form.Item    //抽取
const Option = Select.Option //抽取
/**
 * 添加分类的form组件
 */
class AddForm extends Component {
  render() {
    const { getFieldDecorator } = this.props.form
    return (
      <Form>
        <Item>
          {
            getFieldDecorator('parentId',{
              initialValue:'0'  //设置默认值，默认会选中这个Option
            })(
              <Select>
                <Option value="0">一级分类</Option>
                <Option value="1">电脑</Option>
                <Option value="2">图书</Option>
              </Select>
            )
          }

        </Item>
        <Item>
        {
            getFieldDecorator('categoryName',{
              initialValue:''
            })(
              <Input placeholder="请输入分类名称"></Input>
            )
          }
        </Item>
      </Form>
    );
  }
}

export default Form.create()(AddForm);

````

在Modal中使用：

````react
import UpdateForm from './update-form'
 <Modal
          visible={showStatus === 2}
          title="更新分类"
          onOk={this.updateCategory}
          onCancel={this.handleCancel}
        >
         <UpdateForm />
 </Modal>
````

显示效果：

![image-20200112203301426](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200112203301426.png)

> Select的`onChange`事件

````react
<Select
          value={searchType}
          style={{ width: 150 }}
          onChange={value => this.setState({ searchType: value })}> //value 是选中的value
          <Option value='productName'>按名称搜索</Option>
          <Option value="productDesc">按描述搜索</Option>
        </Select>
````

> Input的`onChange` 事件
>
> input 组件的`onchang`事件传递的是原生的event对象

````react
  <Input
          placeholder="关键字"
          style={{ width: 150, margin: '0 15px' }}
          value={searchName}
          onChange={e => this.setState({ searchName: e.target.value })}
      	
        />
````



#### 子组件向父组件传值

> 　因为这个form表单是封装在子组件中的，当父组件需要form组件的数据的时候就涉及到父子组件传值的问题了。
>
>   子组件向父组件传值，可以通过函数调用的形式传值
>
> 原理：
>
> ​	父组件向子组件传递一个函数，子组件调用这个函数，并且把要传递的参数当成实参，
>
> ​	当子组件调用完这个函数之后，父组件就可以从参数中获取到信息

父组件：

````react
  <Modal
          visible={showStatus === 2}
          title="更新分类"
          onOk={this.updateCategory}
          onCancel={this.handleCancel}
        >
          <UpdateForm 
            categoryName={category.name}   //向子组件传递了一个字符串属性
            setForm={(form) => this.form = form}  //向子组件传递一个函数
            // 保存子组件传递过来的form,并保存到this中
          />
        </Modal>
````

子组件：

````react
import React, { Component } from 'react';
import PropTypes from 'prop-types'
import {
  Form,
  Input
} from 'antd'
const Item = Form.Item

/**
 * 更新分类的form组件
 */
class UpdateForm extends Component {

  static propTypes = {
    categoryName: PropTypes.string.isRequired,
    setForm: PropTypes.func.isRequired
  }
  componentWillMount() {     
    //将form对象通过setForm()传递给父组件
    this.props.setForm(this.props.form) //把需要的信息当成实参传递过去
  }
  render() {
    const { getFieldDecorator } = this.props.form
    const { categoryName } = this.props
    return (
      <Form>
        <Item>
          {
            getFieldDecorator('categoryName', {
              initialValue: categoryName
            })(
              <Input placeholder="请输入分类名称"></Input>
            )
          }
        </Item>
      </Form>
    );
  }
}

export default Form.create()(UpdateForm);
````

这样父组件就可以使用这个属性了

#### antd清除Input组件缓存的信息

> 使用Input组件的时候，不手动删除输入的文字就关闭的话，下次还是会使用缓存住的信息，这样用户体验很差，这时候就用到`resetFields`这个属性了
>
> `resetFields` : 重置一组输入控件的值（为 `initialValue`）与状态，如不传入参数，则重置所有组件

````react
/**
   * 更新分类
   */
  updateCategory = async () => {
    //1.隐藏确认框
    this.setState({
      showStatus: 0
    })
    //准备数据
    const categoryId = this.category._id
    const categoryName = this.form.getFieldValue('categoryName')
    //清除输入数据
    this.form.resetFields()  //在取得数据之后，把Input组件中的数据清除掉，避免缓存问题
    //2.发请求更新列表
  
    const result = await reqUpdataCategorys({ categoryId, categoryName })
    if (result.status === 0) {
      //3.重新显示列表
      this.getCategorys()
    }

  }
````

### 分页列表

#### 1.纯前台分页

- 请求获取数据：一次获取所有数据，翻页时不需要再发请求
- 请求接口：
  - 不需要指定请求参数：页码(pageNum) 和每页数量(pageSize)
  - 响应数据：所有数据的数组

#### 2.基于后台的分页

- 请求获取数据：每次只获取当前页的数据，翻页时要法请求
- 请求接口：
  - 需要指定请求参数：页码(pageNum) 和每页数量(pageSize)
  - 响应数据：当前页数据的数组 + 总记录数(total)

#### 3.如何选择

- 基本根据数据多少来选择

#### Table表格基于后台分页的配置

> 基于后台的分页是要知道总条数才能进行分页的
>
> 在`pagination` 这里配置一个 `total`就可以自动分好页
>
> 点击页码触发的事件：`onChange`,点击页码就会触发这个事件，并且会把页码的参数传递过来

````react
 <Table
          loading={loading}
          dataSource={products}
          columns={this.columns}
          rowKey="_id"
          bordered
          pagination={{
            total,   //配置总页数
            defaultPageSize: PAGE_SIZE,
            showQuickJumper: true,
            onChange: (pageNum) => { this.getProducts(pageNum) } //发送请求，拿到新数据
 			//简写 
            //onChange: this.getProducts
          }}
        />;
````

### List组件用法

> 静态布局用法

````react
import React, { Component } from 'react';
import {
  Card,
  Icon,
  List,
} from 'antd'
import LinkButton from '../../components/link-button';
const Item = List.Item
/**
 * Product详情页面
 */
class ProductDetail extends Component {
  render() {
    const title = (
      <span>
        <LinkButton>
          <Icon
            type="arrow-left"
            style={{ color: 'green', marginRight: 10, fontSize: 20 }}
            onClick={() => this.props.history.goBack()}>

          </Icon>
        </LinkButton>

        <span>商品详情</span>
      </span>
    )
    return (
      <Card title={title} className="product-detail">
        <List>
          <Item>
            <span className="left">商品名称:</span>
            <span>神舟战神</span>
          </Item>
          <Item>
            <span className="left">商品描述:</span>
            <span>i9-9999K 2TSSD,5T硬盘 64G内存 256HZ电竞屏</span>
          </Item>
          <Item>
            <span className="left">商品价格:</span>
            <span>16666元</span>
          </Item>
          <Item>
            <span className="left">所属分类:</span>
            <span>电脑 --> 神舟大法好</span>
          </Item>
          <Item>
            <span className="left">商品图片:</span>
            <span>
              <img className="product-img" src="http://localhost:5000/upload/image-1565434072470.jpg" alt="" />
              <img className="product-img" src="http://localhost:5000/upload/image-1565434276989.jpg" alt="" />
            </span>
          </Item>
          <Item>
            <span className="left">商品详情:</span>
            <span dangerouslySetInnerHTML={{ __html: 'item' }}>
            </span>
          </Item>
        </List>
      </Card>
    );
  }
}

export default ProductDetail;
````

#### ProductHome通过路由将数据传给ProductDetail组件

ProductHome：

````react
 {
        title: '操作',
        width: 100,
        render: (product) => (
          //将product对象使用state传递给目标路由组件 
          <span>
            <LinkButton onClick={()=>this.props.history.push('/product/detail',{product})}>详情</LinkButton>
            <LinkButton>修改</LinkButton>
          </span>
        ),

      }
````

ProductDetail接收：

````react
   const {name,desc,price,detail} = this.props.location.state.product
````

### 在一个await中发送两次请求

> 如果一个 函数中出现了两个await ，那么必须要等到都一个 await执行完成之后才会执行第二个
>
> 这样会效率并不好 
>
> 这时候就用到了 `promise.all()` ,这个方法可以解决这个问题

````react
 async componentDidMount() {
    const { pCategoryId, categoryId } = this.props.location.state.product
    if (pCategoryId === '0') { //一级分类下的商品
      const result = await reqCategory(categoryId)
      const cName1 = result.data.name
      this.setState({
        cName1
      })
    } else { //二级分类下的商品
      /**
       * 通过多个 await方式发多个请求:后面一个请求是在前一个请求成功返回之后才发送
       */
      const result1 = await reqCategory(pCategoryId) //获取一级分类
      const result2 = await reqCategory(categoryId)  //获取二级分类
      const cName1 = result1.data.name
      const cName2 = result2.data.name
      this.setState({
        cName1,
        cName2
      })

    }
  }
````

Promise.all

````react
 async componentDidMount() {
    const { pCategoryId, categoryId } = this.props.location.state.product
    if (pCategoryId === '0') { //一级分类下的商品
      const result = await reqCategory(categoryId)
      const cName1 = result.data.name
      this.setState({
        cName1
      })
    } else { //二级分类下的商品
      /**
       * 一次性发送多个请求，只有都成功了，才正常处理
       */
	const results = await 			      Promise.all([reqCategory(pCategoryId),reqCategory(categoryId)])
    
     const cName1 = results[0].data.name
     const cName2 = results[1].data.name

      this.setState({
        cName1,
        cName2
      })

    }
  }
````

#### 解决Table后台分页更新状态页码丢失问题

> 使用后台分页的时候，如果要更新某个商品的状态，就涉及到要更新这一页的数据，但是又不知道现在是哪一页的数据，所以要在发送请求的函数中保存一下页码

````react
getProducts = async (pageNum) => {
    this.pageNum = pageNum  //保存pageNum，让其它方法可以看得到
    this.setState({
      loading: true
    })
    const { searchName, searchType } = this.state
    //如果搜索关键字有值，说明我们要做搜索分页
    let result
    if (searchName) {
      console.log('发送请求')
      result = await reqSearchProducts({ pageNum, pageSize: PAGE_SIZE, searchName, searchType })
    } else {
      result = await reqProducts(pageNum, PAGE_SIZE)
    }
    console.log(result)
    this.setState({
      loading: false
    })
    if (result.status === 0) {
      const { total, list } = result.data
      //取出分页数据，更新状态，显示分页列表
      this.setState({
        total,
        products: list
      })
    }
  }

  /**
   * 更新指定商品的状态
   */
  updataStatus = async (productId, status) => {
    const result = await reqUpdateStatus(productId, status) //更新状态
    if (result.status === 0) {
      message.success('更新商品成功')
      this.getProducts(this.pageNum) //使用this中记住的pageNum来发送请求，刷新页面
    }
  }

````

### Form表单栅格系统用法

> Form表单如果想用栅格系统来布局，可以使用`formItemLayout`来进行配置

````react
import React, { Component } from 'react';
import {
  Card,
  Form,
  Input,
  Cascader,
  Upload,
  Button,
  Icon
} from 'antd'
import LinkButton from '../../components/link-button';
const { Item } = Form
const { TextArea } = Input
/**
 * product的添加和更新的子路由组件
 */
class ProductAddUpdate extends Component {
  render() {
    //指定Item布局的配置对象
    const formItemLayout = {
      labelCol: { span: 2 },  //左侧label的宽度
      wrapperCol: { span: 8 }, //指定右侧包裹的宽度
    };

    const title = (
      <span>
        <LinkButton>
          <Icon type="arrow-left" style={{ fontSize: 20 }}></Icon>
        </LinkButton>
        <span>添加商品</span>
      </span>
    )

    return (
      <Card title={title}>
        <Form {...formItemLayout} >
          <Item label='商品名称'>
            <Input placeholder="商品名称" />
          </Item>
          <Item label='商品描述'>
            <TextArea placeholder="请输入商品描述" autosize={{ minRows: 2, maxRows: 6 }} />
          </Item>
          <Item label='商品价格'>
            <Input placeholder="商品名称" type='number' addonAfter="元"/>
          </Item>
        </Form>
      </Card>
    );
  }
}

export default ProductAddUpdate;
````

![image-20200117210901481](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200117210901481.png)

#### TextArea的配置

> `autosize={{ minRows: 2, maxRows: 6 }}` 
>
> `autosize`:自动高度
>
> 如果在里面写了对象 ： 
>
> `minRows: 2` : 最小两行
>
> `maxRows: 6`:最多六行

#### Input组件后面加上自定义的字符

> ` addonAfter="元"`   可以在后面加上 元这个字符



### antd中的Cascader级联选择

> `options` : 要显示的数据
>
> `loadData`: 对应一个函数，下一个要显示的数据

静态使用：

````react
import React, { Component } from 'react';
import {
  Card,
  Form,
  Input,
  Cascader,
  Upload,
  Button,
  Icon
} from 'antd'
import LinkButton from '../../components/link-button';
const { Item } = Form
const { TextArea } = Input
/**
 * product的添加和更新的子路由组件
 */
const options = [
  {
    value: 'zhejiang',
    label: 'Zhejiang',
    isLeaf: false,
  },
  {
    value: 'jiangsu',
    label: 'Jiangsu',
    isLeaf: false,
  },
];
class ProductAddUpdate extends Component {

  state = {
    options,
  }
  /**
   * 用于加载下一级列表的回调函数
   */
  loadData = selectedOptions => {
    //得到选择的option对象
    const targetOption = selectedOptions[0]
    //显示loding
    targetOption.loading = true

    //模拟发送请求列表并更新
    setTimeout(() => {
      //隐藏loding
      targetOption.loading = false;
      targetOption.children = [
        {
          label: `${targetOption.label} Dynamic 1`,
          value: 'dynamic1',
        },
        {
          label: `${targetOption.label} Dynamic 2`,
          value: 'dynamic2',
        },
      ]
      //更新 options状态
      this.setState({
        options: [...this.state.options],
      })
    }, 1000);
  }


  render() {

    const { getFieldDecorator } = this.props.form
    return (
      <Card title={title}>
        <Form {...formItemLayout} >
      
          <Item label='商品分类'>
            <Cascader
              options={this.state.options}  //需要显示的列表数据数组
              loadData={this.loadData} //当选择某个列表项目，加载下一级列表的监听回调
           
            />
          </Item>
        </Form>
      </Card>
    );
  }
}

export default Form.create()(ProductAddUpdate);
````

### antd中的Upload组件使用

> `action` :   上传图片的地址
>
> `accept` ： 接受上传文件类型
>
> - 一个有效的不区分大小写的文件名扩展名，以句点（“。”）字符开头。例如：`.jpg`，`.pdf`，或`.doc`。
> - 有效的MIME类型字符串，不带扩展名。
> - 字符串，`audio/*`表示“任何音频文件”。
> - 字符串，`video/*`表示“任何视频文件”。
> - 字符串，`image/*`表示“任何图像文件”。
>
> `listType` ：上传列表的样式
>
> - text
> - picture
> - picture-card
>
> `name` ：发到后台的文件参数名
>
> `fileList`：所有已上传文件的数组
>
> `onPreview`：点击文件链接或预览图标时的回调
>
> `onChange`：上传文件改变时的状态

````react
import React from 'react'
import { Upload, Icon, Modal } from 'antd';

function getBase64(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => resolve(reader.result);
    reader.onerror = error => reject(error);
  });
}

class PicturesWall extends React.Component {
  state = {
    previewVisible: false,  //标识是否显示大图预览Modal
    previewImage: '',  //大图的url
    fileList: [
      {
        uid: '-1', //每一个file都有自己唯一的id
        name: 'image.png', //图片文件名
        status: 'done', //图片状态: done->已上传 uploading->正在上传中 removed-> 已删除
        url: 'https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png',//图片地址
      }
    ],
  };
  /**
   * 隐藏Modal
   */
  handleCancel = () => this.setState({ previewVisible: false });

  handlePreview = async file => {
    console.log('handlePreview', file)
    if (!file.url && !file.preview) {
      file.preview = await getBase64(file.originFileObj);
    }

    this.setState({
      previewImage: file.url || file.preview,
      previewVisible: true,
    });
  };
/**
 * file:当前操作的图片文件(上传/删除)
 * fileList:所有已上传图片文件对象数组
 *
 */
  handleChange = ({file, fileList }) => {
    console.log('handleChange()',file,fileList)
    
    //在操作(上传/删除)过程中更新fileList状态
    this.setState({ fileList })
  };

  render() {
    const { previewVisible, previewImage, fileList } = this.state;
    const uploadButton = (
      <div>
        <Icon type="plus" />
        <div className="ant-upload-text">Upload</div>
      </div>
    );
    return (
      <div>
        <Upload
          action="/manage/img/upload" //上传图片的接口地址
          accept="image/*" //只接收图片格式
          name="image" //请求参数名
          listType="picture-card" //卡片样式
          fileList={fileList} //所有已上传文件的数组
          onPreview={this.handlePreview}
          onChange={this.handleChange}
        >
          {fileList.length >= 8 ? null : uploadButton}
        </Upload>
        <Modal visible={previewVisible} footer={null} onCancel={this.handleCancel}>
          <img alt="example" style={{ width: '100%' }} src={previewImage} />
        </Modal>
      </div>
    );
  }
}

export default PicturesWall
````

### 父组件调用子组件的方法

> 1.子组件调用父组件的方法:将父组件的方法以函数属性的形式传递给子组件，子组件就可以调用
>
> 2.父组件调用子组件的方法: 在父组件中通过ref得到子组件标签对象(也就是组件对象)，调用其方法

### 富文本编辑器

安装

````bash
yarn add react-draft-wysiwyg draftjs-to-html
````

静态使用：

````react
import React, { Component } from 'react';
import { EditorState, convertToRaw } from 'draft-js';
import { Editor } from 'react-draft-wysiwyg';
import draftToHtml from 'draftjs-to-html';
import htmlToDraft from 'html-to-draftjs';
import 'react-draft-wysiwyg/dist/react-draft-wysiwyg.css'
/**
 * 用来指定商品详情的富文本编辑器组件
 */

class RichTextEditor extends Component {
  state = {
    editorState: EditorState.createEmpty(), //创建一个没有内容的编辑对象
  }
  /**
   * 输入过程中实时的回调
   */
  onEditorStateChange = (editorState) => {
    console.log(editorState)
    this.setState({
      editorState,
    });
  };

  render() {
    const { editorState } = this.state;
    return (
      <div>
        <Editor
          editorState={editorState}
          editorStyle={{border:'1px solid black',minHeight:'200px',paddingLeft:'10px'}}
          onEditorStateChange={this.onEditorStateChange}
        />
        <textarea
          disabled
          value={draftToHtml(convertToRaw(editorState.getCurrentContent()))}
        />
      </div>
    );
  }
}



export default RichTextEditor;

````

#### 根据传递过来的标签文本显示在富文本编辑器中

> 如果本次有标签文本传递过来，想让他按照原本的格式显示在富文本编辑器中，就要按照下面这样做

````react
import React, { Component } from 'react'
import PropTypes from 'prop-types'
import { EditorState, convertToRaw, ContentState } from 'draft-js'
import { Editor } from 'react-draft-wysiwyg'
import draftToHtml from 'draftjs-to-html'
import htmlToDraft from 'html-to-draftjs'
import 'react-draft-wysiwyg/dist/react-draft-wysiwyg.css'
/**
 * 用来指定商品详情的富文本编辑器组件
 */

class RichTextEditor extends Component {

  static propTypes = {
    detail: PropTypes.string
  }

  constructor(props) {
    super(props)
    const { detail } = this.props
    //如果有detail传递过来,就证明有带标签的内容，就要这样处理一遍
    if (detail) {
      const contentBlock = htmlToDraft(detail)
      const contentState = ContentState.createFromBlockArray(contentBlock.contentBlocks)
      const editorState = EditorState.createWithContent(contentState)
      this.state = {
        editorState,
      }
    } else {
      this.state = {
        editorState: EditorState.createEmpty() //创建一个没有内容的编辑对象
      }
    }
  }

  /**
   * 输入过程中实时的回调
   */
  onEditorStateChange = (editorState) => {
    this.setState({
      editorState,
    })
  }
  /**
   * 返回输入数据对应的html格式的文本
   */
  getDetail = () => {
    return draftToHtml(convertToRaw(this.state.editorState.getCurrentContent()))
  }
  render() {
    const { editorState } = this.state;
    return (

      <Editor
        editorState={editorState}
        editorStyle={{ border: '1px solid black', minHeight: '200px', paddingLeft: '10px' }}
        onEditorStateChange={this.onEditorStateChange}
      />
    )
  }
}



export default RichTextEditor

````

#### 富文本编辑器中上传图片操作

> 第一步：在标签加上配置`toolbar`
>
> 第二步：修改回调函数

````react
import React, { Component } from 'react'
import PropTypes from 'prop-types'
import { EditorState, convertToRaw, ContentState } from 'draft-js'
import { Editor } from 'react-draft-wysiwyg'
import draftToHtml from 'draftjs-to-html'
import htmlToDraft from 'html-to-draftjs'
import 'react-draft-wysiwyg/dist/react-draft-wysiwyg.css'
/**
 * 用来指定商品详情的富文本编辑器组件
 */

class RichTextEditor extends Component {

  static propTypes = {
    detail: PropTypes.string
  }

  constructor(props) {
    super(props)
    const { detail } = this.props
    //如果有detail传递过来，证明就是点击详情进来的，如果没有，就是点击添加进来的
    if (detail) {
      const contentBlock = htmlToDraft(detail)
      const contentState = ContentState.createFromBlockArray(contentBlock.contentBlocks)
      const editorState = EditorState.createWithContent(contentState)
      this.state = {
        editorState,
      }
    } else {
      this.state = {
        editorState: EditorState.createEmpty() //创建一个没有内容的编辑对象
      }
    }
  }

  /**
   * 输入过程中实时的回调
   */
  onEditorStateChange = (editorState) => {
    this.setState({
      editorState,
    })
  }
  /**
   * 返回输入数据对应的html格式的文本
   */
  getDetail = () => {
    return draftToHtml(convertToRaw(this.state.editorState.getCurrentContent()))
  }

  uploadImageCallBack = (file) => {
    return new Promise(
      (resolve, reject) => {
        const xhr = new XMLHttpRequest()
        xhr.open('POST', '/manage/img/upload')
        const data = new FormData()
        data.append('image', file)
        xhr.send(data)
        xhr.addEventListener('load', () => {
          const response = JSON.parse(xhr.responseText)
          const url = response.data.url //得到图片的Url
          resolve({ data: { link: url } })  //关键代码
        })
        xhr.addEventListener('error', () => {
          const error = JSON.parse(xhr.responseText)
          reject(error)
        })
      }
    );
  }


  render() {
    const { editorState } = this.state;
    return (

      <Editor
        editorState={editorState}
        editorStyle={{ border: '1px solid black', minHeight: '200px', paddingLeft: '10px' }}
        onEditorStateChange={this.onEditorStateChange}
        toolbar={{
          image: { uploadCallback: this.uploadImageCallBack, alt: { present: true, mandatory: true } }
        }}
      />
    )
  }
}



export default RichTextEditor

````

### 配置表格行是否可选择

![image-20200130154352455](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200130154352455.png)

配置地址：`https://ant.design/components/table-cn/#rowSelection`

> `rowSelection`: object
>
> `rowSelection={{ type: 'radio' }}` :配置单选按钮
>
> `selectedRowKeys` : 指定选中项的 key 数组,这个值需要和rowKey配置的值一样

````react
  <Table
          dataSource={roles}
          columns={this.columns}
          bordered
          rowKey='_id'
          pagination={{ defaultPageSize: 5, pageSize: 3 }}
          rowSelection={{ type: 'radio', selectedRowKeys: [role._id] }}
          onRow={this.onRow}
        >
   </Table>
````



### Table中的onRow 事件

> `onRow`：传递一个函数
>
> 可以设置行的属性

````react
 import React, { Component } from 'react';
import {
  Card,
  Button,
  Table
} from 'antd'
class Role extends Component {

  state = {
    roles: [
    ]
  }
 	//====================关键代码========================
  onRow = (role) => {
    return {
      onClick: event => {  //点击的行数
        console.log(role)  //到的点击行的属性
      }
    }
  }
  componentWillMount() {
    this.initColumns()
  }

  render() {
    const { roles } = this.state
    const title = (
      <span>
        <Button type="primary" style={{ marginRight: 10 }}>创建角色</Button>
        <Button type="primary" disabled>设置角色权限</Button>
      </span>
    )

    return (
      <Card
        title={title}
      >
        <Table
          dataSource={roles}
          columns={this.columns}
          bordered
          rowKey='_id'
          pagination={{ defaultPageSize: 5, pageSize: 3 }}
          rowSelection={{ type: 'radio' }}
          onRow={this.onRow} //===========关键代码===========
        >
        </Table>
      </Card>
    );
  }
}

export default Role;
````

### Tree树形控件

> `checkable` :节点前添加 `Checkbox` 复选框
>
> `defaultExpandAll`: 默认展开所有树节点
>
> `checkedKeys` :（受控）选中复选框的树节点（注意：父子节点有关联，如果传入父节点 key，则子节点自动选中；相应当子节点 key 都传入，父节点也自动选中。当设置`checkable`和`checkStrictly`，它是一个有`checked`和`halfChecked`属性的对象，并且父子节点的选中与否不再关联

````react
import React, { Component } from 'react';
import PropTypes from 'prop-types'
import {
  Form,
  Tree,
  Input
} from 'antd'
import menuList from '../../config/menuConfig'

const Item = Form.Item
const { TreeNode } = Tree;

class AuthForm extends Component {

  static propTypes = {
    role: PropTypes.object
  }
  constructor(props) {
    super(props)
    //根据传入角色的menus生成初始状态
    const { menus } = this.props.role
    this.state = {
      checkedKeys: menus
    }
  }


  getTreeNodes = (menuList) => {
    return menuList.reduce((pre, item) => {
      pre.push(
        <TreeNode title={item.title} key={item.key}>
          {item.children ? this.getTreeNodes(item.children) : null}
        </TreeNode>
      )

      return pre
    }, [])
  }
  onSelect = (selectedKeys, info) => {
    console.log('selected', selectedKeys, info);
  };
  //选中某个node时的回调
  onCheck = (checkedKeys, info) => {
    //点击的时候，checkedKeys拿到的是本次所有选择的项
    this.setState({
      checkedKeys
    })
   
  };
  /**
   * 为父组件提交获取最新menus数据的方法
   */
  getMenus = () =>{
    return this.state.checkedKeys
  }
  componentWillMount() {
    this.treeNodes = this.getTreeNodes(menuList)
  }

  render() {
    const { role } = this.props
    const { checkedKeys } = this.state
    const formItemLayout = {
      labelCol: { span: 4 },  //左侧label的宽度
      wrapperCol: { span: 18 }, //指定右侧包裹的宽度
    };
    return (
      <div>
        <Item label="角色名称"  {...formItemLayout}>
          <Input value={role.name} disabled></Input>
        </Item>
        <Item>
          <Tree
            checkable
            defaultExpandAll={true}
            checkedKeys={checkedKeys}
            onCheck={this.onCheck}
          >
            <TreeNode title="平台权限" key="all">
              {this.treeNodes}
            </TreeNode>
          </Tree>
        </Item>
      </div>
    );
  }
}

export default AuthForm;

````



根据menuConfig.js里面的数组生成Tree结构

menuConfig.js:

````js
const menuList = [
  {
    title: '首页',   // 菜单标题名称 
    key: '/home', // 对应的 path 
    icon: 'home', // 图标名称 
  },
  {
    title: '商品', 
    key: '/products',
    icon: 'appstore',
    children: [ // 子菜单列表 
      { title: '品类管理', key: '/category', icon: 'bars' },
      { title: '商品管理', key: '/product', icon: 'tool' },
    ]
  },
  {
    title: '用户管理',
    key: '/user',
    icon: 'user'
  },
  {
    title: '角色管理',
    key: '/role',
    icon: 'safety',
  },
  {
    title: '图形图表',
    key: '/charts',
    icon: 'area-chart',
    children: [
      { title: '柱形图', key: '/charts/bar', icon: 'bar-chart' },
      { title: '折线图', key: '/charts/line', icon: 'line-chart' },
      { title: '饼图', key: '/charts/pie', icon: 'pie-chart' },]
  },
]
export default menuList
````

生成函数

````js
getTreeNodes = (menuList) => {
    return menuList.reduce((pre, item) => {
      pre.push(
        <TreeNode title={item.title} key={item.key}>
          {item.children ? this.getTreeNodes(item.children) : null}
        </TreeNode>
      )

      return pre
    }, [])
  }
````

效果

![image-20200131142500462](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200131142500462.png)

### 菜单权限管理

左侧导航菜单判断

````react
import React, { Component } from 'react';
import { Link, withRouter } from 'react-router-dom'
import logo from '../../assets/images/logo.png'
import menuList from '../../config/menuConfig'  //导航配置数组
import memoryUtils from '../../utils/memoryUtils'
import './index.less'
import { Menu, Icon } from 'antd';

const { SubMenu } = Menu;
class LeftNav extends Component {
    //===================关键代码=========================
  /**
   * 判断当前登陆用户对item是否有权限
   */
  hasAyth = (item) => {
    const { key, isPublic } = item
    const menus = memoryUtils.user.role.menus
    const username = memoryUtils.user.username
    /**
     * 1.如果当前用户是admin
     * 2.如果当前item是公开的，直接返回true
     * 3.当前用户有此item的权限：key
     */
    if (username === 'admin' || isPublic || menus.indexOf(key) !== -1) {
      return true
    } else if (item.children) { //4.如果当前用户有此item的某个子item的权限，这个item也要显示出来
      return !!item.children.find(child => menus.indexOf(child.key) !== -1)
    }
    return false
  }

  /**
   * 根据menu的数据数组生成对应的标签数组
   * 使用map() + 递归调用
   */

  getMenuNodes_map = (menuList) => {
    return menuList.map(item => {
      if (!item.children) {
        return (
          <Menu.Item key={item.key}>
            <Link to={item.key}>
              <Icon type={item.icon} />
              <span>{item.title}</span>
            </Link>
          </Menu.Item>
        )
      } else {
        return (
          <SubMenu
            key={item.key}
            title={
              <span>
                <Icon type={item.icon} />
                <span>{item.title}</span>
              </span>
            }
          >
            {this.getMenuNodes(item.children)}
          </SubMenu>
        )
      }

    })
  }
  getMenuNodes = (menuList) => {
    const path = this.props.location.pathname;
    return menuList.reduce((pre, item) => {

      //如果当前用户有item对应的权限，才需要显示对应的菜单项
      if (this.hasAyth(item)) { //==================关键判断
        if (!item.children) {
          pre.push(
            <Menu.Item key={item.key}>
              <Link to={item.key}>
                <Icon type={item.icon} />
                <span>{item.title}</span>
              </Link>
            </Menu.Item>
          )
        } else {
          //查找一个与当前请求路径匹配的子Item
          const cItem = item.children.find(citem => path.indexOf(citem.key) === 0)
          if (cItem) {
            //如果存在，说明当前item的子列表需要打开
            this.openKey = item.key
          }

          pre.push((
            <SubMenu
              key={item.key}
              title={
                <span>
                  <Icon type={item.icon} />
                  <span>{item.title}</span>
                </span>
              }
            >
              {this.getMenuNodes(item.children)}
            </SubMenu>
          ))
        }
      }


      return pre
    }, [])
  }
  /**
   * 在第一次render()之前执行一次
   * 为第一个render()准备数据(必须同步的)
   */
  componentWillMount() {

    this.menuNodes = this.getMenuNodes(menuList)
  }
  render() {
    //得到当前的路径
    let path = this.props.location.pathname
    if (path.indexOf('/product') === 0) { //说明当前请求的是商品或其子路由路径
      path = '/product'
    }
    const openKey = this.openKey
    console.log(path)
    return (
      <div className="left-nav">
        <Link to="/" className="left-nav-header">
          <img src={logo} alt='' />
          <h1>后台管理</h1>
        </Link>
        <Menu
          selectedKeys={[path]}
          mode="inline"
          theme="dark"
          defaultOpenKeys={[openKey]}
        >
          {this.menuNodes}
        </Menu>
      </div>


    );
  }
}

export default withRouter(LeftNav);
````

> 菜单数组也要进行相应更新

````js
const menuList = [
  {
    title: '首页',   // 菜单标题名称 
    key: '/home', // 对应的 path 
    icon: 'home', // 图标名称 
    isPublic:true  //所有用户都能看到，公开的  //==============要想公开，加上isPublic属性
  },
  {
    title: '商品', 
    key: '/products',
    icon: 'appstore',
    children: [ // 子菜单列表 
      { title: '品类管理', key: '/category', icon: 'bars' },
      { title: '商品管理', key: '/product', icon: 'tool' },
    ]
  },
  {
    title: '用户管理',
    key: '/user',
    icon: 'user'
  },
  {
    title: '角色管理',
    key: '/role',
    icon: 'safety',
  },
  {
    title: '图形图表',
    key: '/charts',
    icon: 'area-chart',
    children: [
      { title: '柱形图', key: '/charts/bar', icon: 'bar-chart' },
      { title: '折线图', key: '/charts/line', icon: 'line-chart' },
      { title: '饼图', key: '/charts/pie', icon: 'pie-chart' },]
  },
]
export default menuList
````

### Echarts使用

安装

````bash
yarn add echarts echarts-for-react
````

> series:[]    => 要显示的数据类型
>
> type:'bar' => 要显示的样式    
>
>  bar => 柱状图
>
>  line => 折线图

使用

````react
import React, { Component } from 'react';
import ReactEcharts from 'echarts-for-react';  //引入组件
import {
  Card,
  Button,
} from 'antd'
export class Bar extends Component {

  state = {
    sales:[5, 20, 36, 10, 10, 20],
    stores:[6, 25, 16, 20, 30, 10]
  }

  /**
   * 返回柱状图的对象配置
   */
  getOption = () => {   //配置项
    return {
      title: {
        text: '异步数据加载示例'
      },
      tooltip: {},
      legend: {
        data: ['销量','库存']
      },
      xAxis: {
        data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
      },
      yAxis: {},
      series: [{
        name: '销量',
        type: 'bar',
        data: [5, 20, 36, 10, 10, 20]
      },
      {
        name: '库存',
        type: 'bar',
        data: [6, 25, 16, 20, 30, 10]
      }
    ]
    }
  }
  render() {
    return (
      <div>
        <Card>
          <Button type="primary">更新</Button>
        </Card>
        <Card title="柱状图一">
          <ReactEcharts option={this.getOption()} /> //使用组件，并传入配置项
        </Card>
      </div>
    );
  }
}

export default Bar;

````

> 柱状图

````react
import React, { Component } from 'react';
import ReactEcharts from 'echarts-for-react';
import {
  Card,
  Button,
} from 'antd'
export class Bar extends Component {

  state = {
    sales: [5, 20, 36, 10, 10, 20],  //销量的数组
    stores: [6, 25, 16, 20, 30, 10]  //库存的数组
  }

  /**
   * 更新数据
   */
  update = () => {
    this.setState(state => ({
      sales: state.sales.map(sale => sale + 1),
      stores: state.stores.reduce((pre, store) => {
        pre.push(store - 1)
        return pre
      }, [])
    }))
  }

  /**
   * 返回柱状图的对象配置
   */
  getOption = (sales, stores) => {
    return {
      title: {
        text: '异步数据加载示例'
      },
      tooltip: {},
      legend: {
        data: ['销量', '库存']
      },
      xAxis: {
        data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
      },
      yAxis: {},
      series: [
        {
          name: '销量',
          type: 'bar',
          data: sales
        },
        {
          name: '库存',
          type: 'bar',
          data: stores
        }
      ]
    }
  }
  render() {
    const { sales, stores } = this.state
    return (
      <div>
        <Card>
          <Button type="primary" onClick={this.update}>更新</Button>
        </Card>
        <Card title="柱状图一">
          <ReactEcharts option={this.getOption(sales, stores)} />
        </Card>
      </div>
    );
  }
}

export default Bar;

````

效果

![image-20200205160041000](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200205160041000.png)

> 折线图

````react
import React, { Component } from 'react';
import ReactEcharts from 'echarts-for-react';
import {
  Card,
  Button,
} from 'antd'
export class Line extends Component {

  state = {
    sales: [5, 20, 36, 10, 10, 20],  //销量的数组
    stores: [6, 25, 16, 20, 30, 10]  //库存的数组
  }

  /**
   * 更新数据
   */
  update = () => {
    this.setState(state => ({
      sales: state.sales.map(sale => sale + 1),
      stores: state.stores.reduce((pre, store) => {
        pre.push(store - 1)
        return pre
      }, [])
    }))
  }

  /**
   * 返回柱状图的对象配置
   */
  getOption = (sales, stores) => {
    return {
      title: {
        text: '异步数据加载示例'
      },
      tooltip: {},
      legend: {
        data: ['销量', '库存']
      },
      xAxis: {
        data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
      },
      yAxis: {},
      series: [
        {
          name: '销量',
          type: 'line',
          data: sales
        },
        {
          name: '库存',
          type: 'line',
          data: stores
        }
      ]
    }
  }
  render() {
    const { sales, stores } = this.state
    return (
      <div>
        <Card>
          <Button type="primary" onClick={this.update}>更新</Button>
        </Card>
        <Card title="柱状图一">
          <ReactEcharts option={this.getOption(sales, stores)} />
        </Card>
      </div>
    );
  }
}

export default Line;

````

![image-20200205160443550](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200205160443550.png)

> 饼图

````react
import React, {Component} from 'react'
import {Card} from 'antd'
import ReactEcharts from 'echarts-for-react'

/*
后台管理的饼图路由组件
 */
export default class Pie extends Component {

  getOption = () => {
    return {
      title : {
        text: '某站点用户访问来源',
        subtext: '纯属虚构',
        x:'center'
      },
      tooltip : {
        trigger: 'item',
        formatter: "{a} <br/>{b} : {c} ({d}%)"
      },
      legend: {
        orient: 'vertical',
        left: 'left',
        data: ['直接访问','邮件营销','联盟广告','视频广告','搜索引擎']
      },
      series : [
        {
          name: '访问来源',
          type: 'pie',
          radius : '55%',
          center: ['50%', '60%'],
          data:[
            {value:335, name:'直接访问'},
            {value:310, name:'邮件营销'},
            {value:234, name:'联盟广告'},
            {value:135, name:'视频广告'},
            {value:1548, name:'搜索引擎'}
          ],
          itemStyle: {
            emphasis: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: 'rgba(0, 0, 0, 0.5)'
            }
          }
        }
      ]
    };

  }

  getOption2 = () => {
    return {
      backgroundColor: '#2c343c',

      title: {
        text: 'Customized Pie',
        left: 'center',
        top: 20,
        textStyle: {
          color: '#ccc'
        }
      },

      tooltip : {
        trigger: 'item',
        formatter: "{a} <br/>{b} : {c} ({d}%)"
      },

      visualMap: {
        show: false,
        min: 80,
        max: 600,
        inRange: {
          colorLightness: [0, 1]
        }
      },
      series : [
        {
          name:'访问来源',
          type:'pie',
          radius : '55%',
          center: ['50%', '50%'],
          data:[
            {value:335, name:'直接访问'},
            {value:310, name:'邮件营销'},
            {value:274, name:'联盟广告'},
            {value:235, name:'视频广告'},
            {value:400, name:'搜索引擎'}
          ].sort(function (a, b) { return a.value - b.value; }),
          roseType: 'radius',
          label: {
            normal: {
              textStyle: {
                color: 'rgba(255, 255, 255, 0.3)'
              }
            }
          },
          labelLine: {
            normal: {
              lineStyle: {
                color: 'rgba(255, 255, 255, 0.3)'
              },
              smooth: 0.2,
              length: 10,
              length2: 20
            }
          },
          itemStyle: {
            normal: {
              color: '#c23531',
              shadowBlur: 200,
              shadowColor: 'rgba(0, 0, 0, 0.5)'
            }
          },

          animationType: 'scale',
          animationEasing: 'elasticOut',
          animationDelay: function (idx) {
            return Math.random() * 200;
          }
        }
      ]
    };
  }

  render() {
    return (
      <div>
        <Card title='饼图一'>
          <ReactEcharts option={this.getOption()} style={{height: 300}}/>
        </Card>
        <Card title='饼图二'>
          <ReactEcharts option={this.getOption2()} style={{height: 300}}/>
        </Card>
      </div>
    )
  }
}

````

![image-20200205160947205](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200205160947205.png)

![image-20200205161002339](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\image-20200205161002339.png)

组件文档`https://github.com/hustcc/echarts-for-react`

echarts文档`https://www.echartsjs.com/zh/option.html#title`

### G2使用

安装

````bash
yarn add bizcharts @antv/data-set
````

> 基于 react 包装 G2 的开源库 
>
> 需要额外下载 @antv/data-set



### 前台404页面

````react
import React, { Component } from 'react';

import { Redirect, Switch, Route } from 'react-router-dom'
import { connect } from 'react-redux'
import { Layout } from 'antd';
import LeftNav from '../../components/left-nav'
import Header from '../../components/header'
import Home from '../home/home'
import Category from '../category/category'
import Product from '../product/product'
import User from '../user/user'
import Bar from '../charts/bar'
import Pie from '../charts/pie'
import Line from '../charts/line'
import Role from '../role/role'
import NotFound from '../not-found/not-found' //引入404组件
const { Footer, Sider, Content } = Layout;
/**
 * 后台管理路由组件
 */

class Admin extends Component {
  render() {
    const user = this.props.user

    //如果内存没有存储user ===> 证明当前没有登陆
    if (!user || !user._id) {
      return <Redirect to="/login" />
    }

    return (
      <Layout style={{ minHeight: '100%' }}>
        <Sider>
          <LeftNav />
        </Sider>
        <Layout>
          <Header>Header</Header>
          <Content style={{ margin: '20px', backgroundColor: '#fff' }}>

            <Switch>
              <Redirect from="/" to="/home" exact={true} />
              <Route path="/home" component={Home} />
              <Route path="/category" component={Category} />
              <Route path="/product" component={Product} />
              <Route path="/user" component={User} />
              <Route path="/charts/bar" component={Bar} />
              <Route path="/charts/pie" component={Pie} />
              <Route path="/charts/line" component={Line} />
              <Route path="/role" component={Role} />
              {/* 上面没有一个匹配的，直接显示404 */}
              <Route component={NotFound} />
            </Switch>

          </Content>
          <Footer style={{ textAlign: 'center', color: '#ccc' }}>推荐使用谷歌浏览器</Footer>
        </Layout>
      </Layout>
    );
  }
}
const mapStateToProps = (state) => {
  return {
    user: state.userReducer
  }
}
export default connect(mapStateToProps)(Admin);
````

### **使用** **BrowserRouter** 的问题

a. 问题: 刷新某个路由路径时, 会出现 404 的错误 

b. 原因: 项目根路径后的 path 路径会被当作后台路由路径, 去请求对应的后台路由, 

但没有 

c. 解决: 使用自定义中间件去读取返回 index 页面展现 