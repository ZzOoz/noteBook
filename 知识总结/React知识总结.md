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



```
// component/Counter/index.js文件
```



## 





## 12、redux的使用

#### **12.1、使用的原则：**

1、唯一数据源

2、状态是只读的

3、数据改变必须通过纯函数完成

#### 12.2、redux的简单实现





## 13、react-router的使用





## 14、antd的配置（官网上有详细的介绍，这里就不多讲了）



