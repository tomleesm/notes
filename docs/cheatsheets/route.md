``` php
Route::get('foo', function(){});
Route::get('foo', 'ControllerName@function');
Route::controller('foo', 'FooController');
```

## 資源路由

``` php
Route::resource('posts','PostsController');
// 資源路由器只允許指定動作通過
Route::resource('photo', 'PhotoController',['only' => ['index', 'show']]);
Route::resource('photo', 'PhotoController',['except' => ['update', 'destroy']]);
// 批次註冊資源路由
Route::resources(['foo' => 'FooController', 'bar' => 'BarController'])
Route::resources(['foo' => 'FooController', 'bar' => 'BarController'], ['only' => ['index', 'show']])
Route::resources(['foo' => 'FooController', 'bar' => 'BarController'], ['except' => ['update', 'destroy']])
```

## 觸發錯誤 

``` php
App::abort(404);
$handler->missing(...) in ErrorServiceProvider::boot();
throw new NotFoundHttpException;
```

## 路由參數 

``` php
Route::get('foo/{bar}', function($bar){});
Route::get('foo/{bar?}', function($bar = 'bar'){});
```

## HTTP 請求方式

``` php
Route::any('foo', function(){});
Route::post('foo', function(){});
Route::put('foo', function(){});
Route::patch('foo', function(){});
Route::delete('foo', function(){});

// RESTful 資源控製器
Route::resource('foo', 'FooController');
// 為一個路由註冊多種請求方式
Route::match(['get', 'post'], '/', function(){});
```

## 安全路由 (TBD)

``` php
Route::get('foo', array('https', function(){}));
```

## 路由約束

``` php
Route::get('foo/{bar}', function($bar){})
        ->where('bar', '[0-9]+');
Route::get('foo/{bar}/{baz}', function($bar, $baz){})
        ->where(array('bar' => '[0-9]+', 'baz' => '[A-Za-z]'))

// 設定一個可跨路由使用的模式
Route::pattern('bar', '[0-9]+')
```

## HTTP 中介軟體 

``` php
// 為路由指定 Middleware
Route::get('admin/profile', ['middleware' => 'auth', function(){}]);
Route::get('admin/profile', function(){})->middleware('auth');
```

## 命名路由

``` php
Route::currentRouteName();
Route::get('foo/bar', array('as' => 'foobar', function(){}));
Route::get('user/profile', [
    'as' => 'profile', 'uses' => 'UserController@showProfile'
]);
Route::get('user/profile', 'UserController@showProfile')->name('profile');
$url = route('profile');
$redirect = redirect()->route('profile');
```

## 路由前綴

``` php
Route::group(['prefix' => 'admin'], function()
{
    Route::get('users', function(){
        return 'Matches The "/admin/users" URL';
    });
});
```

## 路由命名空間

``` php
// 此路由組將會傳送 'Foo\Bar' 命名空間
Route::group(array('namespace' => 'Foo\Bar'), function(){})
```

## 子域名路由

``` php
// {sub} 將在閉包中被忽略
Route::group(array('domain' => '{sub}.example.com'), function(){});
```
