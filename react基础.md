# React

### 安装、初始化项目

```javascript
npm install -g create-react-app //全局安装脚手架

create-reacr-app xxx   //xxx就是项目名称
```

### hello word

#### 最简单的hello word

#### src->index.js

```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'
ReactDOM.render(<App />,document.getElementById('root'))
//上面的代码，我们先引入了React两个必要的文件，然后引入了一个APP组件，目前这个组件还是没有的，需要一会建立。然后用React的语法把APP模块渲染到了root ID上面.这个rootID是在public\index.html文件中的。
```

##### src->App.js

```javascript
import React,{Component} from 'react' 
class App extends Component{
    render() {
        return (
             <div>
                 hello word
             </div>
        )
    }
}
export default App
//import React, {Component} from 'react'
//这其实是ES6的语法-解构赋值，如果你分开写就比较清楚了，你可以把上面一行代码写成下面两行.
//import React from 'react'
//const Component = React.Component
```

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
> createElement在处理的时候，遇到一个组件，返回的对象中：type就不在是字符串标签名了，而是一个函数（类），但是属性还是存在props中
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

 

### Fragment标签讲解

##### React要求必须在一个组件的最外层进行包裹，但是如果每个组件都用div标签包裹有时候会影响布局，比如我们在作`Flex`布局的时候,外层不能有包裹元素，为了解决这个问题，可以使用`<Fragment>`标签

```javascript
import React,{Component,Fragment } from 'react'  //引入

class App extends Component{
    render(){
        return  (
            <Fragment>  //直接可以使用
               <div>list</div>
               <ul>
                   <li>小红</li>
                   <li>小燕</li>
               </ul> 
            </Fragment>
        )
    }
}
export default App 
//再去浏览器查看元素时候，外层是没有 <Fragment> 这个标签的 
```

### 响应式设计和数据的绑定

```javascript
class App extends Component {
    //构造函数
    constructor(props) {
        super(props)  //调用父类的构造函数，固定写法
        this.state = {  
            inputVlaue: '', //input的值
            list: ["小红", "小燕"]
        }
        this.inputChange = this.inputChange.bind(this) //调用函数之前一定要绑定一下this，如果不绑定this的话 ,函数里面的this会是undefined
        this.addList = this.addList.bind(this)
         this.deleteItem = this.deleteItem.bind(this)
    }
    inputChange(e){
        this.setState({
            inputVlaue:e.target.value
        })
       
    }
    addList(){
      this.setState({
            list:[...this.state.list,this.state.inputVlaue],
            inputVlaue:''
        })
    }
     deleteItem(index) {
        //正确的写法
        let list = this.state.list
        list.splice(index, 1)
        this.setState({
            list: list
        })
        //错误的写法
        // this.state.list.splice(index, 1) //React是禁止直接操作state的,
        // this.setState({
        //     list:  this.state.list
        // })
    }
    render() {
        let { list , inputVlaue} = this.state  //赋值解构方式把 list 和inputValue拿出来
        return (
            <Fragment>
                <div>
                 <input value={inputVlaue} onChange={this.inputChange}/>//输入的时候调用函数
                    <button onClick={this.addList}>增加</button>  //点击的时候调用函数
                </div>
                <ul>
                   {list.map((item, index) => {
                        return (
                            <li
                                key={item + index}
                                onClick={() => { this.deleteItem(index) }}>
                                {item}
                            </li>
                        )
                    })}
                </ul>
            </Fragment>
        )
    }
}

export default App
```

#### 连标签一起渲染到页面

##### dangerouslySetInnerHTML

```javascript
 <ul>
                    {list.map((item, index) => {
                        return (
                            <li
                                key={item + index}
                                onClick={() => { this.deleteItem(index) }}
                                dangerouslySetInnerHTML={{__html:item}}  {/*固定写法*/}
                            >
                              
                            </li>
                        )
                    })}
 </ul>
```

#### htmlFor

```javascript
				 <div>
                    <label htmlFor="add">增加:</label> //原生可以用for关键字，但是react中要用hemlFor关键字
                    <input id="add" value={inputVlaue} onChange={this.inputChange} />
                    <button onClick={this.addList}>增加</button>
                </div>
```

### 父子组件传值

##### 父组件

```javascript

import React, { Component, Fragment } from 'react'
import Item from './Item'

class App extends Component {
    constructor(props) {
        super(props)
        this.state = {
            inputVlaue: '',
            list: ["小红", "小燕"]
        }
      
        this.deleteItem = this.deleteItem.bind(this)
    }
    deleteItem(index) {
       
        let list = this.state.list
        list.splice(index, 1)
        this.setState({
            list: list
        })
       
    }
    render() {
        let { list } = this.state
        return (
            <Fragment>
                <ul>
                    {
                        list.map((item, index) => {
                            return (
                                <Item
                                    content={item}  //把内容传递给子组件
                                    deleteItem={() => { this.deleteItem(index) }}//子组件不能操作父组件的值，但是又想改变父组件的值，可以通过传递方法来实现(传递父组件的函数，子组件调用即可)
                                    key={index + item}
                                />
                            )
                        })
                    }
                    {/*  从标签里面把值传递给子组件*/}
                </ul>
            </Fragment>
        )
    }
}

export default App
```

##### 子组件

```javascript
import React, { Component } from 'react';

export class Item extends Component {
    constructor(props) {
        super(props)
    }
    handleClick(){
        console.log('删除index')
    }
    render() {
        let { content ,deleteItem} = this.props; //从props里面接收父组件传递过来的值和方法
        return (
            <li onClick={()=>{deleteItem()}}>{content}</li>//绑定值和绑定方法
        );
    }
}

export default Item;

```

### PropTypes校验传递值

##### 限制接收的类型

```javascript
import React, { Component } from 'react';
import PropTypes from 'prop-types';  //引入

export class Item extends Component {
    constructor(props) {
        super(props)
    }
    handleClick() {
        console.log('删除index')
    }
    render() {
        let { content, deleteItem } = this.props; //从props里面接收父组件传递过来的值
        return (
            <li
                onClick={() => { deleteItem() }}>
               {this.props.name}-{content}
            </li>
        );
    }
}

Item.propTypes = {  //使用方法
    name:PropTypes.string.isRequired,//必须传递并且是字符串类型
    content: PropTypes.string, 
    deleteItem: PropTypes.func 
}

Item.defaultProps = {        //默认值，当父组件没传内容的时候会使用默认的内容
    name:'姓名'
}

export default Item;

```

### ref使用

```javascript
				<div>
                    <label htmlFor="add">增加:</label>
                    <input
                        id="add"
                        value={inputVlaue}
                        onChange={this.inputChange}
                        ref = {(input)=>{this.input = input}}  /* 这里的意思是把input赋值给了 this.input 这样之后就可以用this.input使用这个属性了 */
                    />
                   
                    <button onClick={this.addList}>增加</button>
                </div>
//===========================分割====================================>
inputChange(e) {       
        // this.setState({   //没用ref之前的写法
        //     inputVlaue: e.target.value
        // })
        this.setState({
            inputVlaue: this.input.value  //调用this.input里面的属性
        })

    }
```

#### this.setState异步处理ref

```javascript
 <ul ref={(ul)=>{this.ul = ul}}> //ref绑定 ul
                    {
                        list.map((item, index) => {
                            return (
                                <Item

                                    content={item}
                                    deleteItem={() => { this.deleteItem(index) }}
                                    key={index + item}
                                />
                            )
                        })
                    }
                  
 </ul>
//========================分割==================================>
addList() {
        this.setState({
            list: [...this.state.list, this.state.inputVlaue],
            inputVlaue: ''
        },()=>{
            console.log(this.ul.querySelectorAll('li').length)
        })
    //因为setState是异步的，所以ref获取的DOM属性有时候没有更新那么快
    //因此可以再setState回调函数里面再执行ref之后的操作，可以保证获得正确的信息
    }
```

### 生命周期

#### 1、Initialization:初始化阶段

##### ● 组件开始定义属性（props）和状态(state)

#### 2、Mounting: 挂载阶段

##### ● Mounting阶段叫挂载阶段，伴随着整个虚拟DOM的生成，它里边有三个小的生命周期函数

```javascript
componentWillMount : 在组件即将被挂载到页面的时刻执行。
render : 页面state或props发生变化时执行。
componentDidMount : 组件挂载完成时被执行。

 componentWillMount(){
        console.log('componentWillMount------组件将要挂载到页面的时刻')
    }
    componentDidMount(){
        console.log('componentDidMount------组件挂载完成时刻')
    }
render(){
    console.log('render')
}
//控制台输出 
//componentWillMount------组件将要挂载到页面的时刻
//render
//componentDidMount------组件挂载完成时刻

```

##### 注意：componentWillMount和componentDidMount这两个生命周期函数，只在页面刷新时执行一次，而render函数是只要有state和props变化就会执行

#### 3、Updation: 更新阶段

##### React生命周期中的`Updation`阶段,也就是组件发生改变的更新阶段，这是React生命周期中比较复杂的一部分，它有两个基本部分组成，一个是props属性改变，一个是state状态改变

##### state:

```javascript
shouldComponentUpdate(){
        console.log('shouldComponentUpdate---组件发生改变前执行')
        //需要返回一个true 或者 false 
        //返回false 就不会往下执行了
        return true
    }
    componentWillUpdate(){
         console.log('2-componentWillUpdate---组件更新前，shouldComponentUpdate函数之后执行')
    }
//在组件更新之后执行，它是组件更新的最后一个环节。
    componentDidUpdate(){
        console.log('componentDidUpdate----组件更新之后执行')
    }
render(){
     console.log('3-render-----开始挂载渲染')
}
//1-shouldComponentUpdate---组件发生改变前执行
//2-componentWillUpdate---组件更新前，shouldComponentUpdate函数之后执行
//3-render----开始挂载渲染
//4-componentDidUpdate----组件更新之后执行
```

##### props:子组件生命周期执行

● 凡是组件都有生命周期函数，所以子组件也是有的，并且子组件接收了`props`，这时候函数就可以被执行了。

```javascript
//组件第一次存在于dom中，函数是不会被执行
//如果已经存在于Dom中，函数才会被执行
    componentWillReceiveProps(){
        console.log('子组件-componentWillReceiveProps')
    }
```

● 子组件接收到父组件传递过来的参数，父组件render函数重新被执行，这个生命周期就会被执行。

#### 4、Unmounting: 销毁阶段

##### componentWillUnmount函数

```javascript
//这个函数时组件从页面中删除的时候执行
componentWillUnmount(){
    console.log('child - componentWillUnmount')
}
```

### 优化性能

#### shouldComponentUpdate解决组件重复渲染子组件的问题

```javascript
//shouldComponentUpdate有两个参数：
//1、nextProps:变化后的属性;
//2、nextState:变化后的状态;

//使用：
shouldComponentUpdate(nextProps,nextState){
    if(nextProps.content !== this.props.content){
        return true
    }else{
        return false
    }
   
}
```

### npm

#### npm install -save 和 -save-dev分不清

- `npm install xxx`: 安装项目到项目目录下，不会将模块依赖写入`devDependencies`或`dependencies`。
- `npm install -g xxx`: `-g`的意思是将模块安装到全局，具体安装到磁盘哪个位置，要看 `npm cinfig prefix`的位置
- `npm install -save xxx`：`-save`的意思是将模块安装到项目目录下，并在`package`文件的`dependencies`节点写入依赖。
- `npm install -save-dev xxx`：`-save-dev`的意思是将模块安装到项目目录下，并在`package`文件的`devDependencies`节点写入依赖。

### 发送请求最佳的生命周期函数

##### componentDidMount

```javascript
componentDidMount(){
    axios.post('https://web-api.juejin.im/v3/web/wbbr/bgeda')
        .then((res)=>{console.log('axios 获取数据成功:'+JSON.stringify(res))  })
        .catch((error)=>{console.log('axios 获取数据失败'+error)})
}
```

### EasyMock新建一个接口

##### EasyMock网站:https://www.easy-mock.com/

### React动画 - react-transition-group

##### 安装:npm install react-transition-group --save

##### 三个核心库（或者叫组件）。

- ##### Transition

- ##### CSSTransition

- ##### TransitionGroup

##### 　使用CSSTransition

```javascript
import { CSSTransition } from 'react-transition-group' //引入

//引入后便可以使用了，使用的方法就和使用自定义组件一样,直接写<CSSTransition>，而且不再需要管理className了，这部分由CSSTransition进行管理。

render() { 
    return ( 
        <div>
            <CSSTransition 
                in={this.state.isShow}   //用于判断是否出现的状态
                timeout={2000}           //动画持续时间
                classNames="boss-text"   //className值，防止重复 注意:这里是写classNames
            >
                <div>BOSS级人物-孙悟空</div>
            </CSSTransition>
            <div><button onClick={this.toToggole}>召唤Boss</button></div>
        </div>
        );
}
```

- xxx-enter: 进入（入场）前的CSS样式；
- xxx-enter-active:进入动画直到完成时之前的CSS样式;
- xxx-enter-done:进入完成时的CSS样式;
- xxx-exit:退出（出场）前的CSS样式;
- xxx-exit-active:退出动画知道完成时之前的的CSS样式。
- xxx-exit-done:退出完成时的CSS样式。

##### CSS样式

```css
.input {border:3px solid #ae7000}

.boss-text-enter{
    opacity: 0;
}
.boss-text-enter-active{
    opacity: 1;
    transition: opacity 2000ms;

}
.boss-text-enter-done{
    opacity: 1;
}
.boss-text-exit{
    opacity: 1;
}
.boss-text-exit-active{
    opacity: 0;
    transition: opacity 2000ms;

}
.boss-text-exit-done{
    opacity: 0;
}
```

#### unmountOnExit 属性

##### 给`<CSSTransition>`加上`unmountOnExit`,加上这个的意思是在元素退场时，自动把DOM也删除，这是以前用CSS动画没办法做到的。

```javascript
render() { 
    return ( 
        <div>
            <CSSTransition 
                in={this.state.isShow}   //用于判断是否出现的状态
                timeout={2000}           //动画持续时间
                classNames="boss-text"   //className值，防止重复
                unmountOnExit  //元素退场了，DOM也会被删除
            >
                <div>BOSS级人物-孙悟空</div>
            </CSSTransition>
            <div><button onClick={this.toToggole}>召唤Boss</button></div>
        </div>
        );
}
```

#### TransitionGroup

##### 它就是负责多个DOM元素的动画的，我们还是拿小姐姐这个案例作例子，现在可以添加任何的服务项目，但是都是直接出现的，没有任何动画，现在就给它添加上动画。添加动画，先引入`transitionGrop`

```javascript
import {CSSTransition , TransitionGroup} from 'react-transition-group'//引入

<ul ref={(ul)=>{this.ul=ul}}>
    <TransitionGroup>
    {
        this.state.list.map((item,index)=>{
            return (
                <XiaojiejieItem 
                key={index+item}  
                content={item}
                index={index}
                deleteItem={this.deleteItem.bind(this)}
                />
            )
        })
    }
    </TransitionGroup>
</ul> 

//这个需要放在循环的外边，这样才能形成一个组动画,但是只有这个<TransitonGroup>是不够的，你还是需要加入<CSSTransition>,来定义动画。


```

##### 加入`<CSSTranstion>`标签

```javascript
<ul ref={(ul)=>{this.ul=ul}}>
    <TransitionGroup>
    {
        this.state.list.map((item,index)=>{
            return (
                <CSSTransition
                    timeout={1000}
                    classNames='boss-text'
                    unmountOnExit
                    appear={true}
                    key={index+item}  
                >
                    <XiaojiejieItem 
                    content={item}
                    index={index}
                    deleteItem={this.deleteItem.bind(this)}
                    />
                </CSSTransition>
            )
        })
    }
    </TransitionGroup>
</ul>  
<Boss />
</Fragment>

```

## Redux

### redux处理数据流程

![](C:\Users\Mike\Desktop\React\react笔记\redux.png)

#### redux用法

##### 安装

`npm install --save redux`  或者`yarn add redux`

##### 安装好`redux`之后，在`src`目录下创建一个`store`文件夹,然后在文件夹下创建一个`index.js`文件。

`ndex.js`就是整个项目的`store`文件，打开文件，编写下面的代码。

```javascript
import { createStore } from 'redux'  // 引入createStore方法
const store = createStore()          // 创建数据存储仓库
export default store                 //暴露出去
```

这样虽然已经建立好了仓库，但是这个仓库很混乱，这时候就需要一个有管理能力的模块出现，这就是`Reducers`

在`store`文件夹下，新建一个文件`reducer.js`,然后写入下面的代码。

```javascript
const defaultState = {}  //默认数据
export default (state = defaultState,action)=>{  //就是一个方法函数
    return state
}
```

- `state`: 是整个项目中需要管理的数据信息,这里我们没有什么数据，所以用空对象来表示。

这样`reducer`就建立好了，把reducer引入到`store`中,再创建store时，以参数的形式传递给store。

```javascript
import { createStore } from 'redux'  //  引入createStore方法
import reducer from './reducer'    
const store = createStore(reducer) // 创建数据存储仓库
export default store   //暴露出去
```

##### 在store中为todoList初始化数据

仓库`store`和`reducer`都创建好了，可以初始化一下`todoList`中的数据了，在`reducer.js`文件的`defaultState`对象中，加入两个属性:`inputValue`和`list`。代码如下

```javascript
const defaultState = {
    inputValue: '输入人名',
    list: ['小红', '小燕']
}

export default (state = defaultState, action) => {
    return state
}
```

这就相当于你给`Store`里增加了两个新的数据。

##### 组件获得`store`中的数据

有了store仓库，也有了数据，那如何获得stroe中的数据那？你可以在要使用的组件中，先引入store。 我们todoList组件要使用store，就在`src/TodoList.js`文件夹中，进行引入。

```javascript
import React, { Component } from 'react';
import { Input,Button,List } from 'antd';
import store from './store'  //引入store  



class TodoList extends Component {
    constructor(props){
        super(props)
        console.log(store.getState())
        this.state = store.getState()  //调用redux提供的方法getState获取store 然后赋值给this.state
        
        
    }
    render() {
        return (
            <div>
                <div style={{margin:"20px"}}>
                    <Input
                        placeholder={this.state.inputValue}
                        style={{ width: '250px',marginRight:'10px' }}
                    />
                    <Button type="primary">增加</Button>
                </div>
                <div style={{margin:'10px',width:'300px'}}>
                    <List
                        bordered
                        dataSource={this.state.list}
                        renderItem={item =>(<List.Item>{item}</List.Item>)}
                    ></List>
                </div>
            </div>
        );
    }
}

export default TodoList;
```

#### 配置`Redux Dev Tools`

```javascript
import { createStore } from 'redux'  //  引入createStore方法
import reducer from './reducer'    
const store = createStore(reducer,
window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()) // 创建数据存储仓库 加了之后就可以再浏览器看到redux数据了
export default store   //暴露出去
```

#### 通过Input体验Redux的流程

```javascript
import React, { Component } from 'react';
import { Input,Button,List } from 'antd';
import store from './store'



class TodoList extends Component {
    constructor(props){
        super(props)
        console.log(store.getState())
        this.state = store.getState() //获取store里面的参数api
        
        this.changInputValue = this.changInputValue.bind(this)
        this.storeChange = this.storeChange.bind(this)
        store.subscribe(this.storeChange) //订阅store更新状态
    }
    render() {
        return (
            <div>
                <div style={{margin:"20px"}}>
                    <Input
                        placeholder={this.state.inputValue}
                        style={{ width: '250px',marginRight:'10px' }}
                        onChange={this.changInputValue}
                        value={this.state.inputValue}
                    />
                    <Button type="primary">增加</Button>
                </div>
                <div style={{margin:'10px',width:'300px'}}>
                    <List
                        bordered
                        dataSource={this.state.list}
                        renderItem={item =>(<List.Item>{item}</List.Item>)}
                    ></List>
                </div>
            </div>
        );
    }
    changInputValue(e){ 
        const action = {    //想改变Redux里边State的值就要创建Action了。Action就是一个对象，这个对象一般有两个属性，第一个是对Action的描述，第二个是要改变的值。
            type:'changeInput',
            value:e.target.value
        }
  //action就创建好了，但是要通过dispatch()方法传递给store。我们在action下面再加入一句代码。      
        store.dispatch(action)
    }
    storeChange(){
        this.setState(store.getState())  //数据发生变化，然后setState变化的数据
    }
}

export default TodoList;
```

##### `store`的自动推送策略

`store`只是一个仓库，它并没有管理能力，它会把接收到的`action`自动转发给`Reducer`。我们现在先直接在`Reducer`中打印出结果看一下。打开`store`文件夹下面的`reducer.js`文件，修改代码。

#### Reducer

##### Reducer的两个参数

- **state**: 指的是原始仓库里的状态。

- **action**: 指的是action新传递的状态。

  （**记住：Reducer里只能接收state，不能改变state。**）,所以我们声明了一个新变量，然后再次用`return`返回回去。

```javascript
export default (state = defaultState, action) => {
    //Reducer里只能接收state,不能改变state
    if(action.type === 'changeInput'){
        let newState = JSON.parse(JSON.stringify(state))//深度拷贝state
        newState.inputValue = action.value;
        return newState
    }
    return state
}
```

#### 把Action Types 单度写入一个文件

写`Redux Action`的时候，我们写了很多Action的派发，产生了很多`Action Types`，如果需要`Action`的地方我们就自己命名一个`Type`,会出现两个基本问题：

- ##### 这些Types如果不统一管理，不利于大型项目的服用，设置会长生冗余代码。

- ##### 因为`Action`里的`Type`，一定要和`Reducer`里的`type`一一对应在，所以这部分代码或字母写错后，浏览器里并没有明确的报错，这给调试带来了极大的困难。

把`Action Type`单独拆分出一个文件。在`src/store`文件夹下面，新建立一个`actionTypes.js`文件，然后把Type集中放到文件中进行管理

```javascript
export const CHANGE_INPUT = 'CHANGE_INPUT';
export const ADD_LIST = 'ADD_LIST';
export const REMOVE_ITEM = 'REMOVE_ITEM';

```

#### 引入Reducer并进行更改

```javascript
import {CHANGE_INPUT,ADD_LIST,REMOVE_ITEM} from './actionTypes'
const defaultState = {
    inputValue: '输入人名',
    list: ['小红', '小燕']
}

export default (state = defaultState, action) => {
    //Reducer里只能接收state,不能改变state
    if(action.type === CHANGE_INPUT){
        let newState = JSON.parse(JSON.stringify(state))
        newState.inputValue = action.value;
        return newState
    }
    if(action.type === ADD_LIST){
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.push(newState.inputValue)
        newState.inputValue = ''
        return newState
    }
    if(action.type === REMOVE_ITEM){
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.splice(action.index,1)
        return newState
    }
    return state
}
```

#### 编写`actionCreators.js`文件

在`/src/store`文件夹下面，建立一个新的的文件`actionCreators.js`，先在文件中引入`actionTypes.js`文件

```javascript
import {CHANGE_INPUT,ADD_LIST,REMOVE_ITEM} from './actionTypes'

export const changeInput = (value) => ({
    type:CHANGE_INPUT,
    value
})

export const addList = () => ({
    type:ADD_LIST
})

export const removeItem = (index) => ({
    type:REMOVE_ITEM,
    index
})
```

#### 修改todoList中的代码

```javascript
import React, { Component } from 'react';
import { Input,Button,List } from 'antd';
import store from './store'
import {changeInput,addList,removeItem} from './store/actionCreators'


class TodoList extends Component {
    constructor(props){
        super(props)
        console.log(store.getState())
        this.state = store.getState()
        
        this.changInputValue = this.changInputValue.bind(this)
        this.storeChange = this.storeChange.bind(this)
        this.addList = this.addList.bind(this)
        this.removeItem = this.removeItem.bind(this)
        store.subscribe(this.storeChange)
    }
    render() {
        return (
            <div>
                <div style={{margin:"20px"}}>
                    <Input
                        placeholder={this.state.inputValue}
                        style={{ width: '250px',marginRight:'10px' }}
                        onChange={this.changInputValue}
                        value={this.state.inputValue}
                    />
                    <Button type="primary" onClick={this.addList}>增加</Button>
                </div>
                <div style={{margin:'10px',width:'300px'}}>
                    <List
                        bordered
                        dataSource={this.state.list}
                        renderItem={(item,index) =>(<List.Item onClick={()=>{this.removeItem(index)}}>{item}</List.Item>)}
                       
                    ></List>
                </div>
            </div>
        );
    }
    changInputValue(e){
        store.dispatch(changeInput(e.target.value)) //直接在dispatch里面调用这个函数，传入参数即可使用
    }
    addList(){
       
        store.dispatch(addList())
    }
    removeItem(index){
      
        store.dispatch(removeItem(index))
    }
    storeChange(){
        this.setState(store.getState())  //数据发生变化，然后setState变化的数据
    }
}

export default TodoList;
```

#### 组件UI和业务逻辑的拆分方法

##### 拆分UI组件

然后去`TodoList.js`里把`JSX`的部分拷贝过来， 现在的代码如下(当然现在是不可以使用的，好多东西我们还没有引入，所以会报错):

```javascript
import React, { Component } from 'react';
import { Input, Button, List } from 'antd'; //记得把ui框架的组件也拿过来
class TodoListUi extends Component {
    
    render() { 
        return ( 
            <div>
                <div style={{ margin: "20px" }}>
                    <Input
                        placeholder={this.state.inputValue}
                        style={{ width: '250px', marginRight: '10px' }}
                        onChange={this.changInputValue}
                        value={this.state.inputValue}
                    />
                    <Button type="primary" onClick={this.addList}>增加</Button>
                </div>
                <div style={{ margin: '10px', width: '300px' }}>
                    <List
                        bordered
                        dataSource={this.state.list}
                        renderItem={(item, index) => (<List.Item onClick={() => { this.removeItem(index) }}>{item}</List.Item>)}

                    ></List>
                </div>
            </div>
         );
    }
}
 
export default TodoListUi;

```

但是这并没有`TodoListUI.js`组件所需要的`state`(状态信息)，接下来需要改造父组件进行传递值

#### `TodoList.js`文件的修改

```javascript

import TodoListUi from './TodoListUi'  //引入

    render() {
        return (
            <TodoListUi></TodoListUi> //使用
        );
    }
  
```

##### UI组件和业务逻辑组件的整合

其实整合就是通过属性传值的形式，把需要的值传递给子组件，子组件接收这些值，进行相应的绑定就可以了

TodoList.js的render部分

```javascript
 render() {
        return (
            <TodoListUi
                inputValue={this.state.inputValue}
                changInputValue={this.changInputValue}
                addList={this.addList}
                list={this.state.list}
                removeItem={this.removeItem}
            ></TodoListUi>
        );
    }
```

todoListUi部分

```javascript
import React, { Component } from 'react';
import { Input, Button, List } from 'antd';
class TodoListUi extends Component {
    render() {
        return (
            <div>
                <div style={{ margin: "20px" }}>
                    <Input
                        placeholder={this.props.inputValue}
                        style={{ width: '250px', marginRight: '10px' }}
                        onChange={this.props.changInputValue}
                        value={this.props.inputValue}
                    />
                    <Button type="primary" onClick={this.props.addList}>增加</Button>
                </div>
                <div style={{ margin: '10px', width: '300px' }}>
                    <List
                        bordered
                        dataSource={this.props.list}
                        renderItem={(item, index) => (<List.Item onClick={() => { this.props.removeItem(index) }}>{item}</List.Item>)}

                    ></List>
                </div>
            </div>
        );
    }
}

export default TodoListUi;
```

### Redux中的无状态组件

##### 无状态组件其实就是一个函数，它不用再继承任何的类（class），当然如名字所一样，也不存在`state`（状态）。因为无状态组件其实就是一个函数（方法）,所以它的性能也比普通的`React`组件要好。

#### 无状态组件的改写

把UI组件改成无状态组件可以提高程序性能，具体来看一下如何编写。

1. 首先我们不在需要引入React中的`{ Component }`，删除就好。
2. 然后些一个`TodoListUI`函数,里边只返回`JSX`的部分就好，这步可以复制。
3. 函数传递一个`props`参数，之后修改里边的所有`props`，去掉`this`。

```javascript
import React from 'react';
import { Input, Button, List } from 'antd';

const TodoListUi = (props) =>{
    return (
        <div>
            <div style={{ margin: "20px" }}>
                <Input
                    placeholder={props.inputValue}
                    style={{ width: '250px', marginRight: '10px' }}
                    onChange={props.changInputValue}
                    value={props.inputValue}
                />
                <Button type="primary" onClick={props.addList}>增加</Button>
            </div>
            <div style={{ margin: '10px', width: '300px' }}>
                <List
                    bordered
                    dataSource={props.list}
                    renderItem={(item, index) => (<List.Item onClick={() => {props.removeItem(index) }}>{item}</List.Item>)}

                ></List>
            </div>
        </div>
    );
}

export default TodoListUi;
```

#### Axios异步获取数据并和Redux结合

##### 首先利用easy-mock创建模拟数据

##### 然后引入axios

引入后，在组件的声明周期函数里`componentDidMount`获取远程接口数据。

```javascript
componentDidMount(){
        axios.get('https://easy-mock.com/mock/5dbd28d99e771148eb1d921e/example/List').then((res)=>{
            console.log(res.data.data)
        })
    }
```

#### 获取数据后跟`Redux`相结合（重点）

接下来就可以把得到的数据放到`Redux`的`store`中了，这部分和以前的知识都一样，我就尽量给出代码，少讲理论了。 先创建一个函数，打开以前写的`store/actionCreatores.js`函数，然后写一个新的函数，代码如下：

```javascript
export const getListAction = (data) =>({
    type:GET_LIST,
    data
})
```

回到`TodoList.js`文件，继续编写`axios`中的回调内容，在写之前，记得先把`getListAction`进行引入

```javascript
componentDidMount(){
        axios.get('https://easy-mock.com/mock/5dbd28d99e771148eb1d921e/example/List').then((res)=>{
            store.dispatch(getListAction(res.data.data))
        })
    }
```

现在数据已经通过`dispatch`传递给`store`了，接下来就需要`reducer`处理业务逻辑了

```javascript
export default (state = defaultState, action) => {
   
    if(action.type === GET_LIST){   //在这里设置默认的list值
        let newState = JSON.parse(JSON.stringify(action.data))
        return newState
    }
    if(action.type === CHANGE_INPUT){
        let newState = JSON.parse(JSON.stringify(state))
        newState.inputValue = action.value;
        return newState
    }
    if(action.type === ADD_LIST){
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.push(newState.inputValue)
        newState.inputValue = ''
        return newState
    }
    if(action.type === REMOVE_ITEM){
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.splice(action.index,1)
        return newState
    }

    return state
}
```

### Redux-thunk中间件的安装和配置

##### 安装`Redux-thunk`组件

`Redux-thunk`并不在`Redux`基础组件中，也就是说需要进行新安装。安装使用`npm`就可以了。

```js
yanr add redux-thunk
```

##### 配置`Redux-thunk`组件

1.引入`applyMiddleware`,如果你要使用中间件，就必须在redux中引入`applyMiddleware`.

2.引入`redux-thunk`库

3.要使用**增强函数**。使用增加函数前需要先引入`compose`

```js
import { createStore , applyMiddleware ,compose } from 'redux' 
```

```js
import { createStore , applyMiddleware ,compose } from 'redux'  //  引入createStore方法
import reducer from './reducer'    
import thunk from 'redux-thunk'

const composeEnhancers =   window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ?
    window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({}):compose

const enhancer = composeEnhancers(applyMiddleware(thunk))

const store = createStore( reducer, enhancer) // 创建数据存储仓库
export default store   //暴露出去
```

### Redux-thunk的使用方法

### 在`actionCreators.js`里编写业务逻辑

以前`actionCreators.js`都是定义好的action，根本没办法写业务逻辑，有了`Redux-thunk`之后，可以把`TodoList.js`中的`componentDidMount`业务逻辑放到这里来编写。也就是把向后台请求数据的代码放到`actionCreators.js`文件里。那我们需要引入`axios`,并写一个新的函数方法。（以前的action是对象，现在的action可以是函数了，这就是`redux-thunk`带来的好处）

```js
import axios from 'axios'

export const getTodoList = () =>{
    return ()=>{
        axios.get('https://www.easy-mock.com/mock/5cfcce489dc7c36bd6da2c99/xiaojiejie/getList').then((res)=>{
            const data = res.data
            console.log(data)
        })
    }
}
```

现在我们需要执行这个方法，并在控制台查看结果，这时候可以修改`TodoList.js`文件中的`componentDidMount`代码。

```js
import {getTodoList , changeInputAction , addItemAction ,deleteItemAction,getListAction} from './store/actionCreatores'
---something---
componentDidMount(){
    const action = getTodoList()
    store.dispatch(action)
}
```

然后我们到浏览器的控制台中查看一下，看看是不是已经得到了后端传给我们的数据，如果一切正常，应该是可以得到。得到之后，我们继续走以前的Redux流程就可以了

```js
export const getTodoList = () =>{
    return (dispatch)=>{
        axios.get('https://www.easy-mock.com/mock/5cfcce489dc7c36bd6da2c99/xiaojiejie/getList').then((res)=>{
            const data = res.data
            const action = getListAction(data)
            dispatch(action)
            
        })
    }
}
```

### Redux-saga的安装和配置

##### redux-saga的安装

```text
yarn add redux-saga
```

安装好后，就可以使用了，直接在`/src/store/index.js`里引入`redux-saga`,并创建一个`sagaMiddleware`，代码如下

创建好后，还是用Redux的增强函数进行传递。

```js
import { createStore , applyMiddleware ,compose } from 'redux'  //  引入createStore方法
import reducer from './reducer'   
//------关键代码----start----------- 
import createSagaMiddleware from 'redux-saga' 
const sagaMiddleware = createSagaMiddleware();
//------关键代码----end-----------

const composeEnhancers =   window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ?
    window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({}):compose
//------关键代码----start-----------
const enhancer = composeEnhancers(applyMiddleware(sagaMiddleware))
//------关键代码----end-----------

const store = createStore( reducer, enhancer) // 创建数据存储仓库
export default store   //暴露出去
```

##### 创建`sagas.js`文件并引入

`redux-saga`希望你把业务逻辑单独写一个文件，这里我们在`/src/store/`文件夹下建立一个`sagas.js`文件。

```js
function* mySaga() {} 
export default mySaga;
```

创建好以后直接引入到`index.js`里。

```js
import mySagas from './sagas' 
```

然后执行run方法，让`saga`运行起来。

```js
sagaMiddleware.run(mySagas)
```

`/src/store/index.js`的全部内容。

```js
import { createStore , applyMiddleware ,compose } from 'redux'  //  引入createStore方法
import reducer from './reducer'   
import createSagaMiddleware from 'redux-saga' 
import mySagas from './sagas' 

const sagaMiddleware = createSagaMiddleware();

const composeEnhancers =   window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ?
    window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({}):compose
const enhancer = composeEnhancers(applyMiddleware(sagaMiddleware))
const store = createStore( reducer, enhancer) 
sagaMiddleware.run(mySagas)


export default store  

```

### 使用Redux-saga

#### 编写sagas.js文件(也是业务逻辑)

用`saga`的中间件业务逻辑，就都写在这个`sagas.js`文件里，文件里我们用`mySaga`来作为入口函数。在入口函数中捕获传递过来的`action`类型，根据类型不同调用不同的方法。

测试部分：

```js
import { takeEvery } from 'redux-saga/effects'  
import {GET_MY_LIST} from './actionTypes'

//generator函数
function* mySaga() {
    //等待捕获action
    yield takeEvery(GET_MY_LIST, getList)
}

function* getList(){
  yield  console.log('ok')
}
  
export default mySaga;
```

真正使用：

```js
import { takeEvery ,put } from 'redux-saga/effects'  
import {GET_MY_LIST} from './actionTypes'
import {getListAction} from './actionCreatores'
import axios from 'axios'

//generator函数
function* mySaga() {
    //等待捕获action
    yield takeEvery(GET_MY_LIST, getList)
}

function* getList(){
   
    const res = yield axios.get('https://www.easy-mock.com/mock/5cfcce489dc7c36bd6da2c99/xiaojiejie/getList')
    const action = getListAction(res.data)
    yield put(action)
}
  
export default mySaga;
```

### react-redux

#### react-redux安装

```
yarn add react-redux
```

#### `<Provider>`提供器讲解

`<Provider>`是一个提供器，只要使用了这个组件，组件里边的其它所有组件都可以使用`store`了，这也是`React-redux`的核心组件了

```js
import React from 'react';
import ReactDOM from 'react-dom';
import {Provider} from "react-redux";
import TodoList from './TodoList';
import store from './store'

const App = (
   <Provider store={store}><TodoList /></Provider>  //只要保证Provider是最外层的容器，并且把store传递给它 那么就可以在里面使用了
)

ReactDOM.render(App, document.getElementById('root'));

```

#### `connect`连接器的使用

现在如何简单的获取`store`中数据那？先打开`TodoList.js`文件，引入`connect`，它是一个连接器（其实它就是一个方法），有了这个连接器就可以很容易的获得数据了。

```js
import {connect} from 'react-redux'  //引入连接器
```

这时候暴露出去的就变成了`connect`了，代码如下。

```js
export default connect(xxx,null)(TodoList);
```

#### 映射关系的制作

映射关系就是把原来的state映射成组件中的`props`属性，比如我们想映射`inputValue`就可以写成如下代码。

```js
const stateToProps = (state) =>{
    return {
        inputValue:state.inputValue,
        list:state.list
    }
}
```

这时候再把`xxx`改为`stateToProps`

```js
export default connect(stateToProps,null)(TodoList)
```

然后把`<input>`里的`state`标签，改为`props`,代码如下:

```jsx
 <input value={this.props.inputValue} />
```

半成品代码：

```jsx
import React, { Component } from 'react';
import store from './store/index'
import {connect} from 'react-redux';

class TodoList extends Component {
    constructor(props){
        super(props)
        
    }
    render() {
        return (
            <div>
                <div>
                    <input 
                        value={this.props.inputVlaue}      
                    />
                    <button>按我</button>
                   
                </div>
                <ul>
                    {
                        this.props.list.map((item,index)=>{
                            return <li key={item+index}>{item}</li>
                        })
                    }
                </ul>
            </div>
        );
    }
}

const stateToProps = (state) =>{
    return {
        inputValue:state.inputValue,
        list:state.list
    }
}
const dispatchToProps = () =>{
    return {

    }
}


export default connect(stateToProps,dispatchToProps)(TodoList);
```

#### React-redux的数据修改

##### 编写`DispatchToProps`

要使用`react-redux`，我们可以编写另一个映射`DispatchToProps`

`DispatchToProps`就是要传递的第二个参数，通过这个参数才能改变`store`中的值。

```jsx
const dispatchToProps = (dispatch) => {
    return {
        inputChange(e){
            let action = {
                type:'INPUT_CHANGE',
                data:e.target.value
            }
            dispatch(action) //派发action到store中
        }
}
export default connect(stateToProps, dispatchToProps)(TodoList);
```

有了这个参数之后可以把响应事件改成下面的代码.

```jsx
<input
      value={this.props.inputVlaue}
      onChange={this.props.inputChange.bind(this)}
/>
```

##### 派发`action`到store中

映射关系已经做好了，接下来只要进行`action`的派发和`reducer`对业务逻辑的编写就可以了。

```js
import React, { Component } from 'react';
import store from './store/index'
import { connect } from 'react-redux';

class TodoList extends Component {
   
    render() {
        return (
            <div>
                <div>
                    <input
                        value={this.props.inputVlaue}
                        onChange={this.props.inputChange.bind(this)}
                    />
                    <button onClick={this.props.addItem.bind(this)}>按我</button>

                </div>
                <ul>
                    {
                        this.props.list.map((item, index) => {
                            return <li key={item + index}>{item}</li>
                        })
                    }
                </ul>
            </div>
        );
    }
    
}

const stateToProps = (state) => {
    return {
        inputValue: state.inputValue,
        list: state.list
    }
}
const dispatchToProps = (dispatch) => {
    return {
        inputChange(e){
            let action = {
                type:'INPUT_CHANGE',
                data:e.target.value
            }
            dispatch(action)
        },
        addItem(){
            let action = {
                type:'ADD_ITEM',
            }
            dispatch(action)
        }
    }
}


export default connect(stateToProps, dispatchToProps)(TodoList);
```

派发后就需求在`reducer`里边，编写对应的业务逻辑了。

```js
export default (state = defalutValue,action) =>{
    if(action.type === "INPUT_CHANGE"){
        let newState = JSON.parse(JSON.stringify(state))
        newState.inputValue = action.data
        return newState
    }
    if(action.type === "ADD_ITEM"){
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.push(newState.inputValue)
        newState.inputValue = ''
        return newState
    }

    return state
}
```

#### 精简成UI组件

UI组件(无状态组件)，UI组件就是一个方法，减少很多冗余操作，从而提高程序运行性能。

```jsx
import React from 'react';
import { connect } from 'react-redux';

const TodoList = (props) => {
    let { inputVlaue, inputChange, addItem, list } = props;
    return (
        <div>
            <div>
                <input
                    value={inputVlaue}
                    onChange={inputChange.bind(this)}
                />
                <button onClick={addItem.bind(this)}>按我</button>
            </div>
            <ul>
                {
                    list.map((item, index) => {
                        return <li key={item + index}>{item}</li>
                    })
                }
            </ul>
        </div>
    );
}


const stateToProps = (state) => {
    return {
        inputValue: state.inputValue,
        list: state.list
    }
}
const dispatchToProps = (dispatch) => {
    return {
        inputChange(e) {
            let action = {
                type: 'INPUT_CHANGE',
                data: e.target.value
            }
            dispatch(action)
        },
        addItem() {
            let action = {
                type: 'ADD_ITEM',
            }
            dispatch(action)
        }
    }
}
export default connect(stateToProps, dispatchToProps)(TodoList);
```

##### `connect`的作用是把UI组件（无状态组件）和业务逻辑代码的分开，然后通过connect再链接到一起，让代码更加清晰和易于维护。这也是`React-Redux`最大的有点。

### React Router

#### 安装React Router

```js
yarn add react-router-dom
```

#### 编写一个最简单的路由程序

首先我们改写`src`文件目录下的`index.js`代码

```js
import React from 'react';
import ReactDOM from 'react-dom'
import AppRouter from './AppRouter'

ReactDOM.render(<AppRouter/>,document.getElementById('root'))
```

##### 编写Index组件

先在`/src`目录下建立一个文件夹，起名叫做`Pages`，然后建立一个组件文件`Index.js`。

```js
import React, { Component } from 'react';

class Index extends Component {
    render() {
        return (
            <div>
                <h2>小燕</h2>
            </div>
        );
    }
}

export default Index;
```

##### 编写List组件

```js
import React, { Component } from 'react';

export class List extends Component {
    render() {
        return (
            <div>
                <h2>list-page</h2>
            </div>
        );
    }
}

export default List;

```

##### `AppRouter.js`文件的配置

```js
import React from "react";
import { BrowserRouter as Router, Route, Link } from "react-router-dom";
import Index from './Pages/Index'
import List from './Pages/List'

function AppRouter() {
  return (
    <Router>
        <ul>
            <li> <Link to="/">首页</Link> </li>
            <li><Link to="/list/">列表</Link> </li>
        </ul>
        <Route path="/" exact component={Index} />
        <Route path="/list/" component={List} />
    </Router>
  );
}

export default AppRouter;

```

#### `exact`精准匹配

路径信息要完全匹配成功，才可以实现跳转，匹配一部分是不行的。

把`Index`的精准匹配去掉，你会发现，无论你的地址栏输入什么都可以匹配到`Index`组件，这并不是我们想要的结果。

```js
<Route path="/" component={Index} />
```

所以我们加上了精准匹配`exact`

#### ReactRouter动态传值

##### 在Route上设置允许动态传值

这个设置是以`:`开始的，然后紧跟着你传递的key（键名称）名称。我们来看一个简单的例子。

```js
<Route path="/list/:id" component={List} />
```

##### Link上传递值

设置好规则后，就可以在`Link`上设置值了，现在设置传递的ID值了，这时候不用再添加id了，直接写值就可以了。

```js
 <li><Link to="/list/123">列表</Link> </li>
```

`AppRouter.js`代码。

```js
import React from 'react';
import {BrowserRouter as Router,Route,Link} from 'react-router-dom';
import Index from './Pages/Index'
import List from './Pages/List'


function AppRouter(){
    return (
        <Router>
            <ul>
                <li><Link to="/">首页</Link></li>
                <li><Link to="/list/123">列表</Link></li>
            </ul>
            <Route path="/" exact component={Index} />
            <Route path="/list/:id"  component={List} />
        </Router>
    )
}

export default AppRouter;
```

##### 在List组件上接收并显示传递值

组件接收传递过来的值的时候，可以在声明周期`componentDidMount`中进行，传递的值在`this.props.match`中。我们可以先打印出来,代码如下。

```js
import React, { Component } from 'react';

class List extends Component {
    constructor(props) {
        super(props);
        this.state = {  }
    }
    render() { 
        return (  <h2>List Page</h2> );
    }
    //-关键代码---------start
    componentDidMount(){
        console.log(this.props.match)
    }
    //-关键艾玛---------end
}
 
export default List;
```

然后在浏览器的控制台中可以看到打印出的对象，对象包括三个部分:

- patch:自己定义的路由规则，可以清楚的看到是可以产地id参数的。
- url: 真实的访问路径，这上面可以清楚的看到传递过来的参数是什么。
- params：传递过来的参数，`key`和`value`值。

明白了match中的对象属性，就可以轻松取得传递过来的ID值了。代码如下:

```js
import React, { Component } from 'react';

class List extends Component {
    constructor(props) {
        super(props);
        this.state = {  }
    }
    render() { 
        return (  <h2>List Page->{this.state.id}</h2> );
    }
    componentDidMount(){
       // console.log(this.props.match.params.id)
       let tempId=this.props.match.params.id
        this.setState({id:tempId })
    }
}
 
export default List;
```

这样就实现了动态传值，需要注意的是如果你不传任何东西，是没办法匹配路由成功的。

#### 动态传值案例

##### 模拟一个列表数组

现在可以在`Index`组件里模拟一个列表数组，就相当于我们从后台动态获取到的内容，然后数组中包括文章的`cid`和`title`。直接在state初始化时进行设置，代码如下：

```js
 constructor(props) {
    super(props);
    this.state={
            list:[
                {cid:123,title:'你不知道的javascript'},
                {cid:456,title:'javascript高级程序设计第三版'},
                {cid:789,title:'锋利的jQuery'}
            ]
        }
}
```

有了`list`数组后，再修改一下UI，进行有效的遍历，`Render`代码如下。

```js
 render() { 
    return ( 
        <ul>
            {
                this.state.list.map((item,index)=>{
                    return (
                        <li key={index}> {item.title} </li>
                    )
                })
            }
        </ul>
    )
}
```

列表已经可以在`Index`组件里显示出来了，接下来可以配置`<Link>`了,在配置之前，你需要先引入`Link`组件。

```js
import { Link } from "react-router-dom";
```

引入后直接使用进行跳转就可以，但是需要注意一点，要用`{}`的形式，也就是把`to`里边的内容解析成JS的形式，这样才能顺利的传值过去。

```js
render() { 
    return ( 
        <ul>
            {
                this.state.list.map((item,index)=>{
                    return (
                        <li key={index}>
                            <Link to={'/list/'+item.uid}> {item.title}</Link> 
                        </li>
                    )
                })
            }
        </ul>
    )
}
```

`Index.js`的所有代码.

```js
import React, { Component } from 'react';
import {Link} from 'react-router-dom';
class Index extends Component {
    constructor(props){
        super(props)
        this.state={
            list:[
                {cid:123,title:'你不知道的javascript'},
                {cid:456,title:'javascript高级程序设计第三版'},
                {cid:789,title:'锋利的jQuery'}
            ]
        }
    }
    render() {
        return (
            <div>
              <ul>
                  {
                      this.state.list.map((item,index)=>{
                      return (
                          <li key={item+index}>
                            <Link to={'/list/'+item.cid}>{item.title}</Link>
                          </li>
                      )
                    })
                  }
              </ul>
            </div>
        );
    }
}

export default Index;
```

#### ReactRouter重定向-Redirect使用

- 标签式重定向:就是利用`<Redirect>`标签来进行重定向，业务逻辑不复杂时建议使用这种。
- 编程式重定向:这种是利用编程的方式，一般用于业务逻辑当中，比如登录成功挑战到会员中心页面。

重定向和跳转有一个重要的区别，就是跳转式可以用浏览器的回退按钮返回上一级的，而重定向是不可以的。

#### 标签式重定向

进入`Index`组件，然后`Index`组件,直接重定向到`Home`组件

首先建立一个`Home.js`的页面，这个页面我还是用快速生成的方式来进行编写，代码如下。

```js
import React, { Component } from 'react';

class Home extends Component {
    constructor(props) {
        super(props);
        this.state = {  }
    }
    render() { 
        return (  <h2>我是 Home 页面</h2> );
    }
}
 
export default Home;
```

这个页面非常简单，就是一个普通的有状态组件。

有了组件后可以配置一下路由规则，也就是在`AppRouter.js`里加一个`<Route>`配置，配置时记得引入`Home`组件。

```js
import Home from './Pages/Home'
<Route path="/home/" component={Home} />
```

之后打开`Index.js`文件，从`Index`组件重新定向到`Home`组件，需要先引入`Redirect`。

```js
import { Link , Redirect } from "react-router-dom";
```

引入`Redirect`后，直接在`render`函数里使用就可以了。

```js
 <Redirect to="/home/" />
```

现在就可以实现页面的重定向。

#### 编程式重定向

编程式重定向就是不再利用`<Redirect/>`标签，而是直接使用`JS`的语法实现重定向。他一般用在业务逻辑比较发杂的场合或者需要多次判断的场合。我们这里就不模拟复杂场景了，还是利用上边的例子实现一下，让大家看到结果就好。

比如直接在构造函数`constructor`中加入下面的重定向代码。

```js
 this.props.history.push("/home/"); 
```

#### 后台动态获取路由进行配置

一个后台管理系统，菜单并不是写死的，而是通过后台接口获得的，这时候我们要如何根据后台接口编写我们的路由。

##### 模拟后台得到的JSON数据

我们现在`AppRouter.js`文件里，模拟从后台得到了JSON字符串，并转换为了对象（我们只是模拟，就不真的去远端请求数据了）。模拟的代码如下:

```js
let routeConfig =[
    {path:'/',title:'博客首页',exact:true,component:Index},
    {path:'/video/',title:'视频教程',exact:false,component:Video},
    {path:'/workplace/',title:'职场技能',exact:false,component:Workplace}
]
```

##### 循环出Link区域

这时候一级导航就不能是写死了，需要根据得到的数据进行循环出来。直接使用`map`循环就可以。代码如下：

```js
<ul>
    {
        routeConfig.map((item,index)=>{
            return (<li key={index}> <Link to={item.path}>{item.title}</Link> </li>)
        })
    }
</ul>
```

这时候就可以把所有的Link标签都循环出来了。

##### 循环出路由配置

按照上面的逻辑把`Route`的配置循环出来。代码如下:

```js
{
    routeConfig.map((item,index)=>{
        return (<Route key={index} exact={item.exact} path={item.path}  component={item.component} />)
    })
}
```

总代码

```js
import React from "react";
import { BrowserRouter as Router, Route, Link  } from "react-router-dom";
import Index from './Pages/Index'
import Video from './Pages/Video'
import Workplace from './Pages/Workplace'
import './index.css'

function AppRouter() {
    let routeConfig =[
      {path:'/',title:'博客首页',exact:true,component:Index},
      {path:'/video/',title:'视频教程',exact:false,component:Video},
      {path:'/workplace/',title:'职场技能',exact:false,component:Workplace}
    ]
    return (
      <Router>
          <div className="mainDiv">
            <div className="leftNav">
                <h3>一级导航</h3>
                <ul>
                    {
                      routeConfig.map((item,index)=>{
                          return (<li key={index}> <Link to={item.path}>{item.title}</Link> </li>)
                      })
                    }
                </ul>
            </div>
            
            <div className="rightMain">
                    {
                      routeConfig.map((item,index)=>{
                          return (<Route key={index} exact={item.exact} path={item.path}  component={item.component} />)
                      })
                    }
                
            </div>
          </div>
      </Router>
    );
  }
  
  
  export default AppRouter;
  
```

### React Hooks 简介

2018年底FaceBook的React小组推出Hooks以来，所有的React的开发者都对它大为赞赏。`React Hooks`就是用函数的形式代替原来的继承类的形式，并且使用预函数的形式管理`state`，有Hooks可以不再使用类的形式定义组件了。这时候你的认知也要发生变化了，原来把组件分为有状态组件和无状态组件，有状态组件用类的形式声明，无状态组件用函数的形式声明。那现在所有的组件都可以用函数来声明了。

#### React Hooks 编写形式对比

原始写法:

```js
import React, { Component } from 'react';

class Example extends Component {
    constructor(props) {
        super(props);
        this.state = { count:0 }
    }
    render() { 
        return (
            <div>
                <p>You clicked {this.state.count} times</p>
                <button onClick={this.addCount.bind(this)}>Chlick me</button>
            </div>
        );
    }
    addCount(){
        this.setState({count:this.state.count+1})
    }
}
 
export default Example;
```

React Hooks 写法：

```js
import React, { useState } from 'react';
function Example(){
    const [ count , setCount ] = useState(0);
    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={()=>{setCount(count+1)}}>click me</button>
        </div>
    )
}
export default Example;
```

从这两个程序的对比上可以看出Hooks本质上就是一类特殊的函数，他们可以为你的函数型组件（function component）注入一些特殊的功能。这听起来有点像以前React中的`Mixins`差不多哦。其实是由很多不同，hooks的目的就是让你不再写`class`，让`function`一统江湖。

#### useState 的介绍和多状态声明

##### useState的介绍

> `useState`是react自带的一个hook函数，它的作用是用来声明状态变量。

那我们从三个方面来看`useState`的用法，分别是声明、读取、使用（修改）。这三个方面掌握了，你基本也就会使用`useState`了.

先来看一下声明的方式

```js
const [ count , setCount ] = useState(0);
```

这种方法是ES6语法中的数组解构，这样看起来代码变的简单易懂。

 如果不写成数组解构，上边的语法要写成下面的三行:

```js
let _useState = userState(0)
let count = _useState[0]
let setCount = _useState[1]
```

`useState`这个函数接收的参数是状态的初始值(Initial state)，它返回一个数组，这个数组的第0位是当前的状态值，第1位是可以改变状态值的方法函数。 所以上面的代码的意思就是声明了一个状态变量为count，并把它的初始值设为0，同时提供了一个可以改变`count`的状态值的方法函数。

这时候你已经会声明一个状态了，接下来我们看看如何读取状态中的值。

```js
<p>You clicked {count} times</p>
```

你可以发现，我们读取是很简单的。只要使用`{count}`就可以，因为这时候的count就是JS里的一个变量，想在`JSX`中使用，值用加上`{}`就可以。

最后看看如果改变`State`中的值,看下面的代码:

```jsx
<button onClick={()=>{setCount(count+1)}}>click me</button>
<button onClick={()=>{setCount((count)=>count+1)}}>click me</button> 
//setCount((c)=>c+1) 里面传入一个函数，函数的第一个参数是最新的值
```

直接调用setCount函数，这个函数接收的参数是修改过的新状态值。接下来的事情就交给`React`,他会重新渲染组件。`React`自动帮助我们记忆了组件的上一次状态值，但是这种记忆也给我们带来了一点小麻烦，但是这种麻烦你可以看成规则，只要准守规则，就可以愉快的进行编码。

##### 多状态声明的注意事项

比如现在我们要声明多个状态，有年龄（age）、性别(sex)和工作(work)。代码可以这么写.

```js
import React ,{useState} from 'react';


function Example(){
    const [age,setAge] = useState(22);
    const [sex,setSex] = useState('男');
    const [work,setWork] = useState('前端开发');

    return(
        <div>
            <p>年龄：{age}</p>
            <p>性别：{sex}</p>
            <p>工作：{work}</p>
        </div>
    )
}


export default Example
```

在使用`useState`的时候只赋了初始值，并没有绑定任何的`key`,那React是怎么保证这三个useState找到它自己对应的state呢？

**答案是：React是根据useState出现的顺序来确定的**

比如我们把代码改成下面的样子：

```js
import React, { useState } from 'react';

let showSex = true
function Example(){
    const [ age , setAge ] = useState(22)
    if(showSex){
        const [ sex , setSex ] = useState('男')
        showSex=false
    }
   
    const [ work , setWork ] = useState('前端开发')
    return (
        <div>
             <p>年龄：{age}</p>
            <p>性别：{sex}</p>
            <p>工作：{work}</p>
            
        </div>
    )
}
export default Example;
```

这时候控制台就会直接给我们报错，错误如下：

```text
 React Hook "useState" is called conditionally. React Hooks must be called in the exact same order in every component render 
```

意思就是useState不能在`if...else...`这样的条件语句中进行调用，必须要按照相同的顺序进行渲染。如果你还是不理解，你可以记住这样一句话就可以了：**就是React Hooks不能出现在条件判断语句中，因为它必须有完全一样的渲染顺序**。

### useState的闭包陷阱

示例代码：

```jsx
import React, { useState, useEffect } from 'react'


function MyCountFunc() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    const time = setInterval(() => {
      setCount(count + 1) //=>关键代码，这样设置是不可以的。因为每次进来都会重置count，所以页面只会显示1
    }, 1000)
    return () => clearInterval(time)
  },[])

  return (
    <>
      <h1>{count}</h1>   //在页面渲染，只有1
    </>
  )
}


export default MyCountFunc

```

修改后的代码

```jsx
import React, { useState, useEffect } from 'react'


function MyCountFunc() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    const time = setInterval(() => {
      setCount((c) => c + 1) //里面传入一个函数，函数第一个参数是上一次的值，只要得到上一次的值，那么页面就会实现每秒+1的效果
    }, 1000)
    return () => clearInterval(time)
  }, [])

  return (
    <>
      <h1>{count}</h1>
    </>
  )
}


export default MyCountFunc

```



#### useEffect代替常用生命周期函数

在用`Class`制作组件时，经常会用生命周期函数，来处理一些额外的事情（副作用：和函数业务主逻辑关联不大，特定时间或事件中执行的动作，比如Ajax请求后端数据，添加登录监听和取消登录，手动修改`DOM`等等）。在`React Hooks`中也需要这样类似的生命周期函数，比如在每次状态（State）更新时执行，它为我们准备了`useEffect`。

##### 用`Class`的方式为计数器增加生命周期函数

用原始的方式把计数器的Demo增加两个生命周期函数`componentDidMount`和`componentDidUpdate`。分别在组件第一次渲染后在浏览器控制台打印出计数器结果和在每次计数器状态发生变化后打印出结果。代码如下：

```js
import React, { Component } from 'react';

class Example3 extends Component {
    constructor(props) {
        super(props);
        this.state = { count:0 }
    }
    
    
    componentDidMount(){
        console.log(`ComponentDidMount=>You clicked ${this.state.count} times`)
    }
    componentDidUpdate(){
        console.log(`componentDidUpdate=>You clicked ${this.state.count} times`)
    }

    render() { 
        return (
            <div>
                <p>You clicked {this.state.count} times</p>
                <button onClick={this.addCount.bind(this)}>Chlick me</button>
            </div>
        );
    }
    addCount(){
        this.setState({count:this.state.count+1})
    }
}
 
export default Example3;
```

这就是在不使用Hooks情况下的写法，那如何用Hooks来代替这段代码，并产生一样的效果那。

##### 用`useEffect`函数来代替生命周期函数

在使用`React Hooks`的情况下，我们可以使用下面的代码来完成上边代码的生命周期效果，代码如下： 记得要先引入`useEffect`后，才可以正常使用。

```js
import React, { useState , useEffect } from 'react';
function Example(){
    const [ count , setCount ] = useState(0);
    //---关键代码---------start-------
    useEffect(()=>{
        console.log(`useEffect=>You clicked ${count} times`)
    })
    //---关键代码---------end-------

    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={()=>{setCount(count+1)}}>click me</button>
        </div>
    )
}
export default Example;
```

写完后，可以到浏览器中进行预览一下，可以看出跟`class`形式的生命周期函数是完全一样的，这代表第一次组件渲染和每次组件更新都会执行这个函数。 那这段代码逻辑是什么？我们梳理一下:首先，我们生命了一个状态变量`count`,将它的初始值设为0，然后我们告诉react，我们的这个组件有一个副作用。给`useEffecthook`传了一个匿名函数，这个匿名函数就是我们的副作用。在这里我们打印了一句话，当然你也可以手动的去修改一个`DOM`元素。当React要渲染组件时，它会记住用到的副作用，然后执行一次。等Reat更新了State状态时，它再一词执行定义的副作用函数。

##### useEffect两个注意点

1. React首次渲染和之后的每次渲染都会调用一遍`useEffect`函数，而之前我们要用两个生命周期函数分别表示首次渲染(componentDidMonut)和更新导致的重新渲染(componentDidUpdate)。
2. useEffect中定义的函数的执行不会阻碍浏览器更新视图，也就是说这些函数时异步执行的，而`componentDidMonut`和`componentDidUpdate`中的代码都是同步执行的。个人认为这个有好处也有坏处吧，比如我们要根据页面的大小，然后绘制当前弹出窗口的大小，如果时异步的就不好操作了。

#### useEffect 实现 componentWillUnmount生命周期函数

在写React应用的时候，在组件中经常用到`componentWillUnmount`生命周期函数（组件将要被卸载时执行）。比如我们的定时器要清空，避免发生内存泄漏;比如登录状态要取消掉，避免下次进入信息出错。所以这个生命周期函数也是必不可少的

#### useEffect解绑副作用

学习`React Hooks` 时，我们要改掉生命周期函数的概念（人往往有先入为主的毛病，所以很难改掉），因为`Hooks`叫它副作用，所以`componentWillUnmount`也可以理解成解绑副作用。

打开`Example.js`文件，进行改写代码，先引入对应的`React-Router`组件。

```js
import { BrowserRouter as Router, Route, Link } from "react-router-dom"
```

在文件中编写两个新组件，因为这两个组件都非常的简单，所以就不单独建立一个新的文件来写了。

```js
function Index() {
    return <h2>JSPang.com</h2>;
}
  
function List() {
    return <h2>List-Page</h2>;
}
```

有了这两个组件后，接下来可以编写路由配置，在以前的计数器代码中直接增加就可以了。

```js
return (
    <div>
        <p>You clicked {count} times</p>
        <button onClick={()=>{setCount(count+1)}}>click me</button>

        <Router>
            <ul>
                <li> <Link to="/">首页</Link> </li>
                <li><Link to="/list/">列表</Link> </li>
            </ul>
            <Route path="/" exact component={Index} />
            <Route path="/list/" component={List} />
        </Router>
    </div>
)
```

然后到浏览器中查看一下，看看组件和路由是否可用。如果可用，我们现在可以调整`useEffect`了。在两个新组件中分别加入`useEffect()`函数:

```js
function Index() {
    useEffect(()=>{
        console.log('useEffect=>老弟，你来了！Index页面')
        )
    return <h2>JSPang.com</h2>;
}
  
function List() {
    useEffect(()=>{
        console.log('useEffect=>老弟，你来了！List页面')
    })

    return <h2>List-Page</h2>;
}
```

这时候我们点击`Link`进入任何一个组件，在浏览器中都会打印出对应的一段话。这时候可以用**返回一个函数的形式进行解绑**，代码如下：

```js
function Index() {
    useEffect(()=>{
        console.log('useEffect=>老弟你来了！Index页面')
        return ()=>{
            console.log('老弟，你走了!Index页面')
        }
    })
    return <h2>JSPang.com</h2>;
  }
```

这时候你在浏览器中预览，我们仿佛实现了`componentWillUnmount`方法。但这只是好像实现了，当点击计数器按钮时，你会发现`老弟，你走了!Index页面`，也出现了。这到底是怎么回事那？其实每次状态发生变化，`useEffect`都进行了解绑。

#### useEffect的第二个参数

那到底要如何实现类似`componentWillUnmount`的效果那?这就需要请出`useEffect`的第二个参数，它是一个数组，数组中可以写入很多状态对应的变量，意思是当状态值发生变化时，我们才进行解绑。但是当传空数组`[]`时，就是当组件将被销毁时才进行解绑，这也就实现了`componentWillUnmount`的生命周期函数。

```js
function Index() {
    useEffect(()=>{
        console.log('useEffect=>老弟你来了！Index页面')
        return ()=>{
            console.log('老弟，你走了!Index页面')
        }
    },[])
    return <h2>JSPang.com</h2>;
}
```

为了更加深入了解第二个参数的作用，把计数器的代码也加上`useEffect`和解绑方法，并加入第二个参数为空数组。代码如下：

```js
function Example(){
    const [ count , setCount ] = useState(0);

    useEffect(()=>{
        console.log(`useEffect=>You clicked ${count} times`)

        return ()=>{
            console.log('====================')
        }
    },[])

    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={()=>{setCount(count+1)}}>click me</button>

            <Router>
                <ul>
                    <li> <Link to="/">首页</Link> </li>
                    <li><Link to="/list/">列表</Link> </li>
                </ul>
                <Route path="/" exact component={Index} />
                <Route path="/list/" component={List} />
            </Router>
        </div>
    )
}
```

这时候的代码是不能执行解绑副作用函数的。但是如果我们想每次`count`发生变化，我们都进行解绑，只需要在第二个参数的数组里加入`count`变量就可以了。代码如下：

```js
function Example(){
    const [ count , setCount ] = useState(0);

    useEffect(()=>{
        console.log(`useEffect=>You clicked ${count} times`)

        return ()=>{
            console.log('====================')
        }
    },[count])

    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={()=>{setCount(count+1)}}>click me</button>

            <Router>
                <ul>
                    <li> <Link to="/">首页</Link> </li>
                    <li><Link to="/list/">列表</Link> </li>
                </ul>
                <Route path="/" exact component={Index} />
                <Route path="/list/" component={List} />
            </Router>
        </div>
    )
}
```

这时候只要`count`状态发生变化，都会执行解绑副作用函数，浏览器的控制台也就打印出了一串`=================`。



### useLayoutEffect

useLayoutEffect和useEffect差不多，只不过有一个区别，就是useLayoutEffect先执行，useEffect后执行

● useLayoutEffect是在DOM挂载之前之前

● useEffect是DOM挂载之后执行

它们两的区别就好像是componentWillMount和componentDidMount的区别

#### useContext 让父子组件传值更简单

有了`useState`和`useEffect`已经可以实现大部分的业务逻辑了，但是`React Hooks`中还是有很多好用的`Hooks`函数的，比如`useContext`和`useReducer`。

在用类声明组件时，父子组件的传值是通过组件属性和`props`进行的，那现在使用方法(Function)来声明组件，已经没有了`constructor`构造函数也就没有了props的接收，那父子组件的传值就成了一个问题。`React Hooks` 为我们准备了`useContext`。这节课就学习一下`useContext`，它可以帮助我们跨越组件层级直接传递变量，实现共享。需要注意的是`useContext`和`redux`的作用是不同的，一个解决的是组件之间值传递的问题，一个是应用中统一管理状态的问题，但通过和`useReducer`的配合使用，可以实现类似`Redux`的作用。

`Context`的作用就是对它所包含的组件树提供全局共享数据的一种技术。

#### createContext 函数创建context

引入`createContext`函数，并使用得到一个组件，然后在`return`方法中进行使用。

```js
import React, { useState , createContext } from 'react';
//===关键代码
const CountContext = createContext() //创建共享的组件

function Example4(){
    const [ count , setCount ] = useState(0);

    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={()=>{setCount(count+1)}}>click me</button> 
            {/*======关键代码 */}
            <CountContext.Provider value={count}> //value就是要共享的属性
            </CountContext.Provider>

        </div>
    )
}
export default Example4;
```

这段代码就相当于把`count`变量允许跨层级实现传递和使用了（也就是实现了上下文），当父组件的`count`变量发生变化时，子组件也会发生变化。接下来我们就看看一个`React Hooks`的组件如何接收到这个变量。

#### useContext 接收上下文变量

已经有了上下文变量，剩下的就时如何接收了，接收这个直接使用useContext就可以，但是在使用前需要新进行引入`useContext`（不引入是没办法使用的）。

```js
import React, { useState , createContext , useContext } from 'react';
```

引入后写一个`Counter`组件，只是显示上下文中的`count`变量代码如下：

```js
function Counter(){
    const count = useContext(CountContext)  //一句话就可以得到count
    return (<h2>{count}</h2>)
}
```

得到后就可以显示出来了，但是要记得在`<CountContext.Provider>`的闭合标签中,代码如下。

```js
<CountContext.Provider value={count}>
    <Counter />
</CountContext.Provider>
```

#### useReducer介绍和简单使用

##### reducer到底是什么？

为了更好的理解`useReducer`，所以先要了解JavaScript里的`Redcuer`是什么。它的兴起是从`Redux`广泛使用开始的，但不仅仅存在`Redux`中，可以使用JavaScript来完成`Reducer`操作。那`reducer`其实就是一个函数，这个函数接收两个参数，一个是状态，一个用来控制业务逻辑的判断参数。我们举一个最简单的例子。

```js
function countReducer(state, action) {
    switch(action.type) {
        case 'add':
            return state + 1;
        case 'sub':
            return state - 1;
        default: 
            return state;
    }
}
```

上面的代码就是Reducer，你主要理解的就是这种形式和两个参数的作用，一个参数是状态，一个参数是如何控制状态。

##### useReducer的使用

了解reducer的含义后，就可以讲useReducer了，它也是React hooks提供的函数，可以增强我们的`Reducer`，实现类似Redux的功能

```jsx
import React, { useReducer } from 'react';


function ReduceDemo()          //useReducer要求传入两个参数，第一个是reducer函数，第二个是初始值
    const [count, dispatch] = useReducer((state, action) => { //返回的第一个参数是初始值，第二个是dispatch
        switch (action) {
            case 'add':
                return state + 1;
            case 'sub':
                return state - 1;
            default:
                return state
        }
    }, 0) //第一个参数是reducer函数，第二个参数是初始值 
    return (
        <div>
            <h2>{count}</h2>
            <button onClick={() => { dispatch('add') }}>+</button>
            <button onClick={() => { dispatch('sub') }}>-</button>
             //点击之后传递一个action
         
        </div>
    )
}


export default ReduceDemo
```

### useReducer代替Redux

使用`useContext`和`useReducer`是可以实现类似`Redux`的效果，并且一些简单的个人项目，完全可以用下面的方案代替Redux，这种做法要比Redux简单一些。

##### 理论上的可行性

我们先从理论层面看看替代`Redux`的可能性，其实如果你对两个函数有所了解，只要我们巧妙的结合，这种替代方案是完全可行的。

`useContext`：可访问全局状态，避免一层层的传递状态。这符合`Redux`其中的一项规则，就是状态全局化，并能统一管理。

`useReducer`：通过action的传递，更新复杂逻辑的状态，主要是可以实现类似`Redux`中的`Reducer`部分，实现业务逻辑的可行性。

##### UI部分:

新建一个文件夹：Reudx，在Redux里面新建：index.js，View.js,Buttons.js

View代码：

```jsx
import React from 'react';
function View(){
    return (
        <div style={{color:'blue'}}>
            字体颜色为blue
        </div>
    
    )
}


export default View;
```

Buttons代码：

```jsx
import React from 'react';

function Buttons() {
    return (
        <div>
            <button>红色</button>
            <button>黄色</button>
        </div>
    )
}


export default Buttons;
```

index.js代码：

```jsx
import React from 'react';
import View from './View';
import Buttons from './Buttons';

function Redux() {
    return (
        <>
            <View />
            <Buttons />
        </>
    )
}

export default Redux;
```

##### UI界面可以写好了，接下来就是用createContext和useContext共享数据

首先在Redux文件夹里面新建一个color.js文件，在里面使用createContext创建共享数据并且暴露出来

color.js：

```jsx
import React, { createContext } from 'react';

export const ColorContext = createContext({}) //创建共享组件，并且暴露出去


export const Color = props => {    //创建包裹组件Color并且暴露出去 
    //因为暴露出去的value是一个对象，所以接收的时候也是以对象形式接收
    return (
        <ColorContext.Provider value={{ color: 'red' }}>  
            {props.children}  //   {props.children}     包裹住组件的内容
        </ColorContext.Provider>
    )
}
//因为这里使用了单个暴露语法，所以外面接收的时候要用{ColorContext,Color} 这样接收

```

如果要共享数据，那么就要先把要拿到数据的组件先包裹一下，这样才能在里面获取到数据

在最外围的Index.js文件中进行包裹

index.js:

```jsx
import React from 'react';
import View from './View';
import Buttons from './Buttons';
import { Color } from './color'; //引入Color包裹组件


function Redux() {
    return (
        <>
            <Color>  //进行包裹
                <View />
                <Buttons />
            </Color>
        </>
    )
}

export default Redux;
```

颜色是在View组件中呈现了。那么现在就需要修改View组件中的内容

View.js

```jsx
import React, { useContext } from 'react';
import { ColorContext } from './color';  //引入color.js文件中暴露出来的ColorContext
function View() {
    //因为暴露出去的value是一个对象，所以接收的时候也是以对象形式接收
    const { color } = useContext(ColorContext)
    return (
        <div style={{ color: color }}>  //拿到color，替换掉原来写死的数据
            字体颜色为{color}
        </div>

    )
}


export default View;
```

这样就可以共享数据了，修改了color.js里面的数据，View中的数据也会发生变化。页面也就会改变

##### 接下来就是要配合useReducer实现小型的Redux了

要使用useReducer，那么就要先创建一个reducer函数，为了方便，reducer函数都写在color.js文件中

color.js:

```jsx
import React, { createContext, useReducer } from 'react'; //引入useReducer

export const ColorContext = createContext({}) //创建传递的组件，并且暴露出去

export const UPDATE_COLOR = "UPDATE_COLOR"; //定义一个常量，判断是否改变color


const reducer = (state, action) => {  //创建reducer函数
    switch (action.type) {  //如果传递过来的action.type == UPDATE_COLOR，返回传递过来的action.color
        case UPDATE_COLOR:    
            return action.color
        default:
            return state
    }
}


export const Color = props => {   
    const [color, dispatch] = useReducer(reducer, 'blue') //使用useReducer，第一个参数是reducer函数，第二个初始值
    return (
     
        <ColorContext.Provider value={{ color, dispatch }}>   //把color和dispatch共享出去
            {props.children}
        </ColorContext.Provider>
    )
}


```

在Buttons.js文件中调用dispatch，就可以修改reducer中的数据，那么页面就能随之修改

Buttons.js:

```jsx
import React, { useContext } from 'react';
import { ColorContext, UPDATE_COLOR } from './color'; //引入共享状态需要传入的ColorContext

function Buttons() {
    const { dispatch } = useContext(ColorContext) //得到共享的diaoatch
    return (
        <div>
            <button onClick={() => { dispatch({ type: UPDATE_COLOR, color: "red" }) }}>红色</button>
            <button onClick={() => { dispatch({ type: UPDATE_COLOR, color: "yellow" }) }}>黄色</button>
            {/* 点击的时候 调用dispatch，里面是一个对象，type为常量，color为要传递的颜色*/}
        </div>
    )
}


export default Buttons;
```

数据更改，页面随之更新，小型Redux这样就完成了

### useMemo优化React Hooks程序性能

`useMemo`主要用来解决使用React hooks产生的无用渲染的性能问题。使用function的形式来声明组件，失去了`shouldCompnentUpdate`（在组件更新之前）这个生命周期，也就是说我们没有办法通过组件更新前条件来决定组件是否更新。而且在函数组件中，也不再区分`mount`和`update`两个状态，这意味着函数组件的每一次调用都会执行内部的所有逻辑，就带来了非常大的性能损耗。`useMemo`和`useCallback`都是解决上述性能问题的

#### 性能问题展示案例

建立两个组件,一个父组件一个子组件，组件上由两个按钮，一个是小红，一个是小燕，点击哪个，那个就变成忙碌状态

父组件：

```jsx
import React, { useState } from 'react';
import ChildComponent from './ChildComponent';

function Updata() {
    const [xiaohong, setXiaohong] = useState('小红在等待')
    const [xiaoyan, setXiaoyan] = useState('小燕在等待')
    return (
        <>
            <button onClick={() => { setXiaohong(new Date().getTime()) }}>小红</button>
            <button onClick={() => { setXiaoyan(new Date().getTime() + ',小燕在忙碌') }}>小燕</button>
            <ChildComponent name={xiaohong}>{xiaoyan}</ChildComponent> //传递参数给子组件
        //xiaoyan这个参数可以在子组件中用children接收
        </>
    )
}


export default Updata;
```

子组件：

```jsx
import React, { useMemo } from 'react';

function ChildComponent({ name, children }) {
    function changeXiaohong(name) {
        console.log('小红被更新了.....')
        return name + ',忙碌状态'
    }
    const actionXiaohong = changeXiaohong(name)
    return (
        <>
            <div>{actionXiaohong}</div>
            <div>{children}</div>
        </>
    )
}


export default ChildComponent;
```

写完之后程序可以正常运行，但是有一点是比较重要的，就是不管点小红按钮还是小燕按钮，changeXiaohong()这个函数都会被执行并且更新页面，这样就会有很大的性能损耗问题，我们想要的是点击小燕按钮的时候，这个函数不执行，点击小红按钮的时候这个函数才执行

#### useMemo 优化性能

其实只要使用`useMemo`，然后给它传递第二个参数，参数匹配成功，才会执行。代码如下：

```jsx
import React, { useMemo } from 'react';

function ChildComponent({ name, children }) {
    function changeXiaohong(name) {
        console.log('小红被更新了.....')
        return name + ',忙碌状态'
    }
    const actionXiaohong = useMemo(() => changeXiaohong(name), [name]) //第一个参数是一个函数，把需要优化的函数传递进去作为第一个参数，第二个参数就是(是否要执行的信号)，当name发生变化的时候就执行传递进来的函数，如果name没有发生变化，那么就不会执行这个函数
    //下面这种写法也是可以的
    //const actionXiaohong = useMemo(() => { return changeXiaohong(name) }, [name]) 
    return (
        <>
            <div>{actionXiaohong}</div>
            <div>{children}</div>
        </>
    )
}


export default ChildComponent;
```

### useRef获取DOM元素和保存变量

`useRef`在工作中虽然用的不多，但是也不能缺少。它有两个主要的作用:

- 用`useRef`获取React JSX中的DOM元素，获取后你就可以控制DOM的任何东西了。但是一般不建议这样来作，React界面的变化可以通过状态来控制。
- 用`useRef`来保存变量，这个在工作中也很少能用到，我们有了`useContext`这样的保存其实意义不大，但是这是学习，也要把这个特性讲一下。

##### useRef获取DOM元素

界面上有一个文本框，在文本框的旁边有一个按钮，当我们点击按钮时，在控制台打印出`input`的DOM元素，并进行复制到DOM中的value上。这一切都是通过`useRef`来实现

```jsx
import React, { useRef} from 'react';

function Ref() {
   
    const onButtonClick = () => {
        inputEl.current.value = "小红"
        console.log(inputEl)
    }
  
    return (
        <>
            <input ref={inputEl} type="text" />
            <button onClick={onButtonClick}>按我</button>
        </>
    )

}


export default Ref;
```

##### useRef保存普通变量

这个操作在实际开发中用的并不多，但我们还是要讲解一下。就是`useRef`可以保存React中的变量。我们这里就写一个文本框，文本框用来改变`text`状态。又用`useRef`把`text`状态进行保存，最后打印在控制台上。写这段代码你会觉的很绕，其实显示开发中没必要这样写，用一个state状态就可以搞定，这里只是为了展示知识点。

接着上面的代码来写，就没必要重新写一个文件了。先用`useState`声明了一个`text`状态和`setText`函数。然后编写界面，界面就是一个文本框。然后输入的时候不断变化。

```jsx
import React, { useRef, useState, useEffect } from 'react';

function Ref() {
    const inputEl = useRef(null)
    const onButtonClick = () => {
        inputEl.current.value = "小红"
        console.log(inputEl)
    }
    const [text, setText] = useState('小红');
    const textRef = useRef()
    useEffect(() => {
        textRef.current = text
        console.log('textRef.current', textRef.current)
    })
    return (
        <>
            <input ref={inputEl} type="text" />
            <button onClick={onButtonClick}>按我</button>
            <br />
            <br />
            <input value={text} onChange={(e) => { setText(e.target.value) }} />
        </>
    )

}


export default Ref;
```

### 自定义Hook函数

在实际开发中，为了界面更加美观。获取浏览器窗口的尺寸是一个经常使用的功能，这样经常使用的功能，就可以封装成一个自定义`Hooks`函数，记住一定要用use开头，这样才能区分出什么是组件，什么是自定义函数

新建一个文件`Example.js`,然后编写一个useWinSize,编写时我们会用到`useState`、`useEffect`和`useCallback`所以先用`import`进行引入。

```js
import React, { useState ,useEffect ,useCallback } from 'react';
```

然后编写函数，函数中先用useState设置`size`状态，然后编写一个每次修改状态的方法`onResize`，这个方法使用`useCallback`，目的是为了缓存方法(useMemo是为了缓存变量)。 然后在第一次进入方法时用`useEffect`来注册`resize`监听时间。为了防止一直监听所以在方法移除时，使用return的方式移除监听。最后返回size变量就可以了。

```js
function useWinSize(){
    const [ size , setSize] = useState({
        width:document.documentElement.clientWidth,
        height:document.documentElement.clientHeight
    })

    const onResize = useCallback(()=>{
        setSize({
            width: document.documentElement.clientWidth,
            height: document.documentElement.clientHeight
        })
    },[]) 
    useEffect(()=>{
        window.addEventListener('resize',onResize)
        return ()=>{
            window.removeEventListener('resize',onResize)
        }
    },[])

    return size;

}
```

这就是一个自定义函数，其实和我们以前写的JS函数没什么区别，所以这里也不做太多的介绍。

##### 编写组件并使用自定义函数

自定义`Hooks`函数已经写好了，可以直接进行使用，用法和`JavaScript`的普通函数用起来是一样的。直接在`Example组件使用`useWinSize`并把结果实时展示在页面上。

```js
function Example(){

    const size = useWinSize()
    return (
        <div>页面Size:{size.width}x{size.height}</div>
    )
}

export default Example
```

之后就可以在浏览器中预览一下结果，可以看到当我们放大缩小浏览器窗口时，页面上的结果都会跟着进行变化。说明自定义的函数起到了作用。