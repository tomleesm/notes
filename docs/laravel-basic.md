# Laravel 基礎

只解釋基本觀念，函式語法參考官方文件即可，否則會寫不完

## 路由 Routing

在 `routes/web.php` 中設定路由，以下的範例表示 HTTP  請求 `GET /foo`，回應是 Hello World
``` php
Route::get('foo', function () {
    return 'Hello World';
});
```

1. HTTP 動詞由靜態方法名稱決定：所以 `POST` 請求使用 `Route::post()`，使用 `Route::match(['get', 'head'], '/', function() {})` 表示使用 `GET` 或 `HEAD` 請求 `/`
2. 第一個參數是 URI，開頭可以加上斜線，也可以不加，所以 `Route:get('/foo')` 和 `Route:get('foo')` 都代表 `GET /foo`
3. 第二個參數是匹配到第一個參數的路由規則時，會執行的 Closure 或 Controller 中的 method。一般會執行 method，如下面的範例，而且路由設定中如果有 Closure ，無法最佳化路由載入

``` php
// GET /user 會執行 UserController 的方法 index()
Route::get('/user', 'UserController@index');
```

Laravel 會由上到下匹配路由規則，匹配到就執行。所以有 Route parameter 的路由設定最好放後面。Subdomain Routing 則要放在 `Route::get('/')` 之前，否則不會執行。

``` php
// GET /users/create，會執行 Route::get('users/{username}')
Route::get('users/{username}');
// 導致 Route::get('users/create') 永遠不會執行
Route::get('users/create');
```

## Middleware

## CSRF 防護

## Controllers