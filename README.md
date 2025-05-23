# 闪验uniappx插件项目
## 一.准备工作

### 概述
本文是创蓝闪验SDK在uniappx中使用的集成文档，用于指导 SDK 的集成使用。
### 合规性说明
详见[**SDK初始化合规性指南**](https://doc.chuanglan.com/document/TMEL2UML9YBDH0Y1)

### 能力介绍
**一键登录：SDK获取当前流量卡对应的token，通过服务端可置换当前流量卡的手机号码。**

**本机校验：SDK获取当前流量卡对应的token，提供手机号码，通过服务端可校验提供的手机号是否为当前流量卡的手机号码。**

**`注意：本机校验和一键登录是两个单独的能力，两者的token不能互用，否则会报"应用能力不匹配"。`**
### 前置条件

- 创蓝闪验 SDK 支持中国移动 3/4G/5G、联通 3/4G/5G、电信 4G/5G 的取号能力，在 3G 网络下时延会更高
- 创蓝闪验 SDK 支持单数据网络、数据网络与 WiFi 网络双开，不支持单 WiFi 网络
- 对于双卡手机，创蓝闪验 SDK 只对当前流量卡取号，双卡均未开数据流量 SDK 将会返回错误码

**注意：三网运营商内部执行逻辑不同，必须分别使用三网运营商的卡进行测试，防止功能异常**
### 创建应用

应用的创建流程及AppId的获取，请查看「[账号创建](https://doc.chuanglan.com/document/4UB4MAZI1OVH7X5K)」文档

## 二.一键登录 api
### 0.提升授权页的打开速度

- 建议在执行拉取授权登录页的方法前，提前一段时间调用预取号方法，中间最好有 2-3 秒的缓冲（因为预取号方法需要 1~3s 的时间取得临时凭证）

- 如果条件允许，超时时间设置为 6 秒有助于网络不好的时候取号成功率，SDK 默认 4 秒。
### 1.初始化

**调用 SDK 其他流程方法前，请确保已调用过初始化，否则会返回未初始化。（会采集信息，建议放到同意协议后调用）**

方法原型

```javascript
export function initWithAppId(appId : string,callback : InitCallback) {}
```

参数描述

| 参数         | 类型         | 说明                       |
| ------------ | ------------ | -------------------------- |
| appId        | String       | 创蓝闪验平台获取到的 appId     |
| callback | ShanYanCallback | 初始化回调监听             |

示例代码

```javascript
initWithAppId('your_app_id', (response) => {
// 初始化结果回调函数，参数为包含以下属性的对象:code - 初始化状态码,result - 初始化结果描述
let shanyan_code = response.code;
let shanyan_result = response.result;
console.log(`初始化结果：\nshanyan_code=${shanyan_code}\nshanyan_result=${shanyan_result}`);
}
```

返回参数 code 和 result，含义如下：

| **字段** | **类型** | **含义**                      |
| -------- | -------- | ----------------------------- |
| code     | int      | code 为 1022:成功；其他：失败 |
| result   | String   | 初始化结果描述                |
### 2.预取号
- 【可选方法】获取取号临时凭证；建议在调用拉起授权页前 2-3 秒调用，可以缩短拉起授权页耗时；如果启动 app 就需要展示授权页，预取号和拉起授权页间隔小于1秒，不要调用，否则可能会有异常。

- **请勿与拉起授权登录页同时或之后调用。**
- **避免大量资源下载时调用，例如游戏中加载资源或者更新补丁的时，否则会增加超时概率**

方法原型

```javascript
/**
 * 闪验SDK预取号方法
 * @param callback 预取号结果回调函数
 */
export function getPhoneInfo(callback : ShanYanCallback) {}
```

参数描述

| 参数         | 类型         | 说明                       |
| ------------ | ------------ | -------------------------- |
| callback | ShanYanCallback | 预取号回调监听             |

示例代码

```javascript
getPhoneInfo( (response) => {
// 预取号结果回调函数，参数为包含以下属性的对象:code - 预取号状态码,result - 预取号结果描述
let shanyan_code = response.code;
let shanyan_result = response.result;
const logMessage = `预取号结果：\nshanyan_code=${shanyan_code}\nshanyan_result=${shanyan_result}`;
}
```