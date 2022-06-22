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
# 產生一個回應，body 是 HTML 要求瀏覽器重新導向
# 和 Route::redirect() 一樣
return redirect('/');
# 產生一個回應，body 是 HTML 要求瀏覽器重新導向到上一次瀏覽的網址
# 同時用 withInput() 把輸入都儲存到拋棄式 session
# 參考 HTTP 請求「上一次請求的輸入」
return back()->withInput();
# 產生一個回應，body 是 HTML 要求瀏覽器重新導向到 route('路由名稱') 代表的網址
# 第二個參數是傳送 123 給路由參數，陣列 key 值 user 要和路由參數名稱 {user} 相同
# 如果沒有 key，直接給 [ 123 ] 則會傳給第一個路由參數
return redirect()->route('路由名稱', [ 'user' => 123 ]);
# 如果 Route::get('users/{user}', ...)->name('users.show')
# 產生一個回應，body 是 HTML 要求瀏覽器重新導向 route('users.show') 代表的網址
# route() 可以直接傳入 Eloquent 物件，資料表欄位 id 的值會自動給第一個路由參數 {user}
# 陣列 key 值 user 要和路由參數名稱 {user} 相同
# 如果沒有 key，直接給 [ $user ] 則會傳給第一個路由參數
# 如果要改成查詢其他資料表欄位，參考路由 getRouteKeyName() 和 getRouteKey()
$user = \App\User::findOrFail(1);
return redirect()->route('users.show', [ 'user' => $user ]);
# 跳轉到 UserController 的 show()
# 必須要有路由規則指向 UserController@show
# action() 的第二個參數是傳送 1 給路由參數，陣列 key 值 user 要和路由參數名稱 {user} 相同
# 如果沒有 key，直接給 [ 123 ] 則會傳給第一個路由參數
return redirect()->action('UserController@show', [ 'user' => 1]);
# 產生一個回應，body 是 HTML 要求瀏覽器重新導向 https://www.google.com
return redirect()->away('https://www.google.com');

# 產生一個回應，body 是 HTML 要求瀏覽器重新導向 /dashboard
# 同時把 'Profile updated!' 儲存到 session
# 用存取 session 的方式抓取資料，抓取後資料就刪除了
# 注意，不能用 $request->old('status')，所以 withInput() 和 with() 是不同的
return redirect('dashboard')->with('status', 'Profile updated!');
# with() 傳入陣列，一次儲存多個值
return redirect('dashboard')->with( [ 'status' => 'Profile updated!', 'name' => 'Tom' ] );
```

## 其他回應類型

`response()` 沒有參數時回傳 `Illuminate\Contracts\Routing\ResponseFactory` 的實作物件，此物件提供一些有用的方法來生成各種回應，所以以下的語法不能更換呼叫順序

``` php
<?php
# 如果需要設定 header，並且還需要回傳一個 view 作為回應內容
return response()
    ->view('Hello World', [ 'name' => 'Tom' ], 200)
    ->header('Content-Type', 'text/html');

# 回傳 JSON { "name": "Tom", "status": "OK" }
return response()->json([
    'name' => 'Tom', 
    'status' => 'OK'
]);
# 如果 GET /jsonp?callback=abc123
# 回傳 Content-Type: text/javascript
# body: /**/abc123({"name":"Tom","status":"OK"});
return response()->json([
    'name' => 'Tom', 
    'status' => 'OK'
])->withCallback($request->input('callback'));
# 用 jsonp() 也一樣
return response()->jsonp($request->input('callback'), [
    'name' => 'Tom', 
    'status' => 'OK'
]);
```