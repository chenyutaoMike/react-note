# React基础

### 安装

React脚手架安装

```bash
npm install -g create-react-app
//create-react-app my-app  X 这个命令创建以及过时了
npx create-react-app my-app  //可以使用这个命令创建
cd my-app
npm start
```

### 脚手架目录结构

>   node_modules  当前项目中依赖的包都安装在这里
>        .bin  本地项目中可执行命令，在package.json的scripts中配置对应的脚本即可（其中有一个就是：react-scripts命令）

####  public

> public存放的是当前项目的HTML页面（单页面应用放一个index.html即可，多页面根据自己需求放置需要的页面）

index.html

````html
 <!-- 在react框架中，所有的逻辑都是在JS中完成的（包括页面结构的创建），如果想给当前的页面
          导入一些CSS样式或者IMG图片等内容，我们有两种方式：
          1.在JS中基于ES6 module模块规范，使用import导入，这样webpack在编译合并JS的时候，会把导入
            的资源文件等插入到页面的结构中（绝对不能在JS管控的结构中通过相对目录./或者../，导入资源
            因为在webpack编译的时候，地址就不在是之前的相对地址了）
          2.如果不想再JS中导入（JS中导入的资源最后都会基于webpack编译），我们也可以把资源手动的在HTML
            中导入，但是HTML最后也要基于webpack编译，导入的地址也不建议写相对地址，而是使用 %PUBLIC_URL%
            写成绝对地址
    -->
    <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <link rel="stylesheet" href="%PUBLIC_URL%/css/reset.css">
````

#### src目录

> 项目结构中最主要的目录，因为后期所有的JS、路由、组件等都是放到这里面（包括需要编写的CSS或者图片等）

index.js是当前项目的入口文件

#### .gitignore  

Git提交时候的忽略提交文件配置项

### JSX渲染原理实现

#### createElement

````js
/**
 * 1.创建一个对象（默认有四个属性：type/props/ref,key）,最后要把这个对象返回
 * 2.根据传递的值修改这个对象
 *  type => 传递的type
 *  props => 需要做一些处理：大部分传递props中的属性都赋值给对象的props，有一些比较特殊
 *        => 如果是ref或者key，我们需要把传递的props中的这两个属性，给创建对象的两个属性，而传递的props中把这两个值删除掉
 *  children => 把传递的children作为新创建对象的props中的一个属性
 *   
 */
function createElement(type, props, children) {
  // 创建一个对象，设置一些默认属性值
  let obj = {
    type: null,
    props: {
      children: ''
    },
    ref: null,
    key: null,
  }
  // 用传递的type和props覆盖原有的默认值
  // obj = {...obj,type,props}; 
  obj = { ...obj, type,props: { ...props, children } }

  // 把ref和Key提取出来(并且删除props中的属性)
  'key' in obj.props ? (obj.key = obj.props.key, obj.props.key = undefined) : null;

  'ref' in obj.props ? (obj.ref = obj.props.ref, obj.props.ref = undefined) : null;
  return obj
}
let styleObj = {color:'red'};
console.log(createElement('h1',
  { id: 'titleBox', className: 'title', style: styleObj,ref:'aa',key:'12' },
  '\u6211\u7231\u5c0f\u7ea2'))

/**
 * {
 *  type:'h1',
 *  props:{
 *  id:'titleBox' ,
 *  className:'title',
 *  style:styleObj,
 *  children:'\u6211\u7231\u5c0f\u7ea2'
 * },
 *  ref:null,
 *  key:null,
 *  __proto__:Object.prototype
 * }
 */
````

#### render

````js
/**
 * render:把创建的对象生成对应的DOM元素，最后插入到页面中
 * 
 */
function render(obj, container, callback) {
  let { type, props } = obj || {},
    newElement = document.createElement(type);
  for (let attr in props) {
    if (!props.hasOwnProperty(attr)) break; //不是私有的直接结束遍历
    if (!props[attr]) continue; //如果当前属性没有值，直接不处理即可
    let value = props[attr];
    // className的处理
    if (attr === 'className') {
      newElement.setAttribute('class', value);
      continue;
    }
    // style处理
    if (attr === 'style') {
      if (value === '') continue;
      for (const styleKey in value) {
        if (!value.hasOwnProperty(styleKey)) break;
        newElement['style'][styleKey] = value[styleKey];
      }
      continue;
    }
    // 处理children
    if (attr === 'children') {
      if (typeof value === 'string') {
        let text = document.createTextNode(value);
        newElement.appendChild(text);
      }
      continue;
    }
    newElement.setAttribute(attr, value);//基于setAttribute可以让设置的属性表现在html的结构上
  }
  container.appendChild(newElement);
  callback && callback();
}

render(objJSX, document.querySelectorAll('#root'), () => {
  console.log('ok')
})
````

#### createElement,render复杂操作

````js
/**
 * createElement:创建JSX对象
 *   参数：至少两个 type/props,children这个部分可能没有可能有多个
 *   
 */
function createElement(type, props, ...childrens) {
  let ref, key;
  if ('key' in props) {
    key = props.key;
  }
  if ('ref' in props) {
    ref = props.ref
  }
  return {
    type,
    props: {
      ...props,
      children: childrens.length <= 1 ? (childrens[0] || null) : childrens
    },
    key,
    ref,
  }
}
let styleObj = { color: 'red' };
let objJSX = createElement('h1',
  { id: 'titleBox', className: 'title', style: styleObj, ref: 'aa', key: '12' },
  '\u6211\u7231\u5c0f\u7ea2')
console.log(objJSX)



/**
 * render:把创建的对象生成对应的DOM元素，最后插入到页面中
 * 
 */
function render(obj, container, callback) {
  let { type, props } = obj || {},
    newElement = document.createElement(type);
  for (let attr in props) {
    if (!props.hasOwnProperty(attr)) break; //不是私有的直接结束遍历
    if (!props[attr]) continue; //如果当前属性没有值，直接不处理即可
    let value = props[attr];
    // className的处理
    if (attr === 'className') {
      newElement.setAttribute('class', value);
      continue;
    }
    // style处理
    if (attr === 'style') {
      if (value === '') continue;
      for (const styleKey in value) {
        if (!value.hasOwnProperty(styleKey)) break;
        newElement['style'][styleKey] = value[styleKey];
      }
      continue;
    }
    // 处理children
    if (attr === 'children') {
      /**
       * 可能是一个值：可能是字符串也可能是一个jsx对象
       * 可能是一个数组：数组中的每一项可能是字符串也可能是jsx对象
       */
      // 首先把一个值也变为数组，这样后期统一操作数组即可
      if (!(value instanceof Array)) {
        value = [value]
      }
      value.forEach((item, index) => {
        //  验证Item是什么类型的，如果是字符串就是创建文本节点，如果是对象，我们需要再次执行render方法，把创建的元素放到最开始创建的大盒子中
        if (typeof item === 'string') {
          let text = document.createTextNode(value);
          newElement.appendChild(text);
        } else {
          render(item, newElement)
        }
      })
      continue;
    }
    newElement.setAttribute(attr, value);//基于setAttribute可以让设置的属性表现在html的结构上
  }
  container.appendChild(newElement);
  callback && callback();
}

// render(objJSX, document.querySelectorAll('#root'), () => {
//   console.log('ok')
// })
````

### React组件渲染机制

>  知识点：
>
>  createElement在处理的时候，遇到一个组件，返回的对象中：type就不在是字符串标签名了，而是一个函数（类），但是属性还是存在props中
>
>  render：处理type类型的时候，如果type的类型是函数，那么就会直接执行这个函数，函数(组件)执行之后会返回相应的JS对象（因为组件里面引入的React处理jsx，所以执行完组件函数之后直接能拿到js对象），再通过JS对象生成相应的DOM函数

### 函数式声明组件

````jsx
/**
 * 函数式声明组件
 *  1.函数返回结果是一个新的jsx(也就是当前组件的jsx结构)
 *  2.props变量存储的值是一个对象，包含了调取组件时候传递的属性值（不传递是一个空对象）
 */
import React from 'react';
export default function Dialog(props) {
  let { con = '没有提示', children } = props;
  // children：可能有可能没有，可能只是一个值，也可能是一个数组，可能每一项是一个字符串，也可能是一个对象等（代表双闭合组件中的子元素）
  return (
    <selection>
      <h2>系统提示</h2>
      <div>{con}</div>
      {/* 把属性中传递的子元素放到组件中的指定位置 */}
      {/* {children} */}
      {
        React.Children.map(children, item => item)
      }
    </selection>
  )
}
````

### 创建类式组件

````js
class DialogClass extends Component {
  constructor(props,context,updater){
    super(props) //ES6中的extends继承，一旦使用了constructor,第一行位置必须设置super执行：相当于React.Component.call(this)，也就是call继承，把父类私有属性继承过来
      // =>如果只写super()：虽然创建实例的时候把属性传递进来了，但是并没有传递给父组件，也就是没有把属性挂载到实例上，使用this.props获取的结果是undefined
      // 如果super(props)：在继承父类私有的时候，就把传递的属性挂载到了子类的实例上，constructor就可以使用this.props了

      // => props:当render渲染并且把当前类执行创建实例的时候，会把之前jsx解析出来的props对象中的信息（可能有children）传递给参数props => "调取组件传递的属性"
    
    /**
     * this.props   => 属性集合
     * this.refs    => ref集合（非受控组件中用到）
     * this.context => 上下文
     * 
     */
  }
  render() {
    let {typeVaue="默认提示",content,children} = this.props;
    let objStyle = {
      width: '50%',
      margin: '0 auto'
    }
    return (
      <section className="panel panel-default" style={objStyle}>
      <div className="panel-heading">

        <h3 className="panel-title">{typeVaue}</h3>
      </div>
      <div className="panel-body">
        {content}
      </div>
      {
        children ? <div className="panel-footer">
          {React.Children.map(children, item => item)}
        </div> : null
      }

    </section>
    );
  }
}

export default DialogClass;
````

#### 总结

基于Component类来创建组件

 基于createElement把JSX装换为一个对象，当render渲染这个对象的时候，遇到type是一个函数或者类，不是直接创建元素，而是先把方法执行：

  -> 如果就是函数式声明的组件，就把它当罪普通方法执行（方法中的this是undefined），把函数返回的jsx元素（也是解析后的对象）进行渲染

  -> 如果是类声明式组件，会把当前类 new 它执行，创建类的一个实例(当前本次调取的组件就是它的实例)，执行constructor，会执行this.render()，把render返回的jsx拿过来渲染，所以"类声明式组件，必须有一个render的方法，方法中需要返回一个jsx元素"

  但是不管是哪一种方式，最后都会把解析出来的props属性对象作为实参传递给对应函数或者类

 =====================================

 总结：创建组件有两种方式 "函数式"、"创建类式"

 函数式：

  1.操作简单

  2.能实现的功能也很简单，只是简单的调取和返回jsx而已

 创建类式：

  1.操作相对复杂一些，但是也可以实现更为复杂的业务功能

  2.能够使用生命周期函数操作业务

  3.函数式可以理解为静态组件（组件中的内容调取的时候就已经固定了，很难再修改），而类这种方式，可以基于组件内部的状态来动态更新渲染的内容

### React是如何把JSX元素装换为真实的DOM元素并且添加到页面中？

> 1.基于babel/babel-doader/babel-preset-react-app 把JSX语法便以为react.createElemenet这种模式
>
> ​	=> createElement中至少两个参数，没有伤心
>
> ​	第一个：目前是当前元素标签的标签名（字符串）
>
> ​	第二个：属性（没有给元素设置属性则为null）
>
> ​	其他的：当前元素的所有子元素内容（只要子元素是HTML，就会变为新的createElement）
>
> 2.执行createElement，把传递进来的参数最后处理成为一个对象
>
> ​	{
>
> ​		type:'标签名‘，
>
> ​		props:{
>
> ​			自己设置的那些属性对象（但是对于Key和ref来说是要提取出来的），
>
> ​			children:存放自己的子元素（没有子元素就没有这个属性），如果有多个子元素，就以数组的形式存储信息
>
> ​			}，
>
> ​				ref:null,
>
> ​				key:null
>
> ​	}
>
> 3.把生成的对象交给render进行处理：把对象编程DOM元素，插入到指定的容器中

````js
let createElement = (type, props = {}, ...childrens) => {

  let ref = null, key = null;
  if ('ref' in props) {
    ref = props.ref;
    props.ref = undefined;
  }
  if ('key' in props) {
    key = props.key
    props.key = undefined;
  }
  return {
    type,
    props: {
      ...props,
      // 保证children是一项或者是数组多项
      children: childrens.length <= 1 ? (childrens[0] || '') : childrens
    },
    ref,
    key,
    $$typeof: Symbol(React.element),
    _owner: null,
    _store: {
      validated: false
    },
    _self: null,
    _source: null
  }
}


let render = (objJSX, conntainer, callback) => {
  console.log(objJSX)
  let { type, props } = objJSX;
  let  { children } = props;
   let newElement = document.createElement(type);
  for (const attr in props) {
    let valueJSX = props[attr];
    if (!props.hasOwnProperty(attr)) break;
    if (typeof valueJSX === 'undefined') continue;
    // 事件处理
    let regEvent = /^on/;
    if (regEvent.test(attr)) {
      newElement.addEventListener(attr.toLowerCase().substr(2), valueJSX.bind(undefined));
      continue;
    }

    // 特殊处理
    switch (attr.toUpperCase) {
      case 'CLASSNAME':
        newElement.setAttribute('class', valueJSX);
        break;
      case 'STYLE':
        for (const styAttr in valueJSX) {
          console.log(styAttr,valueJSX[styAttr])
          if (!valueJSX.hasOwnProperty(styAttr)) break;
          newElement.style[styAttr] = valueJSX[styAttr];
        }
        break;
      case 'CHILDREN':
        if (valueJSX instanceof Array) valueJSX = [valueJSX]
        valueJSX.forEach(item => {
          if (typeof item === 'string') {
            newElement.appendChild(document.createTextNode(item));
            return
          }
          render(item, newElement)  //递归调用
        });
        break;

      default:
        newElement.setAttribute(attr, valueJSX);
    }


  }
  conntainer.appendChild(newElement);
  callback && callback();
}


````



### JSX中的事件绑定

````js
render() {
    return (
      <div>
        <button onClick={this.support}>this是谁</button>
      </div>
    )
  }
  support(e){
    // this:undefined（不是理解的操作的元素）
    // e.target:通过事件源可以获取当前操作的元素（一般很少操作，因为框架主要是数据驱动所有DOM的改变）   
  }
````

如果能让方法中的this变成当前类的实例就好了，这样可以操作属性和状态等信息

第一种，调用的时候使用bind绑定this

````js
render() {
    return (
      <div>
        <button onClick={this.support.bind(this)}>this是谁</button>
      </div>
    )
  }
  support(e){
    // this:undefined（不是理解的操作的元素）  
  }
````

第二种，在constructor中使用bind绑定this

````js
  constructor(props) {
    super(props)
    this.support = this.support.bind(this)
  }
````

第三种，箭头函数

````js
support = () => {
    // this:继承上下文中的this（实例），真实项目中，给JSX元素绑定的事件方法一般都是箭头函数，目的是为了保证函数中的this还是实例
  }
````

### 受控组件和非受控组件

> 1.基于数据驱动（修改状态数据，react帮助我们重新渲染视图）完成的组件叫做"受控组件"(受数据控制的组件)
>
> 2.基于ref操作DOM实现试图更新的，叫做“非受控组件”
>
> 建议多使用“受控组件”

### 导入静态图片注意点

> 在react的jsx中需要使用图片等资源的时候，资源的地址不能使用相对地址（因为经过webpack编译后，资源地址的路径已经改变了，原有的相对地址无法找到对应的资源），此时我们需要基于ES6Module或者CommonJS等模块导入规范，把资源当做模块导入进来(或者我们使用的图片地址都是网络地址)

导入图片：

````js
   const img_data = [{
      id:1,
      title:'',
      pic:require('./static/images/1.jpg')
    },
    {
      id:2,
      title:'',
      pic:require('./static/images/2.jpg')
    },
    {
      id:3,
      title:'',
      pic:require('./static/images/3.jpg')
    },
    {
      id:4,
      title:'',
      pic:require('./static/images/4.jpg')
    },
    {
      id:5,
      title:'',
      pic:require('./static/images/5.jpg')
    }
  ];
````





#### Fragment

如果不想用div包裹住组件，那就使用Fragment占位符

```jsx
import React, { Component,Fragment } from 'react';

export class TodoList extends Component {
    render() {
        return (
            <Fragment>
                <input /><button>按我</button>
                <ul>
                    <li>学英语</li>
                    <li>不学英语</li>
                </ul>
            </Fragment>
        );
    }
}

export default TodoList;

```

### this指向问题

函数的this在constructor中绑定this只会执行一次，利于提高性能

```jsx
import React, { Component, Fragment } from 'react';

export class TodoList extends Component {
    constructor(props){
        super(props)
        this.state = {
            inputValue:'',
            list:['小红','小燕']
        }
        
    }
    changInput(e){
       console.log(this) //没有写箭头函数，也没有在调用的时候绑定this，也没有在constructor里面绑定this，所以这次的this指向是undefined
    }
    render() {
        return (
            <Fragment>
                <input value={this.state.inputValue} onChange={this.changInput}/><button>按我</button>
                <ul>
                   {
                       this.state.list.map((item,index)=>{
                           return <li key={item+index}>{item}</li>
                       })
                      }
                </ul>
            </Fragment>
        );
    }
}

export default TodoList;

```

```jsx
 constructor(props){
        super(props)
        this.state = {
            inputValue:'',
            list:['小红','小燕']
        }
        this.changInput = this.changInput.bind(this) //绑定this
    }
    
    changInput(e){
        console.log(this) //现在this指向才正确
        this.setState({
            inputValue:e.target.value
        })
    }
//或者这样写
changinput = () =>{
     console.log(this) //现在this指向也正确 .这种写法不用在constructor写bing绑定
}
```

### setState

immutable的概念

```jsx
import React, { Component, Fragment } from 'react';

export class TodoList extends Component {
    constructor(props) {
        super(props)
        this.state = {
            inputValue: '',
            list: ['小红', '小燕']
        }
        this.changInput = this.changInput.bind(this)
        this.addList = this.addList.bind(this)
        this.removeList = this.removeList.bind(this)
    }

    changInput(e) {

        this.setState({
            inputValue: e.target.value
        })
    }
    addList() {
        this.setState({
            list: [...this.state.list, this.state.inputValue],
            inputValue: ''
        })
    }
    //关键代码=====================>start
    removeList(index) {
        //immutable
        //state 不允许我们做任何的修改
        //所以做法就是复制一份state.list副本，修改副本，再用副本替换原有的state.list
        let list = [...this.state.list];
        list.splice(index, 1)
        this.setState({
            list
        })
    }
    //关键代码==========================> end
    render() {
        return (
            <Fragment>
                <input value={this.state.inputValue} onChange={this.changInput} /><button onClick={this.addList}>按我</button>
                <ul>
                    {
                        this.state.list.map((item, index) => {
                            return <li onClick={() => { this.removeList(index) }} key={item + index}>{item}</li>
                        })
                    }
                </ul>
            </Fragment>
        );
    }
}

export default TodoList;

```

### JSX细节语法

#### dangerouslySetInnerHTML：不转义，以标签形式显示

```jsx
 <ul>
                    {
                        this.state.list.map((item, index) => {
                            return (
                                <li
                                    onClick={() => { this.removeList(index) }}
                                    key={item + index}
                                    dangerouslySetInnerHTML={{__html:item}} //关键用法
                                >
                        
                                </li>
                            )
                        })
                    }
                </ul>
```

#### htmlFor：label标签使用

点击label标签可以聚焦input

```jsx
    <label htmlFor="input">输入内容</label> //关键代码  不能用for了，只能用htmlFor
                <input
                    id="input"
                    value={this.state.inputValue}
                    onChange={this.changInput}
                />
```

### 父子组件传值

父组件

```jsx
import React, { Component, Fragment } from 'react';
import TodoItem from './ToItem' //引入子组件

export class TodoList extends Component {
    constructor(props) {
        super(props)
        this.state = {
            inputValue: '',
            list: ['小红', '小燕']
        }
        this.changInput = this.changInput.bind(this)
        this.addList = this.addList.bind(this)
      this.removeList = this.removeList.bind(this) //传递给子组件之前，要在这里绑定一下this指向
    }


    render() {
        return (
            <Fragment>
                <label htmlFor="input">输入内容</label>
                <input
                    id="input"
                    value={this.state.inputValue}
                    onChange={this.changInput}
                />
                <button onClick={this.addList}>按我</button>
                <ul>
                    //父子组件传值，就是在父组件中以属性的形式把参数传给子组件
                    {
                        this.state.list.map((item, index) => {
                            return (
                                <TodoItem   //循环子组件
                                    key={item + index}
                                    content={item} //把每个item传给子组件
                                    index={index} 
                                    removeList={this.removeList}
           //不仅仅可以传递普通的属性，还可以传递函数给子组件调用，但是传递函数的时候要记得绑定this指向
                                />
                            )
                        })
                    }
                </ul>
            </Fragment>
        );
    }
    changInput(e) {

        this.setState({
            inputValue: e.target.value
        })
    }
    addList() {
        this.setState({
            list: [...this.state.list, this.state.inputValue],
            inputValue: ''
        })
    }
    removeList(index) {
        //immutable
        //state 不允许我们做任何的修改
        //所以做法就是复制一份state.list副本，修改副本，再用副本替换原有的state.list
        let list = [...this.state.list];
        list.splice(index, 1)
        this.setState({
            list
        })
    }
}

export default TodoList;

```

子组件

```jsx
import React, { Component } from 'react';

export class ToItem extends Component {
    render() {
        return (
            //子组件的props属性可以接收全部父组件传递过来的参数
            
            <li
                onClick={() => { this.props.removeList(this.props.index) }}//调用父组件传递过来的参数
            >
                {this.props.content} //用this.props.xxx 接收父组件传递过来的属性
                
            </li>
        );
    }
}

export default ToItem;

```

### 优化

```jsx
addList() {
    //=>setState是异步的
    //=>setState里面使用函数的形式，这样的写法更靠谱
    this.setState((prvState)=>({  //prvState => 就是上次的参数
        list: [...prvState.list, prvState.inputValue],
        inputValue: ''
    }))
    // this.setState({
    //     list: [...this.state.list, this.state.inputValue],
    //     inputValue: ''
    // })
}
```

### 子组件调用父组件的方法

> 将父组件的方法以函数属性的形式传递给子组件，子组件就可以调用

### 



### PropTypes传值类型检验

##### 要求父组件传固定属性的值

用法：

```js
import PropTypes from 'prop-types'; //在子组件引入

ToItem.propTypes = {              //子组件名.propTypes{} 
    content:PropTypes.string,
    index:PropTypes.number,
    removeList:PropTypes.func
}
```

必须传递和默认值

```jsx
ToItem.propTypes = {
    content:PropTypes.string,
    index:PropTypes.number,
    removeList:PropTypes.func,
    tese:PropTypes.string.isRequired  //加入了isRequired,这个值是必须要传递的，如果不传会报错
}
ToItem.defaultProps = {  //默认值，如果父组件没有传递过来这个值，就使用这个默认值，如果有传，就使用传递的值
    test:'hello word'
}

```

```js
//指定一个数组由某一类型的元素组成
ToItem.propTypes = {
    content:PropTypes.string,
    index:PropTypes.oneOfType([PropTypes.number,PropTypes.string])//或者的语法，这里表示Index可以是个number类型的,也可以是string类型的
  
}
```

### props、state与render函数的关系

##### ● 当组件的state或者props发生改变的时候，render函数就会重新执行

##### ● 当父组件的render函数被运行时，它的子组件的render都将被重新运行一次

### 虚拟DOM

React渲染DOM步奏

1. state数据

2. jsx模版

3. 数据 + 模版 生成虚拟DOM(虚拟DOM就是一个JS对象，用它来描述真实DOM)

​     ->`['div',{id:'abc'},['span,{},'hello word']]`

4. 用虚拟DOM的结构生成真实的DOM，来显示

​     ->`<div id='abc'><span>hello word </span></div>`

5. state 发生变化

6. 数据 + 模版 生成新的虚拟DOM (极大的提升了性能)(相当于生成了js对象，而不是真实DOM)

​     ->`['div',{id:'abc'},['span,{},'bye bye']]`

7. 比较原始虚拟DOM和新的虚拟DOM的区别，找到区别是span中的内容

8. 直接操作DOM，改变span中的内容

优点:

1.性能提升了

2.它使得跨端应用得以实现。React Native

#### React.createElement

```jsx
// JSX -> createElement -> 虚拟DOM(JS对象) -> 真实的DOM

<div>item</div>
//相当于
->React.createElement('div',{},'item')
//例如
JSX => <div><span>item</span></div>
底层原理 => React.createElement('div',{},React.createElement('span',{},'item'))
```

#### 虚拟DOM中的Diff算法

同层比对 ，如果第一层发现两个虚拟DOM不一样，那么就终止比对，替换虚拟DOM，如果两个虚拟DOM一致，那么继续比对下去，直到找到不同处，然后将不同的虚拟DOM进行替换

### ref

第一种

```jsx
 <input
    id="input"
    value={this.state.inputValue}
    onChange={this.changInput}
    ref = {(input)=>{this.input = input}} //取到 input 然后赋值给this.input
  />
changInput(e) {
    console.log(this.input.value)  //可以拿到input的DOM属性
   
}
```

第二种

````react
import React, { Component } from 'react';

class RefDom extends Component {
  constructor() {
    super()
    this.HelloDiv = React.createRef()  //第一步
  }
  componentDidMount(){
    console.log(this.HelloDiv.current)  //这个就是divDOM了
  }
  render() {
    return (
      <div>
        <div ref={this.HelloDiv}> //第二步
          hello
         </div>
      </div>
    )
  }
}


export default RefDom

````

### 利用ref实现非受控组件

````react
import React, { Component } from 'react';

class RefDom extends Component {
  constructor() {
    super()
    this.usename = React.createRef()
  }
  clickHandler = () =>{
    console.log(this.usename.current.value)
  }
  render() {
    return (
      <div>
        <div ref={this.HelloDiv}>
          <input type='text' ref={this.usename} />  //这个不是受控组件，可以用ref实现
          <button onClick={this.clickHandler}>按我</button>
         </div>
      </div>
    )
  }
}


export default RefDom

````

### 组合 vs 继承

#### this.props.children

父组件

````react
import React from 'react';
import RefDom from './RefDom'


function App() {
    return (
        <div >
         
        <RefDom>
            <div>这里是父组件传递过来的标签</div>
        </RefDom>
        </div>
    );
}

export default App;
````

子组件

````react
import React, { Component } from 'react';

class RefDom extends Component {
  
 
  render() {
    return (
      <div>
        <p>这个是子组件</p>
        {this.props.children}  //这里就可以显示父组件传递过来的标签了
      </div>
    )
  }
}


export default RefDom

````



### 生命周期函数

> 描述一个组件或者从创建到销毁的过程，我们可以在过程中间基于钩子函数完成一些自己的操作（例如：在第一次渲染完成做什么，或者在二次即将重新渲染之前做什么等...）

[基本流程]

constructor 创建一个组件

componentWillMount 第一次渲染之前

render 第一次渲染

componentDidMount 第一次渲染之后

#### Initialization：组件初始化阶段

一般在constructor构造器里面初始化 state和props

#### Mounting：组件挂载阶段

```js
  // 在组件即将被挂载到页面的时刻自动执行
componentWillMount() {
        console.log('componentWillMount----1')
    }
    //组件渲染
    render(){
        console.log('render----2')
    }
	//组件被挂载到页面之后执行
    componentDidMount() {
        console.log('componentDidMount---3')
    }
```

#### Updation：组件更新阶段

props：

```js
componentWillReceiveProps(nextProps){
    //当组件接收到新的属性的时候自动调用，而且是在render之前
    console.log('componentWillReceiveProps' )  //1
}

//组件被更新之前，他会被自动执行
    shouldComponentUpdate(){
        // 组件是否需要更新，如果需要更新，返回true，如果不需要更新，返回false，这样就不会继续往下执行
        console.log('shouldComponentUpdate')  //2
        return true
    }
// 组件被更新之前，他会自动执行，但是他在shouldComponentUpdate之后
    // 如果shouldComponentUpdate返回true他才执行
    // 如果返回false，这个函数就不会被执行了
    componentWillUpdate(){
        console.log('componentWillUpdate') //3
    }
render(){
                                          //4
}
// 组件更新完成之后会执行
componentDidUpdate(){
                                         //5
    }
```

state:

```js
//组件被更新之前，他会被自动执行
    shouldComponentUpdate(){
        // 组件是否需要更新，如果需要更新，返回true，如果不需要更新，返回false，这样就不会继续往下执行
        console.log('shouldComponentUpdate')  //1
        return true
    }
// 组件被更新之前，他会自动执行，但是他在shouldComponentUpdate之后
    // 如果shouldComponentUpdate返回true他才执行
    // 如果返回false，这个函数就不会被执行了
    componentWillUpdate(){
        console.log('componentWillUpdate') //2
    }
render(){
                                          //3
}
componentDidUpdate(){
                                         //4
    }
```

#### Unmounting: 组件卸载阶段

```js
 // 当这个组件即将被从页面中移除的时候，会被执行
    componentWillUnmount(){
        console.log('componentWillUnmount------GG思密达')
    }
```

#### 有参数的生命周期函数

````js
  shouldComponentUpdate(nextProps,nextState){
    /**
     * 在这个钩子函数中，我们获取的state不是最新修改的，而是上一次的state值
     *  nextProps：最新修改的属性信息
     *  nextState：最新修改的状态信息
     */
  }
  componentWillUpdate(nextProps,nextState){
    // 这里获取的状态是更新之前的(和should相同也有两个参数存储最新的信息)
  }
  componentDidUpdate(){
    // 这里获取的状态是更新之后的
  }
````



### 生命周期使用

##### 发送ajax请求最好的生命周期函数

```js
 componentDidMount(){
        //发送ajax请求
    }
```

阻止没必要的子组件渲染
```js
 shouldComponentUpdate(nextProps, nextState) {
        //nextProps =>接下来我的props会变成什么样
        //nextState =>接下来我的state会变成什么样
        if (nextProps.content !== this.props.content)  { //数据没改变，就会返回false，不会再往下执行
            return true
        } else {
            return false
        }
    }
```

### setState的使用

> setState() 更新状态的两种写法

1). setState(updater,[callback]),

​	updater为返回stateChange对象的函数:(state,props) => stateChange

​	接收的state和props被保证为最新的

2). setState(stateChange,[callback])

​	stateChange为对象，

​	callback是可选的回调函数，在状态更新且界面更新后才执行

总结：

​	对思方式是函数方式的简写方式

​			如果性状态不依赖原状态 ===> 使用对象方式

​			如果新状态依赖于原状态 ===> 使用函数方式

​	如果需要在setState()后获取最新的状态数据，在第二个callback该函数中读取

````react
import React, { Component } from 'react';

class App extends Component {
	state = {
		count: 1
	}
	test1 = () => {
		const count = this.state.count + 1
		this.setState({
			count
		})
		console.log('test1 setState()之后，',this.state.count)
	}
	test2 = () => {
		this.setState(state => ({
			count: state.count + 1
		}))
		console.log('test2 setState()之后，',this.state.count)
	}
	test3 = () => {
		this.setState(state => ({ count: state.count + 1 }), () => {
			console.log('test2 setState() callback',this.state.count)
		})
	}
	render() {
		const { count } = this.state
		return (
			<div>
				<h1>{count}</h1>
				<button onClick={this.test1}>测试1</button>
				<button onClick={this.test2}>测试2</button>
				<button onClick={this.test3}>测试3</button>
			</div>
		);
	}
}

export default App;
````



> setState()更新状态是异步还是同步的？

1). 执行setState()的位置？

​	在react控制的回调函数中：生命周期勾子/ react事件监听回调

​	非react控制的异步回调函数中:定时回调/原生事件监听回调/promise回调/...

2). 异步or 同步？

 	react相关回调中： 异步

​	其它异步回调中：同步

> 关于异步的setState()

1). 多次调用，如何处理？

​	setState({}) : 合并更新一次状态，只调用一次render() 更新界面 --- 状态更新和界面更新都合并了

​	setState(fn) : 更新多次状态，但只调用一次render() 更新界面 --- 状态更新没有合并，但界面更新合并了

2). 如何得到异步更新后的状态数据？

​	在setState() 的callback回调函数

````react
import React, { Component } from 'react';

class App extends Component {
	state = {
		count: 0

	}
	//============异步=============================
	/**
	 * react事件监听回调中，setState() 是异步更新状态
	 */
	update1 = () => {
		console.log('update1 setState()之前', this.state.count)
		this.setState(state => ({ count: state.count + 1 }))
		console.log('update1 setState()之后', this.state.count)
	}
	/**
	 * react生命周期钩子中，setState()是异步更新状态
	 */
	componentDidMount() {
		console.log('componentDidMount setState()之前', this.state.count)
		this.setState(state => ({ count: state.count + 1 }))
		console.log('componentDidMount setState()之后', this.state.count)
	}
	//================同步===================
	/**
	 * 定时回调/原生事件监听回调/promise回调/...
	 */
	update2 = () => {
		setTimeout(()=>{
			console.log('setTimeout setState()之前', this.state.count)
			this.setState(state => ({ count: state.count + 1 }))
			console.log('setTimeout setState()之后', this.state.count)
		})
	}
	update3 = () => {
		const h2 = this.refs.count 
		h2.onclick = () =>{
			console.log('onclick setState()之前', this.state.count)
			this.setState(state => ({ count: state.count + 1 }))
			console.log('onclick setState()之后', this.state.count)
		}

	}
	update4 = () => {
		Promise.resolve().then(value =>{
			console.log('Promise setState()之前', this.state.count)
			this.setState(state => ({ count: state.count + 1 }))
			console.log('Promise setState()之后', this.state.count)
		})
		
	}

	render() {
		console.log('render()', this.state.count)
		const { count } = this.state
		return (
			<div>
				<h1 ref='count'>{count}</h1>
				<button onClick={this.update1}>更新1</button>
				<button onClick={this.update2}>更新2</button>
				<button onClick={this.update3}>更新3</button>
				<button onClick={this.update4}>更新4</button>
			</div>
		);
	}
}

export default App;
````



### 错误边界信息处理

> 封装错误信息组件

````react
import React from 'react';

export default class ErrorBoundary extends React.Component{
  state = {
    hasError : false,
    error:null,
    errorInfo:null
  }
  componentDidCatch(error,errorInfo){
    this.setState({
      hasError:true,
      error:error,
      errorInfo:errorInfo
    })
  }
  render(){
    if(this.state.hasError){
    return <div>{this.props.render(this.state.error,this.state.errorInfo)}</div>
    }
    return this.props.children
  }
}
````

> 使用方式:
>
> 如果你觉得哪一个组件会发生错误，那么就用上面定义好的组件包裹住它

````react
<ErrorBoundary render={(error,errorInfo) => <p>{'加载时候发生错误'}</p>}>
	<Errors />
</ErrorBoundary>
````





### React中实现过渡动画

transition动画

App.js

```jsx
import React, { Component, Fragment } from 'react';
import "./style.css"; //引入CSS
export class App extends Component {
    constructor(props) {
        super(props)
        this.state = {
            show: true
        }
        this.handleToggole = this.handleToggole.bind(this)
    }
    render() {
        return (
            <Fragment>
                <div className={this.state.show ? "show" : "hidden"}>hello</div> //切换类名
                <button onClick={this.handleToggole}>切换</button>
            </Fragment>
        );
    }
    handleToggole() {
        this.setState({
            show: !this.state.show
        })
    }
}

export default App;

```

CSS部分

```css
 .show{
    opacity: 1;
    transition: all 1s ease-in;  //最关键的样式
}
.hidden{
    opacity: 0;
    transition: all 1s ease-in; //最关键的样式
}

```

animation:

CSS部分

```css
.show{
    animation: show-item 2s ease-in forwards;
}
.hidden{
    animation: hide-item 2s ease-in forwards;
}

@keyframes show-item{
    0%{
        opacity: 0;
        color: red;
    }
    50%{
        opacity: 0.5;
        color: green;
    }
    100%{
        opacity: 1;
        color: blue;
    }
}

@keyframes hide-item{
    0%{
        opacity: 1;
        color: red;
    }
    50%{
        opacity: 0.5;
        color: green;
    }
    100%{
        opacity: 0;
        color: blue;
    }
}
```

### react-transition-group

安装

```bash
yarn add react-transition-group
```

使用：

```jsx
import React, { Component, Fragment } from 'react';
import { CSSTransition } from 'react-transition-group'; //引入
import "./style.css";
export class App extends Component {
    constructor(props) {
        super(props)
        this.state = {
            show: true
        }
        this.handleToggole = this.handleToggole.bind(this)
    }
    render() {
        return (

            <Fragment>
                <CSSTransition   //包裹要做动画的元素
                    in={this.state.show} //识别出场入场的动画
                    timeout={1000}  //动画的时间
                    classNames="fade" //CSS类名，要对应CSS中的类名
                    unmountOnExit     //动画结束的时候移除这个DOM
                    onEntered={(e)=>{e.style.color="blue"}} //钩子函数，在某个特定的时间会触发执行这个函数
                    appear={true} //设置为true之后，每次进入页面就会执行入场动画
                >
                    <div className={this.state.show ? "show" : "hidden"}>hello</div>
                </CSSTransition>
                <button onClick={this.handleToggole}>切换</button>
            </Fragment>
        );
    }
    handleToggole() {
        this.setState({
            show: !this.state.show
        })
    }
}

export default App;

```

CSS：

```css
 /* 入场动画 start*/  //类名和 JSX中classNames中对应
.fade-enter {
    opacity: 0;
}
.fade-enter-active{
    opacity: 1;
    transition:  opacity 1s ease-in;
}

.fade-enter-done{
    opacity: 1;
}

/* 入场动画 end*/

/* 出场动画 start */
.fade-exit{
    opacity: 1;
}
.fade-exit-active{
    opacity: 0;
    transition:  opacity 1s ease-in;
}
.fade-exit-done{
    opacity: 0;
}
/* 出场动画 end */

```

多组动画:

TransitionGroup:

```jsx
import React, { Component, Fragment } from 'react';
import { CSSTransition, TransitionGroup } from 'react-transition-group';
import "./style.css";
export class App extends Component {
    constructor(props) {
        super(props)
        this.state = {
            list: ['item']
        }
        this.handleList = this.handleList.bind(this)
    }
    render() {
        return (

            <Fragment>
                <TransitionGroup> //使用TransitionGroup包在外围就可以实现多组动画
                    {
                        this.state.list.map((item, index) => {
                            return (
                                <CSSTransition
                                    timeout={1000}
                                    classNames="fade"
                                    unmountOnExit
                                    onEntered={(e) => { e.style.color = "blue" }}
                                    appear={true}
                                    key={index + item}
                                >
                                    <div>{item}</div>
                                </CSSTransition>
                            )
                        })
                    }
                </TransitionGroup>
                <button onClick={this.handleList}>切换</button>
            </Fragment>
        );
    }
    handleList() {
        this.setState((prvState) => ({
            list: [...prvState.list, 'item']
        }))
    }
}

export default App;

```

### Redux

![1573278835085](C:\Users\Mike\AppData\Roaming\Typora\typora-user-images\1573278835085.png)

#### 最简单的学习方法，就是用redux做todolist

首先创建store文件夹，在store文件夹下面创建index.js 在index.js里面创建store

```js
import { createStore } from 'redux'; //引入创建store的api
import reducer from './reducer'; //引入reducer 管理store的数据(store就是图书管理员，reducer就是里面写的就是整个图书馆的信息)

const store = createStore(
    reducer,
    window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__() //配置Redux DevTools调试用的
    //这句话的意思就是：如果你安装了Redux DevTools，那么就使用这个Redux DevTools
); //连接store和reducer

export default store; //把store暴露出去
```

在store文件夹下面创建reducer.js文件

reducer就是一个函数

```js
const defaultValue = {
    inputValue: '',
    list: ['小红', '小燕']
}
//reducer 是一个函数(相当于笔记本)
//state => 上一次存储的数据
//action => 传递过来的信息

//reducer 可以接收state，但是绝不能修改state
export default (state = defaultValue, action) => {

    if (action.type === 'change_input_value') { //发现用户传递过来的是change_input_value，那么做以下的处理
        let newState = JSON.parse(JSON.stringify(state)) //因为不能直接修改state，所以要拷贝一份state
        newState.inputValue = action.value
        return newState       //替换老的数据
    }
    if (action.type === 'add_todo_item') {
        let newState = JSON.parse(JSON.stringify(state))
        newState.list = [...newState.list, newState.inputValue]
        newState.inputValue = ''
        return newState
    }

    return state  //默认返回的数据
}
```

组件的代码：

里面用到store提供的几个api：

##### ● store.getState() //store提供的api，可以获取store中的数据

##### ●  store.subscribe //store提供的api,订阅store的数据变化，如果store发生了改变，就会调用它

##### ● store.dispatch()  //store提供的api，可以把参数发送给reducer处理

```jsx
import React, { Component } from 'react';

import { Input, Button, List, Typography } from 'antd';
import store from './store'  //引入store


export class TodoList extends Component {
    constructor(props) {
        super(props)

        this.state = store.getState() //store提供的api，可以获取store中的数据
        this.handleInputChange = this.handleInputChange.bind(this)
        this.handleStoreChange = this.handleStoreChange.bind(this)
        this.handleBtnClick = this.handleBtnClick.bind(this)
        store.subscribe(this.handleStoreChange) //subscribe() => store提供的api,订阅store的数据变化，如果store发生了改变，就会调用它
    }

    render() {

        return (

            <div style={{ marginLeft: "300px" }}>

                <Input
                    id="input"
                    placeholder="请输入内容"
                    value={this.state.inputValue}
                    style={{ width: '300px', marginRight: '20px' }}
                    onChange={this.handleInputChange}
                />
                <Button type='primary' onClick={this.handleBtnClick}>按我</Button>

                <List
                    style={{ marginTop: '10px', width: '300px' }}
                    bordered
                    dataSource={this.state.list}
                    renderItem={item => (
                        <List.Item>
                            <Typography.Text mark></Typography.Text> {item}
                        </List.Item>
                    )}
                />
            </div>
        );
    }
    handleInputChange(e) {
        const action = {                     //创建一个action,action是一个对象
            type: 'change_input_value',      //每个action都应该有一个type,告诉reducer应该干什么
            value: e.target.value            //后面可以带上需要修改的参数
        }
        store.dispatch(action)  //通过store提供的api(dispatch) 把action发送给reducer处理
    }
    handleBtnClick() {
        const action = {
            type: 'add_todo_item'
        }
        store.dispatch(action)
    }
    handleStoreChange() {
        this.setState(store.getState())
    }

}

export default TodoList;

```

#### 将action和type抽离出来

可以提高代码的可维护性，降低耦合度(解耦)

1.首先在store文件下创建一个actionTypes.js文件，然后把type的字符串类型都定义成变量。暴露出去

这样做可以快速检查到变量名是否写错

```js
export const CHANGE_INPUT_VALUE = 'change_input_value';
export const ADD_TODO_ITEM = 'add_todo_item';
export const DELETE_TODO_ITEM = 'delete_todo_item'
```

2.然后在store文件下创建一个actionCreators.js文件，用来写生成action的函数

```js
import { CHANGE_INPUT_VALUE, ADD_TODO_ITEM, DELETE_TODO_ITEM } from './actionTypes'//把type引入进来


//这是一个创建action的函数，执行之后会返回一个对象类型的action 
export const getInputChangeAction = (value) =>({
    type:CHANGE_INPUT_VALUE,
    value
})

export const getAddItemAction = () =>({
    type:ADD_TODO_ITEM
})

export const getDeleteItemAction = (index) =>({
    type:DELETE_TODO_ITEM,
    index
})
```

3.再更新组件中的dispatch

```js
import React, { Component } from 'react';
import { Input, Button, List } from 'antd';
import store from './store'  
import { getInputChangeAction, getAddItemAction, getDeleteItemAction } from './store/actionCreators' //引入创建action对象的函数
export class TodoList extends Component {
    constructor(props) {
        super(props)

        this.state = store.getState() //redux提供的api，可以获取store中的数据
        this.handleInputChange = this.handleInputChange.bind(this)
        this.handleStoreChange = this.handleStoreChange.bind(this)
        this.handleBtnClick = this.handleBtnClick.bind(this)
        store.subscribe(this.handleStoreChange) //subscribe() => redux提供的api,订阅store的数据变化，如果store发生了改变，就会调用它
    }

    render() {

        return (

            <div style={{ marginLeft: "300px" }}>

                <Input
                    id="input"
                    placeholder="请输入内容"
                    value={this.state.inputValue}
                    style={{ width: '300px', marginRight: '20px' }}
                    onChange={this.handleInputChange}
                />
                <Button type='primary' onClick={this.handleBtnClick}>按我</Button>

                <List
                    style={{ marginTop: '10px', width: '300px' }}
                    bordered
                    dataSource={this.state.list}
                    renderItem={(item, index) => (
                        <List.Item onClick={this.handleItemClick.bind(this, index)}>
                            {item}
                        </List.Item>
                    )}
                />
            </div>
        );
    }
    handleInputChange(e) {
        // const action = {                     
        //     type: CHANGE_INPUT_VALUE,     
        //     value: e.target.value            
        // }
        store.dispatch(getInputChangeAction(e.target.value))  //上面注释是以前的写法
    }
    handleBtnClick() {
        // const action = {
        //     type: ADD_TODO_ITEM
        // }
        store.dispatch(getAddItemAction())
    }
    handleItemClick(index) {
        // const action = {
        //     type: DELETE_TODO_ITEM,
        //     index
        // }
        // console.log(index)
        store.dispatch(getDeleteItemAction(index))
    }
    handleStoreChange() {
        this.setState(store.getState())
    }

}

export default TodoList;


```

4.最后一步，记得在reducer.js中引入actionTypes.js，更换变量名

```js
import { CHANGE_INPUT_VALUE, ADD_TODO_ITEM, DELETE_TODO_ITEM } from './actionTypes'//引入
const defaultValue = {
    inputValue: '',
    list: ['小红', '小燕']
}


export default (state = defaultValue, action) => {

    if (action.type === CHANGE_INPUT_VALUE) {  //更换
        let newState = JSON.parse(JSON.stringify(state)) 
        newState.inputValue = action.value
        return newState       
    }
    if (action.type === ADD_TODO_ITEM) {
        let newState = JSON.parse(JSON.stringify(state))
        newState.list = [...newState.list, newState.inputValue]
        newState.inputValue = ''
        return newState
    }
    if (action.type === DELETE_TODO_ITEM) {
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.splice(action.index, 1)
        return newState
    }

    return state
}
```

抽离完成，这样代码可维护性大大提高

#### redux使用原则

##### ● store必须是唯一的

##### ● 只有store可以改变自己的内容

##### ● Reducer必须是纯函数

 ->纯函数指的是，给定固定的输入，就一定会有固定的输出，而且不会有任何副作用

 ->所有Reducer里面是不能有异步或者和时间有关系的操作的



### UI组件与容器组件的拆分

容器组件负责逻辑，UI组件负责渲染UI，所有要把他们抽离出来

新建一个TodoListUi.js组件，然后把容器组件的render部分抽离给UI组件

TodoList.js:

```jsx
import React, { Component } from 'react';

import store from './store'  //引入store
import { getInputChangeAction, getAddItemAction, getDeleteItemAction } from './store/actionCreators'
import TodoListUi from './TodoListUi';
export class TodoList extends Component {
    constructor(props) {
        super(props)
        this.state = store.getState()
        this.handleInputChange = this.handleInputChange.bind(this)
        this.handleStoreChange = this.handleStoreChange.bind(this)
        this.handleBtnClick = this.handleBtnClick.bind(this)
        this.handleItemClick = this.handleItemClick.bind(this)
        store.subscribe(this.handleStoreChange)
    }

    render() {
        return <TodoListUi
            inputValue={this.state.inputValue}
            handleInputChange={this.handleInputChange}
            handleBtnClick={this.handleBtnClick}
            list={this.state.list}
            handleItemClick={this.handleItemClick}
        />
    }
    handleInputChange(e) {

        store.dispatch(getInputChangeAction(e.target.value))
    }
    handleBtnClick() {

        store.dispatch(getAddItemAction())
    }
    handleItemClick(index) {

        store.dispatch(getDeleteItemAction(index))
    }
    handleStoreChange() {
        this.setState(store.getState())
    }

}

export default TodoList;

```

TodoListUi.js:

```jsx
import React, { Component } from 'react';
import { Input, Button, List } from 'antd';
class TodoListUi extends Component {
    render() {
        return (
            <div style={{ marginLeft: "300px" }}>

                <Input
                    id="input"
                    placeholder="请输入内容"
                    value={this.props.inputValue}
                    style={{ width: '300px', marginRight: '20px' }}
                    onChange={this.props.handleInputChange}
                />
                <Button type='primary' onClick={this.props.handleBtnClick}>按我</Button>

                <List
                    style={{ marginTop: '10px', width: '300px' }}
                    bordered
                    dataSource={this.props.list}
                    renderItem={(item, index) => (
                        <List.Item onClick={()=>{this.props.handleItemClick(index)}}>
                            {item}
                        </List.Item>
                    )}
                />
            </div>
        );
    }
}

export default TodoListUi;
```

容器和UI组件抽离完成，但是如果想要更好的性能，我们可以把UI组件做成函数组件的形式(无状态组件)，这样性能会有很大的提升

还是在TodoListUi.js文件中，把 class 改成 箭头函数

```jsx
import React from 'react';
import { Input, Button, List } from 'antd';
const TodoListUi = (props) => {
    return (
        <div style={{ marginLeft: "300px" }}>
            <Input
                id="input"
                placeholder="请输入内容"
                value={props.inputValue}
                style={{ width: '300px', marginRight: '20px' }}
                onChange={props.handleInputChange}
            />
            <Button type='primary' onClick={props.handleBtnClick}>按我</Button>

            <List
                style={{ marginTop: '10px', width: '300px' }}
                bordered
                dataSource={props.list}
                renderItem={(item, index) => (
                    <List.Item onClick={() => { props.handleItemClick(index) }}>
                        {item}
                    </List.Item>
                )}
            />
        </div>
    );

}

export default TodoListUi;
```

### redux-thunk

安装:

```tex
yarn add redux-thunk
```

配置:

在store文件夹中index.js：

```js
import { createStore, applyMiddleware, compose } from 'redux';//引入applyMiddleware和compose
import reducer from './reducer';
import thunk from 'redux-thunk';  //thunk也要引入进来

//固定写法  想要redux-thunk和redux-devtools就必须做这一步
//地址-> https://github.com/zalmoxisus/redux-devtools-extension   1.2处查看
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ?
    window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({}) : compose;

const enhancer = composeEnhancers(
    applyMiddleware(thunk),
);

const store = createStore(
    reducer,
    enhancer
);

export default store;
```

> 第二种配置方案：
>
> 可以直接配置使用

````js
import {createStore,applyMiddleware} from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';
import reducer from './reducers';
import thunk from 'redux-thunk';
const store = createStore(reducer, composeWithDevTools(
  applyMiddleware(thunk),
));

export default store

````

配置完之后就可以使用了

为什么要用redux-thunk呢？

##### ● 可以不用在组件中发起ajax请求，转而到actionCreators.js文件中发送请求

##### ● 如果不用redux-thunk中间件的时候，在创建action的时候是不能发送请求的，因为action要求是要返回一个对象形式，而不是函数，但是redux-thunk解决了这个问题，可以返回一个函数

redux-thunk使用：

actionCreators.js:

```js
import { CHANGE_INPUT_VALUE, ADD_TODO_ITEM, DELETE_TODO_ITEM, INIT_LIST_ACTION } from './actionTypes'
import axios from 'axios';
//这是一个创建action的函数，执行之后会返回一个对象类型的action 
export const getInputChangeAction = (value) => ({
    type: CHANGE_INPUT_VALUE,
    value
})

export const getAddItemAction = () => ({
    type: ADD_TODO_ITEM
})

export const getDeleteItemAction = (index) => ({
    type: DELETE_TODO_ITEM,
    index
})

export const initListAction = (data) => ({
    type: INIT_LIST_ACTION,
    data
})
//=========================关键代码====================================================
export const getTodoList = () => {   //因为用了redux-thunk，所以这里可以返回一个函数
    return (dispatch) => {       //这个函数会在TodoList.js dispatch的时候执行，而这个函数里面也有个dispatch参数
        axios.get('/api.json').then((res) => {

            const data = res.data.data;
            const action = initListAction(data)  //创建action
            console.log(action)
            dispatch(action)      //当函数执行的时候 ，这里会dispatch一个对象给store
        })
    }
}
//===========================================================================
```

TodoList.js中的关键代码

在componentDidMount的时候调用

```js
componentDidMount() {
        const action = getTodoList() //现在action接收的是一个函数
        store.dispatch(action)  //因为用了redux-thunk中间件，所以就算action是个函数也能dispatch出去
        //store发现这个是一个函数，就会帮你执行一下这个函数，而这个函数里面又有action和dispatch
        //这样就可以再发送一个dispatch给store了
    }
```

总结：

1.redux中间件是指action和store之间

2.action只能是一个对象，但是使用中间件之后可以是一个函数了

3.中间件就是对dispatch的一个封装升级

4.如果传给dispatch是一个对象的话，那么dispatch就会直接把这个对象给store

5.如果传递给dispatch是一个函数的话，他就会先执行这个函数，然后在执行完函数的时候再根据代码发送给store

##### 注意：

##### 中间件指的是redux的中间件，而不是react的中间件

### redux-saga

安装:

```
 yarn add redux-saga
```

配置：

redux-saga配置比较麻烦，不仅在store中有配置，而且还要新建一个sagas.js文件

store文件下index.js：

```js
import { createStore, applyMiddleware, compose } from 'redux';//引入applyMiddleware和compose
import reducer from './reducer';
import createSagaMiddleware from 'redux-saga'
import todoSagas from './sagas' //这个是sagas.js文件
const sagaMiddleware = createSagaMiddleware()

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ?
    window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({}) : compose;

const enhancer = composeEnhancers(
    applyMiddleware(sagaMiddleware),
);

const store = createStore(reducer, enhancer);

sagaMiddleware.run(todoSagas) //调用这个方法，让saga跑起来

export default store;
```

sagas.js文件：

```js
import { takeEvery, put } from 'redux-saga/effects'
import { GET_INIT_LIST } from './actionTypes'
import axios from 'axios'
import { initListAction } from './actionCreators'

//这段代码的意思就是:
//actiontype === GET_INIT_LIST 的时候，就会执行这个函数
//这个函数执行之后会创建一个action，然后通过put(相当于dispatch)把action发送给store更新数据

function* getInitList() {
    //处理ajax请求失败
    try {
        const res = yield axios.get('/api.json')  //因为这里是个generator函数，所以它会等待结果回来，可以采用同步的方法编程
        const data = res.data.data
        const action = initListAction(data)
        yield put(action)  //这里的put等于 是dispatch的意思
    } catch (e) {
        console.log('网络请求失败')
    }

}

// generator 函数
function* mySaga() {
    yield takeEvery(GET_INIT_LIST, getInitList); //只要actiontype === GET_INIT_LIST,那么就会执行fetchUser这个方法
}

export default mySaga;
```

其他文件照常发送dispatch就可以了

### React-Redux

安装

```
yarn add react-redux
```

#### Provider

Provider的作用：连接组件，在Provider里面的组件都可以获得store

src目录下index.js文件：

```js
import React from 'react';
import ReactDOM from 'react-dom';
import TodoList from './TodoList';
import 'antd/dist/antd.css';
import { Provider } from 'react-redux';  //从react-redux引入Provider
import store from './store'
const App = (
    <Provider store={store}>  //Provider的作用：连接组件，在Provider里面的组件都可以获得store
        <TodoList /> 
    </Provider>
)
ReactDOM.render(<App />, document.getElementById('root'));


```

#### connect

使用这个方法连接store

```jsx
import {connect} from 'react-redux'; //引入
class TodoList extends Component {
    xxx
}
export default connect(null,null)(TodoList); //用connect包裹住组件之后就能连接store了
```

connect的两个参数：

```jsx
import React, { Component } from 'react';
import { getInitList, getInputChangeAction, getAddItemAction, getDeleteItemAction, getTodoList } from './store/actionCreators'
import { connect } from 'react-redux';
class TodoList extends Component {
    constructor(props) {
        super(props)
    }

    render() {

        return (
            <div>
                <div>
                    <input
                        value={this.props.inputValue}
                        onChange={this.props.handleInputChange}
                    />
                    <button onClick={this.props.handleClick}>点我</button>
                </div>
                <ul>
                    {
                        this.props.list.map((item, index) => {
                            return <li key={item + index} onClick={() => { this.props.deleItem(index) }}>{item}</li>
                        })
                    }
                </ul>
            </div>
        )
    }


}
//这两个函数都是要返回一个对象
const mapStateToProps = (state) => { //把store的的数据映射成这个组件的props state这个参数就是store里面的数据
    return {
        inputValue: state.inputValue, //在这里取出了数据，可以在组件内使用props.inputValue获取这个数据
        list: state.list
    }
}

const mapDispatchToprops = (dispatch) => {
    return {
        handleInputChange(e) {
            dispatch(getInputChangeAction(e.target.value))
        },
        handleClick() {
            dispatch(getAddItemAction())
        },
        deleItem(index) {
            dispatch(getDeleteItemAction(index))
        }
    }
}

export default connect(mapStateToProps, mapDispatchToprops)(TodoList);

```

##### 优化

1.使用解构赋值减少一些代码

```jsx
import React, { Component } from 'react';
import { getInitList, getInputChangeAction, getAddItemAction, getDeleteItemAction, getTodoList } from './store/actionCreators'
import { connect } from 'react-redux';
class TodoList extends Component {
    constructor(props) {
        super(props)
    }

    render() {
         //解构赋值减少代码
        const { inputValue, handleInputChange, handleClick, deleItem, list } = this.props;
       
        return (
            <div>
                <div>
                    <input
                        value={inputValue}
                        onChange={handleInputChange}
                    />
                    <button onClick={handleClick}>点我</button>
                </div>
                <ul>
                    {
                        list.map((item, index) => {
                            return <li key={item + index} onClick={() => { deleItem(index) }}>{item}</li>
                        })
                    }
                </ul>
            </div>
        )
    }


}
//这两个函数都是要返回一个对象
const mapStateToProps = (state) => { //把store的的数据映射成这个组件的props state这个参数就是store里面的数据
    return {
        inputValue: state.inputValue, //在这里取出了数据，可以在组件内使用props.inputValue获取这个数据
        list: state.list
    }
}

const mapDispatchToprops = (dispatch) => {
    return {
        handleInputChange(e) {
            dispatch(getInputChangeAction(e.target.value))
        },
        handleClick() {
            dispatch(getAddItemAction())
        },
        deleItem(index) {
            dispatch(getDeleteItemAction(index))
        }
    }
}

export default connect(mapStateToProps, mapDispatchToprops)(TodoList);

```

2.将其封装成无状态组件，提高性能

```jsx
import React, { Component } from 'react';
import { getInitList, getInputChangeAction, getAddItemAction, getDeleteItemAction, getTodoList } from './store/actionCreators'
import { connect } from 'react-redux';


const TodoList = (props) => {
    const { inputValue, handleInputChange, handleClick, deleItem, list } = props;
    return (
        <div>
            <div>
                <input
                    value={inputValue}
                    onChange={handleInputChange}
                />
                <button onClick={handleClick}>点我</button>
            </div>
            <ul>
                {
                    list.map((item, index) => {
                        return <li key={item + index} onClick={() => { deleItem(index) }}>{item}</li>
                    })
                }
            </ul>
        </div>
    )
}

//这两个函数都是要返回一个对象
const mapStateToProps = (state) => { //把store的的数据映射成这个组件的props state这个参数就是store里面的数据
    return {
        inputValue: state.inputValue, //在这里取出了数据，可以在组件内使用props.inputValue获取这个数据
        list: state.list
    }
}

const mapDispatchToprops = (dispatch) => {
    return {
        handleInputChange(e) {
            dispatch(getInputChangeAction(e.target.value))
        },
        handleClick() {
            dispatch(getAddItemAction())
        },
        deleItem(index) {
            dispatch(getDeleteItemAction(index))
        }
    }
}

export default connect(mapStateToProps, mapDispatchToprops)(TodoList);

```

```jsx
export default connect(mapStateToProps, mapDispatchToprops)(TodoList);
```

=>TodoList是一个ui组件,connect把ui组件和业务逻辑结合之后，返回的是一个容器组件了

=>就是connect调用之后返回一个容器组件

### styled-components管理样式

安装：

```
yarn add styled-components
```

使用：

在src根目录下创建一个style.js文件

```js
import { createGlobalStyle } from 'styled-components'


export const Globalstyle = createGlobalStyle`　
body{
　　margin: 0;
　　padding: 0;
    background-color:pink;
　}`
```

在src根目录下的index.js页面引入

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './style'
import App from './App';


ReactDOM.render(<App />, document.getElementById('root'));


```

在要使用的组件里面以标签的形式引入进来

App.js

```jsx
import React, { Component } from 'react';
import {Globalstyle} from './style'  //引入css样式
export class App extends Component {
  render() {
    return (
      <div>
        <Globalstyle />        //使用
        1111
      </div>
    );
  }
}

export default App;

```

这样就可以生效了。

#### Rest.css

为了让浏览器的默认样式统一，我们需要rest.css帮我们重置一下

直接写在style.js里面：

```js
import { createGlobalStyle } from 'styled-components'


export const Globalstyle = createGlobalStyle`　
html, body, div, span, applet, object, iframe,
	h1, h2, h3, h4, h5, h6, p, blockquote, pre,
	a, abbr, acronym, address, big, cite, code,
	del, dfn, em, img, ins, kbd, q, s, samp,
	small, strike, strong, sub, sup, tt, var,
	b, u, i, center,
	dl, dt, dd, ol, ul, li,
	fieldset, form, label, legend,
	table, caption, tbody, tfoot, thead, tr, th, td,
	article, aside, canvas, details, embed, 
	figure, figcaption, footer, header, hgroup, 
	menu, nav, output, ruby, section, summary,
	time, mark, audio, video {
		margin: 0;
		padding: 0;
		border: 0;
		font-size: 100%;
		font: inherit;
		vertical-align: baseline;
	}
	/* HTML5 display-role reset for older browsers */
	article, aside, details, figcaption, figure, 
	footer, header, hgroup, menu, nav, section {
		display: block;
	}
	body {
		line-height: 1;
	}
	ol, ul {
		list-style: none;
	}
	blockquote, q {
		quotes: none;
	}
	blockquote:before, blockquote:after,
	q:before, q:after {
		content: '';
		content: none;
	}
	table {
		border-collapse: collapse;
		border-spacing: 0;
	}
　}`
```

##### 全局的样式设置完成之后，接下来就是设置单个组件的样式了

在src下创建一个文件夹common,在里面再创建一个header文件下，然后创建index.js和style.js文件

路径如下

src

—common

——header

———index.js

———style.js

style.js:

```js
import styled from 'styled-components';  //引入创建标签样式的api

export const HeaderWrapper = styled.div`  //这里的意思就是 创建一个高度56px,pink背景的空div组件，然后把组件暴露出去
    height:56px;
    background:pink;
`

```

index.js：

```jsx
import React, { Component } from 'react';
import { HeaderWrapper } from './style'  //在组件中引入
export class Header extends Component {
    render() {
        return (
            <div>
                <HeaderWrapper>header</HeaderWrapper>  //以标签的形式放在组件中
            </div>
        );
    }
}

export default Header;

```

##### 这样就可以使用styled-components

在styled-components引入图片

先把图片放在一个静态资源文件夹里，然后再style.js引入:

```js
import styled from 'styled-components';
import logoPic from '../../statics/logo.png'; //引入图片，设置成变量

export const HeaderWrapper = styled.div`
	z-index: 1;
	position: relative;
	height: 56px;
	border-bottom: 1px solid #f0f0f0;
`;

export const Logo = styled.div`
	position: absolute;
	top: 0;
	left: 0;
	display: block;
	width: 100px;
	height: 56px;
	background: url(${logoPic}); //在这里使用
	background-size: contain;
`;
```

在index.js文件夹中直接引入使用即可：

```jsx
import { HeaderWrapper ,Logo} from './style'
render() {
        return (
            <div>
                <HeaderWrapper>
                    <Logo />
                </HeaderWrapper>
            </div>
        );
    }

```

在styled-components为a标签设置属性

```js
export const Logo = styled.a.attrs({   //在a后面加.attrs就可以设置属性
    href:'/'
})`
	position: absolute;
	top: 0;
	left: 0;
	display: block;
	width: 100px;
	height: 56px;
	background: url(${logoPic});
	background-size: contain;
`;

```

在styled-components中写上带有类名的样式：

`styled.div`创建出来的是一个组件，我们可以给每个组件加不同的类名(className)，这样每个组件可以拥有不同的样式

style.js:

```js
export const NavItem = styled.div`
	line-height: 56px;
	padding: 0 15px;
	font-size: 17px;
	color: #333;
	&.left {              //用&表示当前组件 这个的意思就是 <NavItem className="left />"
		float: left;
	}
	&.right {
		float: right;
		color: #969696;
	}
	&.active {
		color: #ea6f5a;
	}
`;

```

index.js:

```jsx
  <NavItem className="left">首页</NavItem>
  <NavItem className="left">下载App</NavItem>
  <NavItem className="right">登陆</NavItem>
  <NavItem className="right">Aa</NavItem>
```

### 合并reducer

combineReducers

在项目中，如果reducer写了过多的逻辑也是不太好管理的，我们可以将reducer分离开来，然后在src目录下的store文件夹中用combineReducers这个api将它们合并起来，这样写起来就很方便了

使用方式：

先在header组件中创建一个store文件夹，然后在里面创建reducer.js和index.js

创建index.js是为了导出方便

reducer.js:

```js
const defaultState = {
    focused: true
}


export default (state = defaultState, action) => {
    return state
}
```

index.js:

```js
import reducer from './reducer'
 
export {reducer}  //这样写是为了导出方便，外面的组件可以直接 ../store就可以引入文件了
```

然后最重要的一步，就是处理src下store文件夹的reducer.js

src-store-reducer.js:

```js
import { combineReducers } from 'redux';  //引入redux提供合并reducer的api
import { reducer as HeaderReducer } from '../common/header/store' //把每个不同组件之间的reducer引入进来，因为可能有多个，所以要起对应的名字

const reducer = combineReducers({  //使用combineReducers将其合并
    header: HeaderReducer
})

export default reducer;  //导出给store
```

这样就合并成功了，但是如果要使用的话，还是差一步：

回到header组件中：

```js
const mapStateToProps = (state) => {
    return {
        focused: state.header.focused  //因为有多个reducer，所以取数据的时候要根据reducer的名字来取出相应的数据了
    }
}
```

合并完成

### 用Immutable管理reducer的数据

安装

```
yarn add immutable
```

为什么用immutable：

##### ● reducer不能对原始数据进行修改，要返回一个新的数据回去、

##### ● 但是这种操作是人为约束，有可能不小心就修改了原始的数据

##### ● immutable.js就可以解决这个问题

##### ● immutable => 不可变更的

用法：

reducer.js:

```js
import { SEARCH_FOCUS, SEARCH_BLUR } from './actionTypes'
import { fromJS } from 'immutable'; //引入immutable
const defaultState = fromJS({   //用fromJS包裹这个数据，那么这个数据就变成了immutable对象了
    focused: true
})

//reducer不能对原始数据进行修改，要返回一个新的数据回去、
//但是这种操作是人为约束，有可能不小心就修改了原始的数据
//immutable.js就可以解决这个问题
//immutable => 不可变更的

export default (state = defaultState, action) => {
    switch (action.type) {
        case SEARCH_FOCUS:
            // immutable对象的set方法,会结合之前immutable对象的值和设置的值，返回一个全新的对象
            return state.set('focused',true)  //用immutable提供的set方法设置数据
        case SEARCH_BLUR:
            return state.set('focused',false)
    }
    return state
}
```

因为用了immutable，所以组件中取数据的方式也会变：

header文件夹下的index.js:

```js
const mapStateToProps = (state) => {
    return {
        // focused: state.header.focused  ->以前的做法
        focused: state.header.get('focused')   //->用了immutable之后的做法
        //用immutable提供的get方法获取数据

    }
}
```

### redux-immutable

安装

```
yarn add redux-immutable
```

为什么使用redux-immutable统一管理项目中的reducer：

● 数据流安全

● 编写方式更直观

使用：

在src下总的store文件夹下面的reducer.js文件：

```js
// import { combineReducers } from 'redux';  //以前是从redux引入，现在不用了
import { combineReducers } from 'redux-immutable'; //不用在redux引入，而是在redux-immutable引入
import { reducer as HeaderReducer } from '../common/header/store'

const reducer = combineReducers({   //其他方法是一样的
    header: HeaderReducer
})

export default reducer;
```

因为总的store数据都是immutable数据，所以取数据的时候也会发生改变：

header组件中index.js:

```js
const mapStateToProps = (state) => {
    return {  //state是immutable对象，所以要使用immutable提供的api获取数据
        focused: state.getIn(['header','focused'])        //这两行是相等的，都可以获取到数据
        // focused: state.get('header').get('focused')   //->用了immutable之后的做法

    }
}
```

#### 使用immutable和ajax结合的问题

immutable对象和普通js对象是不一样的，当我们用ajax请求回来的数据替换store中的数据的时候，其实是用普通js对象把immutable对象替换掉了，这是不允许的，所以，我们拿到数据的时候应该先把数据变为immutable对象，这样才能去替换store中的数据，统一数据格式

一般发送请求都是在actionCreator.js中

```js
import { SEARCH_FOCUS ,SEARCH_BLUR,CHANGE_LIST} from './actionTypes';
import axios from 'axios';
import {fromJS} from 'immutable'; //引入immutable

const changeList = (data) =>({
    type:CHANGE_LIST,
    data:fromJS(data)  //拿到数据之后，把数据包装一下变为immutable对象，等待dispatch给reducer
})


export const getList = () =>{
    return (dispatch) =>{
        axios.get('/api/headerList.json').then((res)=>{
          const data = res.data.data;
          const action = changeList(data)
          dispatch(action)  //把包装过后的数据发送给reducer
        }).catch((e)=>{
            console.log('err')
        })
    }
}
```

#### 把immutable对象转换为普通对象

##### toJS()

有时候需要把immutable数据转换成普通数据，这个时候就可以使用immutable提供的toJS()方法了

```js
  const jsList = list.toJS();
```

list是一个 immutable对象，通过转化之后就变为普通对象了

#### immutable一次设置多对象

merge()

接受一个对象，一次可设置多个

```js
state.merge({
                list: action.data,
                totalPage: action.totalPage
            });
```

### 路由

react-router-dom

安装

```
yarn add react-router-dom
```

使用:

```jsx
import {BrowserRouter,Route} from 'react-router-dom'; //引入
import Home from './pages/home'  //子组件
import Detail from './pages/detail'; //子组件
 class App extends Component {
  render() {
    return (
      <Provider store={store}>
        <Globalstyle />
        <Header />
        <BrowserRouter>  //用BrowserRouter包裹要做路由的组件
          <Route  path='/' exact  component={Home}/>  //Route代表每个要渲染的组件 exact代表严格匹配路径
          <Route  path='/detail' exact  component={Detail}/>

        </BrowserRouter>
      </Provider>
    );
  }
}
```

#### Link

页面的跳转：

```jsx
import {Link} from 'react-router-dom';
 <Link to='/detail'>  //to => 要跳转的路由
          xxx

 </Link>
```

注意：

Link不能写在BrowserRouter外部

#### 页面路由参数的传递

#####  第一种：动态路由

跳转的时候带参数：

```jsx
 <Link to={'/detail' + this.state.id}>跳转</Link> //id是动态的
```

但是单单这一步还是不行，还要在路由页面配置一下：

```jsx
  <Route path='/' exact component={Home} />
  <Route path='/detail:id' exact component={Detail} /> //在路径后面加 :id 代表传递参数
```

在跳转之后的detail可以拿到传递过来的id

```react
componentDidMount(){
	console.log(this.props.match.params.id)
}
```

##### 第二种：

跳转页面：

```react
 <Link to={'/detail?id=' + this.state.id}>跳转</Link> //采用问号的形式带参数
```

路由配置

```react
  <Route path='/' exact component={Home} />
  <Route path='/detail' exact component={Detail} /> //路由页面就不用添加 :id了
```

跳转之后的detail页面

```react
console.log(this.props.location.search) //也可以获取，但是连问号也会拿到 => ?id=1  
//这种方法还要自己处理字符串
```

#### 路由重定向

可以用作检查是否登陆，如果登陆了就不跳转，如果没登陆就跳转到登陆页面

用法：

```react
import {Redirect} from 'react-router-dom';

  <Redirect to='/home'/> //to => 要跳到的页面
```

#### withRouter

在异步加载的组件中要使用路由的api，就要用withRouter包裹一下

配合React-redux的connect使用：

```jsx
import {withRouter} from 'react-router-dom';
import {connect} from 'react-redux';

export default connect()(withRouter(Home)); //使用
```



### PureComponent

PureComponent在底层实现了shouldComponentUpdate,这样的话就不用手写shouldComponentUpdate了

使用方法：

```jsx
import React, { PureComponent } from 'react'; //引入

export class Detail extends PureComponent { //使用
    render() {
        return (
            <div>
                Detail
            </div>
        );
    }
}

export default Detail;
```

> PureComponent的基本原理
>
> 1. 重写实现shouldComponentUpdate()
> 2. 对组件的新/旧state和props中的数据进行浅比较，如果都没有变化，返回false,否则返回true
> 3. 一旦shouldComponentUpdate()返回false不再执行用于更新的render()

注意:

##### PureComponent要配合immutable一起使用才不会出现问题，如果没有使用immutable结合PureComponent，会有坑

### 用ref获取文本框中的属性

```jsx
jsx部分
 <input ref={(input) => this.mima = input} />
 <input type="password" ref={(input) => this.pas = input} />
 <button onClick={() => this.props.buttonClick(this.mima, this.pas)}>按我</button>
事件部分
 buttonClick(mima,pas) {
    console.log(mima.value,pas.value)
 }
```

### 按需加载页面

安装

```
yarn add react-loadable
```

在组件中新建一个loadable.js文件：

```jsx
import React from 'react';
import Loadable from 'react-loadable';

const LoadableComponent = Loadable({
  loader: () => import('./'),
    loading(){                   //loading是一个函数，要求返回一个组件
    	return <div>正在加载...</div>
	}
});

export default () => <LoadableComponent />  把组件导出去

/* export default class App extends React.Component {
  render() {
    return <LoadableComponent/>;
  }
} */
```

完成这一步之后，还需要最后一步：

```react
import React, { Component } from 'react';
import { Globalstyle } from './style';
import Header from './common/header'
import { Provider } from 'react-redux';
import { BrowserRouter, Route} from 'react-router-dom';
import Detail from './pages/detail/loadable.js';//这个组件就是异步组件了
import store from './store'
class App extends Component {
  render() {
    return (
      <Provider store={store}>
        <Globalstyle />
        <Header />
        <BrowserRouter>
          <Route path='/' exact component={Home} />
          <Route path='/detail' exact component={Detail} />
        </BrowserRouter>
      </Provider>
    );
  }
}

export default App;

```

