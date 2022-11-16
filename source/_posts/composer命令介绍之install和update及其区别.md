---
title: composer命令介绍之install和update及其区别
tags:
  - composer
id: '105'
categories:
  - - 技术
date: 2017-02-22 11:00:18
---
> composer 是 php 的一个依赖管理工具。它允许你申明项目所依赖的代码库，它会在你的项目中为你安装他们。

然而，对于如何『安装他们』，新手可能并不清楚。网上的答案有的说 composer install，有的说composer update，而这两者似乎都能成功把依赖下载下来并安装好，那么他们究竟有何区别呢？ 

首先要搞清楚的一件事情是，所有的依赖都定义在composer.json中，手册中给出了一些基本用法和例子。你可能已经注意到，在指定版本号的时候，我们并不一定要指明一个精确的版本。那么就有可能发生这么一个情况，对于同一份composer.json，我们在不同时刻拉取到的依赖文件可能不同（因为composer会在满足条件的情况下去拉取最新的那份依赖），从而导致一些异常情况。composer update和composer install正是为了解决这个问题而出现的。 

当你执行composer update的时候，composer会去读取composer.json中指定的依赖，去分析他们，并且去拉取符合条件最新版本的依赖。然后他会把所拉取到的依赖放入vendor目录下，并且把所有拉取的依赖的精确版本号写入composer.lock文件中。

composer install所执行的事情非常类似，只在第一步的时候有差别。当你本地如果已经存在一份composer.lock时，它将会去读取你的composer.lock而非composer.json，并且以此为标准去下载依赖。当你本地没有composer.lock的时候，它所做的事情和composer update其实并没有区别。

这意味着，只要你本地有一份composer.lock，你就可以保证无论过去了多久，你都能拉到相同的依赖。而如果你把它纳入你的项目的版本控制中，那么你就可以确保你项目中的每一个人、每一台电脑，不管什么系统，都能拉取到一模一样的依赖，以减少潜在的依赖对部署的影响。当然，请记得，你应该使用的命令是composer install。 

那什么时候该使用composer update呢？当你修改了你的依赖关系，不管是新增了依赖，还是修改了依赖的版本，又或者是删除了依赖，这时候如果你执行composer install的时候，是不会有任何变更的，但你会得到一个警告信息

> Warning: The lock file is not up to date with the latest changes in composer.json. You may be getting outdated dependencies. Run update to update them.

有人可能会很好奇php是怎么知道我修改了依赖，或者composer.lock已经过期了。很简单，如果你打开composer.lock的话，会发现其中有一个hash字段，这就是当时对应的那份依赖的哈希值。如果值不一致自然而然就知道发生了变更了。 

这时候，你应该去通过composer update来更新下你的依赖了。 如果你不希望影响别的已经安装的依赖，仅仅更新你修改的部分，那你可以通过指定白名单来确定要更新的范围，例如composer update monolog/monolog仅会更新monolog/monlog这个依赖，别的依赖哪怕有更新也会被忽略。
