---
title: 数据分析
header: develop
nav: api
sidebar: data
---
## reportAnalytics

**解释**：自定义分析数据上报接口。使用前，需要在小程序管理后台自定义分析中新建事件，配置好事件名与字段。

### 参数说明：

|参数|	类型|	必填|	说明|
|---|---|---|---|
|eventName|	String|	是|	事件名|
|data|	Object|	是|	上报的自定义数据，key为配置中的字段名，value为上报的数据。|

### 示例代码：
```js
swan.reportAnalytics('purchase', {
  price: 120,
  color: 'red'
})
```