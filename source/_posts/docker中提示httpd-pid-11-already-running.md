---
title: docker中提示httpd (pid 11) already running
tags:
  - docker
categories:
  - - Docker
date: 2019-05-25 12:37:04
---
自己配置了一个Docker，集成PHP环境和Apache环境。由于项目的特殊依赖，环境很特别，所以只能保存成一个镜像了。 

docker有一个规则，就是如果脚本运行完毕，docker就会退出了。所以我前面的博文中写了，脚本启动apache，要以前台进程的形式运行，这样就一直在交互模式中。 

但是今天自己修改了下shell脚本，重新保存了docker镜像后，明明写的是以前台进程模式运行的，但是log里还是提示“httpd (pid 11) already running”，然后退出交互模式了，造成docker也exited了。

百思不得其解，看到很多老外也遇到这个问题，没人回答。借鉴其他人的一些建议，我突然想到一点。 

**在目录 /run/apache2/中有一个apache2.pid存在，这个玩意儿是之前apache启动后生成的pid文件，我在保存镜像的时候一起保存了。**

这样我每次启动，就是被这个文件给影响了。删除它，然后重新保存镜像。 

再试试，终于成功了。
