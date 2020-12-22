# [React Hooks 与 Immutable 数据流实战](https://juejin.cn/book/6844733816460804104)



## react-router-config的使用

是一个辅助react-router的插件，主要是使用配置文件集中式管理路由。

```javascript
import React from 'react'
import { Redirect } from 'react-router-dom'
import Home from '../application/Home';
import Recommend from '../application/Recommend';
import Singers from '../application/Singers';
import Rank from '../application/Rank';

export default [
    {
        path:"/",
        component:Home,
        routes:[
            {
                path:"/",
                exact:true,
                render:() => (
                    <Redirect to={"/recommend"}/>
                )
            },
            {
                path:"/recommend",
                component:Recommend,
            },
            {
                path:"/singers",
                component:Singers,
            },
            {
                path:"/rank",
                component:Rank,
            }
        ]
    }
]
```

### 在App.js中渲染配置文件中的组件

**渲染配置文件中的组件，一次只能渲染配置文件中的一层，在上面的配置文件中即Home组件。**renderRoutes(routes)函数会把routers作为props传入到Home组件中。

```jsx
import React from 'react';

//style
import { IconStyle } from './assets/iconfont/iconfont';
import { GlobalStyle } from './style';
//router
import routes from './routes/index';
import {renderRoutes  } from 'react-router-config'
import { HashRouter } from 'react-router-dom'
//redux
import { Provider } from 'react-redux'
import store from './store/index'


function App() {
  return (
    <Provider store={store}>
      <HashRouter>
      <GlobalStyle></GlobalStyle>
      <IconStyle></IconStyle>
{/* renderRoutes(routes)会把routers作为props传入到Home组件中 */}
      {renderRoutes(routes)}
    </HashRouter>
    </Provider>
    
  );
}

export default App;
```

在Home组件中使用相同的方法渲染它的子路由

```jsx
import React from 'react'
import { renderRoutes } from "react-router-config";

import {
    Top,
    Tab,
    TabItem
} from './style'
import { NavLink } from 'react-router-dom'

const Home = (props) => {
    /* 
    props: {
      history: ...,
      location: ...,
      match: ...,
      route: ...,
      staticContext: ...,
      其他自定义传入的属性: ...
    }
  */
    const { route } = props;
    return (
        <div>
            <Top>
                <span className="iconfont menu">&#xe65c;</span>
                <span className="title">WebApp</span>
                <span className="iconfont search">&#xe62b;</span>
            </Top>
            <Tab>
                <NavLink to="/recommend" activeClassName="selected"><TabItem><span > 推荐 </span></TabItem></NavLink>
                <NavLink to="/singers" activeClassName="selected"><TabItem><span > 歌手 </span></TabItem></NavLink>
                <NavLink to="/rank" activeClassName="selected"><TabItem><span > 排行榜 </span></TabItem></NavLink>
            </Tab>
            {renderRoutes(route.routes)}
        </div>
    )
}

export default React.memo(Home);
```



## useState的使用

参考：https://blog.csdn.net/wu_xianqiang/article/details/105181044

```javascript
const [state, setState] = useState(initialState); // 使用格式
// 返回一个 state，以及更新 state 的函数
```

在初始渲染期间，返回的状态 (`state`) 与传入的第一个参数 (`initialState`) 值相同。

`setState` 函数用于更新 state。它接收一个新的 state 值并将组件的一次重新渲染加入队列。

```javascript
setState(newState);
```

在后续的重新渲染中，useState 返回的第一个值将始终是更新后最新的 state。

### setCount的两种方式（普通方式和函数式更新）

如果新的 state 需要通过使用先前的 state 计算得出，那么可以将函数传递给 `setState`。**该函数将接收先前的 state，并返回一个更新后的值**。下面的计数器组件示例展示了 `setState` 的两种用法：

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  function handleClick() {
    setCount(count + 1)
  }
  function handleClickFn() {
    setCount((prevCount) => {
      return prevCount + 1
    })
  }
  return (
    <>
      Count: {count}
      <button onClick={handleClick}>+</button>
      <button onClick={handleClickFn}>+</button>
    </>
  );
}

```

#### 两种方式的区别

注意上面的代码，`handleClick`和`handleClickFn`一个是通过一个新的 state 值更新，一个是通过函数式更新返回新的 state。现在这两种写法没有任何区别，但是如果是异步更新的话，那你就要注意了，他们是有区别的，来看下面例子：

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  function handleClick() {
    setTimeout(() => {
      setCount(count + 1)
    }, 3000);
  }
  function handleClickFn() {
    setTimeout(() => {
      setCount((prevCount) => {
        return prevCount + 1
      })
    }, 3000);
  }
  return (
    <>
      Count: {count}
      <button onClick={handleClick}>+</button>
      <button onClick={handleClickFn}>+</button>
    </>
  );
}
```

当我设置为异步更新，点击按钮延迟到3s之后去调用`setCount`函数，当我快速点击按钮时，也就是说在3s多次去触发更新，但是只有一次生效，因为 `count` 的值是没有变化的。

当使用函数式更新 state 的时候，这种问题就没有了，因为它可以获取之前的 state 值，也就是代码中的 `prevCount` 每次都是最新的值。

其实这个特点和类组件中 `setState` 类似，可以接收一个新的 state 值更新，也可以函数式更新。如果新的 state 需要通过使用先前的 state 计算得出，那么就要使用函数式更新。

因为setState更新可能是异步，当你在事件绑定中操作 state 的时候，setState更新就是异步的。

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props)
    this.state = { count: 0 }
  }
  handleClick = () => {
    this.setState({ count: this.state.count + 1 })
    this.setState({ count: this.state.count + 1 })
    // 这样写只会加1
  }
  handleClickFn = () => {
    this.setState((prevState) => {
      return { count: prevState.count + 1 }
    })
    this.setState((prevState) => {
      return { count: prevState.count + 1 }
    })
  }
  render() {
    return (
      <>
        Count: {this.state.count}
        <button onClick={this.handleClick}>+</button>
        <button onClick={this.handleClickFn}>+</button>
      </>
    );
  }
}
```

当你在定时器中操作 state 的时候，而 setState 更新就是同步的。

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props)
    this.state = { count: 0 }
  }
  handleClick = () => {
    setTimeout(() => {
      this.setState({ count: this.state.count + 1 })
      this.setState({ count: this.state.count + 1 })
      // 这样写是正常的，两次setState最后是加2
    }, 3000);
  }
  handleClickFn = () => {
    this.setState((prevState) => {
      return { count: prevState.count + 1 }
    })
    this.setState((prevState) => {
      return { count: prevState.count + 1 }
    })
  }
  render() {
    return (
      <>
        Count: {this.state.count}
        <button onClick={this.handleClick}>+</button>
        <button onClick={this.handleClickFn}>+</button>
      </>
    );
  }
}
```

注意这里的同步和异步指的是 setState 函数。因为涉及到 state 的状态合并，react 认为当你在事件绑定中操作 state 是非常频繁的，所以为了节约性能 react 会把多次 setState 进行合并为一次，最后在一次性的更新 state，而定时器里面操作 state 是不会把多次合并为一次更新的。

> 注意：与 class 组件中的 setState 方法不同，useState 不会自动合并更新对象。



## useEffect的使用

参考：https://www.jianshu.com/p/2115c3253e98

参考：https://segmentfault.com/a/1190000020104281

**useEffect类似与class组件里面的 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合**

### 情况1：当useEffect没有第二个参数的时候会不停地调用

### 情况2：当useEffect的第二个参数为空数组时调用一次后就不在调用了

### 情况3：当useEffect的第二个参数里面的数组是一些变量的时候表示这个变量发生变化的时候才会触发

### useEffect的清除

```javascript
useEffect(()=>{
	setTimeout(()=>{
		console.log(count)
	},3000)
	// 清除useEffect 通过return清除
	return () => {}
})
```



## useMemo和useCallback

参考：https://blog.csdn.net/sinat_17775997/article/details/94453167

**useMemo返回的是一个变量，而useCallback返回的是一个函数需要执行**

先简单的看一下useMemo和useCallback的调用签名：

> function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T; function useCallback<T extends (...args: any[]) => any>(callback: T, deps: DependencyList): T;

useCallback和useMemo的参数跟useEffect一致，他们之间最大的区别有是useEffect会用于处理副作用，而前两个hooks不能。

useMemo和useCallback都会在组件第一次渲染的时候执行，之后会在其依赖的变量发生改变时再次执行；并且这两个hooks都返回缓存的值，useMemo返回缓存的变量，useCallback返回缓存的函数。

### useMemo

我们来看一个反例：

```
import React from 'react';



 



 



export default function WithoutMemo() {



    const [count, setCount] = useState(1);



    const [val, setValue] = useState('');



 



    function expensive() {



        console.log('compute');



        let sum = 0;



        for (let i = 0; i < count * 100; i++) {



            sum += i;



        }



        return sum;



    }



 



    return <div>



        <h4>{count}-{val}-{expensive()}</h4>



        <div>



            <button onClick={() => setCount(count + 1)}>+c1</button>



            <input value={val} onChange={event => setValue(event.target.value)}/>



        </div>



    </div>;



}
```

这里创建了两个state，然后通过expensive函数，执行一次昂贵的计算，拿到count对应的某个值。我们可以看到：无论是修改count还是val，由于组件的重新渲染，都会触发expensive的执行(能够在控制台看到，即使修改val，也会打印)；但是这里的昂贵计算只依赖于count的值，在val修改的时候，是没有必要再次计算的。在这种情况下，我们就可以使用useMemo，只在count的值修改时，执行expensive计算：

```
export default function WithMemo() {



    const [count, setCount] = useState(1);



    const [val, setValue] = useState('');



    const expensive = useMemo(() => {



        console.log('compute');



        let sum = 0;



        for (let i = 0; i < count * 100; i++) {



            sum += i;



        }



        return sum;



    }, [count]);



 



    return <div>



        <h4>{count}-{expensive}</h4>



        {val}



        <div>



            <button onClick={() => setCount(count + 1)}>+c1</button>



            <input value={val} onChange={event => setValue(event.target.value)}/>



        </div>



    </div>;



}
```

上面我们可以看到，使用useMemo来执行昂贵的计算，然后将计算值返回，并且将count作为依赖值传递进去。这样，就只会在count改变的时候触发expensive执行，在修改val的时候，返回上一次缓存的值。

### useCallback

讲完了useMemo，接下来是useCallback。useCallback跟useMemo比较类似，但它返回的是缓存的函数。我们看一下最简单的用法：

> const fnA = useCallback(fnB, [a])

上面的useCallback会将我们传递给它的函数fnB返回，并且将这个结果缓存；当依赖a变更时，会返回新的函数。既然返回的是函数，我们无法很好的判断返回的函数是否变更，所以我们可以借助ES6新增的数据类型Set来判断，具体如下：

```
import React, { useState, useCallback } from 'react';



 



const set = new Set();



 



export default function Callback() {



    const [count, setCount] = useState(1);



    const [val, setVal] = useState('');



 



    const callback = useCallback(() => {



        console.log(count);



    }, [count]);



    set.add(callback);



 



 



    return <div>



        <h4>{count}</h4>



        <h4>{set.size}</h4>



        <div>



            <button onClick={() => setCount(count + 1)}>+</button>



            <input value={val} onChange={event => setVal(event.target.value)}/>



        </div>



    </div>;



}
```

我们可以看到，每次修改count，set.size就会+1，这说明useCallback依赖变量count，count变更时会返回新的函数；而val变更时，set.size不会变，说明返回的是缓存的旧版本函数。

知道useCallback有什么样的特点，那有什么作用呢？

使用场景是：有一个父组件，其中包含子组件，子组件接收一个函数作为props；通常而言，如果父组件更新了，子组件也会执行更新；但是大多数场景下，更新是没有必要的，我们可以借助useCallback来返回函数，然后把这个函数作为props传递给子组件；这样，子组件就能避免不必要的更新。

```
import React, { useState, useCallback, useEffect } from 'react';



function Parent() {



    const [count, setCount] = useState(1);



    const [val, setVal] = useState('');



 



    const callback = useCallback(() => {



        return count;



    }, [count]);



    return <div>



        <h4>{count}</h4>



        <Child callback={callback}/>



        <div>



            <button onClick={() => setCount(count + 1)}>+</button>



            <input value={val} onChange={event => setVal(event.target.value)}/>



        </div>



    </div>;



}



 



function Child({ callback }) {



    const [count, setCount] = useState(() => callback());



    useEffect(() => {



        setCount(callback());



    }, [callback]);



    return <div>



        {count}



    </div>



}
```

不仅是上面的例子，所有依赖本地状态或props来创建函数，需要使用到缓存函数的地方，都是useCallback的应用场景。

### 多谈一点：

useEffect、useMemo、useCallback都是自带闭包的。也就是说，每一次组件的渲染，其都会捕获当前组件函数上下文中的状态(state, props)，所以每一次这三种hooks的执行，反映的也都是当前的状态，你无法使用它们来捕获上一次的状态。对于这种情况，我们应该使用ref来访问。