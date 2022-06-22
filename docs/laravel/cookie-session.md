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

- 可以省略參數「保存幾分鐘」
- 注意，經過實測，`response()` 最少要有一個參數，可以是空字串或 null，不能只有 `return response()`

``` php
<?php
return response('訊息')
       ->cookie('名稱', 'cookie值', 保存幾分鐘);
# cookie() 產生 Cookie 物件後加到 response()->cookie()
$cookie = cookie('名稱', 'cookie值', 保存幾分鐘);
return response('訊息')->cookie($cookie);
# 把 cookie 放入佇列，送出 response 前自動加到 Set-Cookie
use Illuminate\Support\Facades\Cookie;
Cookie::queue(Cookie::make('名稱', 'cookie值', 保存幾分鐘));
Cookie::queue('名稱', 'cookie值', 保存幾分鐘);
return response('訊息');
```