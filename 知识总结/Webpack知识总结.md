# Webpack知识总结

书籍：http://www.xbhub.com/wiki/webpack/

知识总结：

**es6引入导出模块：import  export default**

**commonJs引入导出模块：require module.export**

# es6 module 和 commonJs的区别 ？

1、es module 属于静态引入，在编译时引入 （像import _ from 'lodash' 这种的没有定义变量const引入，也就是不需要代码执行时才去引入，而是在代码编译时就可以直接引入了）

2、commonJs 属于动态引入，在执行时引入（因为会定义变量 const a = require("lodash")，这种就是会在代码执行时才会去引入）

3、只有es module才能treeShaking 因为treeShaking就是在代码打包编译的时候执行的



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

参考：https://www.jianshu.com/p/9ba10d6399be

答：filename是entry对象中打包出来后的文件名，

​       而chunkFilename就是未被列在entry中，但有些场景需要被打包出来的文件命名配置。比如按需加载（异步）模块的时候，这样的文件是没有被列在entry中的使用CommonJS的方式异步加载模块

总结：

filename 是对应entry入口的文件

chunkFilename 不在`output.entry`中的文件，但是需要单独打包的文件名 设置使用`require.ensure`或者`import`异步加载模块打包后的名称

# Loader知识

## 1、什么是loader？

答：loader是对特定语言或者文件打包的特定的方案，按照哪种方式打包可以让浏览器识别并且运行

​	

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
                // postcss-loader它负责进一步处理 CSS 文件，比如添加浏览器前缀，压缩 CSS、兼容css样式等。
                loader: ['style-loader', 'css-loader', 'scss-loader', 'postcss-loader'],
            }
        ]
    }
}
```

postcss.config.js

```js
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
npm install --save-dev babel-loader @babel/core @babel/preset-env @babel/polyfill 
```

**babel-loader的作用**：这个是babel和webpack连接的桥梁，当遇到js文件时需要使用loader处理

**@babel/core的作用**：这个时babel的核心，是将js文件转化成抽象语法树（ast）的关键，方便各个插件分析语法进行相应的处理

**注意！！：如果想要将ES6语法转化为ES5语法还是不够的，还要使用@babel/preset-env插件以及@babel/polyfill插件**

**@babel/preset-env的作用：** 将Es6语法转化成Es5语法这个才是最关键的转化语法插件（转换新的js语法，但是一些没有的新的api -——————> 比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如`Object.assign`）都不会转码。），**babel仅仅解析语法，但是是否ES5存在这个变量(比如promise.resolve这个语法es5但是promise并不是es5有的变量)他就不知道了**

**@babel/polyfill的作用：**这个插件的作用就是将一些新的api补充进去使得**@babel/preset-env能转码，是corejs和regenerator的集合

2、配置相关的loader语法

```js
    module: {
        rules: [{
            test: /\.js$/,
            exclude: /node_modules/,
            loader: "babel-loader",
            // 注意点1：option 这些选项存在与.babelrc文件中
            // options: {
            //     presets: [
            //         ["@babel/preset-env", {
            //             targets: {
            //                 chrome: 67
            //             },
            //             // 这个的作用就是当使用到新的js的api时才会通知进行@babel/preset-env转码
            //             useBuiltIns: 'usage'
            //         }]
            //     ]
            // }

            // 注意点2：@babel/preset-env仅仅适用于业务代码的转化
            // 当编写一些组件库或者自己写一些工具时，@babel/preset-env往往因为使用全局变量而污染了
            // 所以使用@babel/plugin-transform-runtime往往更合适(使用类似闭包特性 变量不会污染全局)
            // options在.babelrc文件
        }]
    },
```

.babelrc文件中配置optiions（主要时plugin和preset）

**注意!!：@babel/preset-env可以看作多个plugin的集合只不过是预设了而已**

```js
{
    // 使用@babel/preset-env业务
    "presets": [
        ["@babel/preset-env", {
            // 这个的作用就是当使用到新的js的api时才会通知进行@babel/preset-env转码
            // 而是用来"useBuiltIns": "usage"之后就没有必要再js文件中使用 import "@babel/polyfill" 这个语法 
            "useBuiltIns": "usage",
            "corejs": 3,  // corejs的版本
        }]
    ],
    "plugins": []
    
    //使用@babel/plugin-transform-runtime（编写组件库等）
    // "plugins": [
    //     [
    //         "@babel/plugin-transform-runtime",
    //         {
    //             "absoluteRuntime": false,
    //             "corejs": 2,  // corejs的版本
    //             "version": "^7.7.4"
    //         }
    //     ]
    // ]
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
            template: "./src/index.html",
            filename:"index.html",
        	chunks:['index'] // 只引用index.js文件
        }),
        new HtmlWebpackPlugin({
            template: "./src/other.html",
            filename:"other.html",
        	chunks:['other'] // 只引用other.js文件
        }),
        // 打包前清空dist目录
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



# HMR（热模块更新）

1、新代码生效、网页不刷新，状态不丢失，而对于自动刷新整体网页会刷新且状态会丢失

```js
// webpack.config.js文件
const webpack = require("webpack")
devServer:{
    hot:true // 配置hot为true
}
plugins:[
    new webpack.HotModuleReplacementPlugin()
]
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

## 	2、使用devServer配置(这样做的好处是对于服务器来说，可以发送ajax请求或者做其他处理，对于第1种用watch的方式是不可以的)

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

**注意：**如果遇到一些你不想去使用treeShaking（或者说仅仅是引入而不用去使用的比如一些css文件或者babel/pofill）可以在package.json文件下

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

```js
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
// 直接引入
import _ form 'lodash'
// 异步引入
function go(){
	return import('lodash')
}
```

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

```js
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

2、prefetch、preloading

参考：https://www.jianshu.com/p/235ad6bc5a8e



# webpack和浏览器缓存

**介绍：**通常来说我们打包的文件上传到服务器做生产的时候，用户访问我们的程序回家加载对应的js文件，但是当我们在中途更改了代码再次上传项目的时候，很可能用户已经缓存了之前的js文件而不会更新最新的更改后的js文件（在普通刷新的情况下），那么我们就应该给js打包的文件设置一个hash值，只要在每次更新后hash值都会被刷新，用户也就可以访问到最新的js文件而不去从浏览器缓存中找

```js
// 在开发环境中因为可以有HMR模块更新，因此可以不用设置hash值
    output: {
        filename: "[name].js",
        chunkFilename: "[name].chunks.js"
    }

// 生产环境下加上hash可以在更新文件时，让用户更新文件
    output: {
        filename: "[name]-[contenthash].js",
        chunkFilename: "[name]-[contenthash].chunks.js"
  }
```



# Shimming



# library打包（基于对函数库，工具库的打包）



# 基于TypeScript的一些打包配置（基础）

1、基本配置

```js
// cnpm i ts-loader -D
const path = require("path")
module.exports = {
    mode: "production",
    entry: "./src/index.ts",
    module: {
        rules: [{
            test: /\.ts$/,
            use: "ts-loader",
            exclude: /node_modules/
        }]
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: "tscript.js",
    }
}
```

都是些简单配置，但是最重要的是要新建tsconfig.js文件设置配置

```js
{
    "compilerOptions": {
        "outDir": "./dist",  // 打包到dist
        "module": "es6",  // 使用es6模块引入
        "target": "es5",  // 转译成es5
        "allowJs": true   // 允许引入js模块
    }
}
```

ts语言是一们严格规定类型的语言，但是有一些第三方插件并不能用ts严格规定参数，如lodash、jq等这时可以安装一些模块来让ts支持如：

```js
cnpm i @types/lodash
cnpm i @types/jquery
// 这样安装完成后就可以识别了
```



# 使用webpackDevServer实现请求转发（重要）和解决单页面应用路由的问题

## 1、实现proxy请求转发

​	介绍：在开发环境中可以使用webpack内置的devServer可以便于我们构建一个服务器协助开发，而devServer中的配置proxy是帮助我们实现跨域代理转发请求，让我们可以访问到后台服务器的api

```js
        proxy: {
            // 这里的意思是当访问这个/api的时候事实上就是加上前缀http://www.hyt.com
            // '/api': 'http://www.hyt.com'
            '/api': {
                target: 'http://www.hyt.com',
                // 这个意思是当访问路劲中有header.json改写成demo.json
                // 如果是 /api : '' 意思就是将这个改成空 
                pathRewrite: {
                    "header.json": "demo.json",
                     // 这里的api请求发出去是会变成""
                     "/api" : ''
                }
            }
        }
```

**因为wepback-dev-server实质上底层使用的是[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware)，详细可以查阅[文档](https://github.com/chimurai/http-proxy-middleware#options)。**



## 2、解决单应用路由问题

**使用`historyApiFallback`**



# 优化webpack打包速度的方法

## 1、resolve参数的合理配置

首先：resolve的配置如下

```js
resolve:{
//意思为当你引入如 : import child from "./child" 这种不带后缀的引入方式时，会默认先寻找有关的js文件，后来再找jsx文件，最后找不到才报错
	extensions:['js','jsx'],
    alias:{
        // 对目录起别名，可以直接通过child来引用
        // import a from 'child' 这种方式
		child:path.resolve(__dirname,'../child')
    }
}
```

注意： 但是这种配置选项（extensions）如果太多，会频繁调用nodejs中的底层，影响性能，所以要合理配置

## 2、使用DllPugin提升打包速度（重要）

**介绍**：对一些不必要每次构建的第三方插件进行一次构建后忽略以后的构建，这样会使得下一次的构建速度更快

**使用：**

1、DllPlugin：打包出dll文件

2、DllReferencePlugin：使用dll文件

## 3、控制包文件大小

## 4、利用thread-loader、parallel-webpack、happypack多线程打包

## 5、合理使用source-mao

## 6、结合打包分析分析打包结果

## 7、开发环境的内存编译（devServer将打包内容放入内存）



# 如何编写一个loader？

loader能对某些特定语言（如ts、vue等）的处理、扩展功能，而loader实际上是一个function，下面是一个简单的例子一：

这个我需要使用loader的文件index.js

```js
console.log('hello hyt')
```

我编写了一个叫做replaceLoader的js文件

```js
// 可以是用loader-utils来获取webpack配置中的options参数
// 当然也可以使用this.query来获取参数
const loaderUtils = require("loader-utils")

// loader 实质上是一个函数 注意要用function的方式初始化函数（不能用箭头函数因为this指向问题）
// 而this 会经过webpack的处理 指向要处理源代码（这里就是index.js）
module.exports = function(source) {
    // 将this传进去
    const options = loaderUtils.getOptions(this)
    const result = source.replace('hyt', options.name);
    // 仅仅使用return有时候还不够，可以使用loader的api----> this.callback可以有更多返回的选择
    // return source.replace('hyt', 'huangyuteng')
    // 参数：error（错误），source（源代码），source-map、meta（补充）
    // 这里就相当与return source.replace('hyt', result)
    this.callback(null, result)
}
```

webpack.config.js的配置如下

```js
const path = require("path")
module.exports = {
    mode: "development",
    entry: "./src/index.js",
    module: {
        rules: [{
            test: /\.js$/,
            use: [{
                // 这里就是我要调用的loader，就是上面的js文件
                loader: path.resolve(__dirname, './src/loader/replaceLoader.js'),
                // 传入的options会被loader中的this.options或者使用插件loaderUtils.getOptions(this)接收
                options: {
                    name: 'huangdateng'
                }
            }]
        }]
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: "[name].js",
    }
}
```

最后经过打包后dist目录里面的main.js文件原本的hello hyt 会变成 hello huangyuteng

```js
// 原本
eval("console.log('hello hyt')\n\n//# sourceURL=webpack:///./src/index.js?");
// 通过自己编写loader后变成
eval("console.log('hello huangyuteng')\n\n//# sourceURL=webpack:///./src/index.js?");
```



**这里还是一个简单的例子（对上面的例子进行改造），接下来还有**：当我想要在loader增加一些异步操作不仅仅只有同步操作时需要使用loaderApi中的：this.async

```js
// 对replaceLoader.js改造
const loaderUtils = require("loader-utils")

module.exports = function(source) {
    // 将this传进去
    const options = loaderUtils.getOptions(this)
    // 这里会通过一个异步直接返回 但是会报错 提示：没有返回值，原因也很简单，异步会在同步之后才执行
    setTimeout(() => {
        const result = source.replace('hyt', options.name);
        this.callback(null, result)
    }, 1000);
    // Error:didn't return a Buffer or String
}
```

那么我们就要使用this.async这个方法了

```js
const loaderUtils = require("loader-utils")

module.exports = function(source) {
    // 将this传进去
    const options = loaderUtils.getOptions(this);
    // 这里调用了this.async实际上就就是返回了this.callback
    // 就不会提示错误了
    const cb = this.async()
    setTimeout(() => {
        const result = source.replace('hyt', options.name);
        cb(null, result)
    }, 1000);
}
```

有时候我们引入loader的不想用path.resolve的方法那么可以加上如下配置（webpack.config.js）

```js
resolveLoader:{
	// loader即为你的文件夹名
	modules:['node_modules','./loaders']
},
    module: {
        rules: [{
            test: /\.js$/,
            use: [{
                // 这样就直接引入replaceLoader而不需要path.resolve方法了/loader/replaceLoader
                loader: "replaceLoader"
                // 传入的options会被loader中的this.options或者使用插件loaderUtils.getOptions(this)接收
                options: {
                    name: 'huangdateng'
                }
            }]
        }]
    },
```

最后是一些使用自定义loader的场景比如：

1、当项目需要支持多种语言的时候我们可以对某一个变量（例如title，在html结构中只给一个{{title}}占位）通过在loader里面进行判断，如果是中文将title变成中文，反之对其做其他处理

2、对业务代码中的方法进行try、catch捕获错误，只要判断有function的地方直接try、catch不用一个一个加上语块



# 如何编写一个Plugin插件？

首先编写plugin之前我们知道在使用一个plugin的时候我们需要new一个实例才能使用，那么就可以知道Plugin可以说是一个构造函数，当然也可以是一个类，那么我们就用一个类的方式去编写，

基本的格式（类）：

```js
// plugin 的基本格式 
class MakePlugin {
    // options可以接收传递给插件的参数比如new MakePlugin({name:'hyt'})
    // options.name 就是hyt
    constructor(options) {
        console.log(options.name)
    }
    apply(compiler) {
        // 1、异步
        //compiler 对象代表了完整的 webpack 环境配置 可以看作webpack的实例
        // 当我们想要在webpack打包的过程中做某些事情时就需要使用compiler的一些钩子hooks官网有
        // 比如使用异步的钩子emit 后面要使用tapAsync方法
        compiler.hooks.emit.tapAsync("MakePlugin", (compilation, cb) => {
            // emit是一个钩子 意思为生成资源到 output 目录之前。执行某些操作
            // 这里就是生产一个叫做copyright.txt文件 里面内容是copyright hyt write
            compilation.assets['copyright.txt'] = {
                    source: function() {
                        return 'copyright hyt write'
                    },
                    size: function() {
                        return 21;
                    }
                }
                // 注意！！！！最后记得使用cb()
            cb()
        })




        //2、同步
        // 同步方法就不需要使用tapAsync方法转而使用tap方法
        compiler.hooks.done.tap("MakePlugin", (compilation) => {
            console.log("compilation")
        })
    }
}

module.exports = MakePlugin
```



**当然，在这么多的wepack参数中很难去看到看里面的参数（比如我们想看compilation里面到底有什么东西在里面），这时我们就要借助node的调试了**步骤：

```js
//  package.json中新增脚本 --inspect是启动调试 --inspect-brk是在调试的第一个点打断点
scripts:{
	"debug": "node --inspect --inspect-brk node_modules/webpack/bin/webpack.js",
}
```

**然后运行npm run debug 输入对应的url（如localhost:9229） f12打开工具左上角有node的图标点击就可以进入调试模式看到所有的变量的数据了**

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

#### 1、前端代码为何要进行构建和打包

答：体积更小（Tree-shaking、压缩合并），加载更快

​		编译高级语法和语言(ts、react、vue、sass、es6)

​		兼容性和错误提示（eslint、postcss、polyfill）

​		另外可以有一个高效统一的开发环境、统一的构建流程和产出标准

#### 2、module chunk bundle分别是什么意思，有何区别？

答：module是指任意的文件模块，等价于commonjs中的模块

​       chunks是webpack处理过程中被分组了的modules（有多个模块合成，一个依赖另一个），如代码分割时一个异步加载的chunk可能包含多个module

​       Bunldes是指打包出来的

#### 3、loader和plugin的区别

#### 4、webpack如何实现懒加载

答：import（）

#### 5、webpack常见性能优化

###### 5.1、对与优化构建速度

**比如**：

**优化babel-loader（可用于生产环境）**

```js
{
	test:/\.js$/,
	use:['babel-loader?cacheDirectory'] // 开启缓存优化
	include:path.join(__dirname,'src')
}
```



**IgnorePlugin(避免引入无用模块)（可用于生产环境）**

参考：https://blog.csdn.net/qq_17175013/article/details/86845624



**noParse（可用于生产环境）**：

作用：不去解析属性值代表的库的依赖

参考：https://blog.csdn.net/qq_17175013/article/details/86842321

**noParse和IgnorePlugin区别**：IgnorePlugin直接不引入、代码中没有，而noParse引入，但是打包时不去解析	



**happyPack（开启多进程打包）（可用于生产环境）：**

```js
// webpack.config.js
let path = require("path");
let HappyPack = require('happypack');
module.exports = {
  mode: "development",
  entry: "./src/index.js",
  devServer: {
    ...
  },
  output: {
    ...
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        include: path.resolve("src"),
        use: 'happy/loader?id=js'   //此处将之前配置的babel的一些预设什么的替换为happy/loader。id=js，因为happy也可以打包css,
    ]
  },
  plugins: [
    new HappyPack({
      id: 'js',
      use: [{     // 将js的具体规则放置在此处
        loader: 'babel-loader',
        options: {
          presets: [
            '@babel/preset-env',
            '@babel/preset-react'
          ]
        }
      }]
    })
 ]
};
```



**ParallelUglifyPlugin（可用于生产环境）**：（开启多个子进程，把对多个文件的压缩工作分配给多个子进程去完成，每个子进程其实还是通过 UglifyJS 去压缩代码，但是变成了并行执行）

```js
const path = require('path');
const DefinePlugin = require('webpack/lib/DefinePlugin');
const ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin');

module.exports = {
  plugins: [
    // 使用 ParallelUglifyPlugin 并行压缩输出的 JS 代码
    new ParallelUglifyPlugin({
      // 传递给 UglifyJS 的参数
      uglifyJS: {
        output: {
          // 最紧凑的输出
          beautify: false,
          // 删除所有的注释
          comments: false,
        },
        compress: {
          // 在UglifyJs删除没有用到的代码时不输出警告
          warnings: false,
          // 删除所有的 `console` 语句，可以兼容ie浏览器
          drop_console: true,
          // 内嵌定义了但是只用到一次的变量
          collapse_vars: true,
          // 提取出出现多次但是没有定义成变量去引用的静态值
          reduce_vars: true,
        }
      },
    }),
  ],
}
```

**自动刷新（不可用于生产环境）、**

**热更新（不可用于生产环境）**、

**DllPlugin（不可用于生产环境）**

###### 5.2、对于优化产出代码

1、小图片base64编码（url-loader）

2、bundle加hash

```js
// 在开发环境中因为可以有HMR模块更新，因此可以不用设置hash值
    output: {
        filename: "[name].js",
        chunkFilename: "[name].chunks.js"
    }

// 生产环境下加上hash可以在更新文件时，让用户更新文件
    output: {
        filename: "[name]-[contenthash].js",
        chunkFilename: "[name]-[contenthash].chunks.js"
    }
```

3、懒加载（import）

4、提取公共代码

5、IngroePlugin

6、使用production（production模式自动压缩代码、删除warning、以及自动treeShaking）

7、使用scope-hosting

#### 6、babel-runtime和babel-polyfill区别

