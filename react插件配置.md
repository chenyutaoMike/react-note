# react插件

### 自定义轮播图

````jsx
import React, { Component } from 'react'
import PropTypes from 'prop-types';
import '../static/css/banner.css';
class Banner extends Component {

  // 设置属性的默认值和规则
  static defaultProps = {
    data: [],
    interval: 3000,
    step: 1,
    speed: 300
  }
  static propTypes = {
    data: PropTypes.array,
    interval: PropTypes.number,
    step: PropTypes.number,
    speed: PropTypes.number
  }
  constructor(props) {
    super(props)
    let { step, speed } = this.props;
    this.state = {
      step,
      speed
    };
  }
  componentWillMount() {
    let { data } = this.props;
    if (data.length === 0) return;
    // 克隆数据
    this.cloneData = data.slice(0);
    this.cloneData.push(data[0]);
    this.cloneData.unshift(data[data.length - 1]);
  }
  // 自动轮播
  componentDidMount() {
    this.autoTime = setInterval(this.autoMove, this.props.interval);
  }
  componentWillUpdate(nextProps, nextState) {
    // 向右边界判断：如果最新修改的step索引大于最大索引(说明此时已经是末尾了，不能再向后走了)，我们让其立即回到（无动画）索引为1的位置
    if (nextState.step > (this.cloneData.length - 1)) {
      this.setState({
        step: 1,
        speed: 0
      })
    }
    // 向左边界判断：如果当前最新修改的索引已经小于零，说明不能继续向左走了，我们让其立即回到倒数第二张图片位置（真实的最后一张图片）
    if(nextState.step < 0){
      this.setState({
        step: this.cloneData.length -2 ,
        speed: 0
      })
    }
  }
  componentDidUpdate() {
    // 向右边界判断
    //只有是从克隆的第一张立即切换到真实的第一张后，我们才做如下处理：让其从当前第一张运动到第二张即可
    let { step, speed } = this.state;
    if (step === 1 && speed === 0) {
      // 为啥要设置定时器延迟：CSS3的transiton有一个问题（主栈执行的时候，短时间内遇到两次设置transiton-dration的代码，以最后一次设置的为主）
      let delayTimer = setTimeout(() => {
        clearTimeout(delayTimer);
        this.setState({
          step: step + 1,
          speed: this.props.speed
        })
      }, 0)
    }
    // 向左边界判断：立即回到"倒数第二张"后，我们应该让其向回再运动一张
    if(step === this.cloneData.length -2 && speed === 0){
      let delayTimer = setTimeout(() => {
        clearTimeout(delayTimer);
        this.setState({
          step: step - 1,
          speed: this.props.speed
        })
      }, 0)
    }
   
  
  }
  // 清除定时器
  componentWillUnmount() {
    clearInterval(this.autoTime)
  }
  render() {
    let { data } = this.props;
    let { cloneData } = this;
    // 控制wrapper的样式
    let { step, speed } = this.state;
    let wrapperSty = {
      width: cloneData.length * 1000 + 'px',
      left: -step * 1000 + 'px',
      transition: `all ${speed}ms linear 0ms`
    }
    if (data.length === 0) return;
    return (
      <section 
      className="container" 
      onMouseEnter={this.mouseOver} 
      onMouseLeave={this.mouseOut}
      onClick={this.handleClick}
      >
        <ul className="wrapper" style={wrapperSty} onTransitionEnd={()=> this.isRun = false}>
          {
            cloneData.map((item, index) => {
              return <li key={item.id + index} ><img src={item.pic} alt={item.title} /></li>
            })
          }
        </ul>
        <ul className="focus">
          {
            data.map((item, index) => {
              let tempIndex = step - 1;
              if (step === 0) {
                tempIndex = data.length - 1
              }
              if (step === cloneData.length - 1) {
                tempIndex = 0
              }
              return <li key={item.id + index} className={index === tempIndex  ? 'active' : ''} onClick={() =>this.check(index)}></li>
            })
          }
        </ul>
        <a href="javascript:;" className="arrow arrowLeft"></a>
        <a href="javascript:;" className="arrow arrowRight"></a>
      </section >
    )
  }
  // 向右切换
  autoMove = () => {

    this.setState({
      step: this.state.step + 1
    })
  }
  check =(index) =>{
    console.log(index)
    this.setState({
      step: index+1
    })
  }
  mouseOver =() =>{
    clearInterval(this.autoTime)
    // this.autoTime = setInterval(this.autoMove, this.props.interval);
  }
  mouseOut =() =>{
    clearInterval(this.autoTime)
    this.autoTime = setInterval(this.autoMove, this.props.interval);
  }
  // handleClick事件委托
  handleClick =(e) =>{
    let target = e.target,
        tarTag = target.tagName,
        tarClass = target.className;
    
    if(tarTag === 'A' &&  /(^| +)arrow( +|$)/.test(tarClass)){
      // 防止点击过快
      if(this.isRun) return;
      this.isRun = true;
      if(tarClass.includes('arrowRight')){
        this.autoMove();
        return 
      }
      this.setState({
        step:this.state.step -1
      })
    } 
  }
}

export default Banner;

````



### 使用react-swipe

安装

````bash
yarn add swipe-js-iso react-swipe
````

使用

````js
import React, { Component } from 'react';
import ReactSwipe from 'react-swipe';
import './static/css/index.css';
class App extends Component {
  render() {
    let reactSwipeEl;
    const img_data = [{
      id: 1,
      title: '',
      pic: require('./static/images/1.jpg')
    },
    {
      id: 2,
      title: '',
      pic: require('./static/images/2.jpg')
    },
    {
      id: 3,
      title: '',
      pic: require('./static/images/3.jpg')
    },
    {
      id: 4,
      title: '',
      pic: require('./static/images/4.jpg')
    },
    {
      id: 5,
      title: '',
      pic: require('./static/images/5.jpg')
    }
    ];
    return (
      <div>
        <br />
        <ReactSwipe
          className="carousel"
          swipeOptions={{ auto: 2000,continuous :true }}
          ref={el => (reactSwipeEl = el)}
        >
          {
            img_data.map(item => {
              return <div key={item.id}><img src={item.pic}/></div>
            })
          }
        </ReactSwipe>
      
      </div>
    );
  }
}

export default App;
````

配置手册

`https://github.com/voronianski/swipe-js-iso#config-options`