# Cookie 和 Session

## Cookie

### 抓取 cookie 的值

``` php
<?php
$request->cookie('名稱');

use Illuminate\Support\Facades\Cookie;
Cookie::get('名稱');
```

### 設定 cookie 的值

可以省略參數「保存幾分鐘」

``` php
<?php
return response()
       ->cookie('名稱', 'cookie值', 保存幾分鐘);
# cookie() 產生 Cookie 物件後加到 response()->cookie()
$cookie = cookie('名稱', 'cookie值', 保存幾分鐘);
return response()->cookie($cookie);
# 把 cookie 放入佇列，送出 response 前自動加到 Set-Cookie
Cookie::queue(Cookie::make('名稱', 'cookie值', 保存幾分鐘));
Cookie::queue('名稱', 'cookie值', 保存幾分鐘);
return response();
```