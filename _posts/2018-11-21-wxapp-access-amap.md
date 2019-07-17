---
layout: post
title: 微信小程序之：小程序接入高德地图SDK
date: 2018-11-21 18:32:24.000000000 +09:00
tag: 微信小程序
---



#### 前言
最近在捣腾小程序，想开发一个自己的小程序，过一把独立开发的瘾。

自己设计，自己搞数据，自己找图标，自己决定要或者不要什么功能，完全跟着心走，目前体验下来，感觉还是蛮爽的。
![image1.png](/images/posts/wx-app-amap/image1.png)

虽然我是一个App开发，但我还是蛮注重用户体验的，我会去考虑应用的使用场景，使用习惯，不断的去调整，去优化。这次开发的小程序，从配色、图标选择、UI设计，到页面结构、功能等都花了很多心思。

小程序名字：**杭州地铁通**

欢迎小伙伴们体验，吐槽。

扯远了，回归主题！

#### 小程序地图SDK原理

微信小程序开发，可以使用地图组件`map`，来进行地图显示、定位、显示大头针等基本功能，具体可以看官方文档：[微信小程序map组件](https://developers.weixin.qq.com/miniprogram/dev/component/map.html)。

`map`组件只提供一些基本的“`硬件`”，还需要“`大脑`”来驱动这些“`硬件`”。这个“`大脑`”就是腾讯地图、百度地图和高德地图提供的小程序SDK。其实这些SDK只是提供了一些网络请求接口，请求这些接口，能获得对应结构的网络数据，再驱动`map`组件来渲染。

**比如线路规划**：从A到B，只要确定起点经纬度和终点经纬度，传给SDK，SDK里就会发起网络请求，获得最佳路线，返回给你从A到B所途经的转折点经纬度数组，你把得到的经纬度数组传给`map`组件，`map`就能绘制路线。

#### 高德SDK接口列表

高德地图提供的接口有：

| 方法 | 用途 |
| ------ | ------ |
| getPoiAround(Object) | 获取周边的POI。<br>querykeywords、location、querytypes 字段于 1.1.0 版本新增。 |
| getRegeo(Object) | 获取地址描述信息。<br> location 字段于 1.1.0 版本新增。|
|getWeather(Object)|获取天气情况（实时和预报）。type、city字段于 1.1.0 版本新增。|
|getStaticmap(Object)|获取静态的地图图片。<br>getStaticmap(Object) 方法于 1.1.0 版本新增。|
|getInputtips(Object)|获取提示词。<br>getInputtips(Object) 方法于 1.2.0 版本新增。|
|getDrivingRoute(Object)|获取驾车路线。<br>getDrivingRoute(Object) 方法于 1.2.0 版本新增。|
|getWalkingRoute(Object)|获取步行路线。<br>getWalkingRoute(Object) 方法于 1.2.0 版本新增。|
|getTransitRoute(Object)|获取公交路线。<br>getTransitRoute(Object) 方法于 1.2.0 版本新增。|
|getRidingRoute(Object)|获取骑行路线。<br>getRidingRoute(Object) 方法于 1.2.0 版本新增。|

具体接口用法，参考高德官方文档: [AMapWX基本方法](https://lbs.amap.com/api/wx/reference/core)

### 接入SDK：

**首先：**下载SDK[下载地址](https://lbs.amap.com/api/wx/download)
下载完后，直接将`amap-wx.js`文件拖到工程libs目录下，这个目录随便建的，一定要放到`miniprogram`文件夹内，任何位置都可以，不然找不到。如图：
![image2.png](/images/posts/wx-app-amap/image2.png)

**引用：**
1.在js文件开头引入并声明SDK对象
![image3.png](/images/posts/wx-app-amap/image3.png)

2.把不相关代码删掉后，获得高德当前位置天气情况，大概就是这样子的：
```
// miniprogram/pages/home/home.js
var amapFile = require('../../libs/amap-wx.js');
Page({
    /**
     * 页面的初始数据
     * 本地图："../../images/hz_metro_map.png"
     * 网络图：
     */
    data: {
        weatherInfo: null,
    },
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function() {
        this._getWeather();
    },
    /**
     * 获取天气信息
     */
    _getWeather: function() {
        var that = this;
        var myAmapFun = new amapFile.AMapWX({ key: '高德地图key' });
        myAmapFun.getWeather({
            success: function (data) {
                //成功回调
                console.log(data)
            },
            fail: function (info) {
                //失败回调
                console.log(info)
            }
        })
    }
})
```

这样就拿到了高德的数据，超级简单有木有。

#### 遇到的坑
虽然很简单，但是使用起来，还是发现有坑。使用`getPoiAround`接口时，发现没有分页功能。其实高德服务度接口是有分页功能的，但是小程序SDK里并没有把`page`参数暴露出来，坑啊。我使用的SDK版本是:`sdkversion: "1.2.0"`

解决办法：在`amap-wx.js`文件里，找到`getPoiAround`接口，往参数里添加一个`page`字段即可，如下图：
![image4.png](/images/posts/wx-app-amap/image4.png)

这样就能正常分页了。

到此就说完了，也就那么回事。

欢迎扫上面小程序码，体验我的小程序，多多吐槽，我将继续优化。谢谢！