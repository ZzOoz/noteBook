Koa旧岛服务端总结





# 重点：error中间件的处理

在middleware文件夹中定义一个exception.js文件夹作为全局捕捉的中间件（以下代码都写关键 引入不一定会写）



**注意！注意！注意！ 错误中间件一定要放在所有中间件的首位，因为是用来兜底错误的**

```js
const catchError = (ctx,next){
	try{
	  	await next() // 执行下一个中间件 如果有错误会捕捉到走面下一步
	}catch(error){
		const isHttpException = error instanceof HttpException
        const isDev = global.conf.environment === 'dev'
        
        // 如果是未知错误且是开发模式直接抛错
		if(isDev && !isHttpException){
            throw error
        }
        // 如果是生产环境
        // 判断error是不是HttpException类型错误
        // 如果是则是已知错误 因为HttpException是自己定义的错误类
        // 抛出的错误 如在某一个api抛出ParameterException错误 就会被最外面的catchError捕捉
        // 然后因为是HttpException类型所以会报错下面ctx.body的错误
        // 如果是已知错误
        if(isHttpException){
            ctx.body = {
                msg:error.msg,
                error_code:error.errorCode,
                request:`${ctx.method} ${ctx.path}`
            }
            ctx.status = error.code
        }
        // 未知错误 且生产模式
        else{
            ctx.body = {
                msg:'这是一个未知异常',
                error_code:9999999,
                request:`${ctx.method} ${ctx.path}`
            }
            ctx.status = 500
        }
    }
}


module.exports = catchError
```



# 重点：1.使用requireDirectory自动注册多路由

  1. Cnpm i require-directory 下载npm包

  2. 引入模块 **const** requireDirectory = require('require-directory')

  3. 调用：requireDirectory(module,'./api/v1',{visit:whenLoadModule})

  4. requireDirectory的第三个参数visit为一个回调函数

  5. 这样就可以自动注册多路由（完整代码）

       1. ```js
          const Koa = require('koa')
          const book = require('./api/v1/book')
           // requireDirectory是自动注册所有路由的npm包
          const requireDirectory = require('require-directory')
          const classic = require('./api/v1/classic')
          
          const app = new Koa()
          
          // 调用requireDirectory获取目录下的路由
          requireDirectory(module,'./api/v1',{visit:whenLoadModule})
          
          // 每次使用requireDirectory加载路由的时候会调用这个方法
          function whenLoadModule(obj){
              // 判断这个加入的obj是否为Router类型
              // 如果是则注册路由
              if(obj instanceof Router){
                  app.use(obj.routes())
              }
          }
          
          app.use(classic.routes())
          app.use(book.routes())
          
          app.listen(3000,()=>{
              console.log('程序已启动')
          })
          ```

     ##### 2.使用vscode调式和nodemon自动重启的综合应用

     ​	vscoede

     ##### 3.初始化管理器和使用绝对路



# 重点：2.参数的获取（路径中、查询参数中、header中、body中）

​	1.在koa中获取参数

```js
ctx.param // 获取路径中的参数
ctx.request.query // 获取查询参数
ctx.request.header // 获取头部参数
// 获取body参数需要借助npm插件koa-bodyparser
ctx.request.body
```





##### 编写一个koa Api的思路路径

​	1.首先要获取参数，然后进行参数校验

​	2.然后将获取的参数保存到数据库中去



##### 3.中间件



```js
//为什么要使用以下代码去验证参数
router.post('/register',async (ctx,next)=>{
    // 获取参数，校验参数
    const v = await new RegisterValidator().validate(ctx)
    const user = {
        email:v.get('body.email'),
        password:v.get('body.password2'),
        nickname:v.get('body.nickname')
    }
    // 创建用户
    const u = await User.create(user)
})

//而不去使用以下代码（中间件直接验证？）
router.post('/register',new RegisterValidator(),async (ctx,next)=>{
    // 获取参数，校验参数
    const user = {
        email:v.get('body.email'),
        password:v.get('body.password2'),
        nickname:v.get('body.nickname')
    }
    // 创建用户
    const u = await User.create(user)
})


//
原因:因为在中间件new一个类的时候只会在程序运行的时候实例化一次（中间件的特性），而使用第一种则会每次请求进来都会实例化一次，这会导致每次请求都会有不同的实例化对象，而想要修改这个对象里面挂载的属性的时候就不会互相干扰引起错乱，如果用第二种则会导致混乱（详细视频6-2）
```





##### 4.密码的加密

在/v1/user.js文件下

```js
// 引入bcrypt
const bcrypt = require('bcryptjs')
const Router = require('koa-router')
const router = new Router({
    prefix:'/v1/user'
})

const {RegisterValidator} = require('../../validator/validator')

const {User} = require('../../model/user')

router.post('/register',async (ctx,next)=>{
    const v = await new RegisterValidator().validate(ctx)

    // 对密码进行加密
    // 1.生成盐 参数是成本 越高越安全
    // 2.通过盐来加密 重要的两部
    const salt = bcrypt.genSaltSync(10)
    const pwd = bcrypt.hashSync(v.get('body.password2'),salt)


    const user = {
        email:v.get('body.email'),
        password:pwd, // 加密后的
        nickname:v.get('body.nickname')
    }
    const u = await User.create(user)
})

module.exports = router
```

也可以使用更好的方法（在/model/user.js的password字段定义中）

```js
// 引入bcrypt
const bcrypt = require('bcryptjs')

//。。。
//。。。省略中间

password:{
     type:Sequelize.STRING,
	 // 只要对password进行赋值的话就会调用set函数 val是passowrd
     // 观察者模式的应用：只要password改变我就调用set set是紧盯者password的观察者
     set(val){
       // 对密码进行加密
       // 1.生成盐 参数是成本 越高越安全
       // 2.通过盐来加密
       // 3.调用setDataValue方法设置
       const salt = bcrypt.genSaltSync(10)
       const pwd =bcrypt.hashSync(val,salt)
       this.setDataValue('password',pwd)
 }
```



##### 4.使用sequelize连接数据库

```js
// 这里是连接数据库的文件
const {Sequelize}  = require('sequelize')
const {
    dbname,
    user,
    password,
    host,
    port
} = require('../config/config').database


const sequelize = new Sequelize(dbname,user,password,{
    // 选择类型mysql 前提是要下载npm上面的mysql插件
    dialect:'mysql',
    port,
    host,
    logging:true, // 在终端上显示数据库的操作
    define:{

    },
    timezone:'+8:00'  // 时区
})

module.exports = {
    sequelize
}
```

数据库的目的：持久存储数据，而写入数据库的操作叫做持久化



##### 6.后端Auth权限的验证以及前端小程序通过header传token验证

###### 1.在文件auth.js中定义一个Auth类 使用属性m定义其权限验证

```js
Class Auth{
    constructor(level){
        this.level = level;
        Auth.USER = 8;
        Auth.ADMIN = 8;
        Auth.SUPER_ADMIN = 8;
    }
    
    // 定义属性m来验证用户的权限,使用get方法不能设置参数
    get m(){
        // 返回函数中间件
        return (ctx,netx)=>{
            // ctx.req 是获取node.js原生的封装的request对象
            // ctx.request获取的是koa封装nodejs的request对象
            // 从给定的请求中获取基本的身份验证凭据。 
            //解析Authorization标头，如果标头无效，则返回undefined，
            //否则返回具有name和pass属性的对象。
            let userToken = basicAuth(ctx.req)
            let errorMsg = 'token不合法'

            if(!userToken || !userToken.name){
                throw new global.errs.ForbiddenException(errorMsg)
            }
            
            try{
                // 使用jsonwebtoken来验证token的合法型
                // 第一个参数是token 第二个是secretKey
                // 解析出来的decodeToken有生成token时候的uid和scope权限
				let decodeToken = jwt.verify(userToken.name,global.conf.token.secret)
            }catch(error){
                // 如果出现异常 1.token过期 2.不合法
                // 过期
                if(error.name === 'TokenExpiredError'){
                    errorMsg = 'token过期了'
                }
                
                throw new global.errs.ForbiddenException(errorMsg)
            }
           
            // 通过实例传参来比较权限级别
            if(decode.scope <= this.level){
                errorMsg = '权限不足'
                throw new global.errs.ForbiddenException(errorMsg)
            }
            
            // 通过解密的decodeToken来获取之前放进token的信息
            ctx.auth = {
                uid:decodeToken.uid,
                scope:decodeToken.scope
            }
            await next()
        }
    }
}
```

###### 2.前端小程序token验证（basicAuth方式）

在wx.request方法中

```js
// 这是一个获取最新一期的方法
GetClassic(){
    wx.request({
      url: 'http://localhost:3000/v1/classic/lastest',
      success: function (res) {
        console.log(res)
      },
      header: {
        // 这里就是通过传递用户的tokne来给后端的auth类中m来验证
        // 上面第一点
        Authorization: this._encode() // 通过basic Auth来认证令牌
      }
    })
  },
      
// 定义一个_encode私有方法来通过basic Auth来认证令牌
// base64加密token方法
  _encode() {
    const token = wx.getStorageSync("token")
    // 这里加密过后会在后台通过basicAuth方法解密
    //basicAuth(ctx.req)
    const base64 = Base64.encode(token + ':') //将令牌解析成base64通过Basic Auth来认证
    return 'Basic ' + base64 // 返回规定格式 Authorization: 'Basic:加上token'
  },
```







##### .使用jsonwebtoken生成token令牌

util.js文件中

```js
//引入jwt
const jwt = require('jsonwebtoken')

/....
..../

const generateToken = (uid,scope)=>{
	cosnt secretKey = global.conf.token.secretKey
    cosnt expiresIn = global.conf.token.expiresIn
    
    const token = jwt.sign({
        uid,
        scope
    },secretKey,{
        expiresIn
    })
    
    return token
}
```





##### 7.使用token登录（邮箱验证以及小程序）

 /model/user.js**文件中**

```js
//

class User extends Model{
    //1.邮箱验证 在User模型文件中 
    static async verifiyEamilandPassword(eamil,password){
        //查找是否邮箱存在
        const user = await User.findOne({
            where:{
                eamil:eamil
            }
        })
        
        if(!user){
            throw new NotFindException('账号不存在')
        }
        
        // 如果邮箱存在验证密码
        const correct = bscrypt.compareSync(password,user.password)
        if(!correct){
            throw new NotFindException('密码错误')
        }
        
        //如果都成功返回user
        return user
    }
    
    //2. 微信小程序验证 查找
	static async getUserByOpenId(openId){
        const user = await User.findOne({
            where:{
                openId:openId
            }
        })
        // 在wx.js文件中引用 如果没有就会调用registerBregiyOpendId方法
        return user
    }
    
    //创建
    static async registerBregiyOpendId(opendId){
        const user = await User.create({
            where:{
                openId:openId
         	 }
        })
        return user
    }
}

```



**在token.js文件中**

```js
//引入省略部分
router.post('/token',async (ctx,next)=>{
	const v = await new TokenValidator().validate(ctx)
    let token
    switch (v.get('body.type')) {
        case LoginType.USER_EMAIL: // 邮箱登录验证获取token
            token = await eamilandPassword(v.get('body.account'),v.get('body.secret'))
            break;
        case LoginType.USER_MINI_PROGRAM:
            token = await WxMannager.codeToToken(v.get('body.account'))
            break;
        default:
            throw new global.errs.ParameterException('没有相应的处理函数')
    }
}
            
// 调用验证邮箱方法
function eamilandPassword(account,password){
    // 验证之后生成token user.id为uid 生成权限
    const user = await verifiyEamilandPassword(account,password)
    return generateToken(user.id,Auth.USER) // 生成权限
}

// 微信生成在wx.js文件中
```



**微信生成在wx.js文件中**

```js
Class WxMannager{
	// 通过code生成token
    // 方法在token文件调用 code 即为 account
	static async codeToToken(code){
     	const url = util.format(global.conf.wx.loginUrl,
                             global.conf.wx.appId,
                             global.conf.wx.appSecret,
                             code)
		
        const result = axios.get(url)
        
        if(result.status !== 200){
            throw new ForbiddenException('openId获取失败')
        }
        

        const errorcode = result.data.errcode
        if(result.errorcode !== 0){
            throw new ForbiddenException("openId获取失败"+errorcode)
        }
        
        //调用model中user.js的方法
        let user = await User.getUserByOpenId(result.data.openid)
     	if(!user){
            user = await User.registerByOpenId(result.data.openid)
        }
        
        // 返回成功的token结果 给到token中的LoginType.USER_MINI_PROGRAM中
        return generateToken(user.id,Auth.USER)
     }
}
```





**关于TokenVokalidator部分**

```js
class TokenValidator extends LinValidator{
	constructor(){
        super()
        this.account = [
            new Rule('isLength','长度必须符合',{
				min:6,
                max:32
            })
        ]
        
        this.secret = [
            new Rule('isOptional') // 设为可选
            new Rule('isLength','长度必须符合',{
				min:6,
                max:32
            })
        ]
    }
    
    validateLoginType(vals){
        if(!vals.body.type){
            throw new Error('验证类型必须指定')
        }
        if(!LoginType.isThisType(val.body.type)){
            throw new Error('不存在这种登录类型')
        }
    }
}
```



##### 8.微信生成openID过程

​	由前端生成的code码传给服务端，然后服务端通过调用微信的服务，如果code码正确则会返回用户的openID



**9.微信小程序中使用npm包**

1.在小程序设置选项中找到项目设置=》勾选使用npm模块

2.打开小程序根目录终端输入npm init -y 构建package.json文件

3.安装所需要的npm包

4.在小程序界面打开工具=》点击构建npm







# **业务相关**

### 1.**拿到我喜欢的期刊**



在api文件夹在classic.js中

```js
router.get("/favor",new Auth().m,async (ctx,next)){
    const arts = await Favor.getAllFavorData(ctx.auth.uid)
    ctx.body = arts
}
```

在model文件夹中的favor中定义静态方法

```js
class Favor extends Model{
    
    static async getAllFavorData(uid){
        // 找出type不是400 且uid点赞过的作品
        const arts = Favor.findAll({
            where:{
                uid,
                [Op.not]:400
            }
        }) 
        if(!arts){
            throw new global.errs.NotFindException()
        }
        return await Art.getFavorArtData(arts)
    }
}
```



在model文件中art.js 定义静态方法getFavorArtData

```js
class Art extends Model{
    
    static async getFavorArtData(artList){
        // 分别定义不同类型的期刊 各自是一个空数组
        let favorObj = {
        	100:[],
            200:[],
            300:[]
        }
        
        for(const art of artList){
            // 插入属于这个类型的id
            favorObj[art.type].push(art.art_id)
        }
        
        // 定义一个空数组这个空数组存放各自类型作品的详细信息
        const arts = []
        for(const key in favorObj){
            let ids = favorObj[key]
            if(ids.length === 0){
                continue;
            }
            
            arts.push(await Art._getArtList(ids,key))
        }
        
        // 将key转化成数字 因为对象上面的key是字符串
        key = parseInt(key)
        return flattern(arts)
    }
    
    
    // 私有方法_getArtList
    static async _getArtList(ids,type){
        let arts = []
        const finder = {
			where:{
                id:{
                	[Op.in]:ids
                }
            }
        }
        const scope = 'bh'
        switch (type) {
          case 100:
            arts = await Movie.findAll(finder)
            break
          case 200:
            arts = await Music.findAll(finder)
            break
          case 300:
            arts = await Sentence.findAll(finder)
            break
          case 400:
            break
          default:
            break
        }
        return arts
    }
}
```



# 其他

### 1.toJSON方法

使用toJSON方法可以在数据序列化之前去除一些不必要的字段

在db.js文件中

```js
// 在db文件下定义toJson方法过滤不必要的字段
Model.prototype.toJSON = function(){
    // 先复制一层浅拷贝数据到data 在修改 没必要深拷贝 因为只改一层数据
    let data = clone(this.dataValues)
    unset(data,'deletedAt')
    unset(data,'createdAt')
    unset(data,'updatedAt')
    
    for (key in data) {
        if (key === 'image') {
          console.log(!data[key].startsWith('https') && data[key] !== null)
          if (!data[key].startsWith('https') && data[key] !== null)
            data[key] = global.conf.host + data[key]
        }
      }
    return data
}
```

 