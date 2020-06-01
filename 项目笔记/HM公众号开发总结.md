# 10HM公众号开发总结



## 1、微信js-sdk的使用

步骤：1、首先需要在html模板中引入js文件https://res.wx.qq.com/open/js/jweixin-1.4.0.js

​		   2、然后新建一个wxsdk类，里面包含了要调用sdk的方法和jsApiList接口，在里面首先要请求后台生成所需要的签名signtrue

​		   3、在生成签名之后需要使用wx.config方法传入参数appId、debug、timestamp、nonceStr, signature, jsApiList返回成功

​		   4、成功之后需要调用wx.ready方法里面都是需要使用的sdk接口

​		   5、最后导出wxsdk的实例  export new WxSdk()

```js
// 新建wxSdk类
/**
 * 微信sdk相关方法
 */
import $ajax from './ajax';
import { URL_wxJsConfig } from './api';

class Wxsdk {
    // 步骤一：引入相关的js文件在html模板中
    constructor() {
        this.shareCache = {};
        this.debug = false;
        this.jsApiList = [
            'onMenuShareTimeline', // 分享到朋友圈
            'onMenuShareAppMessage', // 分享给朋友
            'closeWindow', // 关闭当前网页窗口
            'chooseWXPay',
        ];
        this.initConfig();
    }

    /**
     * 初始化sdk配置
     */
    async initConfig() {
        // 步骤2：生成签名、appid等
        const { errorCode, returnObject } = await $ajax({
            url: URL_wxJsConfig,
            method: 'post',
            data: {
                pageUrl: location.href.split('#')[0],
            },
        });
        if (errorCode !== '00') {
            console.error('获取wxsdk配置发生错误');
            return;
        }
        const { appId, timestamp, nonceStr, signature } = returnObject;
        const { debug, jsApiList } = this;
        // 步骤3：调用config方法返回
        wx.config({ debug, appId, timestamp, nonceStr, signature, jsApiList });
    }

    /**
     * 设置微信分享内容
     * @param {Object} options 分享配置
     */
    resetShare(options) {
        this.shareCache = {...this.shareCache, ...options };
        const {
            title,
            desc,
            link,
            imgUrl,
            success,
            cancel,
            fail,
        } = this.shareCache;
        // 步骤4：调用ready方法使用相关的jssdk接口
        wx.ready(() => {
            // 分享到朋友圈
            wx.onMenuShareTimeline({
                title,
                link,
                imgUrl,
                success() {
                    success && success();
                },
                cancel() {
                    cancel && cancel();
                },
                fail() {
                    fail && fail();
                },
            });
            // 分享给朋友
            wx.onMenuShareAppMessage({
                title,
                desc,
                link,
                imgUrl,
                success() {
                    success && success();
                },
                cancel() {
                    cancel && cancel();
                },
                fail() {
                    fail && fail();
                },
            });
        });
    }
}

// 步骤5：导出
export default new Wxsdk();
```



## 2、公众号多页面的配置（基于Vue-cli）

步骤：1、首先清空router、store文件夹、view文件夹；新建page文件夹、utils文件夹；在page文件夹中添加first文件夹里面添加文件				 main.js和App.vue，utils文件夹中添加pageConfig.js文件、page.js和template文件夹里面有index.html文件

​           2、在pageCofig.js中添加配置相关的页面信息路由比如：

```js
module.exports = [
    // 多页面路由信息
	{name:'first'},
	{name:'second'},
]
```

​		   3、在page文件中引入pageConfig文件进行遍历操作最后导出page到vue.config.js配置文件中

```js
const pageConfig = require("./pageConfig.js")
// 初始话pages 导出到vue.config.js文件中
const pages = {}
pageConfig.forEach(({name,title = defaultTitle}) => {
    pages[name] = {
        // 入口文件
        entry:`src/page/${name}/main.js`,
        // 模板
        template:'utils/template/index.html',
        title,
        chunks: ['chunk-vendors', 'chunk-common', name],
    }
})

module.exports = pages

```

​			4、最后在vue.config.js文件中配置pages

```js
const pages = require('./utils/pages');

module.exports = {
  publicPath: './',
  outputDir: process.env.VUE_APP_OUTPUTDIR,
  devServer: {},
  pages,
  productionSourceMap: false,
};

```



## 3、微信报名落地页微信支付

```js
wx.chooseWXPay({
    timestamp: 0, // 支付签名时间戳，注意微信jssdk中的所有使用timestamp字段均为小写。但最新版的支付后台生成签名使用的timeStamp字段名需大写其中的S字符
    nonceStr: '', // 支付签名随机串，不长于 32 位
    package: '', // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=***）
    signType: '', // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
    paySign: '', // 支付签名
    success: function (res) {
        // 支付成功后的回调函数
    }
});
```

步骤：1、开启微信预支付makePreparePay

​           2、该方法中首先创建一个随机的订单号和websocket

​           3、请求后台拿到微信预支付的结果，请求自定义后台，返回的结果里面包含了jsInfo（里面有timeStamp、nonceStr、paySign、signType、package）

​		   4、调用微信的wx.chooseWXPay方法传入timeStamp、nonceStr、paySign、signType、package，在成功、取消、失败的回调中执行相关回调（H5是使用jsInfo的mwebUrl进行页面的跳转，跳转到支付页面）

**webSocket的创建**

```js
// ws.js文件中
/**
 * websocket
 * @returns {Function}
 */
export default opt => {
  const { wsUrl, onOpenCbFun, onCloseCbFun, onMessageCbFun } = opt;
  const ws = new WebSocket(wsUrl);
  ws.addEventListener('open', ev => {
    onOpenCbFun && onOpenCbFun(ev);
  });
  ws.addEventListener('close', ev => {
    onCloseCbFun && onCloseCbFun(ev);
  });
  ws.addEventListener('message', ev => {
    onMessageCbFun && onMessageCbFun(JSON.parse(ev.data));
  });
};
```



```js
    /**
     * 创建webSocket
     */
    createWebSocket: function(token) {
      // 注册websocket对象
      var that = this;

      console.log('创建WebSocket');
      var WSURL =
        'wss://' +
        URL_domain +
        '/webSocketApi/subProg/weixin/wxPayStatus' +
        '?id=' +
        token;
      this.WSURL = WSURL;
      console.log('WebSocket  ==> url :' + WSURL);

      ws({
        wsUrl: WSURL,
        onOpenCbFun: function(openData) {
          console.log(new Date(), 'websocket --- openData === ', openData);
        },
        onCloseCbFun: function() {
          console.log('websocket关闭，重新连接连接并且定时查询订单');
          setTimeout(function() {
            console.error(new Date(), '关闭后3秒重新连接');
            that.createWebSocket(token);
          }, 3000);
        },
        onMessageCbFun: that.onMessageCbFun,
      });
    },
        
    /**
     * webSocket信息回调
     * @param {Object} resp
     */
    onMessageCbFun: function(resp) {
      var result = resp.returnObject;
      console.log('webSocket信息回调结果');
      console.log(result);
      var that = this;

      var status = result.status;
      Toast.clear();
      if (status == 'success') {
        console.log('结果为成功');
        that.onPaySucceed();
      } else {
        console.log('结果为失败');
        that.updateOrderStatus('fail');
      }
    },
   
     /**
     * 创建平台订单号
     */
    getRandomPlatformCode: function() {
      var timestamp = new Date().getTime();
      // var randomSrc = md5.hex_md5(Math.random().toFixed(6));
      // var random = randomSrc.substr(0, 4);
      const random = Math.random()
        .toString(16)
        .substr(2, 4);
      var platformCode = 'wxdt_' + timestamp + random;
      console.log('预支付：platformCode=' + platformCode);
      return platformCode;
    },
```



**makePreparePay方法（支付流程）**

```JS
    /* 
	* 微信预支付
    */
    makePreparePay: function() {.
      console.log('微信预支付...');

      // 产生平台支付流水号
      // 1、创建随机订单号
      var platformCode = this.getRandomPlatformCode();
      this.outTradeNo = platformCode;
      // 2、创建ws
      this.createWebSocket(platformCode); // 创建WS

      var that = this;
      var url = URL_wxPay_v3;

      var openId = this.enrollOpenId;
      var courseId = this.courseId;
      var registerOrigin = this.registerOrigin;
      var pageOrigin = this.pageOrigin;
      var mobile = this.mobile;
      var nickName = this.studentName;

      var teacherId = this.teacherId;
      var consultId = this.consultId;
      var workingWechatId = this.workingWeChatId;
      var grade = this.grade;
      var enrollNickName = this.enrollNickName;
      var wxNickName = this.enrollNickName;

      var isCourseHelper = this.isCourseHelper;

      var resultShowType = this.resultShowType;

      var extInfoJsonbObj = {
        mobile: mobile,
        nickName: nickName,
        registerOrigin: registerOrigin,
        teacherId: teacherId,
        consultId: consultId,
        workingWechatId: workingWechatId,
        grade: grade,
        enrollNickName: enrollNickName,
        isCourseHelper: isCourseHelper,
        resultShowType: resultShowType,
      };
      var extInfoJsonb = JSON.stringify(extInfoJsonbObj);
      console.log(this.courseId, courseId, 'hhhhh');
      var params = {
        openId: openId,
        courseId: courseId,
        registerOrigin: registerOrigin,
        pageOrigin: pageOrigin,
        mobile: mobile,
        nickName: nickName,
        platformCode: platformCode,
        extInfoJsonb: extInfoJsonb,
        wxNickName: wxNickName,
        grade: grade,
      };

      var option = {
        type: 'POST',
        data: params,
        url: url,
        handleError: true,
        isShowLoader: true,
        loaderText: '处理中，请稍后...',
        headers: { 'Content-Type': 'application/json;charset=UTF-8' },
      };
	 
      // 3、请求后台获取结果jsInfo里面
      t.ajax(option, function(resp) {
        console.log('发起预支付结果');
        console.log(resp);
        var errorCode = resp.errorCode;
        var errorMsg = resp.errorMsg;
        if (errorCode == '01') {
          t.showMessage(errorMsg);
          return;
        }

        var result = resp.returnObject;
        var isZeroPay = result.izp;
        var lastPayFee = result.lastPayFee;
        that.lastAmount = lastPayFee;
        console.log('最终实际支付的金额为：' + that.lastAmount);
        that.outTradeNo = result.outTradeNo;

        if (isZeroPay) {
          console.log('0元支付');
          that.lastAmount = 0;
          that.handleZeroPayResult();
        } else {
          console.log('非0元支付');
          // 3、请求后台获取结果jsInfo里面
          var jsapiPayInfo = result.jsInfo;
          // 4、handleWxChoosePay里面调用wxChoosepay方法传入参数支付
          that.handleWxChoosePay(jsapiPayInfo);
        }
      });
    },
```

