# Uni-app知识总结



## 1、Scroll-view组件

主要的属性和方法：

1、scroll-with-animation：在设置滚动条高度时使用动画效果

2、scroll-top：设置竖向滚动条的高度

3、scroll-into-view：根据所设置的滚动方向，滚动到这个子元素id的位置

4、upper-threshold：距顶部/左边多远时（单位px），触发 scrolltoupper 事件

5、scroll-x：允许横向滚动

6、scroll-y：允许纵向滚动

7、@scrolltoupper：滚动到顶部/左边，会触发 scrolltoupper 事件

8、@scrolltolower：滚动到底部/右边，会触发 scrolltolower 事件

9、@scroll：滚动时触发，event.detail = {scrollLeft, scrollTop, scrollHeight, scrollWidth, deltaX, deltaY}



## 2、音频组件控制（使用场景：微信聊天中的语音收听功能可以使用）

使用[uni.createInnerAudioContext()](https://uniapp.dcloud.io/api/media/audio-context?id=createinneraudiocontext)会返回一个innerAudioContext对象，在返回的对象中有属性和方法

其中最主要的有onStart、onEnded、onStop，当到达某一个阶段时方法触发方法

```js
this.AUDIO.onEnded((res)=>{
	console.log(res,'语音自然播放')
	this.playMsgid=null;
});
```



## 3、录音管理（使用场景：微信聊天中的语音功能可以使用）

使用[uni.getRecorderManager()](https://uniapp.dcloud.io/api/media/record-manager?id=getrecordermanager)可以获取到获取**全局唯一**的录音管理器 `recorderManager`。（在h5中录音可能得不到实现）

`recorderManager`中有多个方法和属性

```js
start:开始录音里面有option参数
// 其中option中有duration（时间长度）、sampleRate（采样）、format（音频格式）等
pause:暂停录音
resume:继续录音
stop:停止录音
onStart：开始录音事件
onPause：暂停录音事件

onStop：停止录音事件
//其中有一个e的参数，参数中的e.tempFilePath是录音文件的临时存放路径
还有很多....
```



## 4、修改Popup组件的弹出位置

Popup组件中只存在center、bottom、dialog这些弹出类型，如果想要在左边或者右边弹出popup的时候需要修改原来的组件

方法如下：

1、下载popup组件，引入在Popup.js中找到定义的type，加入所需要的类型

```js
const config = {
	// 顶部弹出
	top:'top',
	// 底部弹出
	bottom:'bottom',
	// 居中弹出
	center:'center',
	// 右侧弹出，这里是我自己自定义加入的
	right:'right',
	// 消息提示
	message:'top',
	// 对话框
	dialog:'center',
	// 分享
	share:'bottom',
}
```

2、在Popup.vue组件中找到methods，定义right的方法

```js
/**
 * 右侧弹出样式处理
 */
right() {
	this.popupstyle = 'right	'
	this.ani = ['slide-right']
	this.transClass = {
		'position': 'fixed',
		'top': 60,
		'right': 0,
		// 'bottom': 0
	}
},
```

3、然后在uni-popup.vue下面的style中增加样式

```js
.uni-right-mask{
	opacity: 1;
}
.right{
    right:0
}
```

4、最后在使用组件的时候使用mode="right"就ok了

```js
<uniPopup ref="select" type="right" v-if="activeTab === 3">
	里面是内容
</uniPopup>
```



## 5、构建聊天界面功能



**5.1、获取聊天记录功能（无数据请求）**

步骤：

1、首先在onLoad中调用getMsgList方法，加载所有的Msg

2、在getMsgList会通过网络请求加载数据（可以写写死数据先），后面通过判断是否为user的信息以及是否为image信息进行对图片的宽高进行设置调用setPicSize方法

3、将data中的msgList赋值

4、最后一种情况是需要将位置滚动到底部，可以将scroll-view组件的scrollTop设置为9999滚动到底部设置scrollAnimation为true进行动画滚动

```js
// 加载初始页面消息
getMsgList(){
	// 消息列表
	//假设这里的msg通过网络请求获取到了为list
	// 获取消息中的图片,并处理显示尺寸
	for(let i=0;i<list.length;i++){
		if(list[i].type=='user'&&list[i].msg.type=="img"){
			list[i].msg.content = this.setPicSize(list[i].msg.content);
			this.msgImgList.push(list[i].msg.content.url);
		}
	}
	this.msgList = list;
	// 滚动到底部
	this.$nextTick(function() {
		//进入页面滚动到底部
		this.scrollTop = 9999;
		this.$nextTick(function() {
			this.scrollAnimation = true;
		});
					
	});
},
//处理图片尺寸，如果不处理宽高，新进入页面加载图片时候会闪
setPicSize(content){
// 让图片最长边等于设置的最大长度，短边等比例缩小，图片控件真实改变，区别于aspectFit方式。
	let maxW = uni.upx2px(350);//350是定义消息图片最大宽度
	let maxH = uni.upx2px(350);//350是定义消息图片最大高度
	// console.log(maxH,maxW,'12312313123131232131231')
	if(content.w>maxW||content.h>maxH){
	let scale = content.w/content.h;
					
	content.w = scale>1?maxW:maxH*scale;
	content.h = scale>1?maxW/scale:maxH;
	// console.log(scale,content.w,content.h,'ccccccccc')
	}
	return content;
},
```



**5.2、实现发送文字功能**

步骤：

1、首先定义一个sendText方法，这个方法里面会处理emoji表情、文本信息返回，

2、往后通过sendMsg方法设置时间、消息id（用于scroll-view跳转到该id中去），将所有的信息填充到msg的固定格式中

3、调用screenMsg方法，里面是有多个不同的switch分支可以判断是text、img、voice、红包等，最后将msg加入到list中遍历信息

```js
			
// 第一步******************
			// 发送文字消息
			sendText(){
				this.hideDrawer();//隐藏抽屉
				if(!this.textMsg){
					return;
				}
				let content = this.replaceEmoji(this.textMsg);
				let msg = {text:content}
				this.sendMsg(msg,'text');
				this.textMsg = '';//清空输入框
			},
			//替换表情符号为图片
			replaceEmoji(str){
				let replacedStr = str.replace(/\[([^(\]|\[)]*)\]/g,(item, index)=>{
					console.log("item: " + item);
					for(let i=0;i<this.emojiList.length;i++){
						// 遍历第一层的emoji表情（里面有几组数组）
						let row = this.emojiList[i];
						// 获取到一栏数组后再次遍历获取单个表情
						for(let j=0;j<row.length;j++){
							// 获取到某一个表情如果和item（即通过正则替换后只剩文字的表情）
							let EM = row[j];
							// 如果alt和item相同
							if(EM.alt==item){
								//在线表情路径，图文混排必须使用网络路径，请上传一份表情到你的服务器后再替换此路径 
								//比如你上传服务器后，你的100.gif路径为https://www.xxx.com/emoji/100.gif 则替换onlinePath填写为https://www.xxx.com/emoji/
								let onlinePath = 'https://s2.ax1x.com/2019/04/12/'
								// 图片拼接 返回路径
								let imgstr = '<img src="'+onlinePath+this.onlineEmoji[EM.url]+'">';
								console.log("imgstr: " + imgstr);
								return imgstr;
							}
						}
					}
				});
				// 返回文本replaceStr
				return '<div style="display: flex;align-items: center;word-wrap:break-word;">'+replacedStr+'</div>';
			},
                
// 第二步*******************
			// 发送消息
			sendMsg(content,type){
				//实际应用中，此处应该提交长连接，模板仅做本地处理。
				// 获取时间、设置最后的lastId用于跳转，填充msg的固定格式
				var nowDate = new Date();
				let lastid = this.msgList[this.msgList.length-1].msg.id;
				lastid++;
				let msg = {type:'user',msg:{id:lastid,time:nowDate.getHours()+":"+nowDate.getMinutes(),type:type,userinfo:{uid:0,username:"大黑哥",face:"/static/img/face.jpg"},content:content}}
				// 发送消息
				this.screenMsg(msg);
				// 定时器模拟对方回复,三秒
				setTimeout(()=>{
					lastid = this.msgList[this.msgList.length-1].msg.id;
					lastid++;
					msg = {type:'user',msg:{id:lastid,time:nowDate.getHours()+":"+nowDate.getMinutes(),type:type,userinfo:{uid:1,username:"售后客服008",face:"/static/img/im/face/face_2.jpg"},content:content}}
					// 本地模拟发送消息
					this.screenMsg(msg);
				},3000)
			},
          		
// 第三步**********************
			// 接受消息(筛选处理)筛选信息的处理方式
			screenMsg(msg){
				//从长连接处转发给这个方法，进行筛选处理
				if(msg.type=='system'){
					// 系统消息
					switch (msg.msg.type){
						case 'text':
							this.addSystemTextMsg(msg);
							break;
						case 'redEnvelope':
							this.addSystemRedEnvelopeMsg(msg);
							break;
					}
				}else if(msg.type=='user'){
					// 用户消息
					switch (msg.msg.type){
						case 'text':
							this.addTextMsg(msg);
							break;
						case 'voice':
							this.addVoiceMsg(msg);
							break;
						case 'img':
							this.addImgMsg(msg);
							break;
						case 'redEnvelope':
							this.addRedEnvelopeMsg(msg);
							break;
					}
					console.log('用户消息');
					//非自己的消息震动
					if(msg.msg.userinfo.uid!=this.myuid){
						console.log('振动');
						uni.vibrateLong();
					}
				}
				this.$nextTick(function() {
					// 滚动到底 滚动到最新的msg的id上去
					this.scrollToView = 'msg'+msg.msg.id
				});
			},
                
// 最后一步****************
             // 添加文字消息到列表通过遍历操作更新
			addTextMsg(msg){
				this.msgList.push(msg);
			},
```



## 5.3、实现拍照发送图片功能

步骤：

1、点击chooseImage方法触发getImage方法

2、getImage方法中调用uni.chooseImage方法选择图片，其中的sourceType可以选择“alumb”或者“camera”，在chooseImage的成功回调中遍历选择的图片使用uni.getImageInfo获取图片信息，最后在getImageInfo的成功回调中调用sendMsg(msg)方法

3、sendMsg也和之前一样获取时间、id号、最后形成msg的固定格式，调用screenMsg方法

4、screenMsg方法通过switch选中img类型，调用addImgMsg方法，addImgMsg里面调用setPicSize，插入信息到msgImgList和msgList中  

```js
// 第一步 通过点击触发****************
// 选择图片发送
            chooseImage(){
                this.getImage('album');
            },
            //拍照发送
            camera(){
                this.getImage('camera');
            },
    
    
// 第二部 调用getImage里面会调用uni.chooseImage和getImageInfo*******************
			//选照片 or 拍照
			getImage(type){
				this.hideDrawer();
				uni.chooseImage({
					// 选择图片是拍照还是从相册中找
					sourceType:[type],
					sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
					success: (res)=>{
						for(let i=0;i<res.tempFilePaths.length;i++){
							uni.getImageInfo({
								// 获取图片信息
								src: res.tempFilePaths[i],
								// 成功回调
								success: (image)=>{
									console.log(image.width);
									console.log(image.height);
									let msg = {url:res.tempFilePaths[i],w:image.width,h:image.height};
									this.sendMsg(msg,'img');
								}
							});
						}
					}
				});
			},
                
// 第三步 调用sendMsg方法更新时间、id和形成固定msg格式 调用screenMsg（msg）方法到达img分支中
			// 接受消息(筛选处理)筛选信息的处理方式
			screenMsg(msg){
				//从长连接处转发给这个方法，进行筛选处理
				if(msg.type=='system'){
					// 系统消息
					switch (msg.msg.type){
						case 'text':
							this.addSystemTextMsg(msg);
							break;
						case 'redEnvelope':
							this.addSystemRedEnvelopeMsg(msg);
							break;
					}
				}else if(msg.type=='user'){
					// 用户消息
					switch (msg.msg.type){
						case 'text':
							this.addTextMsg(msg);
							break;
						case 'voice':
							this.addVoiceMsg(msg);
							break;
                          // 图片到达这里**************************
						case 'img':
							this.addImgMsg(msg);
							break;
						case 'redEnvelope':
							this.addRedEnvelopeMsg(msg);
							break;
					}
					console.log('用户消息');
					//非自己的消息震动
					if(msg.msg.userinfo.uid!=this.myuid){
						console.log('振动');
						uni.vibrateLong();
					}
				}
				this.$nextTick(function() {
					// 滚动到底 滚动到最新的msg的id上去
					this.scrollToView = 'msg'+msg.msg.id
				});
			},
// 第四部调用addImgMsg方法传入msg，调用setPicSize和插入数据到msgImgList和msgList中
			// 添加图片消息到列表
			addImgMsg(msg){
				// 更改图片的大小
				msg.msg.content = this.setPicSize(msg.msg.content);
				// 插入图片的url地址到imagelist中
				this.msgImgList.push(msg.msg.content.url);
				// 插入信息
				this.msgList.push(msg);
			},
```



## 穿透原本组件的样式（比如是scroll-view、textarea等）

在开发的过程中我们想要修改uni-app原生的组件的样式，那么我们可以在vue文件下面新写下一个style标签（在uni-app中style可以有多个）

```js
// 在textarea中穿透原生组件的样式进行修改，注意一定要使用deep和另开启一个style标签
<style scoped>
	/deep/ .uni-textarea-textarea{
		box-sizing: border-box;
		padding:20rpx
	}
</style>
```



## 如何在原生的导航栏中加入自己想要的button样式？

uni-app中自己写的原生导航searchInput和按钮button，可以在page.json文件中进行配置

```js
		{
			"path": "pages/pourOut/pourOut",
			"style": {
                // 导航栏的标题
				"navigationBarTitleText": "心知岛",
				// app中的导航是怎么样的，值得注意的是在h5如果没有配置自己的导航栏是会默认使用app的导航栏的。如果 h5 节点没有配置，默认会使用 app-plus 下的配置。配置了 h5 节点，则会覆盖 app-plus 下的配置。
                  "app-plus": {
                      // 导航栏的样式
					"titleNView": {
                         // type的类型有三种float、default、和透明（单词忘了）
						"type":"default",
                            // 原生搜索框是
						"searchInput":{
                            // align是位置左中右
							"align":"left",
                                // 圆角
							"borderRadius":"20rpx",
							"placeholder":"搜索",
                               // 在pourOut组件中使用了原生导航栏并且使用了onNavigationBarSearchInputClicked事件，那么一定要使用"disabled":true否则无法生效
							"disabled":true   
						},
                            // 这里是配置button的样式可以是原生就有的也可以自定义，具体参考官网
						"buttons":[
							{	"index":"0",
								"background":"rgba(0,0,0,0)",
								"text":"\ue601",
								"fontSrc": "/static/pourout/index/iconfont.ttf",
								"align":"right",
								"fontSize":"40rpx"
							}
						]
					}
				}
			}
		},
```



## 原生导航栏的onNavigationBarSearchInputClicked不生效如何解决？以及如何在原生的导航栏中使用自己想要的字体图标？

**1、当我们在使用了原生导航栏的组件中使用了onNavigationBarSearchInputClicked事件，点击了搜索框是不会生效的，必须要加上disable：true才能生效**

```json
		{
			"path": "pages/pourOut/pourOut",
			"style": {
				"navigationBarTitleText": "心知岛",
				"app-plus": {
					"titleNView": {
						"type":"default",
						"searchInput":{
							"align":"left",
							"borderRadius":"20rpx",
							"placeholder":"搜索",
                               // 在pourOut组件中使用了原生导航栏并且使用了onNavigationBarSearchInputClicked事件，那么一定要使用"disabled":true否则无法生效
							"disabled":true   
						},
						"buttons":[
							{	"index":"0",
								"background":"rgba(0,0,0,0)",
								"text":"\ue601",
								"fontSrc": "/static/pourout/index/iconfont.ttf",
								"align":"right",
								"fontSize":"40rpx"
							}
						]
					}
				}
			}
		},
```

**2、如果想要使用字体图标的话要依照下面的步骤（以阿里的iconfont为例）**

感谢参考：https://segmentfault.com/a/1190000018720472?utm_source=tag-newest

（1）首先进入官网登录 > 点击图标管理 > 找到unicode直接复制在线的链接代码

（2）将代码放到App.vue的style公共样式中去，然后根据iconfont.css来看图标的字体是什么，然后在前面加上u就可以了

```js
		{
			"path": "pages/pourOut/pourOut",
			"style": {
				"navigationBarTitleText": "心知岛",
				"app-plus": {
					"titleNView": {
						"type":"default",
						"searchInput":{
							"align":"left",
							"borderRadius":"20rpx",
							"placeholder":"搜索",
							"disabled":true
						},
						"buttons":[
							{	"index":"0",
								"background":"rgba(0,0,0,0)",
                             	   // 这里使用了字体图标，因为我在App.vue中引入了iconfont.css文件的@font-face
								"text":"\ue601",
								"fontSrc": "/static/pourout/index/iconfont.ttf",
								"align":"right",
								"fontSize":"40rpx"
							}
						]
					}
				}
			}
		},
```



## uni-app中的css变量

当navigationStyle设为custom或titleNView设为false时，原生导航栏不显示，此时要注意几个问题：

- 非H5端，手机顶部状态栏区域会被页面内容覆盖。这是因为窗体是沉浸式的原因，即全屏可写内容。uni-app提供了状态栏高度的css变量

  --status-bar-height

  ，如果需要把状态栏的位置从前景部分让出来，可写一个占位div，高度设为css变量。

  ```html
  <template>
    <view>
        <view class="status_bar">
            <!-- 这里是状态栏 -->
        </view>
        <view> 状态栏下的文字 </view>
    </view>
  </template>    
  <style>
    .status_bar {
        // 这里的--staus-bar-height是指状态栏的高度
        height: var(--status-bar-height);
        width: 100%;
    }
  </style>
  ```

uni-app 提供内置 CSS 变量

| CSS变量             | 描述                   | App                                                          | 小程序 | H5                   |
| :------------------ | :--------------------- | :----------------------------------------------------------- | :----- | :------------------- |
| --status-bar-height | 系统状态栏高度         | [系统状态栏高度](http://www.html5plus.org/doc/zh_cn/navigator.html#plus.navigator.getStatusbarHeight)、nvue注意见下 | 25px   | 0                    |
| --window-top        | 内容区域距离顶部的距离 | 0                                                            | 0      | NavigationBar 的高度 |
| --window-bottom     | 内容区域距离底部的距离 | 0                                                            | 0      | TabBar 的高度        |

**注意：**

- `var(--status-bar-height)` 此变量在微信小程序环境为固定 `25px`，在 App 里为手机实际状态栏高度。
- 当设置 `"navigationStyle":"custom"` 取消原生导航栏后，由于窗体为沉浸式，占据了状态栏位置。此时可以使用一个高度为 `var(--status-bar-height)` 的 view 放在页面顶部，避免页面内容出现在状态栏。
- 由于在H5端，不存在原生导航栏和tabbar，也是前端div模拟。如果设置了一个固定位置的居底view，在小程序和App端是在tabbar上方，但在H5端会与tabbar重叠。此时可使用`--window-bottom`，不管在哪个端，都是固定在tabbar上方。
- 目前 nvue 在App端，还不支持 `--status-bar-height`变量，替代方案是在页面onLoad时通过uni.getSystemInfoSync().statusBarHeight获取状态栏高度，然后通过style绑定方式给占位view设定高度。下方提供了示例代码

具体可以参考官网：[https://uniapp.dcloud.io/frame?id=css%e5%8f%98%e9%87%8f](https://uniapp.dcloud.io/frame?id=css变量)



## uni-app的分包机制

对于小程序来说当一个代码的包大于2MB的时候是不能预览的，那么我们就学要使用分包机制将主包分割这样能加快运行速度的同时也能分包加载，当然当我们使用打包的时候也可压缩代码，减小体积

**分包机制的实现（在package.json文件中）**

```js
	// 分包机制的实现
	"subPackages":[
		{	
			// 我的咨询，比如我要分包的是在page中的my文件下的pourOut文件
			"root":"pages/my/pourOut",
			"pages":[
				// App-我的-倾听师
				{	
                     // 里面的path是pourOut里面的路径
					"path":"myListener/Become",
					"style": {
						"navigationBarTitleText": "APP-成为倾听师"
					}
				},
			]
		}
	]
```



**在分包中报错小程序添加分包的时候报错 “pages不应该在分包 subPackages[*] 中”如何解决？**

在我们添加小程序新的分包的时候很容易会出现这个问题“pages *** 不应该在分包 subPackages[*] 中”

我们如果通过编辑器右键添加page的话，主包中会自动加入page路径，此时我们再去创建新的分包如果没有去掉主包路径就会报错。

解决方案就是去掉主包中的路径就可以了