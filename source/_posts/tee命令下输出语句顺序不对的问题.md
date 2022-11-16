---
title: tee命令下输出语句顺序不对的问题
tags:
  - shell
id: '242'
categories:
  - - Linux
date: 2019-08-06 15:21:48
---
最近写shell脚本的时候，发现在使用了echo语句和read -r -p命令时，输出内容本来是正常的。但在结合了tee命令后，输出内容顺序就会被打乱。

比如：

```bash
#!/bin/bash
#tee_with_read.sh
function tee_test()
{
    echo "***This should be printed first but it is not***"
    read -r -p "Enter input : "
    echo "You entered : $REPLY"
}
tee_test
```

这个命令的运行结果是：

```bash
***This should be printed first but it is not***
Enter input : hello
You entered : hello
```

但是如果把shell脚本修改成：

```bash
#!/bin/bash
#tee_with_read.sh
function tee_test()
{
    echo "***This should be printed first but it is not***"
    read -r -p "Enter input : "
    echo "You entered : $REPLY"
}
tee_test  tee -a logfile
```

输出内容就变成了：

```
Enter input : ***This should be printed first but it is not***
aaa
You entered : aaa
```

查找了相关资料得知：

> 内核启动的时候默认打开这三个I/O设备文件：标准输入文件stdin，标准输出文件stdout，标准错误输出文件stderr，分别得到文件描述符 0, 1, 2。 stderr是不缓存的，stdout是行间缓存的。 read -p 是会输出到stderr区域，但是echo会输出到stdout区域。

因此就会出现顺序颠倒的情况了。

要想解决这个问题，有以下3中解决方法：

1. 在想stderr语句前面输出的echo语句后面加上”> &2“，这样echo语句就会被输出到stderr区域了。

2. 在脚本后面加上”2>&1",这样的意思是，把stderr区域，输出到stdout区域。

3. 可以这么写：

```bash
echo "***This should be printed first, now it will***" ;
echo -n "Enter Input : " ;
read -r ;
```
