---
title: mysql5.6内存过大
tags:
  - mysql
id: '156'
categories:
  - - 默认
date: 2017-08-07 17:50:32
---
在MySql5.5下面新建表的时候发现，一个表中最多只允许一个CURRENT\_TIMESTAMP默认值。如果想设置多个字段默认使用CURRENT\_TIMESTAMP，就需要升级到5.6以上。---这段资料在Stack Overflow查找的。 

所以今天在Ubuntu14.04上升级了MySql，从5.5升级到5.6。升级之后发现，内存占用很大啊！！！ 

找了很多资料，网上的解释很多。 

一般的解释都是，先停止Mysql服务，然后修改/etc/my.cnf配置文件。 

修改，或者添加这三行：

```
performance\_schema\_max\_table\_instances=600
table\_definition\_cache=400
table\_open\_cache=256
```

目的就是把这三行的配置调小一点。这样内存占用就没那么高了。 或者可以直接停止performance\_schema检查服务：

`performance=OFF`

然后再启动Mysql，就发现内存占用没有那么高了。
