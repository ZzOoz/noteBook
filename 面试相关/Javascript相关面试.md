# 面试题相关总结



## 1、Javascript

### A、初级



##### **var、let、const的区别**

答： 1、var和let是变量、const是常量（不可变，但是如果是对象则可以修改内部属性）

​		2、let和const有块级作用域（ES6语法），而var没有块级作用域（ES5语法）

​		3、var有变量提升 、let和const没有变量提升，例如：

```
console.log(a) // undefined 
var a = 100

// 实质上存在变量提升，相当于
var a
console.log(a) // undefined 
a = 100

//如果是let、const的话
console.log(b) // 直接报错，不存在变量提升
let b = 100 
```



函数声明和函数表达式的区别

```js
function fn(){...}  //函数声明
const fn = function(){.....} // 函数表达式

//区别：函数声明会预加载（类似于变量提升）而函数表达式是不会预加载的

//1、函数声明(可以计算出sum的值，因为函数声明会发生预加载)相当于把sum函数提前声明
console.log(sum(10,20))
function sum(a,b){
    return a + b
}
              
//2、函数表达式（报错，表达式无法预加载）
console.log(sum(10,20))
const sum = function(a,b){
    return a + b
}
```

##### js基本类型和引用类型

基本类型：undefined、null、string、number、boolean、symbo（ES6）

普通基本类型：undefined、null、symbol（ES6）

特殊基本包装类型：string、number、boolean

引用类型：Object、Array、RegExp、Date、Function

**区别：引用类型值可添加属性和方法，而基本类型值则不可以。**

基本类型

基本类型的变量是存放在栈内存（Stack）里的
基本数据类型的值是按值访问的
基本类型的值是不可变的
基本类型的比较是它们的值的比较

引用类型

引用类型的值是保存在堆内存（Heap）中的对象（Object）
引用类型的值是按引用访问的
引用类型的值是可变的
引用类型的比较是引用的比较

##### **typeof能判断的类型**

答：值类型：string、number、boolean、symbol、undefined

​									  引用类型：object（仅仅只能判断是否是引用类型无法判断是否为对象或数组）

​									  其他：function



##### **列举强制转换和隐式类型转换**

答：1.强制类型转换：parseInt、parseFloat、toString等

​		2.隐式类型转换：if、==、字符串拼接、逻辑运算if/else

##### **事件冒泡和捕获**

参考：https://blog.csdn.net/weixin_45471827/article/details/99635430

阻止事件冒泡：event.cancelBubble和event.stopPropagation

阻止事件捕获：event.stopPropagation

阻止默认行为：

##### **写一个深度比较的函数isEqual**

要求：obj1和obj2有相同的属性就可以为true，不对引用指针作比较

```js
// 深度比较函数
function isObject(obj) {
    return typeof obj === 'object' &&  obj !== null
}

function isEqual(obj1, obj2) {
    // 如果二者其中之一或两个都不是对象
    // 1路线
    if (!isObject(obj1) || !isObject(obj2)) {
        return obj1 === obj2
    }

    // 2路线 如果用户两个参数都是一样
    if (obj1 === obj2) {
        return true // 直接返回true
    }

    const obj1Keys = Object.keys(obj1)
    const obj2Keys = Object.keys(obj2)


    if (obj1Keys.length !== obj2Keys.length) {
        return false
    }

    // 3路线
    for (let key in obj1) {
        // 对两个参数里面的属性再次递归比较，如果遇到普通值类型走1，如果遇到引用走3
        const res = isEqual(obj1[key], obj2[key])
            // 循环遍历如果出现false，那么退出循环
        if (!res) {
            return false
        }
    }

    return true
}

const obj1 = {
    a: 100,
    b: {
        x: 100,
        y: 200,
    }
}

const obj2 = {
    a: 100,
    b: {
        x: 100,
        y: 200
    }
}

console.log(isEqual(obj1, obj2))  // true
```

​	

##### split和join的区别

答：split是通过参数来对字符串进行划分返回一个数组，join是通过参数对数组进行连接成字符串

```js
'1-2-3'.split('-') // [1,2,3]
[1,2,3].join('-') // '1-2-3'
```



##### **数组API详解**

**1.关于push、pop、unshift、shift**

push：向数组最后添加一个新元素、返回添加元素后一个数组的长度、改变原数组，参数是新元素

pop：向数组最后删除一个元素，返回数组删除的元素，改变原数组，无参数

unshift：向数组前面添加一个新元素、返回添加元素后一个数组的长度、改变原数组，参数是新元素

shift：向数组前面删除一个新元素、返回数组删除的元素，改变原数组，无参数



**2.数组纯函数（不改变原数组，返回一个数组）**

*属于纯函数的有：*

concat：拼接一个数组，生成一个新的数组，不改变原数组，参数是新的数组

slice（切片）：截取一个数组，生成新数组，不改变原数组，参数是 start 和 end （不包括该元素）的 arrayObject 中的元素。

map和filter也是数组纯函数



*不属于纯函数的有：*

forEach、some、every（不返回数组）

splice（剪接）、push、pop、unshift、shift（改变原数组）



##### splice（剪接）和slice（切片）的区别

slice为纯函数（不改变数组和返回数组）、splice不是纯函数

```js
// array-api slice
const res = [1, 2, 3, 4, 5]

const res1 = res.slice(1, 3) // 从1开始切片，到3位置不包括3返回一个新数组 [2,3]
const res2 = res.slice() // 返回一个新数组 [1,2,3,4,5]
const res3 = res.slice(1) // 从1开始切片，到最后  [2,3,4,5]
const res4 = res.slice(-2) // 从末尾开始算，切倒前两个 [4,5]

// array-api splice
const arr1 = res.splice(1, 2, ['a', 'b', 'c']) // 从1开始删除2个，添加[a,b,c]数组 最后原数组结果：[1,[a,b,c],4,5]，返回结果：[2,3]
const arr2 = res.splice(0, 2) // 0开始删除两个，最后原数组结果:[3,4,5] arr2的结果：[1.2]
const arr3 = res.splice(0) // 从0开始剪掉到最后 改变原数组  最后原数组结果:[]   arr3:[1,2,3,4,5]
```



##### [1,2,3].map(parseInt)的结果是什么？为什么？

```js
//首先可以将其拆解为
const res = [1,2,3]
res.map((num,index)=>{
	return parseInt(num,index)
})

// 然后就是将参数传给parseInt一一进行返回新的数组， parseInt的第二参数是按多少进制解析第一个参数
parseInt(1,0) // 第二个参数传参为0，默认使用10进制 结果为1
parseInt(2,1) // 第二个参范围在2~36 结果为NaN
parseInt(3,2) // 二进制只有0和1 而3在二进制中不识别 结果为NaN

// 综上所述结果是[1,NaN,NaN]
```



##### get和post的区别

1、get一般用于查询操作，而post请求一般用于提交数据操作

2、get是在url后面拼接字符串发送请求（浏览器有限制url长度），post是把数据放在请求体发送请求（数据体积更大）

3、post更安全防止CSRF攻击

未完待续。。。



##### 闭包

参考：https://blog.csdn.net/weixin_43586120/article/details/89456183

```js
// 自由变量示例 —— 内存会被释放（不要混淆闭包作为参数被传递的示例，因为这里根本没有将函数作为参数传递）
let a = 0
function fn1() {
    let a1 = 100

    function fn2() {
        let a2 = 200

        function fn3() {
            let a3 = 300
            return a + a1 + a2 + a3  // a、a1、a2 在这里执行完才会被释放
        }
        fn3()
    }
    fn2()
}
fn1()

// 函数作为返回值--内存无法释放，返回的一个函数必须有a的参与
function create() {
     const a = 100
     return function () {
         console.log(a)
     }
 }

const fn = create()
const a = 200
fn() // 100

// 函数作为参数被传递--内存无法释放，作为参数的函数里面的a变量必须参与
function print(fn) {
    const a = 200
    fn()
}
const a = 100
function fn() {
    console.log(a)
}
print(fn) // 100

// 所有的自由变量的查找，是在函数定义的地方，向上级作用域查找
// 不是在执行的地方！！！
```



##### 阻止事件冒泡和默认行为

阻止默认行为：event.preventDefault()	

阻止事件冒泡：event.stopPropagation()和cancelBubble（）

阻止事件捕获：stopPagation（）



##### 解释jsonp的原理，为何不是真正的ajax？

答：首先jsonp是这样的(可以通过script标签传入值和改变值和改变回调函数的名字)，其次jsonp本质上利用的是script标签并没有用XMLHttpRequest对象，所以不是ajax

html文件

```js
<script>
window.abc = function(data){
    console.log(data)
}
<script src="http://localhost:8080/jsop.js?username=xxx&callback=abc"></sciprt>
</sciprt>
```

js文件

```js
abc({
	username: xxx
})
```



##### document load 和ready（DOMContentLoaded）的区别？

答：window.onload // 页面全部资源加载完成才会执行，包括图片视频
DOMContentLoaded // DOM渲染完成就立刻执行，此时图片，视频还可能没有加载完成



##### new Object() 和 Object.create()的区别

{}相当于new Object(),创建了之后会有Object.prototype原型

Object.create(null)创建是没有原型的，如果参数不是null，那么这个参数会作为新创建对象的原型



##### 手写一个trim方法去空格

```js
// trim方法去空格
String.prototype.trim = function(){
    return this.replace(/^\s+/,'').replace(/\s+$/,'')
}
```



##### 手写max函数

```js
function max() {
    const num = Array.prototype.slice.call(arguments)
    let max = 0
    num.forEach((n) => {
        if (n > max) {
            max = n
        }
    })
    return max
}
```



##### 什么是JSON?

答：JSON是一种数据交换格式，本质上是字符串（所以key值都要加双引号），可以用JSON.stringify转字符串、JSON.parse转对象

json里面的数据类型有：数字、字符串、布尔、对象、数组、null

##### 获取url参数的方式、手写将url参数解析为对象

1、location.search方法

2、new URLSearchParams()  新api

```js
// 1.将url参数解析为对象
function urlParse() {
    let res = {}
    let search = location.search.substr(1) // 去掉？号
    search.split('&').forEach(item => {
        let arr = item.split('=')
        let key = arr[0]
        let val = arr[1]
        res[key] = val
    })
    return res
}

// 2.将url参数解析为对象
function urlParse() {
    let res = {}
    let search = new URLSearchParams(location.search)
    search.forEach((val, key) => {
        res[key] = val
    })
    return res
}
```



##### 手写深度去重的函数flatten

```js
// 数组深度去重
function flatten(arrs) {
    const isFlatten = arrs.some((arr) => arr instanceof Array) // 只要一个匹配就会返回true

    if (!isFlatten) { // 表明已经拍平了
        return arrs // 直接返回一个数组
    }
	// 注意： Array.prototype.concat.apply([], arrs)只能处理二维数组 如：[1,3,[4,5]] 变成 [1,3,4,5] 里面再有数组就不行了
    const res = Array.prototype.concat.apply([], arrs) // 如果没有拍平又再次拍平
    return flatten(res) // 再次递归 深度去重
}

const arrs = [1, 2, [3, 4, [5, 6]]]
console.log(flatten(arrs))
```



##### ES5和ES6继承的区别

**ES5：**组合式继承，先创建子类的实例对象，然后再将父类的方法通过call方法添加到this上，通过原型和构造函数的机制来实现

示例：

```jsx
// 定义一个父类
function Parent() {
    this.name = '爸爸'
    this.age = 50
    this.sex = '男'
}
// 在父类的原型上添加一个方法
Parent.prototype.play = function () {
    console.log('去打麻将')
}

// 定义一个子类
function Child(name) {
    this.name = name
    // 第一步：继承父类的属性
    Parent.call(this)
}
// 第二步：实现子类继承父类的方法（儿子从爸爸那里学会了打麻将）
Child.prototype = new Parent()
// 第三步：找回丢失的构造函数
Child.prototype.constructor = Child
```

**ES6：**先创建父类的实例对象this（所以必须先调用父类的super()方法，然后在用子类的构造函数修改this），通过class关键字定义类，类之间通过extends关键字实现继承，子类必须在constructor方法中调用super方法。因为子类没有自己的this对象，而是继承了父类的this对象，然后对其加工，如果不调用super方法，子类得不到this对象。

示例：

```jsx
// 定义一个父类
class Parent {
    constructor(name, sex) {
        this.name = name
        this.sex = sex
    }
    play() {
        console.log(this.name + '打麻将超级垃圾')
    }
    speak() {
        console.log('性别是：' + this.sex)
    }
}
// 父类的实例化
let father = new Parent('老高','男')
father.play() // 打麻将超级垃圾
father.speak() // 性别是男

// 创建一个子类去继承父类
class Child extends Parent{
    constructor(name, sex) {
        // 调用父类的 constructor
        super(name, sex)
    }
}
// 子类的实例化
let son = new Child('小高', '女')
son.play()
son.speak()
```



##### 原生AJAX请求步骤

五步使用法：
 (1).创建XMLHTTPRequest对象
 (2).使用open方法设置和服务器的交互信息
 (3).设置发送的数据，开始和服务器端交互
 (4).注册事件
 (5).更新界面

Get请求：

```jsx
// 第一步：创建异步对象
let xhr = new XMLHttpRequest()
// 第二步：设置请求的url参数，参数1是请求的类型，参数2是请求的url，可以携带参数
xhr.open('get', '/baidu.com?username=1')
// 第三步：设置发送的数据，开始和服务端交互
xhr.send()
// 第四步：注册事件onreadystatechange，当状态改变时会调用
xhr.onreadystatechange = function () {
    if (xhr.readyState === 4 && xhr.status === 200) {
        // 第五步：如果到达这一步，说明数据返回，请求的页面是存在的
        console.log(xhr.responseText)
    }
}
```

POST请求：

```jsx
// 第一步：创建异步对象
let xhr = new XMLHttpRequest()
// post请求一定要添加请求头，不然会报错
xhr.setRequestHeader('Content-type', 'application/x-www-form-urlencoded')
// 第二步：设置请求的url参数，参数1是请求的类型，参数2是请求的url，可以携带参数
xhr.open('post', '/baidu.com')
// 第三步：设置发送的数据，开始和服务端交互
xhr.send('username=1&password=123')
// 第四步：注册事件onreadystatechange，当状态改变时会调用
xhr.onreadystatechange = function () {
    if (xhr.readyState === 4 && xhr.status === 200) {
        // 第五步：如果到达这一步，说明数据返回，请求的页面是存在的
        console.log(xhr.responseText)
    }
}
```



##### 关于事件委托

**什么是事件委托**
 事件委托也叫事件代理，就是利用事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。

**事件委托的作用**
 (1).提高性能：每一个函数都会占用内存空间，只需添加一个时间处理程序代理所有事件，所占用的内存空间更少；
 (2).动态监听：使用事件委托可以自动绑定动态添加的元素，即新增的节点不需要主动添加也可以具有和其它元素一样的事件。

**实现方式**
 我们先来看看，如果不用事件委托，需要绑定多个相同事件的时候是如何实现的：



```xml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    <li class="item">按钮</li>
    <li class="item">按钮</li>
    <li class="item">按钮</li>
    <li class="item">按钮</li>
</ul>
</body>
<script>
    window.onload = function () {
        let lis = document.getElementsByClassName('item')
        for (let i = 0; i < lis.length; i++) {
            lis[i].onclick = function () {
                console.log('用力的点我')
            }
        }
    }
</script>
</html>
```

不使用事件委托，那就要遍历每一个li元素，给每个li元素绑定一个点击事件，这样的做法非常耗费内存，如果有100个、1000个li元素，那对性能的影响是非常大的。

那么使用事件委托是怎么实现的呢？



```xml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul id="wrap">
    <li class="item">按钮</li>
    <li class="item">按钮</li>
    <li class="item">按钮</li>
    <li class="item">按钮</li>
</ul>
</body>
<script>
    window.onload = function () {
        let ul = document.getElementById('wrap')
        ul.onclick = function (ev) {
            // 获取到事件对象
            let e = ev || window.event
            // 如果点击的元素的calssName为item
            if (e.target.className === 'item') {
                console.log('用力的点我')
            }
        }
    }
</script>
</html>
```

这样一来，通过事件委托，只需要在li元素的父元素ul上绑定一个点击事件，通过事件冒泡的机制，就可以实现li的点击效果。并且通过js动态添加li元素，也能绑定点击事件。





##### 关于深拷贝和浅拷贝

**浅拷贝：**只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存

实现：

方法1：直接用=赋值

```bash
let obj1 = {a: 1}
let obj2 = obj1
```

方法2：Object.assign

```jsx
let obj1 = {a: 1}
let obj2 = {}
Object.assign(obj2, obj1)
```

方法3：for in循环只遍历第一层

```jsx
function shallowObj(obj) {
    let result = {}
    for (let key in obj) {
        if (obj.hasOwnProperty(key)) {
            result[key] = obj[key]
        }
    }
    return result
}
let obj1 = {
    a: 1,
    b: {
        c: 2
    }
}
let obj2 = shallowObj(obj1)
obj1.b.c = 3
console.log(obj2.b.c) // 3
```

**深拷贝：**
 方法1：用 JSON.stringify 把对象转换成字符串，再用 JSON.parse 把字符串转换成新的对象

```jsx
let obj1 = {
    a: 1,
    b: 2,
}
let obj2 = JSON.parse(JSON.stringify(obj1))
```

方法2：采用递归去拷贝所有层级属性

```jsx
function deepClone(obj) {
    // 如果传入的值不是一个对象，就不执行
    if (Object.prototype.toString.call(obj) !== '[object Object]') return
    // 根据传入值的类型初始化返回结果
    let result = obj instanceof Array ? [] : {}
    for (let key in obj) {
        if (obj.hasOwnProperty(key)) {
            // 如果obj是个对象，就递归调用deepClone去遍历obj的每一层属性，如果不是对象就直接返回值
            result[key] = Object.prototype.toString.call(obj[key]) === '[object Object]' ? deepClone(obj[key]) : obj[key]
        }
    }
    return result
}
// 改进判断对象的方法
console.log(typeof null === 'object') // true
console.log(Object.prototype.toString.call(null) === '[object Object]') // false
```

方法3：lodash函数库实现深拷贝

```bash
let obj1 = {
    a: 1,
    b: 2,
}
let obj2 = _.cloneDeep(obj1)
```

方法4：通过jQuery的extend方法实现深拷贝

```bash
let array = [1,2,3,4]
let newArray = $.extend(true,[],array) // true为深拷贝，false为浅拷贝
```

方法5：用slice实现对数组的深拷贝

```bash
let arr1 = ["1","2","3"]
let arr2 = arr1.slice(0)
arr2[1] = "9"
console.log(arr2) // ['1', '9', '3']
console.log(arr1) // ['1', '2', '3']
```

方法6：使用扩展运算符实现深拷贝

```bash
let obj1 = {brand: "BMW", price: "380000", length: "5米"}
let obj2 = { ...car, price: "500000" }
```





##### 箭头函数和普通函数的区别

普通函数：
 (1).this总是代表它的直接调用者
 (2).在默认情况下，没找到直接调用者，this指向window，可以用作构造函数，可以有argument对象
 (3).在严格模式下，没有直接调用者的函数中的this是undefined
 (4).使用call，apply，bind绑定，this指的是绑定的对象

箭头函数：
 (1).在使用 => 定义函数的时候，this的指向是定义时所在的对象，而不是使用时所在的对象，bind()、call()、apply()均无法改变指向
 (2).不能用做构造函数，也就是说不能使用new命令，否则就会抛出一个错误
 (3).不能使用arguments对象，但是可以使用…rest参数
 (4).不能使用yield命令
 (5).没有原型属性



##### 为什么js是单线程的呢？

JavaScript作为一门客户端的脚本语言，主要的任务是处理用户的交互，而用户的交互无非就是响应DOM的增删改，使用事件队列的形式，一次事件循环只处理一个事件响应，使得脚本执行相对连续。如果JS引擎被设计为多线程的，那么DOM之间必然会存在资源竞争，那么语言的实现会变得非常臃肿，在客户端跑起来，资源的消耗和性能将会是不太乐观的，故设计为单线程的形式，并附加一些其他的线程来实现异步的形式，这样运行成本相对于使用JS多线程来说降低了很多。



##### 浏览器Event-loop

参考： https://juejin.im/post/5c947bca5188257de704121d

​			https://juejin.im/post/5df631afe51d45581269a7b5#comment