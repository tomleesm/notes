---
hide:
  - toc
---

``` php
App::bind('foo', function($app){ return new Foo; });
App::make('foo');
// 如果存在此類, 則返回
App::make('FooBar');
// 單例模式實例到服務容器中
App::singleton('foo', function(){ return new Foo; });
// 將已實例化的對象註冊到服務容器中
App::instance('foo', new Foo);
// 註冊繫結規則到服務容器中
App::bind('FooRepositoryInterface', 'BarRepository');
// 繫結基本值
App::when('App\Http\Controllers\UserController')->needs('$variableName')->give($value);
// 標記
$this->app->tag(['SpeedReport',  'MemoryReport'],  'reports');
// 給應用註冊一個服務提供者
App::register('FooServiceProvider');
// 監聽容器對某個對象的解析
App::resolving(function($object){});
resolve('HelpSpot\API');
```
