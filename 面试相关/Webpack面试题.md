# Webpack面试题



## filename和chunkname的区别？

filename 是对应entry入口的文件

chunkFilename 不在`output.entry`中的文件，但是需要单独打包的文件名 设置使用`require.ensure`或者`import`异步加载模块打包后的名称



## 什么是loader？

答：loader是对特定语言和文件的打包方案，能够让浏览器识别并且能运行起来



## 高级js语法转ES5的方法？各个插件的作用（babel-loader、@babel/core、babel/preset-env、babel/polyfill）



## 什么是plugin？

**答：**plugin可以在webpack运行到某一时刻的时候帮我们做某一些事情



## html-webpack-plugin的作用？

**答：**会在打包结束后，自动生成一个html文件，并把打包的生成的js自动引入到这个html文件中



## module chunk bundle分别是什么意思，有何区别？

答：module是指任意的文件模块，等价于commonjs中的模块,相当于一个js文件

​       chunks是webpack处理过程中被分组了的modules，多个js文件经过打包后的dist目录下的js文件（有多个模块合成，一个依赖另一个），如代码分割时一个异步加载的chunk可能包含多个module

​       Bunldes是指打包出来的整个文件夹



## prefetch（预获取）和preload（预加载）的区别

参考：https://www.jianshu.com/p/235ad6bc5a8e

- prefetch(预获取): 将来某些导航下可能需要的资源
- preload(预加载)：当前导航下可能需要资源

**preload和 prefetch 指令相比，preload 指令有许多不同之处：**

- preload chunk 会在父 chunk 加载时，以并行方式开始加载。prefetch chunk 会在父 chunk 加载结束后开始加载
- preload chunk 具有中等优先级，并立即下载。prefetch chunk 在浏览器闲置时下载
- preload chunk 会在父 chunk 中立即请求，用于当下时刻。prefetch chunk 会用于未来的某个时刻
- 浏览器支持程度不同

例子

```js
// 在下面的代码下面加上webpackPrefetch或webpackPreload启用实现
document.addEventListener('click',()=>{
    import(/* webpackPrefetch: true */'./math').then(({default:func}) =>{
        func();
    })
})
```



## TreeShaking和Shimming