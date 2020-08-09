# 自定义redux库

### redux语法功能分析

1. redux 库向外暴露下面几个函数
   - createStore(): 接收的参数为 reducer 函数, 返回为 store 对象
   - combineReducers(): 接收包含 n 个 reducer 方法的对象, 返回一个新的 reducer 函数
2. store 对象的内部结构
   - getState(): 返回值为内部保存的 state 数据
   - dispatch(): 参数为 action 对象
   - subscribe(): 参数为监听内部 state 更新的回调函数



### 实现createStore()

> 实现createStore()的功能

````js
/**
 * redux库主模块
 * 1. redux 库向外暴露下面几个函数
   - createStore(): 接收的参数为 reducer 函数, 返回为 store 对象
   - combineReducers(): 接收包含 n 个 reducer 方法的对象, 返回一个新的 reducer 函数
   2. store 对象的内部结构
   - getState(): 返回值为内部保存的 state 数据
   - dispatch(): 参数为 action 对象
   - subscribe(): 参数为监听内部 state 更新的回调函数
 */

/**
 * 
 * 根据指定的reducer函数创建一个store对象
 */
export function createStore(reducer) { //接收一个reducer函数

  //用来存储内部状态数据的变量,初始值为调用reducer函数返回的结果(外部指定的默认值)
  let state = reducer(undefined, { type: '@@redux/init' })
  //用来存储监听state更新回调函数的数组容器
  const listeners = []
  
  /**
   * 返回当前内部的state数据
   */
  function getState() {
    return state
  }
  /**
   * 分发action
   * 1.触发reducer调用，得到新的state
   * 2.保存新的state
   * 3.调用所有已存在的监视回调
   * 
   */
  function dispatch(action) {
    // 1.触发reducer调用，得到新的state
    const newState = reducer(state, action)
    // 2.保存新的state
    state = newState
    
    // 3.调用所有已存在的监视回调
    
    listeners.forEach(listener => listener())
  }
  /** 
   * 绑定内部state改变的监听回调
   * 可以给一个stort绑定多个监听
  */
  function subscribe(listener) {
    //保存到缓存Listener的容器数组中
    listeners.push(listener)
    console.log(listeners)
    console.log('触发subscribe')
  }
  //返回store对象
  return {
    getState,
    dispatch,
    subscribe
  }
}
/**
 * 
 * 整合传入参数对象中的多个reducer函数,返回一个新的reducer
 * 新的reducer管理的总状态:{r1:state1,r2:state2}
 */
export function combineReducers(reducers) {
  return (state, action) => {

  }
}
````

### 实现combineReducers

````js
/**
 * 
 * 整合传入参数对象中的多个reducer函数,返回一个新的reducer
 * 新的reducer管理的总状态:{r1:state1,r2:state2}
 * reuscers的结构：
 * {
 *  count: (state=2,action) => 3
 *  user: (state = {}, action) => {}
 * }
 * 要得到的总状态的结构
 * {
 *  count:count(state.count,action),
 *  user:user(state.user,action)
 * }
 */
// const obj = {
//   count: (state = 2, action) => 3,
//   user: (state = {}, action) => { }
// }

/**
 * forEach版本
 */

export function combineReducers(reducers) {
  //返回一个新的总reducer函数
  //state:总状态
  return (state = {}, action) => {
    //准备一个总状态空对象
    const totalState = {}
    //执行reducers中每个reducer函数得到一个新的子状态，并添加到总状态空对象
    Object.keys(reducers).forEach(key => {
      totalState[key] = reducers[key](state[key], action)
    })
    return totalState
  }
}
/**
 * reduce版本
 */
// export function combineReducers(reducers) {
//   //返回一个新的总reducer函数
//   //state:总状态
//   return (state = {}, action) => {
//     //执行reducers中每个reducer函数得到一个新的子状态，并封装一个对象容器
//     const newState = Object.keys(reducers).reduce((preState, key) => {
//       preState[key] = reducers[key](state[key], action)
//       return preState
//     }, {})
//     return newState
//   }
// }

//简洁版本
export function combineReducers(reducers) {
  return (state = {}, action) => {
    return Object.keys(reducers).reduce((pre, key) => {
      pre[key] = reducers[key](state[key], action)
      return pre
    }, {})
  }
}

````

## 自定义redux-react

##### 1.react-redux 向外暴露了 2 个 API 

- Provider 组件类 

- connect 函数 

##### 2.Provider 组件 

- 接收 store 属性 

- 让所有容器组件都可以看到 store, 从而通过 store 读取/更新状态 

##### 3.connect 函数 

> 接收 2 个参数: mapStateToProps 和 mapDispatchToProps 
>
> mapStateToProps: 为一个函数, 用来指定向 UI 组件传递哪些一般属性 
>
> mapDispatchToProps: 为一个函数或对象, 用来指定向 UI 组件传递哪些函数属性 
>
> connect()执行的返回值为一个高阶组件: 包装 UI 组件, 返回一个新的容器组件 
>
> 容器组件会向 UI 传入前面指定的一般/函数类型属性

#### 代码

````js
/**
 * react-redux库的主模块
 * 1) react-redux 向外暴露了 2 个 API a. Provider 组件类 b. connect 函数 
 * 2) Provider 组件 
 * 接收 store 属性 
 * 让所有容器组件都可以看到 store, 从而通过 store 读取/更新状态 
 * 3) connect 函数 
 * 接收 2 个参数: mapStateToProps 和 mapDispatchToProps 
 * mapStateToProps: 为一个函数, 用来指定向 UI 组件传递哪些一般属性 
 * mapDispatchToProps: 为一个函数或对象, 用来指定向 UI 组件传递哪些函数属性 
 * connect()执行的返回值为一个高阶组件: 包装 UI 组件, 返回一个新的容器组件 
 * 容器组件会向 UI 传入前面指定的一般/函数类型属性
 */
import React from 'react'
import PropTypes from 'prop-types'
/**
 * 用来向所有容器组件提供store的组件类
 * 通过context向所有的容器组件提供store
 */
export class Provider extends React.Component {

  static propTypes = {
    store: PropTypes.object.isRequired //声明接收store
  }
  /**
   * 声明提供context的数据名称和类型
   */
  static childContextTypes = {
    store: PropTypes.object
  }
  /**
   * 向所有有声明子组件提供包含要传递数据的context对象
   */
  getChildContext() {
    return { store: this.props.store }
  }
  render() {
    //返回渲染<Provider> 的所有子节点
    return this.props.children
  }
}

/**
 * connect高阶函数：接收 mapStateToProps 和 mapDispatchToProps ,返回一个高阶组件函数
 * 高阶组件:接收一个UI组件，返回一个容器组件
 */

export function connect(mapStateToProps, mapDispatchToProps) {
  //返回高阶组件函数
  return (UIComponent) => {
    //返回容器组件
    return class ContainerComponent extends React.Component {

      /**
       * 声明接收的context数据的类型
       */
      static contextTypes = {
        store: PropTypes.object
      }
      constructor(props, context) {
        super(props)
        console.log('ContainerComponent', context.store)
        //得到store
        const { store } = context
        //得到包含所有一般属性的对象
        const stateProps = mapStateToProps(store.getState()) //并且把state中的数据传递过去

        //将所有一般属性作为容器组件的状态数据
        this.state = { ...stateProps }

        //得到包含所有函数属性的对象
        let dispatchProps
        //判断传递过来的是否是函数
        if (typeof mapDispatchToProps === 'function') {
          //函数的情况
          dispatchProps = mapDispatchToProps(store.dispatch)//并且把dispatch传递过去
        } else { //对象的情况
          dispatchProps = Object.keys(mapDispatchToProps).reduce((pre, key) => {
            const actionCreator = mapDispatchToProps[key]
            pre[key] = (...args) => store.dispatch(actionCreator(...args)) //参数透传
            return pre
          }, {})
        }

        //保存到组件上
        this.dispatchProps = dispatchProps

        //绑定store的state变化的监听
        store.subscribe(() => {  // store内部的状态数据发生了变化
          console.log('11')
          //更新容器组件 ==>  UI更新
          this.setState(mapStateToProps(store.getState()))
        })
      }

      render() {
        //返回UI组件的标签
        return <UIComponent {...this.state} {...this.dispatchProps} />
      }
    }
  }
}
````

使用：

和react-redux使用方法一样

### 重制版

redux

````js
/**
 * createStore：创建redux容器的
 * @params 
 *    reducer:函数
 * @return
 *    store:{
 *      getState,
 *      dispatch,
 *      subscribe
 *  }
 */

function createStore(reducer) {
  // 创建一个store,state用来存储管理的状态信息，listenAry用来存储事件池中的方法
  // state不用设置初始值，因为第一次dispatch指向reducer，state没有值，走的是reducer中赋值的默认值信息，我们会在创建容器的时候就把dispatch执行一次
  let state,
    listenAry = [];

  function getState() {
    // 1.我们需要保证返回的状态信息不能和容器的state是同一个堆内存(否则外面获取状态信息后，直接可以修改容器中的状态了，这不符合dispatch -> reducer才能修改的规范)
    // 深克隆一份state返回
    return JSON.parse(JSON.stringify(state));
  }
  // 基于dispatch实现任务派发
  function dispatch(action) {
    // 执行reducer,修改容器中的状态信息(接收reducer的返回值，把返回的信息替换原有的state)，值得注意的是：我们是把返回值全部替换state，所以要求reducer中在修改状态之前，要先把原始的状态信息克隆一份，再进行单个的属性修改
    state = reducer(state, action);
    // 容器状态信息经过reducer修改后，通知事件池中的方法依次执行
    for (let i = 0; i < listenAry.length; i++) {
      let item = listenAry[i];
      if (typeof item === 'function') {
        item()
      } else {
        listenAry.splice(i, 1);
        i--;
      }
    }
  }
  dispatch({ type: '@@redux/init' }); //=>创建容器的时候执行一次dispatch，目的是把reducer中的默认状态信息赋值给redux容器中的状态

  // 向事件池中追加方法
  function subscribe(fn) {
    // 向容器中追加方法(重复验证)
    let isExit = listenAry.includes(fn);
    if (!isExit) {
      listenAry.push(fn)
    }
    // 2.返回一个方法：执行放回的方法会把当前绑定的方法在事件池中移除掉
    return function unsubscribe() {
      let index = listenAry.indexOf(fn);
      listenAry[index] = null; //可能会引发数组塌陷

    }
  }
  return {
    getState,
    dispatch,
    subscribe

  }

}

/**
 * combineReducers:reducer合并的方法
 * @param reducers 
 *    对象，对象中包含了每一个板块对象的reducer{xxx:function reducer...}
 * @return reducer函数
 *    返回的是一个新的reducer函数（把这个值赋值给createStore）
 * 特殊处理：合并reducer之后，redux容器中的state也变为以对应对象管理的模式 => {xxx:{}...}
 */

function combineReducers(reducers) {
  // reducers: 传递进来的reducer对象集合
  /**
   *  {
   *    vote:function vote(state ={n:0,m:0},action) {return state;}
   *    personal:function personal(state ={baseInfo=""},action) {return state;}
   *  }
   * 
   */

  return function reducer(state = {}, action) {
    // dispatch派发执行的时候，执行的是返回的reducer，这里也要返回一个最终的state对象替换原有的state，而且这个state中包含每个模块的状态信息 => {vote:...,personal:...}
    // =>合并reducer，其实就是dispatch派发的时候，把每一个模块的reducer都单独执行一遍，把每个模块返回的状态最后汇总在一起，替换容器中的状态信息
    let newState = {};
    for (const key in reducers) {
      if (!reducers.hasOwnProperty(key)) break;
      // reducers[key] 每个模块单独的reducer
      // state[key]:当前模块在redux容器存储的状态信息
      // 返回值是当前模块最新的状态，把它再放到newState中
      newState[key] = reducers[key](state[key], action);
    }
    return newState;
  }
}









// 用法
let reducer = (state = {}, action) => {
  // state:原有状态信息
  // action:派发任务时候传递的行为对象
  switch (action.type) {
    // 更具type执行不同的state修改操作

  }
  return state  //返回的state会替换原有的state
}
let store = createStore(reducer); //create的时候把reducer传递进来，但是此时reducer并没有执行，只有dispatch的时候才执行，通过执行reducer修改容器中的状态
````

react-redux

````js
import React, { Component } from 'react';
import PropTypes, { func } from 'prop-types';
/**
 * Provider:当前项目的"根组件"
 *  1.接收通过属性传递进来的store,把store挂载到上下文中，这样当前项目中任何一个组件中，想要使用redux中的store，直接通过上下文获取即可
 *  2.在组件的render中，把传递给Provider的子元素渲染
 */

class Provider extends Component {
  // 设置上下文信息类型
  static childContextTypes = {
    store: PropTypes.object
  }
  // 设置上下文信息值
  getChildContext() {
    return {
      store: this.props.store
    }
  }

  constructor(props, context) {
    super(props, context)
  }

  render() {
    return this.props.children;
  }


}

/**
 * connect:高阶组件（基于高阶函数：柯理化函数）创建的组件就是高阶组件
 *  @params mapStateToProps mapDispatchToProps
 *    mapStateToProps：回调函数，把redux中的部分状态信息(方法返回的内容)挂载到指定组件的属性上
 *    function mapStateToProps(state) {
 *     // state:redux容器中的状态信息
 *      return {}; //return 对象中有什么，就把它挂载到属性上
 *    }
 *    mapDispatchToProps：回调函数，把一些需要派发的任务方法也挂载到组件的属性上
 *    function mapDispatchToProps(dispatch) {
 *     // dispatch:store中的dispatch
 *      return {
 *          init(){
 *            dispatch({xxx});
 *           }
 *      }; //return 对象中有什么，就把它挂载到属性上(返回的方法中有执行dispatch派发任务的操作)
 *    }
 * @return
 *  返回一个新的函数 connectHOT
 * 
 * ================
 * connectHOT
 *  @param Component
 *    传递进来的是要操作的组件，我们需要把指定的属性和方法都挂载到当前组件的属性上
 *  @return 
 *    返回一个新的组件Proxy(代理组件)，在代理组件中，我们要获取Provider在上下文中存储store，紧接着获取store中的state和dispatch,把mapStateToProps、mapDispatchToProps回调函数执行，接收返回的结果，再把这些结果挂载到Component这个操作组件的属性上
 */

function connect(mapStateToProps, mapDispatchToProps) {
  return function connectHOT(UIComponent) {
    return class Proxy extends Component {
      // 获取上下文中的store
      static contextTypes = {
        store: PropTypes.object
      }
      // 获取store中的state/dispatch,把传递的两个回调函数执行，接收返回的结果
      constructor(props, context) {
        super(props, context);
        this.state = this.queryMountProps();
      }
      // =>基于redux中的subscribe向事件池中追加一个方法，当容器中状态改变，我们需要重新获取最新的状态信息，并且重新把component渲染，把最新的状态信息通过属性传递给component
      componentDidMount() {
        this.context.store.subscribe(() => {
          this.setState(this.queryMountProps())  
        })
      }
      // =>从redux中获取最新的信息，基于回调函数筛选，返回的是需要挂载到组件属性上的信息
      queryMountProps = () => {
        let { store } = this.context,
          state = store.getState();
        let propsState = typeof mapStateToProps === 'function' ? mapStateToProps(state) : {};
        let propsDispatch = typeof mapDispatchToProps === 'function' ? mapDispatchToProps(store.dispatch) : {};
        return {
          ...propsState,
          ...propsDispatch
        }
      }
      // 渲染传递进来的component组件，并且把获取的信息(状态、方法)挂载到组件属性上(当度调取proxy组件的时候传递的属性也给UIComponent)
      render() {

        return <UIComponent {...this.state} {...this.props}/>
      }
    }
  }
}

export {
  Provider,
  connect
}
````

