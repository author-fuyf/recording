---
title: 微信公众号初探
date: 2021-03-20 17:26:13
tags:
- wechat
categories:
- nodejs
description: 公众号接入-基础消息能力
top_img: /img/public-account/index_bg.jpg
cover: /img/public-account/index_bg.jpg
---

## 前言

最初想要做公众号的目的只是想要内嵌一下自己平时写的一些简单的h5页面；提供一个简单大众的入口方便访问和偶尔的展示嘚瑟一下，但不得不说，微信的限制实在相对较为严格，特别的相对于个人开发。个人总结就是，相较于别的开源框架项目，在未明确限制的可容许范围内，其他框架大多没有强加限制，可扩展性兼容性更宽松，而微信基本就是只允许我给你定义放开的api或者功能，不在我考虑之类的一律不允许，个人与企业资质相比较则更为悬殊。小小的吐槽一下，记录我当时的心境；

下面开始；
首先我们需要申请一个个人资质的公众订阅号，微信资质审核很是严格，一般个人资质容易申请的就是个人订阅号，api相关功能会有一定的限制，但仅用作基本功能的尝试一般也问题不大，申请好之后微信公众号后台本身已经提供了比较全面的页面功能供大众使用；基础的菜单栏功能配置、消息回复、文章图文发布都已满足基本需求，但相对功能很单一，不够灵活，所以，需要进入后台管理页面最底部的【开发-基本配置】，第一步：公众号后台服务接入。

## 公众号服务接入

![alt 属性文本](/img/public-account/server_config.jpg)

【服务器地址】就是我们自己的域名地址，/wechat是接口域名，令牌相当于一个check加密字符串用做校验，微信最终会将公众号用户相关操作动作信息转发到我们自己配置的服务接口内，在这之前，需要做服务器公众号之间的接入绑定。在自己的服务端定义对应的接口路由，请求方式为‘GET’,提交时微信会请求当前接口并携带一定参数。

![alt 属性文本](/img/public-account/auth.jpg)

【signature, timestamp, nonce, echostr】大致就是拿到对应参数后加上自己之前定义的令牌key通过特定的排序组合解密完成，校验解密字符串是否与微信端一致从而保证服务安全性，这一步很简单，其实如果想要省事的话不做一系列的解密操作直接返回当前【echostr】也是可以的。
起初让我困惑的是我以为这边的接口仅仅是特定用作校验，其实后续微信转发用户消息也是同样的接口，只不过改为‘POST’请求，只需定义一个接口区分请求方式处理一下就可以。接入成功后，你的公众号就进入了开发者模式，之前页面操作定义的菜单等其他自动回复相关功能就无效了，所有的功能处理全部进入自己的后台服务。


## 事件类型

现在，当前公众号的所有用户触发事件都会通过‘POST’请求将相关消息的XML数据包转发到你的接口中，你首先要做的就是想XML数据包解析，拿到我们真正想要的数据参数。

![alt 属性文本](/img/public-account/message_content.jpg)

实际大致是【handleMsg result】中这样的数据参数，发送方接收方微信则是用户与自己，也需要拿这个参数用作消息回复，Content则是消息内容，需要注意的是MsgType参数，它定义了当前消息类型，大致有普通消息、事件推送两大类；事件推送比如【用户关注、取消关注、自定义菜单链接】相关事件，可以用作用户信息记录统计处理，普通消息则比较多，微信消息发送中的基本功能【文本、图片、语音、视频、位置、链接】等等，需要根据消息类型相对应的判定处理，这里就可以加入我们自己的功能实现逻辑了，比如现在基础实现的一些简单查询功能。

![alt 属性文本](/img/public-account/feature_find.png)

## 消息模板

![alt 属性文本](/img/public-account/msg_template.png)

微信在调用我们的服务转发接口后需要对其作出快速响应，一般是5s,超时则会在客户端报服务故障，消息回复时可以根据需要返回相对应消息类型，此时就需要用到XML消息模板，大致有【文本、图片、语音、视频、音乐、图文】，根据自身需求选择。需要注意的是，类似语音，图片，视频相关消息需要将素材资源通过资源接口上传到微信服务器才可以使用，测试的时候可以拿我们自己发送过来的临时文件id直接返回也可以。

## 阿里云外部服务

![alt 属性文本](/img/public-account/aliy.jpg)

其实真正的功能实现大多还是借助外部功能接口获取，现在接入的类似【天气、星座、智能问答、图文识别】等都是通过阿里云第三方接口完成，需要支付一小部分的功能费用，在自己的消息接收处理中根据关键字或者自定义的逻辑调用外部接口实现转发完成即可。

## 结语

最后，真正接入的其实仅仅是基础消息处理模块，还有很多其他的功能可以尝试。简单记录下来，加深印象，更多的是为了能在周末多利用一下空余时间，激励一下自己。毕竟，博客搭建已经快三个月了。。。O(∩_∩)O哈哈~