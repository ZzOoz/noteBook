# Vue知识总结

## Vue-cli基本使用

```js
//创建新项目
// 基于交互式命令行的方式创建新版vue项目
vue create my-project

// 基于图形化界面的方式创建新版vue项目
vue ui

// 基于2.x的旧模式创建旧版vue项目
npm install -g @vue/cli-init
vue init webpack my project
```



## Vue

# A、基本使用

#### 1.模板（插值、指令）

#### 2.computed（计算属性）和watch（观察动作）

​	**2.1、区别：computed是有缓存的，data不变就不会重新计算，且在v-model双向绑定的时候要同时写get和set函数**

**computed（计算属性）**

```js
data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
  fullName：{
   get(){//回调函数 当需要读取当前属性值是执行，根据相关数据计算并返回当前属性的值
      return this.firstName + ' ' + this.lastName
    },
   set(val){//监视当前属性值的变化，当属性值发生变化时执行，更新相关的属性数据
       //val就是fullName的最新属性值
       console.log(val)
        const names = val.split(' ');
        console.log(names)
        this.firstName = names[0];
        this.lastName = names[1];
   }
   }
 }
```



**watch观察动作**

```js
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
     firstName: function (val) {
     this.fullName = val + ' ' + this.lastName
  },
  lastName: function (val) {
     this.fullName = this.firstName + ' ' + val
   }
   }
```

​	



**总结一下：**

##### computed特性

1.是计算值，
2.应用：就是简化tempalte里面{{}}计算和处理props或$emit的传值
3.具有缓存性，页面重新渲染值不变化,计算属性会立即返回之前的计算结果，而不必再次执行函数

##### watch特性

1.是观察的动作，
2.应用：监听props，$emit或本组件的值执行异步操作
3.无缓存性，页面重新渲染时值不变化也会执行

​	**2.2、watch如何进行深度监听？**

```js
//监听obj对象的变化
watch: {
  obj: {
    handler(newName, oldName) {
    console.log('obj.a changed');
  },
    // 代表在wacth里声明了obj这个方法之后立即先去执行handler方法
    immediate: true,
    // 深度监听
    deep: true
  }
} 
```

​	**2.2.1、监听对象中的单个属性的变化**

```js
    // 监听obj中的a属性的变化
    watch: {
      'obj.a': {
        handler(newName, oldName) {
          console.log('obj.a changed');
        },
        immediate: true,
        // deep: true
      }
    } 
```

​	**2.3、注意：watch监听引用类型，拿不到oldval，当需要监听复杂引用类型时要用深度监听**

#### 3.class和style

##### **3.1、在vue中绑定样式的两种方式?**

答：v-bind:class和v-bind:style

##### **3.2、具体使用**

```js
<template>
  <div>
    <!-- 1.绑定a属性的值，a的值在data中 -->
    <div :class="a">class1</div>
    <!-- 2.Active的存在取决于 isactive的值，为true就说明存在，为false就说明不存 在。 -->
    <div :class="{active:isActive}">class2</div>
    <!-- :class = [“a”,”b”] 
    在数组中元素带不带引号的问题。带了说明是类名，不带就是data中      的属性对应的值-->
    <div :class="['active',a]">class3</div>
    <!-- 1. 注意 style中是一个对象，中间用，隔开。前者是style属性，值是data      中对应的数值 -->
    <div :style="styleData">style1</div>
  </div>
</template>

<script>
export default {
    data(){
        return{
            a:"class1",
            isActive:true,
            styleData:{
                color:'yellow',
                fontSize:"100px"
            }
        }
    }
}
</script>
<style lang="scss">
.class1{
    color:red
}
.active{
    font-size:50px;
}
</style>

```



#### 4.条件

##### **4.1、v-if和v-show的区别和使用场景？**

答：

**区别**：v-if每次条件为true的时候都将dom节点会重新渲染，然后将条件为false的节点销毁，而v-show则是会将所有节点都渲染，在切换的时候只不过将条件为false的节点用display:none隐藏掉

**使用场景**：v-if适用于切换不太频繁的场景，如果切换太频繁了尽量使用v-show

#### 5.循环

##### **5.1、首先注意：v-if和v-for两个指令在同一个标签中是不能同时使用的，原因是因为v-for的优先级高，那么就是在遍历多少个，v-if就要执行多少次，这是不允许的**

##### **5.2、遍历数组和对象**

```js
<template>
  <div>
    <!-- 遍历数组 -->
    <div v-for="(item,index) in array" :key="item.id">
        {{index}}--{{item.name}}--{{item.id}}
    </div>

    <!-- 遍历对象 -->
    <div v-for="(val,key,index) in obj" :key="key">
        {{key}}--{{val.name}}--{{index}}
    </div>
  </div>
</template>

<script>
export default {
    data(){
        return{
            array:[
                {id:1,name:'li',age:12},
                {id:2,name:'li',age:12},
                {id:3,name:'li',age:12},
            ],
            obj:{
                a:{name:"aaa"},
                b:{name:"bbb"},
                c:{name:"ccc"}
            }
        }
    }
}
</script>
<style lang="scss">
.class1{
    color:red
}
.active{
    font-size:50px;
}
</style>

```

#### 6.事件

##### **6.1、vue中的event对象**

​		***6.1.1、event是原生的且事件是被挂载到当前元素***

​		参考文章：https://www.cnblogs.com/gitByLegend/p/10836922.html

```js
<template>
  <div>
 	  // 与demo2(222,$event) 一样
      <div @click="demo(222)">ggogg1</div>
      <!-- 想要及传入参数又有event对象可以在括号里面加上$event就可以了，当然也可以不传，绑定window.event-->
      <div @click="demo2(222,$event)">ggogg2</div>
      <!-- 当绑定事件后面加上圆括号时，默认不传入event对象 -->
      <div @click="demo3()">ggogg3</div>
      <!-- 在绑定事件不加上圆括号时，默认传入event对象 -->
      <div @click="demo4">ggogg4</div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  methods:{
    demo(val){
        console.log(window.event === event) // true
        console.log(typeof event)  // object
    },
    demo2(val,event){
        console.log(event) // 具体信息
        console.log(window.event === event) // true
        console.log(typeof event)  // object
    },
    demo3(event){
        console.log(event) // undefined
        console.log(window.event === event) // false
        console.log(typeof event)  // undefined
    },
    demo4(event){
        console.log(event) // 具体信息
        console.log(window.event === event) // true
        console.log(typeof event)  // object
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

```

##### **6.2.事件修饰符**

参考：https://www.cnblogs.com/xuqp/p/9406971.html

参考：https://blog.csdn.net/microopithecus/article/details/82863592

**6.2.1、stop：阻止事件冒泡、capture：捕获事件，从外到内触发事件、prevent：阻止默认行为、self：只会触发自己范围内的事件，不包含子元素、once：只会触发一次**

**6.2.2、用`v-on:click.prevent.self`会阻止所有的点击，而`v-on:click.self.prevent`只会阻止对元素自身的点击。**

```js
    <div @click="alert(1)">
      <a href="/#" @click.prevent.self="alert(2)">
        <div @click="alert(3)"></div>
      </a>
    </div>
```

点击div3，会alert3,alert1。不但阻止了alert(2)，还阻止了a的默认跳转。

因为点击的时候会先prevent，阻止默认事件，阻止了跳转;然后判断是否是self，因为点击到的是div3，所以不是self，阻止了alert(2)。

```js
    <div @click="alert(1)">
      <a href="/#" @click.self.prevent="alert(2)">
        <div @click="alert(3)"></div>
      </a>
    </div>
```

点击div3，会alert3,alert1,跳转到/#。只阻止了alert(2)。

因为会先判断self，点击到div3，不是self，所以不会执行click事件，就不会执行 阻止默认事件和alert(2) ，所以可以跳转



6.2.3、**native修饰符**：就是在父组件中给子组件绑定一个原生的事件，就将子组件变成了普通的HTML标签，不加'. native'事件是无法触 发的。

组件绑定事件时

1. 普通组件绑定事件不能添加.native, 添加后事件失效

2. **自定义组件**绑定事件**需要添加.native**, 否则事件无效

```js
    <div class="container" >
        <!--原生事件的根部原生-->
        <my-kirin-local @click.native="show(event)" :son_data=" parent_data">（这里取消.native，事件无效）这里的内容会被替换成template内容，如果v-on:事件后加了.native方法，事件会有效（因为是事件的根部--原生事件）</my-kirin-local>
        <p>
            <!--普通html标签-->
            <buton @click = "show(event)" >（这里不加.native，事件有效）这是一个普通的html标签，如果v-on:事件后加了.native方法，反而就失去效果，因为不是原生事件</buton>
        </p>
    </div>
    
    const app =  new Vue({
        el:".container",
        data:{
            parent_data:["first","secend"],
        },
        methods: {
            show:function() {
                console.log("show");
            }
        },
});

```



#### 7.表单

##### 7.1、v-model

##### **7.2、表单修饰符（lazy、trim、number）**

​	参考：https://www.cnblogs.com/mmzuo-798/p/9301121.html

​	7.2.1、lazy:使用了这个修饰符将会从“input事件”变成change事件进行同步

​	7.2.2、number

首先谁明这个number并不是限制用户的输入,而是将用户输入的数据尝试绑定为 js 中的 number 类型

举个例子，如果用户输入`300`，data 中绑定的其实是`'300'`(string)，添加 number 指令后可以得到 `300`(number)的绑定结果。
而如果用户输入的不是数字，这个指令并不会产生任何效果。

​	7.2.3、trim可以用来过滤前后的空格

# B、组件

#### 1.生命周期

**1.1、created和mounted的区别：**

答：created生命周期代表vue的实例已经初始化完成，但是还没有被渲染到页面上，而mounted是表示页面已经被渲染完成了，也可以操作dom了，通常可以用来ajax请求

**1.2、beforeDestory可以做的事情：**

答：可以销毁子组件、销毁自定义事件（event.$off）不去占用内存

#### 2.组件间的通信

参考文章:https://segmentfault.com/a/1190000019208626#item-6

**2.0、父子组件间通信（$parent和$children）**

**2.1、父子组件间通信（prop和$emit）**

**2.2、兄弟组件间通信（event.$emit和event.$on）**

**介绍**：这里实现通过在一个js文件中初始化一个新的实例(new Vue)将其导出，然后通过引用这个实例调用$on（监听事件的组件同时接收值）和$emit（需要触发事件的组件同时可以传值）互相传值

**示例**：

index.js文件

```js
import Vue from 'vue'

export default new Vue()
```



```js
// bro1.vue组件
<template>
  <div>
      <div>我是bro1组件 {{title}}</div>
  </div>
</template>

<script>
import Event from './index'
export default {
  name: 'bro1',
  data(){
      return{
          title:""
      }
  },
  mounted(){
      Event.$on('brotherEvent',(val)=>{
          this.title = val
          console.log(val)
      })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

```



```js
// bro2.vue组件
<template>
  <div>
    <div>我是bro2组件</div>
    <button @click="sendEventFromBro2">bro2</button>

  </div>
</template>

<script>
import Event from './index'
export default {
    name: 'bro2',
    data(){
        return{
            title:'我是bro2组件的title'
        }
    },
    methods:{
      sendEventFromBro2(){
          Event.$emit('brotherEvent',this.title)
      }
  } 
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

```

引用bro1和bro2组件的主组件**(点击bro2的事件会触发bro1的监听成功将title值变成bro2传过去的值)**

```js
<template>
  <div>
  <bro1 /> 
  <bro2 />
  </div>
</template>

<script>
import bro1 from '../components/EventBus/bro1.vue'
import bro2 from '../components/EventBus/bro2.vue'

export default {
  name: 'EventBus',
  components: {
    bro1,
    bro2
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

```

**2.3、跨级组件通信（$attr和$listener）**

**介绍**： $attrs--继承所有的父组件属性（除了组件内prop属性、class 和 style ）

​			inheritAttrs：默认值true,继承所有的父组件属性（除props的特定绑定）作为普通的HTML特性应用在子组件的根元素上，如果你不希望组件的根元素继承特性设置inheritAttrs: false,但是class属性会继承

​			$listeners--属性，它是一个对象，里面包含了作用在这个组件上的所有监听器（不含.native修饰符），你就可以配合 `v-on="$listeners"` 将所有的事件监听器指向这个组件的某个特定的子元素。

**示例**：

```js
// index.vue
<template>
  <div>
    <h2>我是index.vue</h2>
    <child-com1
      :foo="foo"
      :boo="boo"
      :coo="coo"
      :doo="doo"
      title="前端工匠"
	  @Event="sendEvent" 
    ></child-com1>
  </div>
</template>
<script>
const childCom1 = () => import("./childCom1.vue");
export default {
  components: { childCom1 },
  data() {
    return {
      foo: "Javascript",
      boo: "Html",
      coo: "CSS",
      doo: "Vue"
    };
  },
  methods:{
      sendEvent(){
          console.log("我是祖先组件传来的事件")
      }
  }
};
</script>
```

childCom1.vue中

```js
// childCom1.vue
<template class="border">
  <div>
    <p>foo: {{ foo }}</p>
    <p>childCom1的$attrs: {{ $attrs }}</p>
    <child-com2 v-bind="$attrs"></child-com2>
  </div>
</template>
<script>
const childCom2 = () => import("./childCom2.vue");
export default {
  components: {
    childCom2
  },
  inheritAttrs: false, // 可以关闭自动挂载到组件根元素上的没有在props声明的属性
  props: {
    foo: String // foo作为props属性绑定
  },
  created() {
    // 注意这里已经没有foo属性了，foo已经是被childCom1当作prop属性了
    console.log(this.$attrs); // { "boo": "Html", "coo": "CSS", "doo": "Vue", "title": "前端工匠" }
  }
};
</script>
```

childeCom2.vue

```js
// childCom2.vue
<template>
  <div class="border">
    <p>boo: {{ boo }}</p>
    <p>childCom2: {{ $attrs }}</p>	
    <!-- 这里要绑定listeners事件就可以监听所有的祖先事件啦 -->
    <child-com3 v-bind="$attrs" v-on="$listeners"></child-com3> 
  </div>
</template>
<script>
const childCom3 = () => import("./childCom3.vue");
export default {
  components: {
    childCom3
  },
  inheritAttrs: false,
  props: {
    boo: String
  },
  created() {
  // 注意这里已经也没有boo属性了
    console.log(this.$attrs); // {"coo": "CSS", "doo": "Vue", "title": "前端工匠" }
  }
};
</script>
```

childCom3.vue

```js
// childCom3.vue
<template>
  <div class="border">
    <p>childCom3: {{ $attrs }}</p>
  </div>
</template>
<script>
export default {
  props: {
    coo: String,  // 成功传到
    title: String // 成功传到
  },
  methods:{
	demo(){
		this.$emit("Event") // 触发祖先组件的事件
    }
  }
};
</script>
```

**2.4、跨级组件通信（$provide和$inject）**

**介绍**：以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。换句人话：祖先组件中通过provider来提供变量，然后在子孙组件中通过inject来注入变量

2.4.1、$provide和$inject需要成对使用，同时在后代组件inject注入后改变祖先组件的属性后是不会响应式改变值的

2.4.2、



## Vue-router

## Vuex