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
      <!-- 与第二种一样 -->
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
  name: 'EventDemo',
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
    // 但如果是下面这种的
    // demo3(){
    //     console.log(event) // 具体信息
    //     console.log(window.event === event) // false
    //     console.log(typeof event)  // undefined
    // },
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

Father.vue

```js
<template>
  <div>
      <Chlid />
  </div>
</template>

<script>
import Chlid from './Child.vue'
export default {
  name: "",
  provide(){
      return {
          for:'demo'
      }
  },
  components:{
    Chlid
  },
}
</script>

```

Chlid.vue

```js
<template>
  <div>
      <div>{{demo}}</div>
  </div>
</template>

<script>
export default {
  name: "",
  inject:['for'],
  data(){
    return{
        demo:this.for
    }
  }
}
</script>
```



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

```

##### 4.2、异步组件

参考：https://segmentfault.com/a/1190000012138052（建议看）

参考：https://www.w3cplus.com/vue/async-vuejs-components.html

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

参考：https://www.jianshu.com/p/4b55d312d297

**介绍**：keep-alive是一种缓存组件的方式，可以对已有组件的状态保存（在切换到下一个组件的时候）

**使用**：可以在例如tab组件切换的时候使用

**例子**：



### 6、mixin

**参考：**https://www.jianshu.com/p/4b55d312d297

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

总结： *Vue.js* 是如何在我们修改 `data` 中的数据后修改视图了。简单回顾一下，这里面其实就是一个“`setter -> Dep -> Watcher -> 生成render函数产生vnode-> patch -> 视图`”的过程。

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

##### 1、路由模式(hash、history)

1.1、hash默认（默认）如：http://localhost:8800/#/user/10

1.2、history模式如：http://localhost:8800/user/10

后者需要server支持，无特殊需求选择前者

##### 2、路由配置（动态路由、懒加载）如：

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



##### 3、route和router的区别？

route：代表当前路由信息对象，可以获取到当前路由的信息参数
router：代表路由实例的对象，包含了路由的跳转方法，钩子函数等

# Vuex 

1、基本概念

2、用于Vue

2.1、dispatch

2.2、commit

2.3、mapState、mapGetter、mapActions、mapMutations



# 面试题

以下是一些面试常问的题目，在上面可能有答案，只不过统一整理以下

### 1、为什么要在v-for中使用key？

**答：**在v-for循环中使用key（不要使用index和random），原因是因为diff算法是通过tag和key来判断，是否是同一个node，这样做会减少渲染不必要的node的次数，提升渲染性能

### 2、描述Vue父子组件的执行顺序？

**答：**线上

https://www.cnblogs.com/yuliangbin/p/9348156.html

https://www.cnblogs.com/zmyxixihaha/p/10714217.html

### 3、Vue组件间的通信（知识点中已详细描述）

### 4、描述组件渲染和更新过程

### 5、v-model实现双向绑定的原理

**答：**实际上v-model只不过是v-bind和v-on的语法糖，v-bind通过绑定value值，而v-on是通过监听input事件（value=$event.target.value）来触发value的改变，而value的改变也会是的input事件里面的value值发生改变，然后通过data的更新触发render函数

### 6、组件data为何是一个函数？

**答：**每一个组件都是一个vue实例，如果使用对象的方式 给组件内data赋值，那么当复用组件改变data时会影响所有的组件，如果使用函数的形式那么就是给每一个组件的data设为一个私有变量，不会相互影响

```
// data
data() {
  return {
	message: "子组件",
	childName:this.name
  }
}

// new Vue
new Vue({
  el: '#app',
  router,
  template: '<App/>',
  components: {App}
})

复制代码
```

因为组件是用来复用的，且 JS 里对象是引用关系，如果组件中 data 是一个对象，那么这样作用域没有隔离，子组件中的 data 属性值会相互影响，如果组件中 data 选项是一个函数，那么每个实例可以维护一份被返回对象的独立的拷贝，组件实例之间的 data 属性值不会互相影响；而 new Vue 的实例，是不会被复用的，因此不存在引用对象的问题

### 7、如何将组件所有props传递给子组件

**答：**使用$props

```js
<User v-bind="$props">
```

### 8、自己实现一个v-model

```js
<template>
  <div>
      <!-- 自己实现一个自定义v-mode -->
      <input type="text" 
             :value="content"
             @input="this.$emit('change',$event.target.value)"/>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  model:{
      props:['content'],
      event:'change'
  },
  props:{
      content:String
  }
}
</script>
```



### 9、何时使用keep-alive？谈谈你对keep-alive的理解？

https://www.jianshu.com/p/4b55d312d297

**注意：exclude的优先级高于include	**

答：1、缓存组件，不需要重复渲染

​		2、如多个静态tab页切换

​		3、优化性能



### 10、何时需要使用beforeDestory？

答：1、解绑自定义事件event.$off

​		2、清除定时器

​		3、解绑自定义dom事件（addListener），如window scroll等



### 11、vuex中action和mutation的区别？

答：1、action处理异步，mutation不能

​		2、mutation做原子操作

​		3、action可以整合多个mutation



### 12、vue中如何监听数组的响应式变化？

答：1、Object.defineProperty不能监听数组的变化

​		2、需要重新定义原型，重写push、pop等函数（上面知识点有详细）

​		3、proxy可以原生支持监听数组变化



### 13、Vue中常见性能优化方式

1、合理使用v-show和v-if

2、合理使用computed

3、v-for中使用key（优化渲染）

4、自定义事件、dom事件及时销毁

5、合理使用异步组件和keep-alive

6、data层级不要太深（使用definePropertype需要递归到最底部监听，消耗性能）

7、webpack层面的优化

8、图片懒加载等





### 14、直接给一个数组项赋值，Vue 能检测到变化吗？

参考：https://www.jianshu.com/p/e6e8c45e7fd6

由于 JavaScript 的限制，Vue 不能检测到以下数组的变动：

- 当你利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`
- 当你修改数组的长度时，例如：`vm.items.length = newLength`

为了解决第一个问题，Vue 提供了以下操作方法：

```js
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
// vm.$set，Vue.set的一个别名
vm.$set(vm.items, indexOfItem, newValue)
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```

为了解决第二个问题，Vue 提供了以下操作方法：

```js
// Array.prototype.splice
vm.items.splice(newLength)
```



### 15、父组件可以监听到子组件的生命周期吗？

比如有父组件 Parent 和子组件 Child，如果父组件监听到子组件挂载 mounted 就做一些逻辑处理，可以通过以下写法实现：

```js
// Parent.vue
// 通过监听子组件的生命周期在doSometing函数中做某些事情
<Child @mounted="doSomething"/>
    
// Child.vue
mounted() {
  this.$emit("mounted");
}
```

以上需要手动通过 $emit 触发父组件的事件，更简单的方式可以在父组件引用子组件时通过 @hook 来监听即可，如下所示：

```js
//  Parent.vue
<Child @hook:mounted="doSomething" ></Child>

doSomething() {
   console.log('父组件监听到 mounted 钩子函数 ...');
},
    
//  Child.vue
mounted(){
   console.log('子组件触发 mounted 钩子函数 ...');
},    
    
// 以上输出顺序为：
// 子组件触发 mounted 钩子函数 ...
// 父组件监听到 mounted 钩子函数 ...     
```

当然 @hook 方法不仅仅是可以监听 mounted，其它的生命周期事件，例如：created，updated 等都可以监听。





### 16、Vue 是如何实现数据双向绑定的？

Vue 数据双向绑定主要是指：数据变化更新视图，视图变化更新数据，如下图所示：



![1.png](https://user-gold-cdn.xitu.io/2019/8/19/16ca75871f2e5f80?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

即：

- 输入框内容变化时，Data 中的数据同步变化。即 View => Data 的变化。
- Data 中的数据变化时，文本节点的内容同步变化。即 Data => View 的变化。

其中，View 变化更新 Data ，可以通过事件监听的方式来实现，所以 Vue 的数据双向绑定的工作主要是如何根据 Data 变化更新 View。

Vue 主要通过以下 4 个步骤来实现数据双向绑定的：

实现一个监听器 Observer：对数据对象进行遍历，包括子属性对象的属性，利用 Object.defineProperty() 对属性都加上 setter 和 getter。这样的话，给这个对象的某个值赋值，就会触发 setter，那么就能监听到了数据变化。

实现一个解析器 Compile：解析 Vue 模板指令，将模板中的变量都替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，调用更新函数进行数据更新。

实现一个订阅者 Watcher：Watcher 订阅者是 Observer 和 Compile 之间通信的桥梁 ，主要的任务是订阅 Observer 中的属性值变化的消息，当收到属性值变化的消息时，触发解析器 Compile 中对应的更新函数。

实现一个订阅器 Dep：订阅器采用 发布-订阅 设计模式，用来收集订阅者 Watcher，对监听器 Observer 和 订阅者 Watcher 进行统一管理。

以上四个步骤的流程图表示如下，如果有同学理解不大清晰的，可以查看文章[《0 到 1 掌握：Vue 核心之数据双向绑定》](https://juejin.im/post/5d421bcf6fb9a06af23853f1)，有进行详细的讲解、以及代码 demo 示例。



![1.png](https://user-gold-cdn.xitu.io/2019/8/19/16ca75871f729d89?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 17、 vue为什么要求组件模板只能有一个根元素

我们在使用单文件组件时，一般这样写：

<template>
<div class='component'></div>
</template>
为什么template下必须有一个根元素？

首先看一看template这个标签，这个标签是html5的新标签，有三个特性：

隐藏性：不会显示在页面中
任意性：可以写在页面的任意地方
无效性： 没有一个根元素包裹，任何HTML内容都是无效的
每一个.Vue的单文件组件本质就是一个vue实例，既然是一个vue实例，它就要有一个入口，如果有多个div，就不无法指定这个vue实例的根入口。

就像一个HTML文档只能有一个根元素一样，多个根元素必将导致无法构成一颗树，所以解释了 <template></template>只有一个<div>根元素。



### 18、在.vue文件中style是必须的吗？那script是必须的吗？为什么？

```js
都不是必须的
如果是普通组件那么只能是一个静态html
如果是函数式组件， 那么可以直接使用props等函数式组件属性
```

### 19、vue-router如何响应路由参数的变化？

问题：为什么要响应参数变化？

- 切换路由，路由参数发生了变化，但是页面数据并未及时更新，需要强制刷新后才会变化。
- 不同路由渲染相同的组件时（组件复用比销毁重新创建效率要高），在切换路由后，当前组件下的生命周期函数不会再被调用。

解决方案：

1. 使用 watch 监听

```js
watch: {
    $route(to, from){
        if(to != from) {
            console.log("监听到路由变化，做出相应的处理");
        }
    }
}
```

1. 向 router-view 组件中添加 key

   ```js
   <router-view :key="$route.fullPath"></router-view>
   ```

   $route.fullPath 是完成后解析的URL，包含其查询参数信息和hash完整路径



### 20、如何获取路由参数？

如果使用`query`方式传入的参数使用`this.$route.query` 接收
如果使用`params`方式传入的参数使用`this.$router.params`接收



### 21、在vue组件中怎么获取到当前的路由信息？

如果是template中获取直接 `$route` 即可
如果是script中获取 `this.$route`
可以 `console.log(this.$route)` 查看其详细信息



### 22、怎么样实现动态加载路由？

router.addRoutes()方法



### 23、vue-router是用来做什么的？它有哪些组件？

vue-router路由，通俗来讲主要是来实现页面的跳转，通过设置不同的path，向服务器发送的不同的请求，获取不同的资源。
主要组件：router-view、router-link

**router-view 是显示当前路由地址对应的内容**

**router-link 是跳转的一种方式，相当于a标签 可以通过to来跳转指定页面，同时也可以用tag属性指定对应的标签，使用key来响应路由参数变化**



### 24、你有写过vuex中store的插件吗？在什么场景下使用？

```js
const MyPlugin = store => {
	store.subscribe((mutation,state)=>{
		// 每次mutation执行后调用这个钩子
	})
}
```

使用场景：当多个操作都会触发某一个mutation时可以在这个逻辑里面新增一些方法



### 25、Vuex中的action和mutation的区别？

mutations可以直接修改state，但只能包含同步操作，同时，只能通过提交commit调用(尽量通过Action或mapMutation调用而非直接在组件中通过this.$store.commit()提交)
actions是用来触发mutations的，它无法直接改变state，它可以包含异步操作，它只能通过store.dispatch触发



### 26、不使用vuex会带来什么问题？

1.传参数时对于多层嵌套的组件将会非常繁琐，对于兄弟组件更是无法传递（只能使用EventBus自定义事件，但是大型项目就很难了）
2.当不同视图的行为需要去修改数据时，无法追踪到数据的变更方向，导致无法维护代码



### 27、请求数据写在组件内还是vuex的action中？

具体情况具体分析，如果需要将数据保存在state中以及为了复用数据的话在acion中，如果不是这种情况放在组件内请求



### 28、Vuex如何监听数据的变化？

参考：https://segmentfault.com/q/1010000012405776z



### 29、vue单页面应用刷新网页后vuex的state数据丢失的解决方案

参考：https://blog.csdn.net/guzhao593/article/details/81435342



### 30、Proxy与Object.defineProperty的优劣对比?

Proxy的优势如下:

- Proxy可以直接监听对象而非属性
- Proxy可以直接监听数组的变化
- Proxy有多达13种拦截方法,不限于apply、ownKeys、deleteProperty、has等等是`Object.defineProperty`不具备的
- Proxy返回的是一个新对象,我们可以只操作新的对象达到目的,而`Object.defineProperty`只能遍历对象属性直接修改
- Proxy作为新标准将受到浏览器厂商重点持续的性能优化，也就是传说中的新标准的性能红利

Object.defineProperty的优势如下:

- 兼容性好,支持IE9



### 31、什么是虚拟DOM?实现？

- 虚拟DOM本质上是JavaScript对象,是对真实DOM的抽象
- 状态变更时，记录新树和旧树的差异
- 最后把差异更新到真正的dom中



### 32、vue-router中实现页面跳转的方式？谈谈你对router-link的了解？

**router-link是vue-router的一个插件，是进行页面跳转的，类似于a标签，但是可以通过tag标签对默认标签进行渲染，通过to来跳转**

第一种是通过router-link的方式进行跳转

```js
// 字符串
<router-link to="apple"> to apple</router-link>
// 对象
<router-link :to="{path:'apple'}"> to apple</router-link>
// 命名路由
<router-link :to="{name: 'applename'}"> to apple</router-link>
//直接路由带查询参数query，地址栏变成 /apple?color=red
<router-link :to="{path: 'apple', query: {color: 'red' }}"> to apple</router-link>
// 命名路由带查询参数query，地址栏变成/apple?color=red
<router-link :to="{name: 'applename', query: {color: 'red' }}"> to apple</router-link>
//直接路由带路由参数params，params 不生效，如果提供了 path，params 会被忽略
<router-link :to="{path: 'apple', params: { color: 'red' }}"> to apple</router-link>
// 命名路由带路由参数params，地址栏是/apple/red
<router-link :to="{name: 'applename', params: { color: 'red' }}"> to apple</router-link>
```

第二种是通过编程式导航使用this.$router.push（replace、go）等

```js
// 字符串
router.push('apple')
// 对象
router.push({path:'apple'})
// 命名路由
router.push({name: 'applename'})
//直接路由带查询参数query，地址栏变成 /apple?color=red
router.push({path: 'apple', query: {color: 'red' }})
// 命名路由带查询参数query，地址栏变成/apple?color=red
router.push({name: 'applename', query: {color: 'red' }})
//直接路由带路由参数params，params 不生效，如果提供了 path，params 会被忽略
router.push({path:'applename', params:{ color: 'red' }})
// 命名路由带路由参数params，地址栏是/apple/red
router.push({name:'applename', params:{ color: 'red' }})
```





33、vue-router如何配置重定向和404页面？

**重定向**

重定向也是通过 `routes` 配置来完成，下面例子是从 `/a` 重定向到 `/b`：

```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})
```

重定向的目标也可以是一个命名的路由：

```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: { name: 'foo' }}
  ]
})
```

甚至是一个方法，动态返回重定向目标：

```js
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: to => {
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}
  ]
})
```



配置404页面

```js
// 在配置路由的最后添加 最后最为匹配
{
	path:*,
	component:NotFound
	name:'404'
}
```



### 33、vuex中的action中的context对象有什么？

**里面包含了state、commit、getter、dispatch、rootState、rootGetters(后面两个是模块的时候)属性**