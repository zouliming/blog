---
title: docker启动后又立刻退出
tags:
  - docker
id: '221'
categories:
  - - Docker
date: 2019-03-28 15:29:22
---
自己搭建了一套PHP容器，当然，很特殊的容器，融合了很多扩展。 

经常遇到一个奇怪的问题就是容器启动后又退出了。 

结合自己找到的多方资料，联合实际情况，在这里一一解释出来，供大家参考吧。 

首先，我用的docker-compose，需要配置command参数，我的容器里也的确写了一个entrypoint.sh脚本，默认容器启动后会执行这个sh脚本。脚本的主要内容就是启动apache，并且以前台进程的形式运行。内容如下：

```
apache2ctl -D FOREGROUND
```

之前容器直接退出，我去看log，也就是简单的一条：

```
 httpd (pid 11) already running
```

当时我很懵B。日志写的apache服务已经启动了，但是为什么我这边测试的结果是，容器却是**exit with code 0**状态，php环境也根本没有启动成功。 

在这里不得不感叹一句，人就是这样，同样的问题，过几天再看，突然间就会有思路了，难道这就是传说中的悟性？ 

在这里我就不解释我奇妙的思维路线了，我直接解释原因吧。 

**docker容器比较特别的一点是，命令一执行完，容器也就退出了。这一点非常关键。**

**为什么apache要用foreground模式来运行呢，就是为了让它一直在交互模式中，这样docker就不会退出了。** 

**我的log里提示httpd (pid 11) already running，是检测到之前apache进程还在运行中，那么apache2ctl -D FOREGROUND命令自然是启动失败的，那么进程就不会进入交互模式，那么entrypoint.sh就执行结束，就导致docker直接退出了。 **

导致之前容器镜像中的apache进程还在运行中的因素比较多，比如docker被直接**ctrl+c**结束了，没有用**docker-compose down**来关闭，销毁当前的镜像，那么旧的image可能会影响到新的容器镜像的启动。所以推荐大家要用**docker-compose down**的方式来关闭容器。 

那么之前提到的问题怎么解决呢？其实很简单，只要保证**entrypoint.sh**一直在执行中就行了，方法有很多，比如有网友说每过一秒输出一句话：

```
"while true; do echo hello world; sleep 1; done"
```

当然，我不会用这样的办法。我直接在apache2ctl -D FOREGROUND命令前加一个apache2ctl stop就行了。 

这样就可以保证**apache2ctl -D FOREGROUND**以前台进程模式运行了。你注意看**docker log**就会发现，有时候日志里会有一个：

```
httpd (pid 11?) not running
```

就是我的**apache2ctl stop**执行时，发现没有apache进程引起的，当然，这不会引起任何问题。 

对了，**docker运行的时候记得加-d参数，这样以后台模式运行docker**，不然默认是以交互模式运行docker的，在windows中，我还没找到退出交互模式的方法，有知道的小伙伴给我告诉我一下。
