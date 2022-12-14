---
title: OAuth2.0的简化模式和授权码模式不冲突吗？
tags:
  - oauth
id: '122'
categories:
  - - PHP
date: 2017-04-06 18:33:37
---
最近在学习OAuth2.0，对这个没有概念的人可以看看阮一峰的博客：[理解OAuth2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html) 

其中最常用的两种模式，就是授权码模式，和简化模式。 

**很多人都会问，为什么授权码模式中，需要返回一次授权码，然后还要再拿着授权码再去请求服务器，才能拿到token呢，为什么要这么麻烦？直接一次请求就返回给我token不就行了吗？** 

一开始我也有这样的疑惑，后来仔细看看就明白了。第一次请求是web跳转到passport端，然后passport端通过浏览器跳转到应用端，此时是web跳转，url完全是暴露的，所以token直接显示在url里是非常危险的一件事情。万一你的机器被人截获请求了呢？不就泄密了？对吧。 

所以第一次请求返回给你的授权码，这个授权码就算泄密了也没关系。因为还有第二次请求，第二次请求是从后台去抓包获取请求的，即用户是看不出来的。同时passport端拿到授权码，还会和URI核对，以及应用端核对，这样就保证了安全性了。 

（看懂了没？） 

**那么我的第二个问题来了？那为什么还要有简化模式？** 

简化模式是直接一次请求就返回了Token？这个不就和上面所讲的一大堆矛盾了吗？ 

后来我查了下资料才搞懂。应用端在像认证端发送请求后，认证端的确返回了token，但是注意了，是在Hash部分。 

啥叫Hash部分，我来给你举个例子。 比如这样一段链接：

www.example.com/cb#access\_token=4fhslh239004hds&state=xyz&token\_type=example&expires\_in=3600 

从#号以后的，都叫Hash部分。 

浏览器跳转，只会跳转到www.example.com/cb这样的地方，后面的一大堆都留在浏览器端了。就算有坏人截取了你的请求，也只看到www.example.com/cb这样的数据，看不到别的了。

而应用端就不一样了，它拿到服务端发送过来的代码，就可以通过这段代码提取Hash部分的数据了。 我们假如这段代码是这样的：

```
window.location.hash
```

这样不就拿到了access\_token了。 

**为啥还要这么麻烦的让服务端返回给我提取Hash部分的代码呢？我直接不就可以写js拿到了？**

我也不知道，难道是为了支持自定义Hash内容的加密方式？欢迎在评论里帮我补充。
