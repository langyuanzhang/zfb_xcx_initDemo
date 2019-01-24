# zfb_xcx_initDemo
支付宝小程序 初始化Demo



###### 获取授权
```js
Authorized() {
    var that = this;
    //授权
    my.getAuthCode({
    scopes: 'auth_user', // 主动授权（弹框）：auth_user，静默授权（不弹框）：auth_base
    success: function (res) {
        // 认证成功
        // 调用自己的服务端接口，让服务端进行后端的授权认证，并且种session，需要解决跨域问题
        // 发起网络请求
        my.httpRequest({
        url: 'https://www.123.com/ali_postopenid', // 该url是自己的服务地址，实现的功能是服务端拿到authcode去开放平台进行token验证
        data: {
            authCode: res.authCode
        },
        method: 'POST',
        success: (ress) => {
            // 授权成功并且服务器端登录成功
            if (ress.data.code == '200') {
            //写入缓存
            my.setStorageSync({
                key: 'userInfo',
                data: {
                user_id: ress.data.data.user_id,
                avatar: ress.data.data.avatarUrl,
                nickName: ress.data.data.nickname,
                }
            });
            //成功后处理
            var userInfo = {
                nickName: ress.data.data.nickname,
                avatar: ress.data.data.avatarUrl,
            }
            that.setData({
                userInfo: userInfo,
                iflogin: 1
            })
            my.hideLoading();
            } else {
            my.showToast({
                content: '系统繁忙',
            });
            my.hideLoading();
            }
        },
        });

    },
    fail: () => {
        // 根据自己的业务场景来进行错误处理
        my.showToast({
        content: '您已取消授权，没有权限使用该功能',
        });
        my.hideLoading();
    },
    })
},
```

###### App.js
```js
App({
  todos: [
    { text: 'Learning Javascript', completed: true },
    { text: 'Learning ES2016', completed: true },
    { text: 'Learning 支付宝小程序', completed: false },
  ],
  onLaunch() {
  },

  //全局变量（含授权后的用户信息 app.globalData.userInfo.user_id）
  globalData: {},

  //封装请求 url:地址，data：传输参数，callback：回调方法
  postDatas: function (getOrPost, url, data, callback = function () { }) {
    var that = this;
    // 发起网络请求
    my.httpRequest({
      url: url,
      data: data,
      method: getOrPost,
      success: function (res) {
        if (res.data.code != 200) {
          //服务器错误***
          if (res.data.code == 400) {
            callback(res.data);
          }
          else {
            my.showToast({
              content: res.data.msg,
            });
            return false;
          }

        }
        callback(res.data);
      }
    })
  },

});
```

###### 调用请求方法
```js
//查找
getarea(user_id) {
    var that = this;
    //请求的参数，
    var datas = {
        user_id: user_id,
    };
    //请求的路径
    var postUrl = CONFIG.API_URL.ali_getarea;//请求的路径
    app.postDatas('GET', postUrl, datas, function (res) {
        console.log(res)
    })
},
```

###### Utils/config.js
```js
const BASE = 'https://www.123.com';
const API_BASE = BASE + '/addons/shuiguo/shop/api/';

const appId = "2018020102123333";
const appKey = "RSA(SHA256)";

const CONFIG = {
  API_URL: {
    URL: BASE,
    ali_getarea:BASE +'/ali_getarea',  
  }
}

module.exports = CONFIG;
```

###### 使用富文本
```html
<import src="../../aParse/wxParse.axml" />
<view class="container">
  <view class="box">
    <view class="wxParse">
      <template is="wxParse" data="{{wxParseData:article.nodes}}" />
    </view>
  </view>
</view>
```

```js
import aParse from '../../aParse/wxParse.js';
const CONFIG = require('../../utils/config.js');
var app = getApp();
Page({
  data: {},
  onLoad() {
    var that = this;
    //请求使用指南富文本
    var datas = {};//请求的参数，
    var postUrl = CONFIG.API_URL.ali_getguide;//请求的路径
    app.postDatas('GET', postUrl, datas, function (res) {
      var article = res.data.guide;
      aParse.wxParse('article', 'html', article, that,50);
    })
  },
});
```


