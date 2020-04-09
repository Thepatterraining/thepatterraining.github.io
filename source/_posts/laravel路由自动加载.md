---
title: laravel路由自动加载
date: 2020-04-06 16:15:00
tags: ['php','laravel']
category: php
article: laravel路由自动加载
---

# laravel路由自动加载

laravel 自带的路由文件有四个
- api.php 文件存放 api 路由，会自动加载api前缀和一些中间件。
- channels.php 文件用于注册应用支持的所有事件广播频道。
- console.php 文件用于定义所有基于闭包的控制台命令，每个闭包都被绑定到一个控制台命令并且允许与命令行 IO 方法进行交互，尽管这个文件并不定义 HTTP 路由，但是它定义了基于控制台的应用入口（路由）。
- web.php 如果应用无需提供无状态的、RESTful 风格的 API，那么路由基本上都要定义在 web.php 文件中。会自动加载web中间件。

我们常用的无非是api和web路由，一开始我们可以都写在里面，那当程序不断扩大，路由达到几千个，几万个甚至更多，放在一个文件里显示难以维护，难以查找。

这时候我们需要把路由分到不同的路由文件中去，我们在routes目录下创建api文件夹，来存放相关的api路由。

<!--more-->

这时候我们自己创建的路由文件是不被框架认可的，不被加载的。那我们怎么做呢，最简单的方法是加载到api.php路由文件内。

```php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

require base_path('routes/api/user.php');  //加载api文件夹下的用户路由

/*
|--------------------------------------------------------------------------
| API Routes
|--------------------------------------------------------------------------
|
| Here is where you can register API routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| is assigned the "api" middleware group. Enjoy building your API!
|
*/

Route::middleware('auth:api')->get('/user', function (Request $request) {
    return $request->user();
});



```

但是这样的方式有着很大的缺点，难道我们每增加一个路由文件，都要修改这个api.php文件嘛？

我们还有更好的方式，在laravel中，服务提供者是一个很重要的模块，其实这里的所有路由都是通过`RouteServiceProvider.php`这个服务提供者来加载的。所以我们只需要更改这个服务提供者就可以了。

这里面有一个`mapApiRoutes`函数来加载api路由，我们可以写一个函数`requireRoutes`来加载我们自己创建的路由。然后在`mapApiRoutes`函数里面调用。

```php
 /**
     * Define the "api" routes for the application.
     *
     * These routes are typically stateless.
     *
     * @return void
     */
    protected function mapApiRoutes()
    {
        Route::prefix('api')     //前缀
            ->middleware('api')  //中间件
            ->namespace($this->namespace)  //命名空间
            ->group(base_path('routes/api.php'));

        $this->requireRoutes('routes/Api');
    }

    /**
     * 遍历文件夹
     */
    private function requireRoutes($path) {
        $dirs = scandir(base_path($path));
        foreach ($dirs as $dir) {
            if (is_dir(base_path($path.'/'.$dir))) {
                if($dir=='.' || $dir=='..'){//判断是否为系统隐藏的文件.和..  如果是则跳过否则就继续往下走，防止无限循环再这里。
                    continue;
                }
                $this->requireRoutes($path.'/'.$dir);
            } else {
                //文件，加载进来
                Route::prefix('api')
                ->middleware('api')
                ->middleware('jwtCheck')
                ->namespace($this->namespace)
                ->group(base_path($path.'/'.$dir));
            }
        }
    }
```