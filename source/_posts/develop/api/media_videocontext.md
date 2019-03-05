---
title: 视频组件控制
header: develop
nav: api
sidebar: media_videocontext
---
createVideoContext
---
**解释：** 创建并返回 video 上下文 `videoContext` 对象。通过 videoId 跟一个 video 组件绑定，通过它可以操作一个 video 组件。

**参数：** videoId

**videoContext 对象的方法列表：**

|方法 | 参数 | 说明 |
|---- | ---- | ---- |
|play  |  无 |  播放  |
|pause |  无  | 暂停  |
|seek  |  position   | 跳转到指定位置（单位：s）    |
|sendDanmu |  danmu  | 发送弹幕，danmu 包含两个属性 text、color。  |
|requestFullScreen  | 无  | 进入全屏  |
|exitFullScreen | 无 |  退出全屏|
|showStatusBar | 无 |  显示状态栏，仅在iOS全屏下有效。|
|hideStatusBar | 无 |  隐藏状态栏，仅在iOS全屏下有效。|

**示例：**

```html
<view>
    <video id="myVideo" src="https://example.baidu.com/xxxx"></video>
</view>
```
```js
const myVideo = swan.createVideoContext('myVideo');
myVideo.play();
```

#### 错误码

**Andriod**

|错误码|说明|
|--|--|
|202|解析失败，请检查参数是否正确。 |
|1001|执行失败|

**iOS**

|错误码|说明|
|--|--|
|202|解析失败，请检查参数是否正确&。|