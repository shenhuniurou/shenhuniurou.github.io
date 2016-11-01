---
layout: post
title: 微信小程序开发笔记2
category: 技术分享
tags: 微信小程序
---


## 文件结构
框架程序包含一个描述整体程序的 app 和多个描述各自页面的 page。一个框架程序主体部分由三个文件组成，必须放在项目的根目录，如下：

| 文件 | 必填 | 作用 |
|:------|:----|:-----|
|app.js|是|小程序逻辑|
|app.json|是|小程序公共设置|
|app.wxss|否|小程序公共样式表|

一个框架页面由四个文件组成，分别是：

| 文件类型 | 必填 | 作用 |
|:------|:----|:-----|
|js|是|页面逻辑|
|json|否|页面配置|
|wxss|否|页面样式表|
|wxml|是|页面结构|


**注意：为了方便开发者减少配置项，我们规定描述页面的这四个文件必须具有相同的路径与文件名。**

## 配置
我们使用app.json文件来对微信小程序进行全局配置，决定页面文件的路径、窗口表现、设置网络超时时间、设置多 tab 等。
以下是一个包含了所有配置选项的简单配置app.json：

```json
{
  "pages": [
    "pages/index/index",
    "pages/logs/logs"
  ],
  "window": {
    "navigationBarTitleText": "Demo", 
    "navigationBarBackgroundColor": "#019DD6",
    "navigationBarTextStyle": "white" 
  },
  "tabBar": { 
    "list": [{
      "pagePath": "pages/index/index",
      "text": "首页"
    }, {
      "pagePath": "pages/logs/logs",
      "text": "日志"
    }]
  },
  "networkTimeout": { 
    "request": 10000,
    "downloadFile": 10000
  },
  "debug": true 
}
```

其中，pages里面第一个页面将会是小程序启动时显示的那个页面，所以配置的时候要注意，一般我们会把启动页splash写在第一行。
app.json 配置项列表说明：

| 属性 | 类型 | 必填 | 描述 |
|:------|:----|:-----|:----|
|pages|Array|是|设置页面路径
|window|Object|否|设置默认页面的窗口表现
|tabBar|Object|否|设置底部 tab 的表现
|networkTimeout|Object|否|设置网络超时时间
|debug|Boolean|否|设置是否开启 debug 模式

各个属性的详细说明：

### pages
接收一个数组，每一项都是字符串，来指定小程序由哪些页面组成。每一项代表对应页面的【路径+文件名】信息，**数组的第一项代表小程序的初始页面**。小程序中新增/减少页面，都需要对 pages 数组进行修改。
文件名不需要写文件后缀，因为框架会自动去寻找路径.json、.js、.wxml、.wxss的四个文件进行整合。

### window
用于设置小程序的状态栏、导航条、标题、窗口背景色。

| 属性 | 类型 | 默认值 | 描述 |
|:------|:----|:-----|:----|
|navigationBarBackgroundColor|HexColor|#000000|导航栏背景颜色，如"#000000"
|navigationBarTextStyle|String|white|导航栏标题颜色，仅支持 black/white
|navigationBarTitleText|String|Wechat|导航栏标题文字内容
|backgroundColor|HexColor|#ffffff|窗口的背景色
|backgroundTextStyle|String|dark|下拉背景字体、loading 图的样式，仅支持 dark/light
|enablePullDownRefresh|Boolean|false|是否开启下拉刷新，详见[页面相关事件处理函数](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/page.html?t=1476197491005#页面相关事件处理函数)。

### tabBar

如果我们的小程序是一个多 tab 应用（客户端窗口的底部有tab栏可以切换页面），那么我们可以通过 tabBar 配置项指定 tab 栏的表现，以及 tab 切换时显示的对应页面。
tabBar 是一个数组，**只能配置最少2个、最多5个 tab**，tab 按数组的顺序排序。

**属性说明：**

| 属性 | 类型 | 必填 | 默认值 | 描述 |
|:------|:----|:-----|:-----|:----|
|color|HexColor|是||tab 上的文字默认颜色
|selectedColor|HexColor|是||tab 上的文字选中时的颜色
|backgroundColor|HexColor|是||tab 的背景色
|borderStyle|String|否|black|tabbar上边框的颜色， 仅支持 black/white
|list|Array|是||tab 的列表，详见 list 属性说明，最少2个、最多5个 tab

其中 list 接受一个数组，数组中的每个项都是一个对象，其属性值如下：

| 属性 | 类型 | 必填 | 描述 |
|:------|:----|:-----|:----|
|pagePath|String|是|页面路径，必须在 pages 中先定义
|text|String|是|tab 上按钮文字
|iconPath|String|是|图片路径，icon 大小限制为40kb
|selectedIconPath|String|是|选中时的图片路径，icon 大小限制为40kb

![](http://offfjcibp.bkt.clouddn.com/tabBar.jpg)

### networkTimeout
可以设置各种网络请求的超时时间。

**属性说明：**

| 属性 | 类型 | 必填 | 描述 |
|:------|:----|:-----|:----|
|request|Number|否|[wx.request](https://mp.weixin.qq.com/debug/wxadoc/dev/api/network-request.html?t=1476197491005)的超时时间，单位毫秒
|connectSocket|Number|否|[wx.connectSocket](https://mp.weixin.qq.com/debug/wxadoc/dev/api/network-socket.html?t=1476197491005)的超时时间，单位毫秒
|uploadFile|Number|否|[wx.uploadFile](https://mp.weixin.qq.com/debug/wxadoc/dev/api/network-file.html?t=1476197491005#wxuploadfileobject)的超时时间，单位毫秒
|downloadFile|Number|否|[wx.downloadFile](https://mp.weixin.qq.com/debug/wxadoc/dev/api/network-file.html?t=1476197491005#wxdownloadfileobject)的超时时间，单位毫秒

### debug
可以在开发者工具中开启 debug 模式，在开发者工具的控制台面板，调试信息以 info 的形式给出，其信息有Page的注册、页面路由、数据更新、事件触发。可以帮助开发者快速定位一些常见的问题。

### page.json
每一个小程序页面也可以使用.json文件来对本页面的窗口表现进行配置。 页面的配置比app.json全局配置简单得多，只是设置 app.json 中的 window 配置项的内容，页面中配置项会覆盖 app.json 的 window 中相同的配置项。页面的.json只能设置window相关的配置项，以决定本页面的窗口表现，所以无需写window这个键，如：

```json
{
  "navigationBarBackgroundColor": "#ffffff",
  "navigationBarTextStyle": "black",
  "navigationBarTitleText": "微信接口功能演示",
  "backgroundColor": "#eeeeee",
  "backgroundTextStyle": "light"
}
```

## 注册程序
### App
`App()`函数用来注册一个小程序。接受一个 object 参数，其指定小程序的生命周期函数等。
**object参数说明：**

|属性|类型|描述|触发时机
|:--|:--|:----|:----|
|onLaunch|Function|生命周期函数--监听小程序初始化|当小程序初始化完成时，会触发 onLaunch（全局只触发一次）
|onShow|Function|生命周期函数--监听小程序显示|当小程序启动，或从后台进入前台显示，会触发 onShow
|onHide|Function|生命周期函数--监听小程序隐藏|当小程序从前台进入后台，会触发 onHide
|其他|Any|开发者可以添加任意的函数或数据到 Object 参数中，用`this`可以访问||

**前台、后台定义：**

当用户点击左上角关闭，或者按了设备Home键离开微信，小程序并没有直接销毁，而是进入了后台；当再次进入微信或再次打开小程序，又会从后台进入前台。只有当小程序进入后台一定时间，或者系统资源占用过高，才会被真正的销毁。

**示例代码：**

```javascript
App({
  onLaunch: function() { 
    // Do something initial when launch.
  },
  onShow: function() {
      // Do something when show.
  },
  onHide: function() {
      // Do something when hide.
  },
  globalData: 'I am global data'
})
```

**App.prototype.getCurrentPage()**
getCurrentPage()函数用于获取当前[页面的实例](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/page.html?t=1476766391616)。
**getApp()**
我们提供了全局的getApp()函数，可以获取到小程序实例。

```javascript
// other.js
var appInstance = getApp()
console.log(appInstance.globalData) // I am global data
```

**注意：**
- `App()`必须在`app.js`中注册，且不能注册多个。
- 不要在定义于App()内的函数中调用`getApp()`，使用`this`就可以拿到`app`实例。
- 不要在`onLaunch`的时候调用getCurrentPage()，此时`page`还没有生成。
- 通过`getApp()`获取实例之后，不要私自调用生命周期函数。

## 注册页面
### Page
`Page()`函数用来注册一个页面。接受一个 object 参数，其指定页面的初始数据、生命周期函数、事件处理函数等。
**object 参数说明：**

|属性|类型|描述|
|:--|:--|:----|
|[data](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/page.html?t=1476259068206#初始化数据)|Object|页面的初始数据
|onLoad|Function|生命周期函数--监听页面加载
|onReady|Function|生命周期函数--监听页面初次渲染完成
|onShow|Function|生命周期函数--监听页面显示
|onHide|Function|生命周期函数--监听页面隐藏
|onUnload|Function|生命周期函数--监听页面卸载
|onPullDownRefresh|Function|页面相关事件处理函数--监听用户下拉动作
|其他|Any|开发者可以添加任意的函数或数据到 object 参数中，用`this`可以访问

**示例代码：**

```javascript
//index.js
Page({
  data: {
    text: "This is page data."
  },
  onLoad: function(options) {
    // Do some initialize when page load.
  },
  onReady: function() {
    // Do something when page ready.
  },
  onShow: function() {
    // Do something when page show.
  },
  onHide: function() {
    // Do something when page hide.
  },
  onUnload: function() {
    // Do something when page close.
  },
  onPullDownRefresh: function() {
    // Do something when pull down
  },
  // Event handler.
  viewTap: function() {
    this.setData({
      text: 'Set some data for updating view.'
    })
  }
})
```

### 生命周期函数
- onLoad: 页面加载
  - 一个页面只会调用一次。
  - 参数可以获取wx.navigateTo和wx.redirectTo及<navigator/>中的 query。
- onShow:页面显示
  - 每次打开页面都会调用一次。
- onReady: 页面初次渲染完成
  - 一个页面只会调用一次，代表页面已经准备妥当，可以和视图层进行交互。
  - 对界面的设置如wx.setNavigationBarTitle请在onReady之后设置。详见[生命周期](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/page.html?t=1476197492481#生命周期)
- onHide: 页面隐藏
  - 当navigateTo或底部tab切换时调用。
- onUnload: 页面卸载
  - 当redirectTo或navigateBack的时候调用。

### 页面相关事件处理函数
- onPullDownRefresh: 下拉刷新监听用户下拉刷新事件。
  - 需要在config的[window
](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/config.html?t=1476766392127#window)选项中开启enablePullDownRefresh。
  - 当处理完数据刷新后，[wx.stopPullDownRefresh
](https://mp.weixin.qq.com/debug/wxadoc/dev/api/ui-other.html?t=1476766392127)可以停止当前页面的下拉刷新。

##### 事件处理函数
除了初始化数据和生命周期函数，Page 中还可以定义一些特殊的函数：事件处理函数。在渲染层可以在组件中加入[事件绑定](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/event.html?t=1476766392127)，当达到触发事件时，就会执行 Page 中定义的事件处理函数。

**示例代码：**

```java
<view bindtap="viewTap"> click me </view>
Page({
  viewTap: function() {
    console.log('view tap')
  }
})
```

### Page.prototype.setData()
`setData`函数用于将数据从逻辑层发送到视图层，同时改变对应的`this.data`的值。
**注意：**
- 直接修改 this.data 无效，无法改变页面的状态，还会造成数据不一致。
- 单次设置的数据不能超过1024kB，请尽量避免一次设置过多的数据。

### setData()参数格式
接受一个对象，以 key，value 的形式表示将 this.data 中的 key 对应的值改变成 value。其中 key 可以非常灵活，以数据路径的形式给出，如array[2].message，a.b.c.d，并且不需要在 this.data 中预先定义。

**示例代码：**

```xml
<!--index.wxml-->
<view>{{text}}</view>
<button bindtap="changeText"> Change normal data </button>
<view>{{array[0].text}}</view>
<button bindtap="changeItemInArray"> Change Array data </button>
<view>{{object.text}}</view>
<button bindtap="changeItemInObject"> Change Object data </button>
<view>{{newField.text}}</view>
<button bindtap="addNewField"> Add new data </button>
```

```javascript
//index.js
Page({
  data: {
    text: 'init data',
    array: [{text: 'init data'}],
    object: {
      text: 'init data'
    }
  },
  changeText: function() {
    // this.data.text = 'changed data'  // bad, it can not work
    this.setData({
      text: 'changed data'
    })
  },
  changeItemInArray: function() {
    // you can use this way to modify a danamic data path
    this.setData({
      'array[0].text':'changed data'
    })
  },
  changeItemInObject: function(){
    this.setData({
      'object.text': 'changed data'
    });
  },
  addNewField: function() {
    this.setData({
      'newField.text': 'new data'
    })
  }
})
```

### 文件作用域
在 JavaScript 文件中声明的变量和函数只在该文件中有效；不同的文件中可以声明相同名字的变量和函数，不会互相影响。
通过全局函数[getApp()
](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/app.html?t=1476259068206#getapp)可以获取全局的应用实例，如果需要全局的数据可以在App()中设置，如：

```javascript
// app.js
App({
  globalData: 1
})
// a.js
// The localValue can only be used in file a.js.
var localValue = 'a'
// Get the app instance.
var app = getApp()
// Get the global data and change it.
app.globalData++
// b.js
// You can redefine localValue in file b.js, without interference with the localValue in a.js.
var localValue = 'b'
// If a.js it run before b.js, now the globalData shoule be 2.
console.log(getApp().globalData)
```

### 模块化
我们可以将一些公共的代码抽离成为一个单独的 js 文件，作为一个模块。模块只有通过module.exports或者exports才能对外暴露接口。
**需要注意的是：**
- exports是module.exports的一个引用，因此在模块里边随意更改exports的指向会造成未知的错误。所以我们更推荐开发者采用module.exports来暴露模块接口，除非你已经清晰知道这两者的关系。
- 小程序目前不支持直接引入node_modules, 开发者需要使用到node_modules时候建议拷贝出相关的代码到小程序的目录中。

```javascript
// common.js
function sayHello(name) {
  console.log(`Hello ${name} !`)
}
function sayGoodbye(name) {
  console.log(`Goodbye ${name} !`)
}
module.exports.sayHello = sayHello
exports.sayGoodbye = sayGoodbye
```

在需要使用这些模块的文件中，使用`require(path)`将公共代码引入
```javascript
var common = require('common.js')
Page({
  helloMINA: function() {
    common.sayHello('MINA')
  },
  goodbyeMINA: function() {
    common.sayGoodbye('MINA')
  }
})
```

### 数据绑定
Page中的数据再wxml中访问只需要用`{{}}`，例如：
```java
<!--wxml-->
<view> {{message}} </view>
// page.js
Page({
  data: {
    message: 'Hello MINA!'
  }
})
```

### 列表渲染
```java
<!--wxml-->
<view wx:for="{{array}}"> {{item}} </view>
// page.js
Page({
  data: {
    array: [1, 2, 3, 4, 5]
  }
})
```

### 条件渲染
```xml
<!--wxml-->
<view wx:if="{{view == 'WEBVIEW'}}"> WEBVIEW </view>
<view wx:elif="{{view == 'APP'}}"> APP </view>
<view wx:else="{{view == 'MINA'}}"> MINA </view>
// page.js
Page({
  data: {
    view: 'MINA'
  }
})
```

### 模板
```javascript
<!-- 列表模板start -->
<template name="items">
	<navigator url="../detail/detail?id={{id}}" class-hover="navigation-hover">
		<view class="list-img">
			<image class="in-image" src="{{image}}" background-size="cover" model="scaleToFill"></image>
		</view>
		<view class="list-info">
			<view class="list-title"> {{title}} </view>
			<view class="list-desc"> {{desc}} </view>
			<view class="list-time"> {{time}} </view>
		</view>
	</navigator>
</template>
<!-- 列表模板end -->

<!-- 循环输出数据 -->
<view wx:for="{{newslist}}" class="list">
	<template is="items" data="{{...item}}"/>
</view>
```

模板拥有自己的作用域，只能使用data传入的数据。

### 事件
```javascript
<view bindtap="add"> {{count}} </view>
Page({
  data: {
    count: 1
  },
  add: function(e) {
    this.setData({
      count: this.data.count + 1
    })
  }
})
```

#### 事件详解

事件分类:事件分为冒泡事件和非冒泡事件：
- 冒泡事件：当一个组件上的事件被触发后，该事件会向父节点传递。
- 非冒泡事件：当一个组件上的事件被触发后，该事件不会向父节点传递。

WXML的冒泡事件列表：

|类型|触发条件|
|:---|:-------|
|touchstart|手指触摸
|touchmove|手指触摸后移动
|touchcancel|手指触摸动作被打断，如来电提醒，弹窗
|touchend|手指触摸动作结束
|tap|手指触摸后离开
|longtap|手指触摸后，超过350ms再离开
**注：除上表之外的其他组件自定义事件都是非冒泡事件，如[<form/>
](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/component/form.md?t=1476197492610)的submit事件，[<input/>
](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/component/input.md?t=1476197492610)的input事件，[<scroll-view/>
](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/component/scroll-view.md?t=1476197492610)的scroll事件，(详见各个[组件](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/component/index.md?t=1476197492610))**

#### 事件绑定
事件绑定的写法同组件的属性，以 key、value 的形式。

- key 以bind或catch开头，然后跟上事件的类型，如bindtap、catchtouchstart
- value 是一个字符串，需要在对应的 Page 中定义同名的函数。不然当触发事件的时候会报错。

`bind`事件绑定不会阻止冒泡事件向上冒泡，catch事件绑定可以阻止冒泡事件向上冒泡。
如在下边这个例子中，点击 inner view 会先后触发handleTap3和handleTap2(因为tap事件会冒泡到 middle view，而 middle view 阻止了 tap 事件冒泡，不再向父节点传递)，点击 middle view 会触发handleTap2，点击 outter view 会触发handleTap1。

```xml
<view id="outter" bindtap="handleTap1">
  outer view
  <view id="middle" catchtap="handleTap2">
    middle view
    <view id="inner" bindtap="handleTap3">
      inner view
    </view>
  </view>
</view>
```

##### 事件对象
如无特殊说明，当组件触发事件时，逻辑层绑定该事件的处理函数会收到一个事件对象。

**事件对象的属性列表：**

|属性|类型|说明|
|:--|:---|:---|
|[type](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/event.html?t=1476197493113#type)|String|事件类型
|[timeStamp](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/event.html?t=1476197493113#timeStamp)|Integer|事件生成时的时间戳
|[target](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/event.html?t=1476197493113#target)|Object|触发事件的组件的一些属性值集合
|[currentTarget](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/event.html?t=1476197493113#currenttarget)|Object|当前组件的一些属性值集合
|[touches](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/event.html?t=1476197493113#touches)|Array|触摸事件，触摸点信息的数组
|[detail](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/event.html?t=1476197493113#detail)|Object|额外的信息

- touches
touches是一个触摸点的数组，每个触摸点包括以下属性：

|属性|说明|
|:-----|:---|
|pageX,pageY|距离文档左上角的距离，文档的左上角为原点 ，横向为X轴，纵向为Y轴
|clientX,clientY|距离页面可显示区域（屏幕除去导航条）左上角距离，横向为X轴，纵向为Y轴
|screenX,screenY|距离屏幕左上角的距离，屏幕左上角为原点，横向为X轴，纵向为Y轴

- detail
特殊事件所携带的数据，如表单组件的提交事件会携带用户的输入，媒体的错误事件会携带错误信息，详见[组件](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/wxml/wxml-component.md?t=1476197492610)定义中各个事件的定义。

### API文档
[API文档地址](https://mp.weixin.qq.com/debug/wxadoc/dev/api/?t=1476197491461)
