# Webpack知识总结

知识总结：

**es6引入导出模块：import  export default**

**commonJs引入导出模块：require module.export**

首先注意：浏览器中不会识别es6模块引入语法（import export），如果想要浏览器识别es6语法需要安装webpack-cli(能够让我们使用webpack命令)，然后通过npx webpack index.js（文件名）可以将es6翻译出来

```js
// 这个是demo.js文件
//1.npm install webpack-cli --save安装
//2.npx webpack demo.js 将文件翻译出浏览器识别的语法 demo.js是打包的入口文件
import Header from './header.js' // 在浏览器中不会识别es6引入语法

new Header()
```

```js
// 这个是header文件
function Header(){
    console.log('header')
}
export default Header
```

## 1、什么是webpack？

答：webpack虽然是可以翻译模块导出语法，但实际上基于node环境的模块打包工具，

​		他的工作是：分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（Scss，TypeScript等），并将其打包为合适的格式以供浏览器使用。

## 2、npx 是什么？

答：npx 会帮你执行依赖包里的二进制文件。npx 会自动查找当前依赖包中的可执行文件（node_modules），如果找不到，就会去 PATH 里找。如果依然找不到，就会帮你安装。 



## 3、webapck基本配置（webpack.config.js）

```js
// webpack 配置文件
// 通过命令行npx webpack 可以直接打包entry指定的文件
// 而不用使用npx webpack 文件名  的命令方式
// 如果webpack配置文件不是webpack.config.js的话 可以使用npx webpack --config (webpack配置文件名) 的方法打包
const path = require('path')
module.exports = {
    // 入口文件
    entry: './src/index.js',
    output: {
        // 打包出口文件名
        filename: "bundle.js",
        path: path.resolve(__dirname, 'dist')
    }
}
```



# Entry和outPut

1、多入口配置

```js
    // 入口文件
    entry: {
        main: './src/index.js',
        sub: './src/index.js'
    },
    output: {
        // 打包出口文件名
        // filename: "bundle.js",
        // 存在多入口文件时，需要使用多出口占位符
        // publicPath是当打包成功后会在所有的js文件的前面加上前缀
        publicPath: 'this-is-publicPath',
        filename: '[name].js',
        path: path.resolve(__dirname, 'dist')
    },
```

2、chunkFilencame和filename的区别？

答：filename是entry对象中打包出来后的文件名，

​       而chunkFilename就是未被列在entry中，但有些场景需要被打包出来的文件命名配置。比如按需加载（异步）模块的时候，这样的文件是没有被列在entry中的使用CommonJS的方式异步加载模块

总结：

filename 是对应entry入口的文件

chunkFilename 不在`output.entry`中的文件，但是需要单独打包的文件名 设置使用`require.ensure`或者`import`异步加载模块打包后的名称

比如：

```

```



# Loader知识

## 1、什么是loader？

答：loader是对特定语言或者文件打包的特定的方案，按照哪种方式打包可以让浏览器识别并且运行

## 2、url-loader和file-loader的区别？

答：1、url-loader依赖file-loader
       2、当使用url-loader加载图片，图片大小小于上限值，则将图片转base64字符串，；否则使用file-loader加载图片，      都是为了提高浏览器加载图片速度。
       3、使用url-loader加载图片比file-loader更优秀

6、css-loader和style-loader的功能？

答： css-loader会根据多个css文件之间的关系进行分析并且将其所有变成一个统一的css

​		style-loader会将这个统一的css文件挂载到html的head标签里面变成css样式

## 3、loader的相关配置

webpack.config.js文件

```js
// webpack 配置文件
// 通过命令行npx webpack 可以直接打包entry指定的文件
// 而不用使用npx webpack 文件名  的命令方式
// 如果webpack配置文件不是webpack.config.js的话 可以使用npx webpack --config (webpack配置文件名) 的方法打包
const path = require('path')
    // 使用loader打包样式、图片
module.exports = {
    // 入口文件
    entry: './src/index.js',
    output: {
        // 打包出口文件名
        filename: "bundle.js",
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [{
                test: /\.(jpg|png|gif)$/,
                // loader: "file-loader",
                // 可以使用url-loader
                loader: "url-loader",
                options: {
                    // []中的值是占位符意思为按照原来的图片名和后缀打包
                    name: '[name].[ext]',
                    // 这是打包后的文件夹名/dist/images
                    outputPath: 'images/',
                    // limit是文件大小 大于这个值url-loader会按照file-loader打包
                    limit: 10240
                }
            },
            {
                test: /\.css$/,
                // loader 从右往左执行
                use: ['style-loader',
                    {
                        loader: 'css-loader',
                        options: {
                            modules: true,
                        }
                    }
                ],
            },
            {
                test: /\.scss$/,
                // loader 从右往左执行
                // postcss-loader它负责进一步处理 CSS 文件，比如添加浏览器前缀，压缩 CSS 等。
                loader: ['style-loader', 'css-loader', 'scss-loader', 'postcss-loader'],
            }
        ]
    }
}
```

postcss.config.js

```
module.exports = {
    plugins: [
        // 对于css3的新特性会为其加入新前缀
        require('autoprefixer')
    ]
}
```



## 4、配置Es6转Es5语法的loader（重要）

1、安装babel-loader和 @babel/core

```js
npm install --save-dev babel-loader @babel/core
```

**babel-loader的作用**：这个是babel和webpack连接的桥梁，当遇到js文件时需要使用loader处理

**@babel/core的作用**：这个时babel的核心，是将js文件转化成抽象语法树（ast）的关键，方便各个插件分析语法进行相应的处理

**注意！！：如果想要将ES6语法转化为ES5语法还是不够的，还要使用@babel/preset-env插件以及@babel/polyfill插件**

**@babel/preset-env的作用：** 将Es6语法转化成Es5语法这个才是最关键的转化语法插件（转换新的js语法，但是一些没有的新的api -——————> 比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如`Object.assign`）都不会转码。）

**@babel/polyfill的作用：**这个插件的作用就是将一些新的api补充进去使得**@babel/preset-env能转码

2、配置相关的loader语法

```js
module: {
  rules: [
      // 剔除掉node_modules中的js文件
    { test: /\.js$/, exclude: /node_modules/, loader: "babel-loader" }
  ]
}
```



# Plugin

## 1、什么是plugin？

**答：**plugin可以在webpack运行到某一时刻的时候帮我们做某一些事情

## 2、html-webpack-plugin的作用？

**答：**会在打包结束后，自动生成一个html文件，并把打包的生成的js自动引入到这个html文件中

## 3、html-webpack-plugin和clean-webpack-plugin的基本配置

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
plugins: [
        // 打包后以/src/index.html文件为模板作为打包后的html文件
        new HtmlWebpackPlugin({
            template: "./src/index.html"
        }),
        // 打包前清空dist目录
        new CleanWebpackPlugin()
    ]
}
```



# devtool（source-map）

**介绍：**source-map是一种映射关系，通过这种映射关系在代码发生错误的时候可以定位的源代码错误的位置而不是打包后的代码的错误位置，可以给开发者更好的开发体验

**注意：**

​			配置source-map，打包后会出现一个.map后缀的文件这个是一个映射关系的文件

​			有inline前缀的文件会将.map文件合并到打包后的js文件中去

​			有module前缀的source-map类型都可以映射除了业务代码之外的位置（如第三方插件）

​		    而有cheap前缀的一般来说都只会精确定位到错误代码的行数，而不会定位第几列，打包速度更快

​			eval性能最好，但不适用于代码较为复杂的时候

**使用建议：**

​		1.当在开发模式下，建议使用cheap-module-eval-source-map

```js
    mode: 'development',
    devtool: 'cheap-module-eval-source-map',
```

​		2.当在生产模式下，建议使用cheap-module-source-map

```js
	mode: 'production',
    devtool: 'cheap-module-source-map',
```



# DevServer配置

实时监听更新代码的方式：

## 1、在package.json文件的script脚本下运行添加命令

```js
 // 加上--watch可以监听代码的变化并且更新
 "scripts": {
    "bundle": "webpack --watch"
  },
```

## 2、使用devServer配置(这样做的好处是对于服务器来说，可以发送ajax请求或者做其他处理，对于第1种用watch的方式是不可以的)

```js
// 1、需要先安装webpack-dev-server
// 2、在webpack.config.js文件下进行配置
    devServer: {
        contentBase: path.join(__dirname, "dist"), // 服务器运行的目录
        port: 9000
    },
        
// 3、运行脚本npm run server 脚本配置 "server":"webpack-dev-server"
```



# treeShaking（按需引入模块功能）

**介绍**：treeShaking是一种按需引入功能的方式，它可以减少引入不必要或者没有引用的功能，增加代码的打包速度

**注意**：treeShaking只支持es6的模块引入方式

**配置步骤**（production模式下没必要配置因为已经配置好了，development模式下可以配置）

webpack.config.js文件下

```js
// mode为development
    // 在开发模式下配置treeShaking 但是在生产模式下已经配置好了没必要在配置
    optimization: {
        usedExports: true
    },
```

**注意：**如果遇到一些你不想去使用treeShaking（或者说仅仅是引入而不用去使用的比如一些css文件或者babel/pofill）可以在package.json文件下设置

```js
{	
    // 可以为false 或者写入你不想要去treeShaking的文件
  "sideEffects": [
    "./src/some-side-effectful-file.js"
  ]
}

```



# development和production的分别打包

webpack.common.js

```js
// 公共webpack配置
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require("clean-webpack-plugin")
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin') //生产模式下对css代码压缩

module.exports = {
    entry: './src/index.js',
    output: {
        // 打包出口文件名
        filename: "bundle.js",
        path: path.resolve(__dirname, 'dist'),
        chunkFilename: "[name].chunks.js"
    },
    // 在开发模式下配置treeShaking 但是在生产模式下已经配置好了没必要在配置
    optimization: {
        minimizer: [
            // 对css代码进行压缩
            new OptimizeCSSAssetsPlugin({})
        ],
        usedExports: true,
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: "./src/index.html"
        }),
        new CleanWebpackPlugin()
    ]
}
```



webpack.dev.js

```js
// 开发环境下的配置
const path = require('path')
const webpack = require('webpack')
const merge = require('webpack-merge')
const commonConfig = require('./webpack.common')
const devConfig = {
    // 入口文件
    mode: 'development',
    devServer: {
        contentBase: path.join(__dirname, "dist"), // 服务器运行的目录
        open: true, // 自动打开浏览器访问9000网页
        port: 9000,
        hot: true
    },
    module: {
        rules: [{
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "babel-loader",
        }, {
            test: /\.css$/,
            use: [
                'style-loader',
                'css-loader'
            ]
        }]
    },
    // 在开发模式下配置treeShaking 但是在生产模式下已经配置好了没必要在配置
    // optimization: {
    //     usedExports: true,
    //     splitChunks: {
    //         // 代码分割的类型 async异步 initial同步 all全都分割，同步分割需要配合chcheGroups的vendors选项
    //         chunks: "all",
    //         // 分割的最小占用内存
    //         minSize: 0,
    //         // 当一个模块至少用了多少次才进行代码分割
    //         // 比如这里的意思是当有一个js文件需要引入目标分割文件时就需要被分割
    //         minChunks: 1,
    //         //按需加载时并行请求的最大数量 也就是最多进行5次
    //         maxAsyncRequests: 5,
    //         //入口点的最大并行请求数
    //         maxInitialRequests: 3,
    //         // 分割后文件名连接符
    //         automaticNameDelimiter: '~',
    //         name: true,
    //         // 分割同步代码时的走的方向
    //         cacheGroups: {
    //             // 同步分割
    //             vendors: {
    //                 // 将所有node_modules的插件都打包在这个vendors里面
    //                 // 符合node_modules模块里面的
    //                 test: /[\\/]node_modules[\\/]/,
    //                 // 优先级与vendors对比（都符合的情况下）
    //                 priority: -10,
    //                 // 可以配置分割后的文件名 vendors.js
    //                 // filename: 'vendors.js'
    //             },
    //             // 如果不是第三方插件的情况下
    //             default: {
    //                 filename: 'default.js',
    //                 minChunks: 1,
    //                 // 优先级与vendors对比（都符合的情况下）
    //                 priority: -20,
    //                 // 如果一个模块已经被引入了，那么忽略这个模块
    //                 reuseExistingChunk: true
    //             }
    //         }
    //     }
    // },
    plugins: [
        // 热模块更新
        new webpack.HotModuleReplacementPlugin()
    ]
}

module.exports = merge(commonConfig, devConfig)
```

webpack.prod.js

```
// 生产环境下的配置
const merge = require('webpack-merge')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const commonConfig = require('./webpack.common')
const prodConfig = {
    // 入口文件
    mode: 'production',
    module: {
        rules: [{
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "babel-loader",
        }, {
            test: /\.css$/,
            use: [
                MiniCssExtractPlugin.loader,
                'css-loader'
            ]
        }]
    },
    plugins: [
        new MiniCssExtractPlugin({
            // Options similar to the same options in webpackOptions.output
            // both options are optional
            filename: "[name].css",
            chunkFilename: "[id].css"
        })
    ],
}

module.exports = merge(commonConfig, prodConfig)
```

最后再package.json上面配置script脚本

```js
    "scripts": {
        "dev-build": "webpack --config webpack.dev.js --mode development",

        "build": "webpack --config webpack.prod.js --mode production"
    },
```



# webpack和code spliting（代码分割）

**注意：**首先代码分割和webpack本身并没有直接的关系，只不过webpack自身有这种设置

## 1、webpack实现JS代码分割有两种方式：

1.1、同步代码：需要在webpack.config.js中做设置

```js
optimization：{
	splitChunks:{
		chunks:"all"
}
```

1.2、异步代码：无需做配置，会自动进行代码分割（可以参考官网的例子）

1.3、事实上，无论同步还是异步代码分割都是基于SplitChunksPlugin这个插件实现的，而这个插件仅仅是默认异步代码的分割

## 2、SplitChunksPlugin配置参数详解

```js
splitChunks: {
    // 代码分割的类型 async异步 initial同步 all全都分割，同步分割需要配合chcheGroups的vendors选项
    chunks: "async", 
    // 分割的最小占用内存
    minSize: 30000,
    // 当一个模块至少用了多少次才进行代码分割
    // 比如这里的意思是当有一个js文件需要引入目标分割文件时就需要被分割
    minChunks: 1,
    //按需加载时并行请求的最大数量 也就是最多进行5次
    maxAsyncRequests: 5,
    //入口点的最大并行请求数
    maxInitialRequests: 3,
    // 分割后文件名连接符
    automaticNameDelimiter: '~',
    name: true,
    // 分割同步代码时的走的方向
    cacheGroups: {
        // 同步分割
        vendors: {
            // 将所有node_modules的插件都打包在这个vendors里面
            // 符合node_modules模块里面的
            test: /[\\/]node_modules[\\/]/,
                // 优先级与vendors对比（都符合的情况下）
            priority: -10
            // 可以配置分割后的文件名 vendors.js
            // filename:'vendors.js'
        },
        // 如果不是第三方插件的情况下
    default: {
            minChunks: 2,
                // 优先级与vendors对比（都符合的情况下）
            priority: -20,
                // 如果一个模块已经被引入了，那么忽略这个模块
            reuseExistingChunk: true
        }
    }
}
```



## 3、webpack实现对css进行代码分割（MiniCssExtractPlugin插件）官网上有详细实例

**注意**：MiniCssExtractPlugin插件一般适用于生产环境，因为并不支持HMR，不支持热模块加载，并且需要使用MiniCssExtractPlugin.loader来代替style-loader，并且在打包的时候注意是否配置了treeShaking，在package.json中对sideEffect加入对所有css文件忽略treeShaking

```
    // 正确配置module和plugin都要配置
    module: {
        rules: [{
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "babel-loader",
        }, {
            test: /\.css$/,
            use: [
                MiniCssExtractPlugin.loader,
                'css-loader'
            ]
        }]
    },
    plugins: [
        new MiniCssExtractPlugin({
            // Options similar to the same options in webpackOptions.output
            // both options are optional
            filename: "[name].css",
            chunkFilename: "[id].css"
        })
    ],
```

如果想对css文件进行压缩的话可以使用optimize-css-assets-webpack-plugin插件

```js
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin') //生产模式下对css代码压缩
///.....

    optimization: {
        minimizer: [
            // 对css代码进行压缩
            new OptimizeCSSAssetsPlugin({})
        ],
    },
```



# Chunk是什么？

答：chunk是指webpack打包过后的js文件，一个js文件对应一个chunk



# 打包分析、preloading、prefetching 

1、打包分析：

![image-20200409173401333](C:\Users\hyt\AppData\Roaming\Typora\typora-user-images\image-20200409173401333.png)

可以对webpack打包后的项目文件进行分析，在package.json文件中新增script命令例如

```js
scripts:{
	"dev-build":"webpack --profile --json > stats.json --config"
}
```



# 面试题

## 1、基本配置

1.1、拆分配置和merge

1.2、启动本地服务

通过在开发环境配置的js文件里加入devServer配置选项配置本地服务

## 2、高级配置

## 3、优化打包效率

## 4、优化产出代码

## 5、构建流程概述

## 6、Babel简述

## 7、面试总结

1、前端代码为何要进行构建和打包

2、module chunk bundle分别是什么意思，有何区别？

3、loader和plugin的区别

4、webpack如何实现懒加载

5、webpack常见性能优化

6、babel-runtime和babel-polyfill区别

