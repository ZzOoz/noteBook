# React知识总结



## 1、创建create-react-app

```js
npx create-react-app my-app
cd my-app
npm start
```

## 2、定义组件的方式

**2.1、使用箭头函数**

```js
import react from 'react'
import ReactDOM from 'react-dom'

// 创建组件的第一种方式：使用箭头函数，但是这个名字要大写开始。
// 利用props传入参数，同时使用{}来渲染数据
const createApp = (props)=>{
	return (
    	<div>
             {/* 只要在jsx里要插入js的代码，就加一层花括号即可，注释也是js，所以这里的注释就加了一层花括号 */}
        	<p title={props.title}>hello {props.title}!</p>
        </div>
    )
}


const app = createApp({
    title:"world"
})

ReactDOM.render(
	app,
    document.querySelector("#root")
)


//也可以使用这种方式
const App = (props)=>{
    return (
    	<div>hello {props.title}</div>
    )
}

ReactDOM.render(
	<App title="1999"/>,
    document.querySelector("#root")
)
```



**2.2、使用类的方式创建组件**

```js
import React, { Component } from 'react'
import {render} from 'react-dom'

class App extends Component {
    render() {
        return (
            <div>
                Hello {this.props.title}                
            </div>
        )
    }
}

render(
    <App title="world"/>,
    document.querySelector("#root")
)

```



## 3、jsx的原理

jsx：是一种扩展。需要在jsx中写js的话需要在{}中写

本质上是使用React.createElement()的方法，比如

```js
// 创建一个App的节点
class App extends Component {
// 使用了render方法
    render() {
        return (
        // 括号内的不是真正的js语法，这个jsx
            <div className="div1">
                <h1>Hello</h1>
            </div>
        )
    }
}

// 而本质上是调用了React.createElement方法，是这种方式创建节点
class App extends Component {
    render() {
        return (
            React.createElement(
                "div",
                {
                    className:"div1"
                },
                React.createElement(
                    'h1',
                    null,
                    'Hello'
                )
            )
        )
    }
}
```



4、定义组件样式（可以使用classname和styled-components插件）

例子：

```js
import React, { Component } from 'react'
import {render} from 'react-dom'
import classNames from 'classnames'
import styled from 'styled-components'

// 引入index.css
import './index.css'

// 定义一css组件样式 使用styled-components插件
const Title = styled.h1`
    color:red
`

// react组件中的样式
class App extends Component {
    render() {
        return (
            <div className="div1">
                {/* 使用classname插件 需要index.css文件*/}
                <h1 className={classNames('a',{b:true,c:false})}>Hello</h1>
                {/* 使用styled-components插件 */}
                <Title>我是h2</Title>
            </div>
        )
    }
}

render(
    <App></App>,
    document.querySelector('#root')
)
```



## 4、组件中的props和props-types（prop检验）

**props是组件外部的组件传递到组件本身的数据**

使用props-types对传入组件中的props中的数据进行校验

主App组件（引入TodoList和TodoInput）

```js
import React, { Component } from 'react'
import {
    TodoList,
    TodoInput
} from './components'

export default class App extends Component {
    render() {
        return (
            <div>
                <TodoList desc="待办事项"></TodoList>
                <TodoInput inputText="输入input"></TodoInput>
            </div>
        )
    }
}

```

4.1、在函数式组件中使用props

```js
import React from 'react'
import PropsTypes from 'prop-types'

// 在函数中使用props-types
export default function TodoInput(props) {
    return (
        <div>
            {props.inputText}
        </div>
    )
}

TodoInput.propTypes = {
    inputText:PropsTypes.string.isRequired
}
TodoInput.defaultProps = {
    inputText:'默认输入'
}
```

4.2、在类中使用props（在App组件中传入desc数据要使用this.props.desc才能获取）

```js
import React, { Component } from 'react'
import PropsTypes from 'prop-types'

// 在类中使用props-types进行数据检验
export default class TodoList extends Component {
    static propTypes = {
        // 必须且为字符串
        desc:PropsTypes.string
    }
    static defaultProps = {
        desc:'默认描述'
    }
    render() {
        return (
            <div>
                {this.props.desc}
            </div>
        )
    }
}
```



## 5、组件中的state属性

state属性是组件自身的属性，与props不同，props是从组件外部传入，state是自己特有的

**在组件中，仅仅只有类组件具有state属性，具有自身的状态，因为类可以保存变量，而函数组件funciton不能保存变量，已执行完成就会将变量销毁**

```js
import React, { Component } from 'react'
import {
    TodoList,
    TodoInput
} from './components'

export default class App extends Component {
    state = {
        desc:'11',
        inputText:'输入'
    }
    render() {
        return (
            <div>
                <TodoList desc={this.state.desc}></TodoList>
                <TodoInput inputText={this.state.inputText}></TodoInput>
            </div>
        )
    }
}

```

5.1、组件的分类

（1）根据定义的方式可以分为：**函数式组件、类组件**

（2）根据是否具有state可以分为：**状态组件（类组件）、无状态组件（函数式）**

（3）仅仅具有props的组件，而没有state属性的可以看作是**受控组件**，而既有props也有state可以看作**非受控组件**



6、dangerouslySetInnerHTML的使用（对于一些富文本的使用，比如说后台传来一连串的html：**<p><span>hhhh</span></p>**等）

```jsx
import React, { Component } from 'react'
import {
    TodoList,
    TodoInput
} from './components'

export default class App extends Component {
    state = {
        article:'<div>哈哈哈我是article</div>'
    }
    render() {
        return (
            <div>
                {
                    // 如果是想要吧标签嵌入
                    <div dangerouslySetInnerHTML={{__html:this.state.article}}></div>
                }
            </div>
        )
    }
}

```



## 6、setState的使用方法（是异步的）

```js
import React, { Component } from 'react'

export default class Like extends Component {
    constructor(){
        super()
        this.state = {
            isLike:false
        }
    }

    handleLike = ()=>{
        // 这是使用setState的第一种方法
        // this.setState({
        //     isLike:!this.state.isLike
        // })
        
        // 第二种方法如下
        //setState中第一个参数是一个函数，函数的两个参数是上一次的state和props
        //setState中第二个参数是函数，可以获取最新的state
        this.setState((prevState,props)=>{
            return {
                isLike:!prevState.isLike
            }
        },()=>{
            //由于setState是异步的，如果想获取到最新的state，应该在这个回调里面获取
           console.log(this.state) 
        })
    }

    render() {
        return (
            <span onClick={this.handleLike}>
                {this.state.isLike ? '取消喜欢':'我喜欢'}
            </span>
        )
    }
}

```



## 7、react中的事件（重要！！）以及他的this指向问题

参考：https://blog.csdn.net/a1059526327/article/details/104327176



## 8、生命周期问题

**注意：生命周期只会存在于类组件中，而函数式组件是没有的，生命周期本质上是一些回调函数，而函数式组件是通过传参才能调用回调函数**

**首先：constructor只会执行一次，pureComponent会进行一层浅比较**

**常见的生命周期：**

**（1）constructor**

**使用场景：**通常给this.state赋值来初始化内部的state（不要在该生命周期内使用this.setState）、对事件处理函数进行绑定实例

**注意事项：**不要在这个生命周期内进行setState操作、避免将props的值赋值给state（没有必要且props改变不会影响state）

**（2）componentDidMount**

**使用场景：**通常进行网络请求获取数据的生命周期函数、添加订阅事件的地方（取消订阅事件的时候在componentWillUnmount中）、可以使用setState方法（但会有性能问题）

**（3）componentDidUpdate**

**使用场景：**在首次渲染的时候是不会执行的，只有在数据更新的时候才会更新，当组件更新后，可以在此处对 DOM 进行操作。如果你对更新前后的 props 进行了比较，也可以选择在此处进行网络请求。（例如，当 props 未发生变化时，则不会执行网络请求）。

**注意事项：**虽然setState可以在该生命周期可以调用，但是他必须包裹在一个条件语句中，如果不这样做会陷入死循环，还会导致额外的渲染、如果 [`shouldComponentUpdate()`](https://react.docschina.org/docs/react-component.html#shouldcomponentupdate) 返回值为 false，则不会调用 `componentDidUpdate()`。

**（4）componentWillUnmount()**

**使用场景：**`componentWillUnmount()` 会在组件卸载及销毁之前直接调用。在此方法中执行必要的清理操作，例如，清除 timer，取消网络请求或清除在 `componentDidMount()` 中创建的订阅等。

`componentWillUnmount()` 中**不应调用 `setState()`**，因为该组件将永远不会重新渲染。组件实例卸载后，将永远不会再挂载它。



不常见的生命周期：



## 9、react-hooks（新）用于提供函数组件能够像类组件一样使用生命周期和拥有状态的功能

```js
import React,{useState, useEffect} from 'react'
import {render} from 'react-dom'

function App() {
    // 使用useState来定义变量和方法，setCount是方法可以改变变量
    const [count,setCount] = useState(0)
    // useEffect类似于生命周期，可以在组件初次渲染和更新时触发
    useEffect(()=>{
        document.title = `${count}`
    })
    return (
        <div>
            <div>{count}</div>
            <button onClick={()=>{setCount(count + 1)}}>添加</button>
        </div>
    )
}
render(
    <App />,
    document.querySelector('#root')
)

```



## 10、context上下文（用于跨组件通信）

实现一个计算器例子：

```js
// 这是一个cotext的例子
import React,{ Component,createContext} from 'react'
import {render} from 'react-dom'

// 引入createContext解构出来Provider和Consumer
const {
    Provider,
    Consumer:CounterConsumer
} = createContext()

// 在Provider外面包裹一层CounterProvider
class CounterProvider extends Component{
    constructor(){
        super()
        this.state = {
            count:100
        }
    }

    // 计算加一方法
    increment = () => {
        this.setState({
            count:this.state.count + 1
        })
    }

    // 计算减一方法
    decrement = () => {
        this.setState({
            count:this.state.count - 1
        })
    }

    render(){
        return (
            // 将所有的值通过value传入
            <Provider value={{
                count:this.state.count,
                onIncrement:this.increment,
                onDecrement:this.decrement
            }}>
                {this.props.children}
            </Provider>
        )
    }
}

class CounterBtn extends Component{
    render(){
        return (
            // CounterConsumer里面必须接受一个function
            // 判断handleEvent的类型
            <CounterConsumer>
                {
                    ({onIncrement,onDecrement}) => {
                        const handlerEvent = this.props.type === 'increment' ? onIncrement : onDecrement
                        return (
                            <button type="button" onClick={handlerEvent}>{this.props.children}</button>
                        )
                    }
                }
            </CounterConsumer>
        )
    }
}


class Counter extends Component{
    render(){
        return (
            // 这里也是在CounterConsumer里面放一个方法
            <CounterConsumer>
                {/* 给counter的值 */}
                {
                    ({count})=>{
                        return (
                            <span>{count}</span>
                        )
                    }
                }
            </CounterConsumer>
        )
    }
}

class App extends Component {
    render() {
        return (
            <>
                <CounterBtn type="decrement">-</CounterBtn>
                <Counter></Counter>
                <CounterBtn type="increment">+</CounterBtn>
            </>
        )
    }
}

render(
    // 在APP最外面给一层CounterProvider可以共享所有的数据
    <CounterProvider>
        <App />
    </CounterProvider>,
    document.querySelector("#root")
)

```

将这个文件分解成一个目录结构

```js
// index.js文件
// 这是一个cotext的例子
import React from 'react'
import {render} from 'react-dom'
import App from './App'
import {CounterProvider} from './CounterStore'


render(
    // 在APP最外面给一层CounterProvider可以共享所有的数据
    <CounterProvider>
        <App />
    </CounterProvider>,
    document.querySelector("#root")
)

```



```js
// App.js文件
import React, { Component } from 'react'
import CounterBtn from './components/CounterBtn'
import Counter from './components/Counter'
export default class App extends Component {
    render() {
        return (
            <>
                <CounterBtn type="decrement">-</CounterBtn>
                <Counter></Counter>
                <CounterBtn type="increment">+</CounterBtn>
            </>
        )
    }
}

```



```js
// counterStore.js文件
import React, { Component,createContext } from 'react'


// 引入createContext解构出来Provider和Consumer
const {
    Provider,
    Consumer:CounterConsumer
} = createContext()

// 在Provider外面包裹一层CounterProvider
class CounterProvider extends Component{
    constructor(){
        super()
        this.state = {
            count:100
        }
    }

    // 计算加一方法
    increment = () => {
        this.setState({
            count:this.state.count + 1
        })
    }

    // 计算减一方法
    decrement = () => {
        this.setState({
            count:this.state.count - 1
        })
    }

    render(){
        return (
            // 将所有的值通过value传入
            <Provider value={{
                count:this.state.count,
                onIncrement:this.increment,
                onDecrement:this.decrement
            }}>
                {this.props.children}
            </Provider>
        )
    }
}

export {
    CounterProvider,
    CounterConsumer
}

```



```js
// component/CounterBtn/index.js文件
import React, { Component } from 'react'
import {CounterConsumer} from '../../CounterStore'

export default class CounterBtn extends Component{
    render(){
        return (
            // CounterConsumer里面必须接受一个function
            // 判断handleEvent的类型
            <CounterConsumer>
                {
                    ({onIncrement,onDecrement}) => {
                        const handlerEvent = this.props.type === 'increment' ? onIncrement : onDecrement
                        return (
                            <button type="button" onClick={handlerEvent}>{this.props.children}</button>
                        )
                    }
                }
            </CounterConsumer>
        )
    }
}

```



```js
// component/Counter/index.js文件
import React, { Component } from 'react'
import {CounterConsumer} from '../../CounterStore'
export default class Counter extends Component{
    render(){
        return (
            // 这里也是在CounterConsumer里面放一个方法
            <CounterConsumer>
                {/* 给counter的值 */}
                {
                    ({count})=>{
                        return (
                            <span>{count}</span>
                        )
                    }
                }
            </CounterConsumer>
        )
    }
}

```





## 11、HOC和使用装饰器

HOC是高阶组件，将一个组件包裹在一个函数里面，这个函数可以传值进入到该组件

```js
import React, { Component } from 'react'

// 参数是一个组件
const withCopyRight = (YourComponents)=>{
    return class withCopyRight extends Component {
        render() {
            return (
                <div>
                    <YourComponents />
                    <div>著作商标</div>
                </div>
            )
        }
    }
}

export default withCopyRight
```



**使用装饰器的方法步骤**

![image-20200714101652617](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200714101652617.png)

## 12、redux的使用

**redux是什么？（它跟vuex状态管理很像，都是使用单一对象进行管理，使用单例模式）**

```
（1）Web 应用是一个状态机，视图与状态是一一对应的。

（2）所有的状态，保存在一个对象里面。
```

**redux的基本概念和api**

**store：存放的地方数据，使用createStore来创建一个store，store里面包含了getState、subscribe、dispatch方法**

```
cosnt store = createStore(reducer) // createStore接受reducer作为参数，注意reducer是一个函数
```

**state：可以理解为是某一个组件里面的数据，比如cartlist组件里面有他自己的state**

**dispatch:可以理解为是view向action发送通知说你要改变state了**

```
store.dispatch(action) // 接受action，action里面有type和payload
```

**action：他是惟一能够改变state的唯一方法，是view视图层发出的通知(而这个通知就是使用dispatch)，通知state要发生变化了**

**actionCreateor：实际上是一个函数，简单来说就是可以传参的action**

```js
// 这就是一个actionCreator
export const add = (id) => {
	return {
		type:'INCREMENT',
		payload:{
        	id // 可传参
        }
	}
}

// 普通的action
const add = {
    type:"INCREMENT"
}
```



**reducer：可以理解为对state各个阶段的处理方式，是一个处理器，但是每次都必须要 返回一个新的state**

```js
export default (state=initState,action) => {
	switch(action.type){
		case 'INCREMENT':
			return {
				...state,
				count:state.count + 1
			}
   		case 'DECREMENT':
			return {
				...state,
				count:state.count - 1
			}
		default:
			return state  // 这里就不是break咯，应该返回一个state
	}
}
```



#### **12.1、使用的原则：**

1、唯一数据源

2、状态是只读的

3、数据改变必须通过纯函数完成

#### 12.2、redux的简单实现

简单步骤：1、创建reducers

​				  2、合并reducers

​                  3、createStore传入合并后的reducers

​                  4、使用Provider将store传入，通常在<App />外面包裹一层Provider

​                  5、使用connect(mapStateToProps,{...actionCreators})(YourComponents)

​                  6、使用actionCreators建立action

​                  7、修改reducer

**redux处理同步和异步操作：**

**同步操作**：actionCreator =》 自动dispatch(actionCreator())  =》 触发reducer操作  =》改变store的值  =》view更新

**异步操作**：actionCreator =》 经过middleware处理生成新的action =》手动dispatch(action) =》 触发reducer操作  =》改变store的值  =》view更新

在处理异步操作时需要安装中间件(middleware)

```
cnpm i redux-thunk -S  // 安装react-thunk
```





## 13、react-router的使用

BrowerRouter和hashRouter的区别

react-router的基本使用（注意：当路由匹配第一个路由之后，第二个路由就不会在匹配了）

**path的使用：**

通配符的使用规则：

```
（1）：paramsName（例子：/user/:id或者/:name/:id等）
:paramName匹配URL的一个部分，直到遇到下一个/、?、#为止。这个路径参数可以通过this.props.params.paramName取出。
（2）使用（）
()表示URL的这个部分是可选的。
（3）使用*
*匹配任意字符，直到模式里面的下一个字符为止。匹配方式是非贪婪模式。
（4）使用**
** 匹配任意字符，直到下一个/、?、#为止。匹配方式是贪婪模式。
```

**browerHistory和hashHistory的区别和使用**

参考这篇文章：https://www.cnblogs.com/nangezi/p/11490778.html

数据埋点的多种设置方式：

1.第一种是使用ajax请求（这种方式的确定是有延迟，成功率不高，而且请求频繁）

2.第二种使用img标签，当用户点击的时候可以使用src属性自带参数，请求后台，后台返回一个不同大小的像素的图片来定义状态（这种做法兼容性好）

3.第三种使用sendBecon方法，这种兼容性低，但是成功率高



## 14、antd的配置（官网上有详细的介绍，这里就不多讲了）



## 15、react-redux的使用