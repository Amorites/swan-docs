---
title: 授权流程
header: develop
nav: third
sidebar: pro
---
第三方平台（TP）是指帮助小程序运营者进行开发、管理、运营小程序并从此获取收益的平台；可在<a href="https://smartprogram.baidu.com/mappconsole/main/login ">小程序首页</a>进行注册，成为**企业类型**的智能小程序后即可创建第三方平台。**小程序运营者，可以一键授权给第三方平台，通过第三方平台来完成业务。**
<!-- 目前智能小程序第三方平台为邀请制，如需申请，请发邮件至：smartprogram_bd@baidu.com -->
<p>小程序授权给第三方平台的技术实现流程如下图：
![图片](../../../img/third.png)



## 授权流程详解

下面对各API和机制进行介绍（特别注意，所有API调用需要验证调用者IP地址。只有在第三方平台申请时填写的白名单IP地址列表内的IP地址，才能合法调用，其他一律拒绝）：

|顺序|功能	|API的作用|
|---|---|---|
|1|推送 ticket|	出于安全考虑，在第三方平台创建审核通过后，小程序TP平台服务器每隔10分钟会向第三方的消息接收地址推送一次 ticket，用于获取第三方平台接口调用凭据,接收到后必须直接返回字符串 success。|
|2|获取第三方平台 access_token|	第三方平台通过自己的 client_id 和 ticket（每10分钟推送一次的安全 ticket ）来获取自己的接口调用凭据（access_token）|
|3|获取预授权码 pre_auth_code|	第三方平台通过自己的接口调用凭据（access_token）来获取用于授权流程准备的预授权码（pre_auth_code）。|
|4|引导小程序管理员对第三方平台进行授权|	根据 client_id, pre_auth_code, redirect_uri生 成授权二维码。|
|5|获取授权码 authorization_code	|引导小程序管理员扫码授权，在回调 URL 参数中返回授权码 (authorization_code) 和过期时间。|
|6|使用授权码换小程序的接口调用凭据和授权信息|	通过授权码和自己的接口调用凭据（access_token），换取小程序的接口调用凭据（access_token和用于前者快过期时用来刷新它的 refresh_token）和授权信息（授权了哪些权限等信息）。|
|7|获取（刷新）授权小程序的接口调用凭据|	通过自己的接口调用凭证 access_token 和 refresh_token 来刷新小程序的接口调用凭据。|
|8|获取小程序基础信息|	当小程序对第三方进行授权、取消授权、更新授权时，将通过事件推送告诉开发者。|

### 1、 推送ticket协议

在第三方平台创建审核通过后，小程序 TP 平台服务器会向其“授权事件接收 URL ”每隔10分钟定时推送 ticket ,用于获取第三方平台调用凭据，接收到后必须直接返回字符串 success。

post 数据示例：
```json
{
  "Nonce": "4464221",
  "TimeStamp": "1535551395",
  "Encrypt": "/X6AhuNc6ObfSXaA0/DI2SH4m4NAhaaLrYDxyeARIXnGkVYwjhSz/cK8ZLvPcbsHi6d8spK+brQwHw5+t55o4NuZj09x1TT2G6hkCQwU/R9ejDIv9yFI292XwkTMVkQ6dnZIwpvRbPmsV2EX6cagRl3C5KVlFme+6b4SS3aDat7dmpNyxjb6MdYCdZnzp4CmgbramsS0BtA/tnNgqKZ1VA==",
  "MsgSignature": "ba394c04acece6c2c0edee058c5dddf82474b8ee"
}
```
其中 Encrypt 参数需要使用 AES 秘钥解密，解密算法见文档最后<a href="https://smartprogram.baidu.com/docs/develop/third/apppage/#附录">附录</a>部分，解密后数据如下：
```json
{
    "Ticket" : "ticket",  //ticket内容
    "FromUserName" : "SmartAPP",    //固定为SmartApp
    "CreateTime" : "1413192605",    //时间戳
    "MsgType" : "ticket",           //固定为ticket
    "Event" : "push"                //固定为push
}
```
参数说明

|参数名	|描述|
|---|---|
|Ticket|	ticket内容|
|FromUserName|	固定为SmartApp|
|CreateTime|	时间戳|
|MsgType|	固定为ticket|
|Event|	固定为push|
### 2、 获取第三方平台access_token

第三方平台 access_token 是第三方平台的接口调用凭据，也叫做令牌（access_token）。每个令牌是有效期一个月，且令牌的调用次数有限，请第三方平台做好令牌的管理，在令牌过期之前进行刷新。

接口调用请求说明：
```
GET https://openapi.baidu.com/public/2.0/smartapp/auth/tp/token?client_id=OdxUiUVpVxH2Ai7G02cIjXGnnnMEUntD&ticket=8e329bc7e5fc432740d2e7e76a39c0e3
```
参数说明

|参数名	|类型|	是否必须|	描述|
|---|---|---|
|client_id|	string|	是|	分配给第三方平台key id。|
|ticket	|string	|是|	第三方平台服务器推送的 ticket，此 ticket 会定时推送，具体请见“1、 推荐 ticket协议”。|
返回值说明

|字段名|	类型|	描述|
|---|---|---|
|access_token|	string|	获取到的接口调用凭证|
|expires_in	|int|	凭证有效时间，单位：秒。|
|scope|	string|	拥有的权限说明|
错误情况下:

|字段名|	类型|	描述|
|---|---|---|
|errno|	int|	错误码；关于错误码的详细信息请参考 http://developer.baidu.com/wiki/index.php?title=docs/oauth/error 。|
|msg|	string|	错误描述信息，用来帮助理解和解决发生的错误。|
返回值示例
```json
{
    "errno":0,
    "msg":"success",
    "data" :{
        "access_token": "42.12835b16c449ae00f7d9a61570516b4f.2592000.1535536744.aPk4Eh420Yt-2JdTBB_F-34gJWz93WxN4e9rQhN",
        "expires_in": 2592000,
        "scope": "smartapp_tp_smtapp_common public"
    }
}
```
出错时返回
```json
{
    "errno": 502, 
    "msg":"Client authentication failed"
}
```
### 3、获取预授权码pre_auth_code

openapi 用于获取预授权码，预授权码用于小程序授权时的第三方平台方安全验证。

接口调用请求说明
```
GET https://openapi.baidu.com/rest/2.0/smartapp/tp/createpreauthcode?access_token=42.89210dcaa616b575cdca56f978cefbc2.2592000.1535617875.Wf0l2sXgdy5SabS_wP00-34gJWz93WxN4e9rQhN
```
参数说明

|参数名	|类型	|是否必须|	描述|
|---|---|---|---|
|access_token|	string|	是|	TP的access_token，可参考：获取第三方平台接口调用凭据tp_access_token|
返回值说明

|字段名|	类型|	描述|
|---|---|---|
|pre_auth_code|	string|	预授权码|
|expires_in	|int|	凭证有效时间，单位：秒，默认20分钟。|
返回值示例
```json
{
    "errno": 0,
    "msg": "success",
    "data": {
        "pre_auth_code": "c210YXBwMjAzMTQxODMzMThiMDlhMzhlZmEzMGM2MjAzY2NjMGQ5MTBlNGNmZWI1",
        "expires_in": 1200
    }
}
```
错误情况下:

|字段名|	类型|	描述|
|---|---|---|
|error	|string	|错误码；关于错误码的详细信息请参考 http://developer.baidu.com/wiki/index.php?title=docs/oauth/error 。|
|error_description|	string|	错误描述信息，用来帮助理解和解决发生的错误。|
### 4、引导小程序管理员对第三方平台进行授权

将用户浏览器重定向的如下授权页面，生成授权二维码 。
页面地址
```
https://smartprogram.baidu.com/mappconsole/tp/authorization?client_id=OdxUiUVpVxH2Ai7G02cIjXGnnnMEUntD&redirect_uri=http://cp01-xstp-otp5-4.epc.baidu.com:8220/mappconsole/main/apps&pre\_auth\_code=c210YXBwMTk4NjM0Mjg1NGFhMTRiMDMyNWQyMGE3ZGE0OWQ1ODE0OWQ1OGM0YzY4
```
参数说明

|参数名称|	类型|	是否必须|	描述|
|---|---|---|---|
|client_id|	string|	是|	分配给第三方的 client id|
|pre_auth_code	|string	|是	|预授权码|
|redirect_uri|	string|	是	|回调URI|
### 5、授权回调，获取授权码 authorization_code

授权流程完成后，授权页会自动跳转进入回调地址，并在 URL 参数中返回授权码和过期时间`(redirect_url?authorization_code=xxx&expires_in=3600)`

### 6、使用授权码换小程序的接口调用凭据和授权信息

openapi 用于使用授权码换取小程序的授权信息，并换取 access_token和refresh_token。 授权码的获取，需要在用户在第三方平台授权页中完成授权流程后，在回调 URI 中通过 URL 参数提供给第三方平台方。
**说明**：
小程序可以自定义选择部分权限授权给第三方平台，因此第三方平台开发者需要通过该接口来获取小程序具体授权了哪些权限，而不是简单地认为自己声明的权限就是小程序授权的权限。

接口调用请求说明
```
GET https://openapi.baidu.com/rest/2.0/oauth/token?access_token=ACCESS_TOKEN&code=AUTH_CODE&grant_type=app_to_tp_authorization_code
```
参数说明

|参数名|	类型|	是否必须|	描述|
|---|---|---|---|
|access_token|	string|	是|	TP的access_token，第三方平台接口调用凭据|
|code|	string|	是	|授权码|
|grant_type|	string	|是|	固定字符串： app_to_tp_authorization_code|
返回值说明

|字段名|	类型|	描述|
|---|---|---|
|access_token|	string|	授权小程序的接口调用凭据|
|refresh_token|	string|	接口调用凭据刷新令牌，有效期10年，使用后失效|
|expires_in|	int|	Access Token的有效期，单位：秒，默认1小时|
返回值示例
```js
{
    access_token: "45.1d4146fdea08ab043a2d291b0e2d86ca.3600.1536147748.C1Q38_EEfQjeNhZ1diO5d7hX8Dx_-mVMFst84kTtF6Sn4je",
    refresh_token: "46.4d79bd6882af6d2bb238b2f851f3a00f.315360000.1851504148.C1Q38_EEfQjeNhZ1diO5d7hX8Dx_-mVMFst84kTtF6Sn4je",
    expires_in: 3600
}
```
错误情况下:

|字段名|	类型|	描述|
|---|---|---|
|error|	string|	错误码；关于错误码的详细信息请参考 http://developer.baidu.com/wiki/index.php?title=docs/oauth/error|
|error_description|	string|	错误描述信息，用来帮助理解和解决发生的错误|
### 7、获取（刷新）授权小程序的接口调用凭据

该API用于在授权方令牌（access_token）失效时，可用刷新令牌（refresh_token）获取新的令牌。请注意，此处TP的access_token有效期一个月，开发者需要自行进行token的缓存，避免token的获取次数达到每日的限定额度(额度限定尚未做)。当换取refresh_token后建议保存。

接口调用请求说明
```
GET https://openapi.baidu.com/rest/2.0/oauth/token?access_token=ACCESS_TOKEN&refresh_token=REFRESH_TOKEN&grant_type=app_to_tp_refresh_token
```
参数说明

|参数名	|类型	|是否必须|	描述|
|---|---|---|---|
|access_token	|string	|是	|TP的access_token，第三方平台接口调用凭据|
|refresh_token|	string|	是|	接口调用凭据刷新令牌，有效期10年，使用后失效|
|grant_type	|string|	是	|固定字符串： app_to_tp_refresh_token|
返回值说明

|字段名|	类型|	描述|
|---|---|---|
|access_token|	string|	授权小程序的接口调用凭据|
|refresh_token|	string	|接口调用凭据刷新令牌|
|expires_in	|int|	小程序的Access Token的有效期，单位：秒，默认1小时|
返回值示例
```js
{
    access_token: "45.c1cb2c4ddd225536ca80d70875a9f60d.3600.1536148028.FiKQ1VSLjMjS7uaJZlCdbOcjcasQ-mVMFst84kTtF6Sn4je",
    refresh_token: "46.045cabb3f09efe6c8fa570de94a41773.315360000.1851504428.FiKQ1VSLjMjS7uaJZlCdbOcjcasQ-mVMFst84kTtF6Sn4je",
    expires_in: 3600
}
```
错误情况下:

|字段名	|类型	|描述|
|---|---|---|
|error	|string|	错误码；关于错误码的详细信息请参考 http://developer.baidu.com/wiki/index.php?title=docs/oauth/error|
|error_description|	string|	错误描述信息，用来帮助理解和解决发生的错误|
### 8、获取小程序基础信息

接口调用请求说明
```
GET https://openapi.baidu.com/rest/2.0/smartapp/app/info?access_token=ACCESS_TOKEN
```
参数说明

|参数名	|类型|	是否必须|	描述|
|---|---|---|---|
|access_token|	string|	是|	授权小程序的接口调用凭据|
返回值说明

|字段名|	类型|	描述|
|---|---|---|
|app_id|	long|	小程序的appid|
|app_name|	string|	小程序的名称|
|app_desc|	string|	小程序的介绍内容|
|photo_addr|	string|	小程序logo|
|qualification|	object|	小程序账号对应的主体信息|
|category|	array|	小程序的行业信息|
|min_swan_version|	string	|开发者工具最低版本|
|status|	int|	小程序的状态|
错误情况下:

|字段名	|类型|	描述|
|---|---|---|
|errno	|int|	错误码|
|msg|	string|	错误描述信息，用来帮助理解和解决发生的错误|
返回值示例
```js
{
  "errno": 0,
  "msg": "success",
  "data": {
        "app_id": 111111,
        "app_name": "小程序",
        "app_desc": "1531812276", //描述
        "photo_addr": "[{\"cover\":\"https:\\/\\/b.bdstatic.com\\/searchbox\\/mappconsole\\/image\\/20180416\\/1523870283-34303.jpg\"}]",
        "qualification": {   //主体信息
            "name": "",  // 主体名称
            "type": 1, //  主体类型： 1：个人 2 企业 3： 政府 4：媒体  5：其他， 个人暂不开放
            "satus": 1,  // 0:未操作 1：通过 2：审核中 3：审核失败 4：推送失败
            "ad_type":  1, // 高级认证类型,0:未做高级认证、1:对公验证、2:活体验证
            "ad_status": 1  // 高级认证状态,1:通过、3:失败
        },
        "category": [   // 行业息息
            {
                "category_id": ,   // 二级行业id
                "category_name": ,   // 二级行业名称
                "category_desc":,    // 二级行业说明
                "parent": {     // 一级行业
                    "category_id": ,  // 一级行业id
                    "category_name": , // 一级行业名称
                    "category_desc": // 一级行业说明
                }
            }
        ],
        "min_swan_version": ,  // 开发者工具最低版本
        "status":  // -1代表封禁，1代表正常，2代表审核中，4代表暂停服务
  }
}
```