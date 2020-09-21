## HM小程序总结（基于uni-app开发）

### **0.基本**：

vuex中index.js的获取用户信息

```js
actions:{
    // 获取用户基本信息
    async getUserInfo({ commit }) {
      // 抽取storeage信息
      let userInfo = uni.getStorageSync('userInfo')
      // 不存在那么就使用login的Api进行登录，登录成功返回code码
      if (!userInfo) {
        const [, {code}] = await uni.login({
          provider: 'weixin'
        })
        // 使用code码请求返回unionId和openId 设置storage
        const { unionid, openid } = await ajax({
          method: 'GET',
          url: URL_login,
          data: code
        })
        userInfo = { unionid, openid }
        uni.setStorageSync('userInfo', userInfo)
      }
       // 触发同步mutation传值
      commit('setUserInfo', userInfo)
    },
},
mutations: {
    // 触发setUserInfo 扩展运算符对象补充
 setUserInfo(state, payload) {
    state.userInfo = {
       ...state.userInfo,
       ...payload
     }
  },
}
```





### 1、积分商城

#### 1.1、进度条功能

html页面中

```html
<div class="wish__progress">
    <!-- 这里绑定的是一个计算属性 class="wish__progress--active"是一个进度条样式-->
    <div class="wish__progress--active" :style="progressStyleStr"></div>
  	<div class="wish__text">{{ progressText }}</div>
</div>
```

css

```css
.wish__progress {
  position: relative;
  border-radius: 10rpx;
  margin-top: 16rpx;
  width: 100%;
  background-color: rgb(238, 238, 238);
}
.wish__progress--active {
  border-radius: 10rpx;
  height: 30rpx;
  background-image: linear-gradient(
    to right,
    rgb(255, 215, 96),
    rgb(255, 158, 6)
  );
}
.wish__text {
  position: absolute;
  top: 0;
  left: 24rpx;
  right: 24rpx;
  bottom: 0;
  font-size: 20rpx;
  line-height: 30rpx;
}
```

script中

```js
//新增一个计算属性绑定style和文本属性
computed(){
	progressStyleStr(){
        // 获取当前是宝石还是金豆数量
        const current = this.likeGoods.isGem ? this.Gems:this.beans
        // 获取当前需要兑换的总数量
        const {price} = this.likeGoods
        // 计算百分比
        const percent = Math.min((current/price)*100,100)
        // 返回宽度
        return `width:${percent}%`
    }
    progressText() {
      const current = this.likeGoods.isGem ? this.gems : this.beans
      const { price } = this.likeGoods
      const unit = this.likeGoods.isGem ? '宝石' : '金豆'
      return `${current} / ${price}${unit}`
    },
}
```

#### 1.2、添加心愿单功能

步骤：

​	1、首先需要遍历所有的商品，构成一个goodList，通过遍历操作将所有的good遍历出来显示

​    2、这里的遍历可以建立一个组件good-item，绑定key、item本身、active（是否是我喜欢的显示心形图标）

​    3、最后绑定事件点击进入兑换click和喜欢事件like（参数是item的id）

```js
// 封装一个组件good-item遍历goods列表
<GoodsItem
   v-for="item in goods"
   :key="item.id"
   :item="item"
   :active="item.id === likeGoods.id"
   @click="handleGoodsClick(item.id, item.isGem)"
   @like="handleGoodsLike(item.id)"
/>
 

// 方法handleGoodsClick
    async handleGoodsClick(id, isNeedLogin) {
      // 如果是宝石就需要登录 如果是金豆就不需要登录
      isNeedLogin && (await this.checkSignUp())
      uni.navigateTo({
        url: `/pages/MallConfirm/MallConfirm?id=${id}`
      })
    },
// 方法handleGoodsLike
    async handleGoodsLike(exchangeGoodsId) {
      await ajax({
        method: 'GET',
        url: URL_goodsLike,
        data: {
          unionId: this.userInfo.unionid,
          exchangeGoodsId
        }
      })
      uni.showToast({
        icon: 'none',
        title: '已加入心愿单'
      })
      // 加入心愿单之后需要刷新心愿单的状态
      // 更新likeGoodTips（获取good需要做的步骤）和likeGood（喜欢商品的信息）
      this.getAssetsStat()
    },

// 方法getAssetsStat
    async getAssetsStat() {
      // 网络请求获取data 更新信息
      const data = await ajax({
        method: 'GET',
        url: URL_taskStat,
        data: {
          unionId: this.userInfo.unionid,
          userId: this.childId
        }
      })
      const {
        beansNum,
        gemNum,
        exchangeGoodsId,
        goodsType,
        coverImage,
        needIntegral,
        goodsName,
        needFansNum,
        needFriendsNum,
        ontimeNum,
        finishBaseWorkNum,
        finishExpandWorkNum,
        selectionNum
      } = data
      this.beans = beansNum
      this.gems = gemNum
      // 这里的likeGoodsTip是
      this.likeGoodsTips = {
        needFansNum,
        needFriendsNum,
        ontimeNum,
        finishBaseWorkNum,
        finishExpandWorkNum,
        selectionNum
      }
      // 喜欢的商品信息
      this.likeGoods = goodsTranslator({
        exchangeGoodsId,
        goodsType,
        coverImage,
        needIntegral,
        goodsName
      })
    },
```

#### 1.3、根据选择的商品类型更新商品分类

步骤：1、每当用户通过picker组件选择商品类型的时候（选择金豆类型或者宝石类型）那么就会刷新商品类型

​		   2、可以在picker中定义一个事件@change=“handleCategoryChange”

```js
// 通过定义一个方法 handleCategoryChange 每次最后都会调用getGoodsList方法 参数可为空 也可以是1或2金豆或者宝石
	   <picker
          @change="handleCategoryChange"
          :value="categoryIndex"
          :range="categorys"
          range-key="v"
        >
          <div class="category">
            <div class="category__text">{{ categorys[categoryIndex].v }}</div>
            <div class="category__icon"></div>
          </div>
        </picker>

//方法handleCategoryChange
    async getGoodsList(goodsFlag = '') {
      const goods = await ajax({
        method: 'GET',
        url: URL_goodsList,
        data: {
          goodsFlag
        }
      })
      this.goods = goods.map(item => goodsTranslator(item))
    },
        
    /**
     * @see 改变商品列表类型时候的事件
     */
    handleCategoryChange(e) {
      const { value } = e.detail
      this.categoryIndex = value
      this.getGoodsList(this.categorys[value].k)
    },
```



#### 1.4、设置地址功能

步骤： 1、与修改个人信息一样，使用input绑定电话、姓名等，使用picker的mode=“region”模式选择地区

​			2、首先在created使用getAddress方法，里面通过网络请求返回的data中查看是否有之前设置过的地址

​            3、如果没有那么直接返回return，如果有那么就将这个json字符串解析出来展示数据

​            4、保存功能是将所有的input数据和picker作为参数通过网络请求传递

```js
    async getAddress() {
      const {
        receiverName,
        receiverTelPhone,
        receiverAddressJsonb
      } = await ajax({
        method: 'GET',
        url: URL_addressGet,
        data: {
          unionId: this.userInfo.unionid
        }
      })
      // 1.首先先获取先前的address地址 没有直接返回
      if (!receiverAddressJsonb) {
        return
      }
      // 2.有地址则将值赋值
      this.name = receiverName
      this.phone = receiverTelPhone
      const { province, city, district, other_info } = JSON.parse(
        receiverAddressJsonb
      )
      // address地址就是详细地址信息
      this.address = other_info
      // region是地区信息
      this.region = [province, city, district]
    },

async handleSave() {
      await ajax({
        method: 'POST',
        url: URL_addressSave,
        data: {
        // 将所有的数据拆分传递参数 post请求
          unionId: this.userInfo.unionid,
          province: this.region[0],
          city: this.region[1],
          district: this.region[2],
          other_info: this.address,
          receiverTelPhone: this.phone,
          receiverName: this.name
        }
      })
      uni.navigateBack()
    }
```



### 2、个人中心功能

**功能：展示设置个人信息、作品、证书、跳转联系班主任、任务中心、积分商城、修改个人信息**

#### 2.1、展示个人信息功能

步骤：首先在Vuex中的index.js的action中新建方法**getChildList**，判断是否成功如果成功获取用户状态、证书、作品，**这里统一封装成initPage在onShow调用**，后面通过getter的childInfo进行渲染，可以跨组件使用信息了，可以通过open-data组件展示用户信息

```js
// vuex中的index.js中的action****************************//
    /**
     * @see 获取孩子列表信息
     */
    async getChildList({ dispatch, commit, state }) {
      // 1.首先触发getUserInfo获取unionId和openId的对象
      await dispatch('getUserInfo')
      // 2.请求url 参数unionId
      const list = await ajax({
        method: 'GET',
        url: URL_getUserList,
        data: {
          unionId: state.userInfo.unionid
        }
      })
      // 为空返回false
      if (!list.length) {
        return false
      }
      // 同步触发childList和ChildId
      commit('setChildList', list)
      commit('setChildId')
      return true
    },
        
// vuex中的index.js的mutation*********************************//
    setChildList(state, list) {
      state.childList = list
    },
    setChildId(state, childId) {
      // storageChildId是获取用户之前已经登录切换的id
      let storageChildId = uni.getStorageSync('childId')
      // 通过之前获取到的childList中使用map方法返回一个用userId构建的数组
      // 如果里面不包含storageChildId的值 将其初始话为0
      if (!state.childList.map(i => i.userId).includes(storageChildId)) {
        storageChildId = 0
      }

      // 如果storageChildId不生效那么就直接获取childList的第一个
      const { userId: firstChildId } = state.childList[0] || {}
      const id = childId || storageChildId || firstChildId || 0
      if (id === 0) return
      state.childId = id
      // 设置storeage
      uni.setStorageSync('childId', id)
    },
        
// vuex中的index.js的getter********************************//
getters: {
    // 返回对应id的child信息通过childeList，childList在action中已经获取了 这里是获取usreId相同的信息，这样就可以跨组件传递
    childInfo: state =>
      state.childList.find(i => i.userId === state.childId) || {},
},
```



#### 2.2、跳转联系班主任、任务中心、积分商城、作品、证书功能

步骤：使用uni.navigateTo方法跳转指定路由，也可以传入id跳转指定页面

```js
uni.navigateto({
	url: '/pages/TaskCenter/TaskCenter'
})
```

#### 2.3、联系班主任中的复制微信号、拨打电话功能

步骤：复制微信号使用的是uni.setClipboardData

```js
// 使用uni.setClipboardData
handleCopyWechatID() {
      uni.setClipboardData({
        data: this.wxNo
    })
 },
```

步骤：拨打电话使用的是uni.makePhoneCall

```js
// 使用uni.makePhoneCall
handlePhoneCall() {
      uni.makePhoneCall({
        phoneNumber: this.teacherMobile
      })
    },
```



#### 2.4、个人信息修改

##### 2.4.1、头像的修改

步骤：使用[uni.chooseImage](https://uniapp.dcloud.io/api/media/image?id=chooseimage)和[uni.uploadFile](https://uniapp.dcloud.io/api/request/network-file?id=uploadfile)

```js
    // 上传头像方法  
    async handleAvatarClick() {
      // 1.使用uni.chooseImage选中一张图片
      const [, { tempFilePaths }] = await uni.chooseImage({
        count: 1
      })
      // 2.使用uploadFile对服务器进行请求返回data
      const [, { statusCode, data }] = await uni.uploadFile({
        url: URL_uploadImage,
        filePath: tempFilePaths[0],
        name: 'file'
      })
      // 3.判断
      if (statusCode !== 200) {
        uni.showToast({ icon: 'none', title: '上传发生错误' })
        return
      }
      const { errorCode, errorMsg, returnObject } = JSON.parse(data)
      if (errorCode !== '00') {
        uni.showToast({ icon: 'none', title: errorMsg })
        return
      }
      // 4.更新
      this.headPortrait = returnObject
    },
```

##### 2.4.2、其他（姓名、昵称、年级、性别等）

步骤：通过input标签的v-model双向绑定数据（如：姓名、昵称、密码等），通过picker来修改年级、性别等，通过handleSave方法修改

```vue
<input class="input" v-model="academicName" />
// 修改性别、年级使用picker组件，change事件赋值、value是选中的index索引、range是数据源、range-key是选择框里面的选项
<picker
    @change="handleSexChange"
    :value="sexIndex"
    :range="sexList"
    range-key="v"
  >
    <div class="input input__picker">{{ sexList[sexIndex].v }}</div>
</picker>
```

修改函数handSave

```js
async handleSave() {
   const {
      childId,
      headPortrait,
      academicName,
      userNickName,
      age,
      gradeIndex,
      sexIndex,
      password
 } = this
      const grade = this.gradeList[gradeIndex].k
      const sex = this.sexList[sexIndex].k
      await ajax({
        method: 'POST',
        url: URL_updateUserInfo,
        data: {
          userId: childId,
          headPortrait,
          academicName,
          userNickName,
          age,
          grade,
          sex,
          password
        }
      })
      uni.showToast({
        icon: 'success',
        title: '修改成功'
      })
    },
```



### 3、兑换记录功能（MallRecord）

步骤：1、首先遍历数组通过v-for="（v,i）in ['金豆明细'，’兑换记录‘]"分成两个tab

​           2、设置一个变量active绑定值，0代表金豆明细、1代表兑换记录，当i和active相等的时候具有tab__item--active属性

​		   3、使用template模板active=0是显示金豆明细详情、active=1时显示兑换记录详情，进行数据渲染

```vue
// 步骤1和2
<div
  v-for="(v, i) in ['金豆明细', '兑换记录']"
  :key="i"
  class="tab__item"
  :class="active === i && 'tab__item--active'"
  @click="handleTab(i)"
 >
    {{ v }}
</div>

// 方法handleTab 切换tab
handleTab(val){
    this.active = val
}
```

```js
 // 步骤3
 <template v-if="active === 0">
      <div v-if="!beans.length" class="no-record">
        <div class="no-record__icon no-record__icon--bean"></div>
        <div class="no-record__text">暂无金豆明细~</div>
        <div class="no-record__btn" @click="toTaskCenter">去赚金豆</div>
      </div>
      <div v-else class="panel">
        <div v-for="item in beans" :key="item.id" class="bean__item">
          <div>
            // 这里时使用了drpType过滤器
            <div class="bean__title">{{ item.drpType | drpType }}</div>
            <div class="bean__time">{{ item.createTimestamp }}</div>
          </div>
          <div class="bean__price">
            <image class="bean__unit" :src="beanSrc" mode="aspectFit" />
            <div class="bean__number">{{ item.awardNum }}</div>
          </div>
        </div>
      </div>
    </template>
    <template v-if="active === 1">
      <div v-if="!exchanges.length" class="no-record">
        <div class="no-record__icon no-record__icon--record"></div>
        <div class="no-record__text">暂无兑换记录~</div>
      </div>
      <div v-else class="panel">
        <div
          v-for="(item, index) in exchanges"
          :key="index"
          class="exchange__item"
        >
          <image
            class="exchange__cover"
			// 这里也使用了过滤器
            :src="item.cover_image | coverSrc(item.goods_type)"
            mode="aspectFill"
          />
          <div class="exchange__info">
            <div class="exchange__title">{{ item.goods_name }}</div>
            <div class="exchange__detail">
              <div class="exchange__content">
                <div class="exchange__price">
                  <div class="exchange__number">{{ item.need_integral }}</div>
                  <image
                    v-if="item.goods_type === 'minAppBeans'"
                    class="exchange__unit"
                    :src="beanSrc"
                    mode="aspectFit"
                  />
                  <image
                    v-else
                    class="exchange__unit"
                    :src="gemSrc"
                    mode="aspectFit"
                  />
                </div>
                <div class="exchange__time">{{ item.create_timestamp }}</div>
              </div>
              <div class="exchange__btn">
                  // 这里也使用了过滤器
                {{ item.material_object_exchange_status | exchangeStatus }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </template>


// 过滤器的使用
  filters: {
      // 这里的val是指在|前面的值 通过过滤器进行进一步处理
      // 这里的通过maps[vals]可以返回maps里面的值
    drpType(val) {
      const maps = {
        0: '完成6节入门课奖励',
        1: '粉丝奖励',
        2: '好友奖励',
        3: '每日签到奖励'
      }
      return maps[val] || ''
    },
      // 这里的val是指在|前面的值 通过过滤器进行进一步处理
      // 这里的通过maps[vals]可以返回maps里面的值
    exchangeStatus(val) {
      const maps = {
        application: '申请处理中',
        processing: '申请处理中',
        delivery: '发货中',
        confirm: '确认收货'
      }
      return maps[val] || ''
    },
     // 这里是通过image的src拼接过滤
    coverSrc(val, goodsType) {
      const path =
        goodsType === 'minAppBeans' ? ORIGIN : `${ORIGIN}/subProg/upload/goods`
      return `${path}/${val}`
    }
  },
```

