
## 登录

* Facebook
* Twitter
* Instagram
* Line
* Google
* Email
* Amazon

### Facebook

[文档地址](https://developers.facebook.com/docs/facebook-login)

* CocoaPods 
* 下载SDK

> 最新版 "4.39.0" 登录返回报错 errcode=3. "4.38.0" 可用


### Twitter

[文档地址](https://github.com/twitter/twitter-kit-ios/wiki/Installation)

* 使用 CocoaPods 集成，会因为依赖静态库报错，通过hook Podfile 文件，在pre_install 中禁用静态库检查。
* 下载SDK [手动集成](https://github.com/twitter/twitter-kit-ios/wiki/Installation#install-twitter-kit-manually)

### Instagram

[英文文档地址](https://www.instagram.com/developer/authentication/) 需要翻墙

* 通过请求链接，handleURL 方式，启动 Instagram，或者打开网页授权。

### Google

[文档地址](https://developers.google.cn/identity/sign-in/ios/start)

* CocoaPods
* 下载SDK 手动集成

### Youtube

是Google的一个服务，需要在 Google 后台打开权限。

[接入文档](https://developers.google.com/youtube/v3/quickstart/ios)

* CocoaPods. 依赖 Google 的库

###  Amazon

[接入文档](https://login.amazon.com/ios)

* 下载SDK，手动集成

### Line

[接入文档](https://developers.line.biz/en/docs/ios-sdk/)

主要东南亚使用，Line 有支付功能。账号注册问题。

* CocoaPods 集成
* 手动集成

## 支付

* PayPal
* Visa/master
* ApplePay

### PayPal

[接入文档](https://paypal.github.io/paypalnativecheckout-docs/)

* CococaPods 集成
* 手动集成

### WorldPay

[接入文档](https://www.worldpay.com/us/developers/apidocs/getstartipc.html)

### Visa

没找到前端接入方式。使用 PayPal/ApplePay 接入卡片支付。

### ApplePay 

网上好多