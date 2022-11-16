---
title: docker-compose生成的容器立刻退出,exited with code 0
tags:
  - docker
categories:
  - - Docker
date: 2019-03-04 17:06:52
---
### **问题**

自己配置好了一个docker，可以单独运行，但是用docker-compose.yml 管理项目，启动docker-compose up的时候，却exited with code 0 了，生成的容器立刻退出了。

网上找了很多资料，相关甚少。

### **解决**

Docker镜像的缺省命令是 `bash`，如果不加 `-it`，`bash`命令执行了自动会退出，加 `-it`后docker命令会为容器分配一个伪终端，并接管其 `stdin/stdout`支持交互操作，这时候 `bash`命令不会自动退出 

像不使用 `docker-compose`,我们会执行类似如下的命令

 `docker run -it --name node node` 

但 `docker-compose`需要额外配置下 

需要在 `docker-compose.yml`中包含以下行:

```
stdin_open: true
tty: true
```

第一个对应于 `docker run`中的 -i ,第二个对应于 -t . 

如果你不想停留在bash命令页面，这样执行命令即可：

```bash
docker-compose up -d
```
