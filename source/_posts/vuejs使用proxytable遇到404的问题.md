---
title: vuejs使用proxytable遇到404的问题
tags:
  - nodejs
  - proxytable
  - vue
id: '159'
categories:
  - - 技术
date: 2017-08-18 17:50:08
---
在使用vuejs的时候，使用proxytable，可以转发请求，而且支持跨域。 

文档：https://vuejs-templates.github.io/webpack/proxy.html 

代码例子：

```
proxyTable: {
  '/list': {
    target: 'http://api.xxxxxxxx.com',
    changeOrigin: true,
    pathRewrite: {
      '^/list': '/list'
    }
  }
}
```

这样我们在写url的时候，只用写成 ** /list/1**  就可以代表  **api.xxxxxxxx.com/list/1** changeOrigin参数，如果设置为true，那么本地会虚拟一个服务端接收你的请求并代你发送该请求，这样就不会有跨域问题了。 

好了，我们开始说重点： 

今天我在开发的时候遇到了404的问题，知道最后才发现配置根本没有问题，问题是出现在别的地方： 

1. 修改了proxyTable的配置，一定要重新执行 `npm run dev`命令。

2. 如果你是在window上执行npm命令，**一定要使用windows系统自带的cmd终端**，不要使用第三方的终端，比如bash，git bash等。**这些终端在Ctrl+C操作的时候，并不会终止掉已经运行的node.js程序**。而且你会发现重新 `npm run dev`的时候，会提示端口已经被占用。我就是被这个给坑了。

3. 记住，proxytable只在开发环境中使用哦。
