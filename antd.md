# Antd使用

### 安装

```bash
$ npm install antd --save
或者
$ yarn add antd
```

### 最简单的使用

> 最简单的时候，没有按需加载，而是直接引入就使用

在src目录下index.js文件中:

````js
import React from 'react';
import ReactDOM from 'react-dom';
import 'antd/dist/antd.css';  //直接引入
import App from './App';


ReactDOM.render( < App / > , document.getElementById('root'));
````

那么在组件中就可以直接使用了：

````react
import React from 'react';
import { Button } from 'antd'; //引入 
function App() {
    return (
        <div >
            <Button type="primary">Primary</Button> //使用
            <Button>Default</Button>
            <Button type="dashed">Dashed</Button>
            <Button type="danger">Danger</Button>
            <Button type="link">Link</Button>

        </div>
    );
}

export default App;
````

### 在脚手架中按需加载antd

>  第一种：手动按需加载，会比较麻烦
>
> 缺点比较明显，每次引入一个组件都要引入相应的样式

````react
import React from 'react';
import Button from 'antd/es/button'; //引入Button
import 'antd/es/button/style/css'; //引入Button的样式
import Pagination  from 'antd/es/pagination'; //引入Pagination
import 'antd/es/pagination/style/css'; //引入Pagination的样式
function App() {
    return (
        <div >
            <Button type="primary">Primary</Button>
            <Button>Default</Button>
            <Button type="dashed">Dashed</Button>
            <Button type="danger">Danger</Button>
            <Button type="link">Link</Button>
            <Pagination  defaultCurrent={1} total={50} />
        </div>
    );
}

export default App;
````

> 第二种：使用npm run eject 弹出脚手架配置项进行按需加载

1. 先把.git文件删除

2. 运行 `npm run eject`命令，弹出配置项

3. 安装`npm install babel-plugin-import --save-dev` 插件

4. 安装完成之后配置根目录下的`package.json`文件的babel处，在persets后面添加

   ````json
   "babel": {
       "presets": [
         "react-app"
       ],     //在这里加入下面的内容
       "plugins": [   
         ["import", {
           "libraryName": "antd",
           "libraryDirectory": "es",
           "style": "css" 
         }]
       ]
     }
   ````

   antd配置说明：https://ant.design/docs/react/introduce-cn

