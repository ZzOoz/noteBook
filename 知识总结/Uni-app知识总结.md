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

