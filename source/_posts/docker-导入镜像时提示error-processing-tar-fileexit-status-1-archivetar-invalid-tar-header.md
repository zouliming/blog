---
title: >-
  docker 导入镜像时提示Error processing tar file(exit status 1): archive/tar: invalid
  tar header
tags:
  - docker
id: '227'
categories:
  - - Docker
date: 2019-04-03 14:17:53
---

用了Docker是方便许多，公司的前端需要帮忙搭建PHP环境，我可以直接将docker导出，导入就搞定了。 至于docker中镜像和容器的区别，save和export的区别，请自行去网上学习一下。 我今天这里讲到的是，很多人会遇到的在import tar的时候出现错误信息：

```
Error response from daemon: Error processing tar file(exit status 1): archive/tar: invalid tar header
```

很多外国人也在github issue上提出了问题，相关的解决方案很难找，不过还是被我找到了。 这个是windows系统的powershell的bug，解决的方法很简单，在export 的时候，请使用如下的命令：

```
docker export f377314ce10d -o xxx.tar
```

而不是

```
docker export f377314ce10d xxx.tar
```

其实就是增加了一个 -o 的参数。但是就能解决这个问题。