---
layout: post
title:  跨平台应用开发趋势和Webassembly带来的变化
categories: [web, mobile, app, webassembly, OS]
comments_id: 6
excerpt: 移动互联网应用已经发展10年有余，随着技术和商业环境的变化，业界对跨平台的universal app框架的需求越来越强烈，出现了React Native，Flutter, Weex这样的跨平台框架，随着W3C正式标准化Webassembly，Web又成为创新的核心，也是universal app实现的一个重要路径


秉持“Code once run everywhere” 理念的跨平台应用开发技术从移动应用诞生之始就一直有需求，也出现了cordova，react native，flutter，weex等开源的框架技术。除了代码共享，开发便利等因素，跨平台应用开发还存在其他一些更强烈的诉求。
#### 1、随着个人消费者设备逐渐增多，用户的手机，pad，pc，电视，手表，汽车，AR/VR设备等设备存在强烈统一用户体验，数据共享，无缝交互的需求。
#### 2、基于app store的中心化分发方式，从最初的app质量安全保证，提供分发和应用内购支付的便利，演变为信息控制中枢，垄断了用户和app的关系，不但成为垄断平台，也成为地缘政治的管制手段。最近epic game和apple，google就epic的游戏内购发生争执，遭到两大store下架，迄今还没有和解，用户喜爱的fortinet等游戏不能得到更新，另一边tiktok、微信等企业在地缘政治争端中被无端当作人质，曾经自由开放的互联网变成被两大app商店垄断的市场。
#### 3、另一方面，随着super app的兴起， app需要以不同的形式被super app集成和触发，例如在微信，美团，地图应用等入口平台进行小程序形式的分发，新的交互方式例如：语音识别，要求能够直达app内的服务，app需要有足够的平台适应能力，应用逻辑和UI呈现都能够适配不同平台的需求。

这些驱动力和相应的技术平台的进步会对目前的原生app生态产生重要影响，未来跨平台的应用，可以称为universal app，将会在开发方式和分发方式上满足上述诉求，基于目前跨平台应用开放呈现出来的趋势，下面总结一下universal app的技术需求。

Universal App的产品需求。参考云计算的12 factor app需求，由类似的地方
##### UI的声明式描述？？
##### 平台适配能力，显示尺寸，布局方式，交互方式，组件外观
##### MVVM模式的单向数据流动驱动UI， UI 受state驱动刷新，state是immutable，通过function改变
##### 平台能力的包装，对系统API访问的一致化包装，
##### 原子化服务能力，App的feature可以route直达，deeplink，提供URL服务定位
##### View对数据的依赖通过model结构，model包含本地和远端数据，model使用data graph对应后端和前端数据，减少串行化开销
##### 离线服务和在线服务不区分，离线运行需要的asset缓存，数据访问通过model，model提供本地储存和缓存以及后台更新能力（service worker）
##### 个人云，个人敏感数据的存储使用服务化机制，可以使用本地化存储机制
##### 用户登陆和认证，使用mobile ID是首选
