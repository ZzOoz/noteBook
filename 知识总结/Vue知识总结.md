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

# C、Vue高级特性

### 1、自定义v-model

### 2、$nextTick

**介绍**：Vue是异步渲染的，即data改变之后DOM不会立刻渲染，$nextTick会在DOM渲染之后被触发，以获取最新的dom节点

2.1、vue组件更新之后如何获取最新的DOM?

答：通过nextTick

2.2、例子：

```js
<template>
  <div>
      <ul ref="ulList" >
          <li v-for="(item,index) in list" :key="index">{{item}}</li>
      </ul>
      <button type="button" @click="insert">插入</button>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data(){
      return {
          list:[
              {name:'a'},
              {name:'b'},
              {name:'c'}              
          ]
      }
  },
  methods:{
      insert(){
          this.list.push({name:'d'})
          this.list.push({name:'e'})
          this.list.push({name:'f'})
          
          // 因为vue是异步渲染的，如果用这种方法是获取不到最新dom的ulList的长度的
          // 此时dom还没有更新
          //  let ulList = this.$refs.ulList.childNodes.length

          // 要用nextTick方法等到dom更新在计算
          // 此时就可以拿到最新的dom,无论多少次更新data，vue的重新渲染都只会有一次
          this.$nextTick(()=>{

              let ulList = this.$refs.ulList.childNodes.length
              console.log(ulList,'长度')
          })
          
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

```



### 3、slot（插槽）

**介绍**：通俗的说就是父组件想往子组件插入内容，那么就会通过插槽来插入内容到子组件

##### **3.1、普通插槽**

**介绍**：当子组件内加入slot标签（内容自定义）当父组件在子组件内没有插入内容就会使用子组件自己slot标签的默认内容，当父组件有内容插入进去那么就会使用父组件插入的内容

**例子**：

```

```



##### 3.2、作用域插槽

**介绍**：当父组件需要使用子组件的数据来显示在父组件本身时，就要用到作用域插槽

**例子**：

父组件

```js
<template>
  <div>
      <ScopeSlot>
          <template v-slot="slotProps">
              <!-- 获取子组件的data中的sonlist的name -->
              <!-- 此时已经引入name属性 -->
              {{slotProps.slotData.name}}
          </template>
      </ScopeSlot>
  </div>
</template>

<script>
import ScopeSlot from '../components/SlotDemo/ScopeSlot'
export default {
  name: 'ScopeSlot12',
  components:{
      ScopeSlot
  },
  data(){
      return {
          list:{
              name:'我是父组件',
              number:12,
              scope:1
          }
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

```



子组件











##### 3.3、具名插槽

介绍：子组件内有各个名字的组件，父组件可以调用相应的名字嗲用子组件的插槽

例子：

父组件调用子组件的插槽名

```js
<template>
  <div>
      <NameSlot>
          <div name="header"></div>
          <div></div>
          <div name="footer"></div>

      </NameSlot>
  </div>
</template>

<script>
import NameSlot from '../components/SlotDemo/NameSlot'
export default {
  name: 'NameSlot1',
  components:{
      NameSlot
  },
  data(){
      return {

      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

```

子组件

```js
<template>
  <div>
      <slot name="header">我是头部</slot>
      <div>我是具名插槽子组件</div>
      <slot name="footer">我是底部</slot>

  </div>
</template>

<script>
export default {
  name: 'NameSlot'
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

```



### 4、动态&异步组件

##### **4.1、动态组件**

**介绍**：vue提供了一种componet标签绑定is并且切换属性值可以动态更换不同类型的组件

**使用场景**： 1.:is="component-name" 用法

​					2.需要根据数据，动态渲染的场景，即组件类型不确定

**例子**：

```js
<template>
  <div>
      <!-- 结果渲染出home、about、EventBus组件 -->
      <div v-for="(item,key) in componentsData" :key="key">
          <!-- 动态组件遍历componentsData通过item来动态切换组件 -->
          <component :is="item.type" />
      </div>
  </div>
</template>

<script>
import Home from './Home'
import About from './About'
import EventBus from './EventBus'

export default {
    components:{
        About,
        Home,
        EventBus
    },
    data(){
        return {
            componentsData:{
                1:{type:"Home"},
                2:{type:'About'},
                3:{type:'EventBus'}
            }
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>

```

##### 4.2、异步组件

参考：https://segmentfault.com/a/1190000012138052（建议看）

**介绍**：当需要加载一个比较大的组件或者说想要在使用时才加载组件时可以使用异步组件加载的方式

**使用**：1.import()函数、2.按需加载，异步加载大组件

**简单例子**：

```js
//按需异步加载组件
<template>
  <div>
    <div>按需异步加载组件</div>
    <Home v-if="showFlag"/>
    <button type="button" @click="show">显示加载异步组件</button>
  </div>
</template>

<script>
 // 我们通常使用这种方式来加载组件的
 // 但是当组件太大或者是想在必要的时候加载那么就可以使用异步加载的方式
// import Home from './Home'  
export default {
  name: 'AsyncComponents',
  components:{
      // 这种就是异步加载的方式
      Home:()=>import('./Home')
  },
  data(){
      return {
          showFlag:false
      }
  },
  methods:{
      show(){
          this.showFlag = !this.showFlag
      }
  }
}
</script>

```



### 5、keep-alive

**介绍**：keep-alive是一种缓存组件的方式，可以对已有组件的状态保存（在切换到下一个组件的时候）

**使用**：可以在例如tab组件切换的时候使用

**例子**：



### 6、mixin

**介绍**：多个组件有相同的逻辑，可以抽取出来，同时可以简化代码，在组件mixins选项中可以加入多个mixin

**例子**：

mixin.js(多个组件中的共同属性)

```js
export default {
    data() {
        return {
            city: '中山'
        }
    },
    methods: {
        showName() {
            alert(this.name)
        }
    }
}
```

mixin组件

```js
<template>
  <div>
      <div>
          {{name}}--{{age}}--{{city}}
      </div>
      <button type="button" @click="showName">显示名称</button>
  </div>
</template>

<script>
import Mixin from './mixin'
export default {
    mixins:[Mixin],// 可以引入多个mixin
    data(){
    return{
        // 此处加入mixin之后已经有city属性了以及showName方法
        name:'hyt',
        age:12
    }
  }
}
</script>
```

# D、vue原理

### 1、组件化和MVVM

##### **1.1、组件化是什么？**

**答**：组件化的意思时将一个项目（代码）划分多个组件，通过组件的任意组合来实现项目的功能

##### 1.2、MVVM为什么会出现？是什么？

**1、为什么会出现？**

在很久以前就已经有组件化的概念，但是那时候在模板渲染完成之后页面数据就是如此了，如果要更新数据就需要操作DOM，但是因为操作DOM是一件非常消耗性能的事情，同时处理起来很难有条理，那么就诞生了数据驱动视图的概念，意思就是不用去关注dom操作，我们只需要关注数据的变化来改变视图的显示，于是就出现了vue、react等框架，他们的能够实现数据驱动视图的原理时MVVM（vue）、setData（react）

**2、是什么？**

MVVM是一种实现数据驱动视图的一种方式，M是model层（可以看作数据data）、V是视图层（可以看做页面dom等）、VM是视图模型（可以看作是连接M和V的一种桥梁，例如methods、事件、mixin等）

### 2、响应式原理（这是实现数据驱动视图的第一步）

##### 2.1、响应式是什么？

答：组件data数据一旦发生变化，会立即触发视图的更新

##### 2.2、实现响应式原理的核心api是什么？

答：Object.defineProperty方法，Vue3中使用Proxy

##### 2.3、Object.defineProperty方法实现响应式

###### 2.3.1、监听对象，监听数组、深度监听

```js
// 简单实现vue响应式原理

let data = {
    name: 'hyt',
    age: 12,
    list: ['a', 'b', 'c'],
    city: {
        provice: '广东'
    }
}

// 视图更新类方法
function updateView() {
    console.log('视图更新')
}

// 深度监听数组的方法
let arrayProto = Array.prototype
    // 新创建一个对象，但是在扩展原型上同名的方法是不会影响原有的
let newArray = Object.create(arrayProto);
['push', 'pop', 'unshift', 'shift'].forEach(methodName => {
    newArray[methodName] = function() {
        updateView();
        // 这里实质上是Array.prototype.原型方法.call(this,arguments)
        arrayProto[methodName].call(this, ...arguments)

    }
})

function observer(target) {
    // 如果不是对象且不是null 直接返回值 这里只是实现监听对象和监听数组
    if (typeof target !== 'object' && target !== null) {
        return target
    }

    // 如果是数组直接将原型付给target
    if (Array.isArray(target)) {
        target.__proto__ = newArray
    }

    console.log(target)
        // 遍历循环target获取key同时也可以获取值
    for (const key in target) {
        // 如果是新增属性这里是监听不到的 因为实在调用observer方法在先 
        // console.log(key)
        // 对target中的数据进行监听

        defineReactive(target, key, target[key])
    }
}

// 核心api 实现响应式
function defineReactive(target, key, value) {
    // 如果要监听对象需要再次递归调用observer方法
    // 如果不使用observer方法 如例子一样监听这里只是到了data.city这里是没有办法到达data.city.province
    observer(value) // 因此在这里调用observer递归到最底部

    Object.defineProperty(target, key, {
        get: function() {
            return value
        },
        set: function(newVal) {
            // 这里也需要监听因为 set的时候又可能是data.age = {num:12}
            if (newVal !== value) {
                observer(key)
                value = newVal;
                // 设置完成后更新视图
                updateView()
            }

        }
    })
}


//调用监听data的方法
observer(data)

// 1.监听完成后修改数据 仅仅是已经存在的值
data.name = 'tyh';
data.age = 100

// 2.新增属性、删除属性是不能发生视图更新
// 这时候就需要使用Vue.set 和 Vue.delete
data.gender = 1

// 3.深度监听对象
data.city.provice = '广东东'

// 4.深度监听数组
data.list.push('d')
console.log(data.list)
```



###### 2.3.2、几个缺点

​	1.深度监听，需要递归到底，一次性计算量大

​	2.无法监听新增属性/删除属性（需要使用Vue.set和Vue.delete）

​	3.无法原生监听数组，需要特殊处理

### 3、vdom和diff算法

##### 3.1、vdom诞生背景：（基于数据驱动视图，为了更加高性能地操作DOM）

​		1、DOM操作非常耗性能，

​		2、以前是用jq可以自行控制DOM操作时机，Vue和react是数据驱动视图如何有效控制DOM操作？

​		3、因为js执行速度非常快，那么可以将js模拟DOM结构，计算最小的变更操作dom

##### 3.2、vdom结构

这里是一个html片段（dom结构）

```html
    <div id="div1" class="container">
        <p>vdom</p>
        <ul style="font-size:20px">
            <li>a</li>
        </ul>
    </div>
```

通过vdom形式表示

```js
{
    tag: 'div',
    props: {
        className: 'container',
        id: 'div1'
    }
    children: [{
            tag: 'p',
            children: 'vdom'
        },
        {
            tag: 'ul',
            props: { style: 'font-size:20px' },
            children: [{
                tag: "li",
                children: 'a'
            }]
        }
    ]
}
```

##### 3.4、diff算法的规则优化时间复杂度O(n)

1、只比较同一层级，不跨级比较

2、tag不相同，则直接删掉重建、不再深度比较

3、tag和key，两者都相同，则认为是相同节点，不在深度比较

**注意：普通的树diff比较时间复杂度为O(n*3)**

##### 3.5、通过snabbdom学习dom以及diff算法（简单了解原理）

### 4、模板编译

**介绍**：1、首先模板不是html，他有指令、插值、js表达式、能实现循环判断，2、其次html是标签语言，只有js能实现循环判断，3、那么模板一定是转换成js代码，而转换的方式就是模板编译

##### **4.1、前置知识:js的with语法**

1、改变{}内自由变量的查找规则，当作obj属性来查找

2、如果找不到匹配的obj属性，就会报错

3、with要慎用，他打破了作用域规则，可读性变差

例：

```js
// 关于js的with语法
// with语法里面的自由变量可以通过with后面的obj属性查找
const obj = { a: 1, b: 2 }
with(obj) {
    console.log(a) // 1
    console.log(b) // 2
    console.log(c) // 报错
}

console.log(obj.a) // 1
console.log(obj.b) // 1
console.log(obj.c) // 报错
```

##### 4.2、vue template complier 将模板编译成render函数

1.、这个complier模板编译为render函数，执行render函数返回vnode

2、基于vnode在执行patch和diff

3、使用vue-cli是是基于vue-loader编译成render，会在开发环境编译模板

**注意：整个流程就是从template模板编译到render函数，通过render函数生成vnode，在通过vnode中发生渲染和更新**

### 5、组件渲染过程

##### 5.1、初次渲染过程

答：1、解析模板为render函数（vue-cli中在开发环境已经完成，vue-loader）

​		2、触发响应式，监听data属性getter和setter

​		3、执行render函数，生成vnode

​		4、patch（element，vnode）将element和vnode关联显示视图

##### 5.2、更新过程

答：1、修改data，触发setter（此前在getter中已被监听到了）

​		2、重新执行render函数，生成newVode

​		3、更新节点 patch（vnode，newVnode）

##### 5.3、异步渲染

答：1、nextTick是等待DOM更新渲染后再次执行

​		2、会去收集data的修改，一次性更新视图

​		3、这样会减少DOM的操作次数，提高性能

### 6、前端路由

##### 6.1.前端hash模式特点：

1、hash变化会触发网页跳转，即浏览器的前进后退

2、hash变化不会刷新页面，spa的特点

3、hash永远不会提交到server

4、window.onhashchange方式监听hash变化 

##### 6.2.H5 history模式特点：

1.用url规范的路由，同样也是跳转时页面不会刷新

2.history.pushState和window.onpopstate监听变化

3、需要server端支持

# Vue-router

1、路由模式(hash、history)

1.1、hash默认（默认）如：http://localhost:8800/#/user/10

1.2、history模式如：http://localhost:8800/user/10

后者需要server支持，无特殊需求选择前者

2、路由配置（动态路由、懒加载）如：

```js
const User = {
	template:'<div>{{$route.params.id}}</div>'
}
const router = new VueRouter({
	routes:[
	//配置动态参数 可以命中/user/10 /user/20等
		{path:'/user/:id',component:User}
    // 路由懒加载
        {
        	path:'/user/:id',
        	components:()=>import('../components/user')
        }
	]
})
```



# Vuex 

1、基本概念

2、用于Vue

2.1、dispatch

2.2、commit

2.3、mapState、mapGetter、mapActions、mapMutations