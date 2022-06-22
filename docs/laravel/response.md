# HTTP 回應

## 產生回應和設定 header
``` php
<?php
return 'Hello World';
# 產生 JSON 回應 [ 1, 2, 3 ]
return [ 1, 2, 3 ];
# 產生 JSON 回應 { "name": "Tom"}
return [ "name" => "Tom" ];
# 自訂 HTTP 狀態碼和 header
# 注意，經過實測，response() 最少要有一個參數，可以是空字串或 null
# 不能只有 return response()
return response('Hello World', 200)
     ->header('Content-Type', 'text/plain')
     ->header('X-Header', 'header value');
return response('Hello World', 200)
     ->withHeaders([
        'Content-Type' => 'text/plain',
        'X-Header' => 'header value'
    ]);

# middleware cache.headers 快速設定 header Cache-Control
# etag：response body 的 MD5 值將會自動設定到 header ETag
# Cache-Control: max-age=2628000, public
# ETag: "b10a8db164e0754105b7a99be72e3fe5" <-- Hello World 的 md5 checksum
->middleware('cache.headers:public;max_age=2628000;etag')
```

## 不加密與簽署 cookie

在 `app/Http/Middleware/EncryptCookies.php` 設定 `$except` 屬性

``` php
<?php
protected $except = [
  'cookie_name_A', 'cookie_name_B'
];
```

## 重新導向

``` php
<?php
# 產生一個回應，包含 HTML 要求瀏覽器重新導向
# 和 Route::redirect() 一樣
return redirect('/');
# 產生一個回應，包含 HTML 要求瀏覽器重新導向到上一次瀏覽的網址
# 同時用 withInput() 把輸入都儲存到拋棄式 session
# 參考 HTTP 請求「上一次請求的輸入」
return back()->withInput();
# 產生一個回應，包含 HTML 要求瀏覽器重新導向到 route('路由名稱') 代表的網址
# 第二個參數傳遞 123 給第一個路由參數
return redirect()->route('路由名稱', [ 'id' => 123 ]);
# 如果 Route::get('users/{user}', ...)->name('users.show')
# route() 可以直接傳入 Eloquent 物件，資料表欄位 id 的值會自動給第一個路由參數 {user}
# 如果要改成查詢其他資料表欄位，參考路由 getRouteKeyName() 和 getRouteKey()
$user = \App\User::findOrFail(1);
return redirect()->route('users.show', [ $user ]);
```