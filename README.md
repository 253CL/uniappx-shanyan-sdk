# 一.准备工作

## 概述

本文是创蓝闪验SDK在uniappx/uniapp中使用的集成文档，用于指导 SDK 的集成使用。

## 合规性说明

详见[**SDK初始化合规性指南**](https://doc.chuanglan.com/document/TMEL2UML9YBDH0Y1)

## 能力介绍

**一键登录：SDK获取当前流量卡对应的token，通过服务端可置换当前流量卡的手机号码。**

**本机校验：SDK获取当前流量卡对应的token，提供手机号码，通过服务端可校验提供的手机号是否为当前流量卡的手机号码。**

**`注意：本机校验和一键登录是两个单独的能力，两者的token不能互用，否则会报"应用能力不匹配"。`**

## 前置条件

- 创蓝闪验 SDK 支持中国移动 3/4G/5G、联通 3/4G/5G、电信 4G/5G 的取号能力，在 3G 网络下时延会更高

- 创蓝闪验 SDK 支持单数据网络、数据网络与 WiFi 网络双开，不支持单 WiFi 网络

- 对于双卡手机，创蓝闪验 SDK 只对当前流量卡取号，双卡均未开数据流量 SDK 将会返回错误码

**`注意：三网运营商内部执行逻辑不同，必须分别使用三网运营商的卡进行测试，防止功能异常`**


## 创建应用

应用的创建流程及AppId/AppKey的获取，请查看「[账号创建](https://doc.chuanglan.com/document/4UB4MAZI1OVH7X5K)」文档

# 二.一键登录API

## 引用方式

```javascript
	import { PrivacyListItem, AuthAndroidUIConfigure, AuthIOSUIConfigure, BaseUIConfig, ShanYanUIConfigure, WidgetsConfig, ShanYanSDKModule } from "../../uni_modules/cl-shanyan"
	import { WidgetItem, StatusBarItem, SystemNavBarItem, NavBarItem, LogoItem, NumberItem, SLoganItem, LoginButtonItem, PrivacyItem, PrivacyList, DialogThemeItem, CLLayoutUIItem, ConfigItem, CLEdgeInsetsItem, CLSizeItem } from "../../uni_modules/cl-shanyan/utssdk/interface.uts"	
    const shanYanSDKModule = new ShanYanSDKModule();
```   

## 1.初始化

**调用 SDK 其他流程方法前，请确保已调用过初始化，否则会返回未初始化。（会采集信息，建议放到同意协议后调用）**

调用示例

```javascript
shanYanSDKModule.initWithAppId('your_appId', (response) => {
// 初始化结果回调函数
let code = response.code;
let message = response.message;
});
```

参数描述

| 参数         | 类型         | 说明                       |
| ------------ | ------------ | -------------------------- |
| appId        | String       | 创蓝闪验平台获取到的 appId     |
| initCallback| CLResultCallBack | 初始化结果监听函数，接收结果对象CLResultResponse，详见【CLResultResponse描述】|


## 2.预取号

- 【可选方法】获取取号临时凭证；建议在调用拉起授权页前 2-3 秒调用，可以缩短拉起授权页耗时；如果启动 app 就需要展示授权页，预取号和拉起授权页间隔小于1秒，不要调用，否则可能会有异常。

- **请勿与拉起授权登录页同时或之后调用。**

- **避免大量资源下载时调用，例如游戏中加载资源或者更新补丁的时，否则会增加超时概率**

调用示例

```javascript
shanYanSDKModule.getPhoneInfo( (response) => {
// 预取号结果回调函数
let code = response.code;
let message = response.message;
let telecom = response.data?.telecom;
let protocolName = response.data?.protocolName;
let protocolUrl = response.data?.protocolUrl;
});
```
参数描述

| 参数         | 类型         | 说明                       |
| ------------ | ------------ | -------------------------- |
| phoneInfoCallback | CLResultCallBack| 预取号结果监听，接收结果对象CLResultResponse，详见【CLResultResponse描述】|


## 3.拉起授权页&获取token

- 拉起授权页方法将会调起运营商授权页面。**已登录状态请勿调用 。**

- **每次调用拉起授权页方法前均需先调用授权页配置方法，否则授权页可能会展示异常。**

- 必须保证上一次拉起的授权页已经销毁再调用，否则 SDK 会返回请求频繁。

- 拉起一次授权页，登录按钮最多只能点击 4 次，第五次默认会置灰，不返回信息。

调用示例


```javascript
//授权页UI配置详见【授权页配置说明】
const uiConfigure = this.getUIConfig();
shanYanSDKModule.quickAuthLoginWithConfigure(uiConfig, (response) => {
		// 拉起授权页结果回调函数
		let code = response.code;
		let message = response.message;
		}, (response) => {
		// 授权结果回调函数，包括获取token结果和点击返回按钮结果
		let code = response.code;
		let message = response.message;
        //当code为1000时，返回token字段。
		let token = response.data?.token;
		});
```

参数描述

| 参数         | 类型         | 说明                       |
| ------------ | ------------ | -------------------------- |
| uiConfig        | ShanYanUIConfigure       | 授权页配置对象， 详见[【授权页配置说明】](https://doc.chuanglan.com/document/7TGX955VG53CWKHY) |
| openLoginAuthCallback| CLResultCallBack | 启动授权页结果监听，接收结果对象CLResultResponse，详见【CLResultResponse描述】|
| oneKeyLoginCallback| CLResultCallBack  | 授权结果监听，接收结果对象CLResultResponse，详见【CLResultResponse描述】   |


## 4.置换手机号


当【**4.拉起授权页&获取token**】授权结果监听外层 code 为 1000 时，会获取到置换手机号所需的 token。请将token传递给App服务端，由服务端参考「[服务端](https://doc.chuanglan.com/document/SY88R0I8TCUPR1M3)」文档来实现获取手机号码的步骤。

## 5.其他API

### a.设置 log 开关

需要初始化之前调用，开启后能够打印SDK内部更多日志信息，主要用于开发对接过程中排查问题。

调用示例

```javascript
//true：开启；false：关闭；默认：false
shanYanSDKModule.setDebug(true);
```

### b.设置预取号超时时间

需要初始化之前调用，用于设置预取号超时时间，不建议设置小于 4 的值，否则可能会导致超时的概率增加。

调用示例

```javascript
//单位秒，默认：4。
shanYanSDKModule.setTimeOutForPreLogin(6);
```

### c.清理预取号缓存

预取号成功后默认会有本地缓存，调用此方法可以清理本地预取号缓存。

调用示例

```javascript
shanYanSDKModule.clearScripCache();
```
### d.销毁授权页

在授权页正在展示的情况下，调用此方法可以关闭授权页。

调用示例

```javascript
shanYanSDKModule.finishAuthActivity();
```

# 三.本机校验API

## 1.初始化

**同一键登录初始化，如果本机校验和一键登录都需要使用时，只需调用一次初始化。**


## 2.本机校验获取 token

在初始化执行之后调用，本机号校验可以不写界面，如果需要界面需自行实现，该方法可以在多个需要校验的页面中调用。

调用示例


```javascript
shanYanSDKModule.startAuthentication((response) => {
						// 本机校验结果回调函数
	let code = response.code;
	let message = response.message;
    //当code为1000时，返回token字段。
	let token = response.data?.token;
});
```

参数描述


| 参数         | 类型         | 说明                       |
| ------------ | ------------ | -------------------------- |
| authCallback | CLResultCallBack| 本机校验获取token结果监听，接收结果对象CLResultResponse，详见【CLResultResponse描述】|


## 3.校验手机号

当本机校验获取token监听的code 为 1000 时，会获取到检验手机号所需的 token。请将token传递给App服务端，由服务端参考「[服务端](https://doc.chuanglan.com/document/SY88R0I8TCUPR1M3)」文档来实现检验手机号码的步骤。


# 四.返回结果及状态码描述

## 1.CLResultResponse描述

CLResultResponse为初始化、预取号、拉起授权页&获取token、本机校验等方法的返回结果对象，包含参数释义如下：

| **字段** | **类型** | **含义**                      |
| -------- | -------- | ----------------------------- |
| code     | int      | 外层码。  1000：成功；其他：失败 。|
| message   | string   | 外层描述 ，结果简述。            |
| innerCode   | string   | 内层码，详细状态码，主要用于失败时排查问题 。           |
| innerDesc   | string   | 内层描述，结果详细描述，主要用于失败时排查问题。             |
| data   | ResultData   | 初始化、拉起授权页时为空，其他方法见参数描述。 |
| data> telecom  | string   | 运营商类型：CMCC（移动）；CUCC（联通）；CTCC（电信）； UNKNOW/其他（无 SIM 卡或非三网运营商卡）。`注意：只在预取号方法成功时有值`|
| data> protocolName  | string   |  当前运营商名称。`注意：只在预取号方法成功时有值`|
| data> protocolUrl  | string   |  当前运营商协议链接。`注意：只在预取号方法成功时有值`|
 data> token  | string   |  一键登录或本机校验的token。`注意：只在获取token成功时有值`|

## 2.状态码描述

此表为 SDK 外层返回码，如需查看内层码及服务端返回码，请查看官网[[返回码]](https://doc.chuanglan.com/document/XYLLRYR6TQGBWY38)文档


| 返回码 | 返回码描述                                              |
| ------ | ------------------------------------------------------- |
| 1000   | 成功                                 |
| 1001   | 运营商返回错误                                          |
| 1002   | 运营商信息获取失败，请结合 result 查看具体失败原因      |
| 1003   | 一键登录获取 token 失败，请结合 result 查看具体失败原因 |
| 1004   | 未初始化 |
| 1005   | 预取号请求失败，请结合 result 查看具体失败原因 |
| 1006   | 无法识别sim卡或没有sim卡 |
| 1007   | 网络请求失败，请结合 result 查看具体失败原因            |
| 1008   | 数据流量不稳定            |
| 1011   | 点击返回，用户取消免密登录                              |
| 1014   | SDK 内部异常，请结合 result 查看具体失败原因            |
| 1016   | APPID 为空                                              |
| 1019   | 其他错误，请结合 result 查看具体失败原因                |
| 1022   | 网络初始化、预取号成功                                  |
| 1023   | 初始化、预取号失败，请结合 result 查看具体失败原因      |
| 1031   | 请求过于频繁                                            |
| 1032   | 用户禁用                                                |
| 2003   | 本机号校验返回失败，请结合 result 查看具体失败原因      |