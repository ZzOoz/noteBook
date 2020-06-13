# TypeScript知识总结

## 1、对于静态类型的理解

```js
// 这是一个静态类型
const count:number = 213
// 当我在使用count这个变量的时候可以使用number类型的方法,同理字符串、对象类型等也是如此
count.fixed()

## 静态类型

静态类型分为两种：基础类型和对象类型

```js
// 基础类型有number，string，void，symbol，boolean，void等
const count:number = 123
const teacher:string = 'hello'

// 对象类型有对象、类、数组、函数等

//对象
const teach:{
	name:string,
	age:number
} = {
	name:'李四'，
    age:12
}

// 类类型
class Person{}

const hyt:Person = new Person()

// 数组
const num:number[] = [1,2,3]

// 函数
const getTotal:()=>number = () => {
	return 123
}
```



## 类型注解和类型推断

**类型注解：我们自己通过定义类型来告诉TS变量是什么类型**

**类型推断：TS会自动去尝试分析变量得类型**

如果Ts能够自动分析变量类型，我们就什么都不需要做，但是如果Ts不能推断那么我们就需要使用类型注解了

```ts
// 这里可以通过类型注解判断类型
let count：number = 123
count = 123

// 这里通过类型推断判断
let countInfernence = 123

// 但是这里就不能通过类型推断来判断了,有可能是各种类型 
function getTotal(first,second){
	return first + second
}
getTotal(1,2)

// 需要这样
function getTotal(first:number,second:number){
	return first + second
}
getTotal(1,2)
```



## 函数返回值的相关类型

常见的有：string、number、never、boolean、void等

注意：never类型是指永远不能够执行到底的意思

```ts
// never类型
function errorEmitter():never{
	// 这里抛出错误了 无法执行到最后
	throw new Error()
	// 这里就不会执行了 因此是一个never类型
	console.log('1233')
}

function errorEmitter():never{
	// 这里无限循环了 无法执行到最后
	while(true){}
	// 这里就不会执行了 因此是一个never类型
	console.log('1233')
}

// 还有就是关于解构的函数类型, 这里如果要给结构的类型定义一个规范的格式就需要这样使用
function add({first,second}:{first:number,second:number}):number{
    return first + second
}
// 这里就可以判断结构里面的值的类型
console.log(add({first:1,second:2}))
// 这样会报错
console.log(add({first:"12",second:"2"}))
```

