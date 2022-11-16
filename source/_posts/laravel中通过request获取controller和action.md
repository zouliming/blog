---
title: Laravel中通过Request获取Controller和Action
tags:
  - laravel
id: '133'
categories:
  - - PHP
date: 2017-05-05 17:24:06
---
最近有个工作任务，是在Laravel项目中增加ACL权限控制。 

所谓ACL权限控制，就是对控制器和Action进行控制访问，当然我觉得这种权限控制思想没有RBAC灵活。 

在Laravel中要增加权限拦截，自然是增加中间件来控制。 

所以我在Kernel.php中增加了permission的middleware，然后新建一个PermissionMiddleware来实现handle方法，进行拦截。 

但是写到一半遇到难题了，如何通过Request参数来获取对应的Controller和Action呢？ 

Laravel的API文档可没写的那么全，自己看源码太花时间。好在网上找到了资料，在这里记录一下。

```
<?php
namespace App\Http\Middleware;
use Closure;
class BeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        //1.是否登录
        if(!Auth::login()) {
            return returnError('未登录', 400);
        }
        //2.按路由获取到controller和method
        $actions = $request->route()->getAction();
        $actionName = isset($actions['controller']) ? $actions['controller'] : '';
        $actionArr = explode('@', $actionName);
        $controllerArr = explode('\\', $actionArr[0]);
        $controller = array_pop($controllerArr);
        $controller = strtolower(str_replace('Controller', '', $controller));
        $method = $actionArr[1];
        //3.判断对应的controller和method是否有权限
        if( $this->checkAuth($controller, $method) ) {
            return $next($request);
        } else {
            return returnError('未授权', 401);
        }
    }
}
```
