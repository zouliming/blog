---
title: Laravel支持通过token来获取用户登录信息
tags:
  - laravel
id: '125'
categories:
  - - PHP
date: 2017-04-14 18:28:10
---
现在很多公司都在做前后端分离，而通过Api来判断用户是否登录，则是通过Token的形式。 

Laravel自带的获取用户信息逻辑，是需要数据库中有一张User表。而目前很多公司的做法是，应用系统并没有这样的User表，User数据是记录在Passport系统的。 

那么如何修改代码，来实现通过Token来调用Passport系统来获取用户信息呢？下面我来讲讲。 

Laravel的默认路由里，**api.php**的**middleware**是这样配置的：**'middleware'=>'auth:api'** 

这个**“auth:api”**是什么意思呢？** 

****auth**对应的是**app\\Http\\Kernel.php里的$routeMiddleware**变量的auth这个key，也就是通过auth这个key，指到了Kernel 文件中配置的Class类。**

**api**对应的是config\auth.php中的guards配置的api这个key的配置信息。（实际上api代表的是auth对应的那个类的参数，同学们看下auth那个类的源代码就明白了） 

那么我们来看看config/auth.php**中的guards配置吧。

```php
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'token',
            'provider' => 'users',
        ],
    ],
```

其中**token**代表的是**TokenGuard.php**，有兴趣的朋友可以看看 **{project}/vendor/laravel/framework/src/Illuminate/Auth/TokenGuard.php** 这段源码。

**provider**的**users**，对应的是**config\\auth.php**中的**providers**中的配置里，**users**的配置内容。如果我们要自定义一个用户提供器，可以这么写：

```php
    'providers' => [
        // 'users' => [
        //     'driver' => 'eloquent',
        //     'model' => App\User::class,
        // ],
        'users' => [
            'driver' => 'our_provider'
        ],
        // 'users' => [
        //     'driver' => 'database',
        //     'table' => 'users',
        // ],
    ],
```

我定义了一个**our\_provider**的用户提供器，接着我们来实现它。（关于自定义providers和自定义guards的说明，可以看看官方文档。但是官网写的太少了，很多人看不明白，我这里就直接贴代码给你看）

首先，需要在**app/Providers/AuthServiceProvider.php**文件中的boot方法里加上一段代码，如下：

```
    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::provider('our_provider', function($app, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\UserProvider...

            return new App\Authentication\UserProvider();
        });
    }
```

我在**app**目录下新建了一个文件夹，叫**Authentication**，里面新建了两个类，**User**类和**UserProvider**类。上面的代码里，最后return的就是我新建的**UserProvider**类。 

通过查看**TokenGuard**的user方法，我得知，token守卫是通过**retrieveByCredentials**方法来返回用户信息的，所以我们的**UserProvider**要着重实现这个方法。 

以下是代码例子：

```
<?php
namespace App\Authentication;

use App\Repositories\AuthRepository;
use Illuminate\Contracts\Auth\UserProvider as IlluminateUserProvider;
use Illuminate\Contracts\Auth\Authenticatable;

class UserProvider implements IlluminateUserProvider{
    /**
     * @param  mixed $identifier
     * @return \Illuminate\Contracts\Auth\Authenticatablenull
     */
    public function retrieveById($identifier)
    {
        // Get and return a user by their unique identifier
    }

    /**
     * @param  mixed $identifier
     * @param  string $token
     * @return \Illuminate\Contracts\Auth\Authenticatablenull
     */
    public function retrieveByToken($identifier, $token)
    {
        // Get and return a user by their unique identifier and "remember me" token
    }

    /**
     * @param  \Illuminate\Contracts\Auth\Authenticatable $user
     * @param  string $token
     * @return void
     */
    public function updateRememberToken(Authenticatable $user, $token)
    {
        // Save the given "remember me" token for the given user
    }

    /**
     * Retrieve a user by the given credentials.
     *
     * @param  array $credentials
     * @return \Illuminate\Contracts\Auth\Authenticatablenull
     */
    public function retrieveByCredentials(array $credentials)
    {
        //根据通行证去返回用户信息
        $user = null;
        if(isset($credentials['api_token'])){
            $token_info = AuthRepository::getTokenInfo($credentials['api_token']);
            if($token_info){
                $user = new User();
                $user->setAttributes($token_info['user_info']);
            }
        }
        return $user;
    }

    /**
     * Validate a user against the given credentials.
     *
     * @param  \Illuminate\Contracts\Auth\Authenticatable $user
     * @param  array $credentials
     * @return bool
     */
    public function validateCredentials(Authenticatable  $user, array $credentials)
    {
        // Check that given credentials belong to the given user
    }

}
```

我在**retrieveByCredentials**方法中根据**api\_token**参数，然后去调用相关信息拿到用户信息，最后返回一个**User**类。记住，这里的**User**类也是我新建的，它必须继承**Authenticatable**接口。 

以下是**User**类代码：

```
<?php

namespace App\Authentication;

use Illuminate\Contracts\Auth\Authenticatable;

class User implements Authenticatable
{
    private $attributes = array();

    public function setAttributes($attributes)
    {
        $this->attributes = $attributes;
    }

    /**
     * @return string
     */
    public function getAuthIdentifierName()
    {
        // Return the name of unique identifier for the user (e.g. "id")
        return 'id';
    }

    /**
     * @return mixed
     */
    public function getAuthIdentifier()
    {
        // Return the unique identifier for the user (e.g. their ID, 123)
        $identifier_name = $this->getAuthIdentifierName();
        return $this->attributes[$identifier_name];
    }

    /**
     * @return string
     */
    public function getAuthPassword()
    {
        // Returns the (hashed) password for the user
    }

    /**
     * @return string
     */
    public function getRememberToken()
    {
        // Return the token used for the "remember me" functionality
    }

    /**
     * @param  string $value
     * @return void
     */
    public function setRememberToken($value)
    {
        // Save a new token user for the "remember me" functionality
    }

    /**
     * @return string
     */
    public function getRememberTokenName()
    {
        // Return the name of the column / attribute used to store the "remember me" token
    }
}
```

要注意的是，User类一定要实现两个方法，分别是**getAuthIdentifierName**和**getAuthIdentifier**。

* **getAuthIdentifierName**方法，返回User类的主键名称。
* **getAuthIdentifier**方法，返回User类的主键值。

我们在Laravel调用**Auth::id()**，其实调用的就是**getAuthIdentifier**这个方法。 

只要完成这些步骤，就可以试试用**Auth::user()**来获取当前用户信息了。 

_本文是我自己查阅了一些资料整理得来，如需转载，请附注原文出处。有疑问，请给我留言。_
