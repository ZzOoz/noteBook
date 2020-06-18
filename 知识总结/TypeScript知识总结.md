# TypeScript知识总结



## 将js文件转化为ts文件

```js
// 首先npm
npm install typescript -g
// 后面使用tsc将js文件解析成ts文件
tsc demo.js

// 还有一种方法是下载ts-node
npm install ts-node -g
// 直接运行
ts-node demo.js
```



## 对于静态类型的理解

```js
// 这是一个静态类型
const count:number = 213
// 当我在使用count这个变量的时候可以使用number类型的方法,同理字符串、对象类型等也是如此
count.fixed()

## 静态类型

静态类型分为两种：基础类型和对象类型

​```js
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



## 数组和元组

数组：数组的定义方式

```js
// 1.这里定义一个数组，里面可以是字符串和数字
const arr:(number|string)[] = [1,'2','3']
// 2.这里定义一个字符串数组
const arr2:string[] = ['1','2']
// 3.这里定义一个undefined数组，下面有字符串是会报错的
const arr3:undefined[] = [undefined,'2']
// 4.这里也可以使用type类型别名
type Teacher = {
    name:string,
    age:number
}

const objArr: Teacher[] = [
    //这里就规定了要符合teacher的格式
    {name:'hyt',age:12},
    {name:'hhh',age:33}
]

// 5.使用class类,这样也是允许的
class Teacher{
    name: string;
    age:number;
}

const objArr2:Teacher[] = [
    new Teacher(),
    {
        name:'hhhh',
        age:12
    }
 
]
```

元组：元组是一个可以限制数组类型同时可以限制数组数量的方式，有组织的数组，你需要以正确的顺序预定义数据类型。

```js
// 这里硬性规定了数组里面只能是字符串、数字、数字
const teacherInfo:[string,number,number]=['hyt',12,33]
// 这里是一个二维数组
const teacher:[string,number,number][] = [
    ['hyt',12,33],
    ['hyt',12,33],
    ['hyt',12,33],
]
```



## Interface接口

### Interface接口和type类型别名

在Ts中，接口和类型别名是很相似的，但是也是有区别的，比如interface接口是不能代表基础类型，而type可以

```js
/1、
// 两者很相似
interface Person{
	name:string
}

type Person{
	name:string 
}

/2、
// 但是类型别名可以定义基础类型，而接口不可以
type Str = String

// 这样是会报错的
interface Str1 = String

/3、
// 继承方面
// 接口继承接口
interface Person{
    name:string,
}

interface man extends Person{
    age:12
}
    
// 接口继承类型别名
type hobby = {
    type:String
}    

interface woman extends hobby {
    age:13
}
    
// 类型别名继承接口
interface Person{
    name:String,
}

type a = Person & {
    age:12
}
    
// 类型别名继承类型别名
type hobby = {
    type:String
}
    
type b = hobby & {
    age:11
}
```



### Interface接口可选属性和只读属性

步骤：可选属性在之后面加上问好、只读属性在属性前加上readonly

```ts
interface Person{
	name:string,
	age?:number,
	readonly gender:boolean
}
```

### 类中使用interface接口和函数接口

```js
// 类中使用接口
interface Person{
	name:string,
	age?:number,
	readonly gender:boolean,
	say():string,
}

class A implements Person{
	name:'hyt'
	say(){
        return 'hello'
    }
}

// 定义函数接口
interface sayHi{
    // 意思时参数时string，返回值时string
    (str:string):string
}

const say:SayHi = (str:string) => {
    return str
}
```



### Interface接口的注意事项

**1.当将一个变量直接赋值给函数中会实行强校验，因此在不想使用强校验的情况下，请将变量先作为缓存，再将属性传进去**

```js
interface Person{
	name:String,
	age?:Number,
	readonly gender:Boolean
}

const getPersonName = (person:Person):void => {
	console.log(person.name)
}

const setPersonName = (person:Person,name:String):void => {
	person.name = name
}

// 我们多了一个hobby的属性，当我把值缓存在这里时 再传到方法中是不会报错的
const person = {
    name:'hyt',
    age:12,
    gender:false,
    hobby:'PE'
}

// getPersonName({
//     name:'hyt',
//     age:12,
//     gender:false,
//     // 直接这样作为变量传入时会报错的,因为这里会做强类型校验，
//     // 如果想让这个成功通过这个校验，在接口中定义[propsName:string]:stirng
//     // 意思时key为字符串，value为字符串
//     hobby:'PE'
// })

// 但是这样是不会报错，这里不会做强类型校验
getPersonName(person)

```



**2.当编译ts文件时，interface并不会转化成js语法，实际上只是ts将其作为提示的工具而已**



## 类型断言

定义：可以用来手动指定一个值的类型。

定义的方法有两种

```js
const str = 'this is a string'
// 使用as
const num:number = (str as string).length
// 使用尖括号
const num1:number = (<string>str).length
```



## 泛型



## TS中的类

### super关键字

ts中的super：ts中的super可以继承父类中的方法，如果想要获取到父类之前的方法或者属性，在子类中使用，那么可以使用super

```ts
class F{
    name = 'hyt';
    say(){
        return this.name
    }
}

class S extends F{
    say(){
    	// 覆盖了父类的方法，使用了super继承之前的方法加上自己的逻辑
        return super.say() + 'hello'
    }
}

const s = new S()
console.log(s.sayHi())
```



### Ts中类的访问类型（public、protected、private）

**public:允许我在类内和类外被调用**

**private：允许我在类内被调用，其余的不可以**

**protected：允许我在类内以及继承的子类中使用**

```js
class A{
    //**public:允许我在类内和类外被调用*
    public name:string
    //**private：允许我在类内被调用，其余的不可以**
    private age:number
    //**protected：允许我在类内以及继承的子类中使用**
    protected gender:boolean
}

class B extends A{
    say(){
        // name时public可以使用
        console.log(this.name);
        // age是private不可以使用
        console.log(this.age);
        // gender是protected可以使用
        console.log(this.gender)
    }
}

const a = new A()
// name时public可以使用
console.log(a.name)
// age是private不可以使用
console.log(a.age)
// gender是protected不可以使用
console.log(a.gender)
```



### Ts中类的constructor

**constructor会在实例化一个实例的时候会被调用比如：const a = new Person 会调用Person这个类的constructor**

```js
// 使用constructor的方法之一
// class A {
//     name:string
//     constructor(name:string){
//         this.name = name
//     }
// }
//还可以简化
class A{
    constructor(public name:string){
        this.name = name
    }
}

const a = new A('hyt')
console.log(a.name)
```

```js
 class A{
     constructor(public name:string){

     }
 }

 class B extends A{
    // 继承了父类之后使用构造器construcor后要使用super再次调用父类的构造器
    // 同时按照父类的的传参来传参
    constructor(public age:number){
         super('hyt')
     }
 }

 const b = new B(22)	
```



### getter和setter

**类中的getter作用是获取一个值，setter是改变一个值，他们的使用场景是当获取（改变）一个私有属性的时候我们可能不能再类外直接使用，那么可以借助getter和setter**

```ts
class H{
    // 这样做的好处是我们不能改变私有变量，但是可以通过改变get、set
    // 来间接改变_name 保证数据的安全
    constructor(private _name:string){}
    get name(){
        return this._name + 'gogo'
    }
    set name(name:string){
        this._name = name
    }
}

const h = new H('hyt')
// 这里调用的是类中的get name
console.log(h.name)
// 这里是调用类中的set name
h.name = 'yuteng'
console.log(h.name)
```



### 静态属性和方法static

**static是一个关键字、意思是当使用static的时候会将属性或者方法挂在类本身而不是实例**

使用static来实现一个单例模式

```ts
// 单例模式
class SingleMode{
    // 创建一个静态、私有的属性只有在类内使用并且挂在类上
    private static instance:SingleMode
    constructor(public name:string){}

    // 新建实例的时候使用这个方法判断是否已经创建
    static getInstanceMode(){
        if(!this.instance){
            this.instance = new SingleMode('yuteng')
        }
        return this.instance
    }
}

const Single = SingleMode.getInstanceMode()
const s1 = SingleMode.getInstanceMode()
console.log(s1 === Single)
console.log(Single.name)
```



### 抽象类abstruct

1、首先抽象类是不能实例化的，它仅仅是一个抽象的东西，具体不知道如何实现，需要在他所继承的子类中得到实现

2、抽象类中有抽象方法，抽象方法也是不能够得到具体实现，但是每当子类继承了抽象类之后需要强制实现这个抽象方法

3、抽象类中除了抽象方法之外还可以实现一些其他的方法和属性，这些都不是抽象的

```ts
// 抽象类是将一些共有的属性和方法收集，如果是一些方法是对于每个子类共有
// 但是具体实现要分别在他们其中具体实现就需要使用抽象类
abstract class AFunc{
    width:number
    getType(){
        return 'AFunc'
    }
    // 抽象方法中不能有具体实现
    abstract getArea():number
}

class BFunc extends AFunc{
    // 子类所继承的抽象类要实现具体的抽象方法
    getArea(){
        return 123
    }
}

const bb = new BFunc()
```



## TypeScript的编译运转过程的进一步理解

如果我们想要将自己的ts文件打包成js文件上传使用，那么可以在package.json中新增build命令，运行

```js
"build":"tsc" // 将所有ts文件编译成js文件
```

如果想要编译到自己想要的目录那么可以在tsconfig.json文件中找到outDir修改，例如修改

```js
"outDir": "./build",                        /* Redirect output structure to the directory. */
```



tsconfig.json文件详解

注意：当我们运行tsc命令+文件名的时候是不会走tsconfig.json的配置的，只有我们运行直接运行tsc命令的时候才会走这个配置，但是值得一提的是，如果使用ts-node命令+文件名运行的话，会走tsconfig.json这个文件的配置

```js
{
//                   /*当我们想要使用tsc运行某些特定的文件时那么就可以使用include，意思是运行当前目录的demo.ts文件变异成js  当然也可以使用exclude和file来除去不需要编译的文件和想要编译的文件*/    
  "include":["./demo.ts"], 
  "compilerOptions": {
      "removeComments": true  // 编译时去除注释
      "noImplicitAny": true,  // 是否需要显式的类型注解any，如果是true,那么在不明确指定any类型会报错
  	  "strictNullChecks": true,  // 是否强制对null进行校验检查，是就会检查
 	  "outDir": "./",    // 是指运行tsc命令后输出的文件位置
       "rootDir": "./",  // 是指使用tsc命令是编译的文件位置在哪里	
       "checkJs": true,  // 是否允许对js文件进行检测
   	   "allowJs": true,  // 是否允许想ts文件一样对js文件进行打包编译（es6转es5，默认打包target是es5）
       "noUnusedLocals": true,     // 是否对未被使用的变量进行提示
        "noUnusedParameters": true, // 是否对未被使用的函数参数进行提示
       "sourceMap":true // 开启sourceMap
   }
}

```



## 联合类型和类型保护

```ts
interface Bird{
    fly:boolean,
    eat:()=>{}
}

interface Dog{
    fly:boolean,
    bark:()=>{}
}

// 当我们使用联合类型Bird | Dog的时候，通过使用animal是无法使用bark和eat方法的
// 因为ts无法确认animal是什么类型

// 1.那么我们可以使用类型断言的方式进行类型保护
function getAnimal(animal:Bird | Dog){
    if(animal.fly){ 
        // 如果fly为true，那么类型断言成Bird
        (animal as Bird).eat()
    }else{
        // 否则就是Dog类型
        (animal as Dog).bark()
    }
}

// 2.使用in语法进行类型保护
function getAnimal2(animal:Bird | Dog){
    // 如果eat方法在animal中
    if("eat" in animal){ 
        // 那么类型断言成Bird
        (animal as Bird).eat()
    }else{
        // 否则就是Dog类型
        (animal as Dog).bark()
    }
}

// 3.使用typeof的方法进行类型保护
function add(first:number | string,second:number | string){
    if(typeof first === 'string' && typeof second === 'string'){
        return first + second
    }
}

// 4.使用instanceof的方法进行类型保护

// 只有类才有instanceof的语法 interface是没有的
class NumObj {
    count:number
}

function addSecond(first:object | NumObj,second:object | NumObj){
    if(first instanceof NumObj && second instanceof Number){
        return first.count + second.count
    }
    return 0
}
```



## 枚举类型

使用场景：对于多个状态的不同推断可以使用枚举类型

```ts
enum Status{
    // 枚举默认从0开始，如果改变了一个值得枚举数字，那么就会从这个改变的开始递增
    onLine,
    offLine,
    deleted
}

// 这样可以反推出枚举的值
console.log(Status[0])

function getResult(status){
    if(status === Status.onLine){
        return 'onLine'
    }else if(status === Status.offLine){
        return 'offLine'
    }else if(status === Status.deleted){
        return 'deleted'
    }
}

// 这样也可以
console.log(getResult(0))
// 这样也可以
console.log(getResult(Status.onLine))
```



## 泛型

**泛型是指泛指的类型，通常用来复用多种不同的类型**



### **函数泛型**

```ts
// 泛型
function join<T>(first:T,second:T){
    return `${first}${second}`
}

// params是一个数组，里面的格式是泛型约定的
function map<T>(parmas:T[]){
    return parmas
}

// 同时还可以定义多种泛型
function each<T,P>(first:T,second:P){
    return `${first}${second}`
}

// 通过制定T中的类型是string 来确定first和second的类型（他们都是用来T作为他们的类型）
join<string>('1','2')
join<number>(1,2)

// 当我们不指定泛型的时候ts会进行类型推断
join('1','2')

// 使用map函数
map<string>(['123'])

// 定义多个泛型
each<string,number>('123',123)



// 如何使用泛型作为一个具体的类型注解
// 首先<T>(params:T) => T是一个类型注解 是使用了T泛型这个概念
// 其次等号后面是函数的具体实现
const func:<T>(params:T) => T = <T>(params:T)=>{
    return params
}

// 相当于
function Hello<T>(params:T):T{
    return params
}

const func1:<T>(params:T) => T = Hello
```



### 类的泛型

```ts
// 类泛型

interface Item{
    name:string
}

// T extends Item 的意思是T要继承item里面的所有的东西
class DataManager<T extends Item>{
    constructor(private data:T[]){}
    getItem(index:number):string{
        return this.data[index].name
    }
}
```



## 使用parcel打包Ts代码

首先使用npm安装parcel的包

```js
npm i parcel@next
```

后面在package.json中加入命令

```js
"dev":"parcel ./src/index.html" // 使用parcel运行index.html
```

后面在index.html中引入demo.ts后就可以直接使用npm run dev来跑一个服务器了 