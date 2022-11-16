---
title: Laravel中使用Pusher的一个暗坑
tags:
  - laravel
  - php
  - pusher
id: '150'
categories:
  - - PHP
date: 2017-07-17 19:16:06
---
最近在研究 `Laravel`中的事件广播系统，就拿Pusher作为驱动器来练练手，结果调入了各种坑中。

1. 遇到过环境变量加载不到的问题，可能和中间使用过 `php artisan config:cache`和 `php artisan config:clear` 有关，就算 `clear`之后，我对pusher设置的 `env`配置也是无效的，最后还是通过新增了其他名称的 `env`配置项，才生效的，原因我也不清楚，可能加载机制和文件写时间有关系吧。
2. Laravel的官方文档说，如果要使用Pusher对事件进行广播，需要安装Pusher PHP SDK，命令如下 ` composer require pusher/pusher-php-server`
   可惜在调用的时候，却提示“_Pusher not found_”。 最后我在vendor文件夹里找到Pusher类包，**最后手动在config/app.php文件的aliases项目里添加了对Pusher的配置**，才解决此Bug。
   `'Pusher' => \Pusher\Pusher::class`
3. 最后就是一个非常奇葩的问题了。`Pusher`的类也有了，配置也配置正确，可惜怎么都发送不了事件。最后查看了源代码，发现是调用了 `Pusher`的 `trigger`方法，返回的false。我想，这是官网的Pusher类库啊，不会有错啊！可惜我找了很多资料之后，发现，妈的，官方的代码也会有错。**这里没有设置 `SSL`验证为 `false`，可能是SSL验证引起的。**于是加上两行代码再试：

```php
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER,false);
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST,false);
```

消息终于成功发送了。 

通过在Pusher官网的 `Debug Console`模块，可以看到新增的事件消息，就代表事件广播成功了。接下来怎么在客户端消费广播信息，就是另一件事情了。
