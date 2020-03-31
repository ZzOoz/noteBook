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



##### **typeof能判断的类型**

答：值类型：string、number、boolean、symbol、undefined

​									  引用类型：object（仅仅只能判断是否是引用类型无法判断是否为对象或数组）

​									  其他：function



##### **列举强制转换和隐式类型转换**

答：1.强制类型转换：parseInt、parseFloat、toString等

​		2.隐式类型转换：if、==、字符串拼接、逻辑运算if/else



##### **写一个深度比较的函数isEqual**

要求：obj1和obj2有相同的属性就可以为true，不对引用指针作比较

```js
// 深度比较函数
function isObject(obj) {
    return typeof obj === 'object' && typeof obj !== null
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

shift：向数组最后删除一个新元素、返回数组删除的元素，改变原数组，无参数



**2.数组纯函数（不改变原数组，返回一个数组）**

*属于纯函数的有：*

concat：拼接一个数组，生成一个新的数组，不改变原数组，参数是新数组

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
const arr1 = res.splice(1, 2, ['a', 'b', 'c']) // 从1开始到2（不包括） 补充新元素
const arr2 = res.splice(0, 2) // 0开始到2（不包括），不补充新元素  [1,2]
const arr3 = res.splice(0) // 从0开始剪掉到最后 改变原数组  [1,2,3,4,5]
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

阻止事件冒泡：event.stopPropagation()



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

