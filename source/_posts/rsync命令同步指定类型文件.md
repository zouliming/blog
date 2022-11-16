---
title: rsync命令同步指定类型文件
tags:
  - rsync
id: '246'
categories:
  - - Linux
date: 2020-01-20 17:11:36
---
我之前有个需求，需要复制A目录下的指定文件类型到B目录，包含子目录，并且只复制指定文件类型，发现很多人都不会。 

后来发现有rsync这个命令可以满足。 

但在使用过程中发现了一些问题，而且网上很多人的文章都是从51cto博客复制的，关键源文章还是错误的！害的我浪费了一些时间，我特意把各种例子都试了一次。 

假设我的A目录结构时这样的：

```
a
├── index.php
└── zhang
    ├── config.php
    ├── index.html
    └── test.pem
```

  我只需要复制pem后缀的文件类型，别的统统忽略。 测试结果如下： 成功

```
rsync -vzrtopg --include "*.pem" --exclude="*.*" a/ b/
rsync -vzrtopg --include "*.pem" --exclude "*.*" a/ b/
rsync -vzrtopg --include="*.pem" --exclude="*.*" a/ b/
同步a目录，以及其子目录下的所有pem文件
rsync -vzrtopg --include="*.pem" --include="*/" --exclude="*" a/ b/
```

  失败

```
rsync -vzrtopg --include "*.pem" --exclude "*" a/ b/
rsync -vzrtopg --include=".pem" --exclude="*" a/ b/
rsync -vzrtopg --include=".pem" --exclude="*.*" a/ b/
rsync -vzrtopg --include=".pem" --exclude=* a/ b/
rsync -vzrtopg --include=".pem" --exclude=*.* a/ b/

仅同步一级目录下的pem文件
rsync -vzrtopg --include="*.pem" --exclude="*" a/ b/

会把其他类型文件也同步过去
rsync -vzrtopg --include="*.pem" a/ b/
```

  大家自己根据例子去揣摩要点吧。
