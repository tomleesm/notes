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

## Session

### 設定

把 session 儲存到資料庫

``` bash
# 建立資料表以儲存 session
php artisan session:table
php artisan migrate
# 在 .env 設定 SESSION_DRIVER=database
```

使用 Redis 儲存 session

``` bash
# 安裝 redis
sudo apt install redis-server
sudo systemctl status redis-server.service
redis-cli ping # 輸出 PONG
sudo systemctl enable redis-server.service

# 設定密碼
sudo vim /etc/redis/redis.conf
搜尋 requirepass foobared，並移除開頭的 # 註解
# 產生密碼。最後會產生固定兩個==，複製那之前的字串
openssl rand 100 | openssl base64 -A
# foobared 改成設定的密碼
sudo systemctl restart redis-server.service

# 安裝 php redis extension
sudo apt install php-redis
sudo service php7.3-fpm restart

# 安裝 predis/predis
composer require predis/predis
# 在 .env 設定 SESSION_DRIVER=redis 和 REDIS_PASSWORD=密碼
```

### 抓取 session 的值

``` php
<?php
# 抓取索引值爲 key 的 session
$value = $request->session()->get('key');
# 抓取索引值爲 key 的 session，沒有的話給預設值
$value = $request->session()->get('key', 'default');
$value = $request->session()->get('key', function() {
    return 'default';
});

# 全域函式 session()
$value = session('key');
$value = session('key', 'default');

# 陣列 [ 'key' => 'value', ...] 包含所有的 session
$data = $request->session()->all();
# 抓取索引值爲 key 的 session 值，然後刪除這個 session
# 沒有這個 session 的話，給預設值
$value = $request->session()->pull('key', 'default');
```

### 設定 session 的值

``` php
<?php
# 索引值 key，值爲 value
$request->session()->put('key', 'value');
session( [ 'key' => 'value' ] );

# 設定 session user 的值是陣列 [ 'team' => [ 'Tom', 'Alice' ] ]
$user = [
  'team' => [ 'Tom', 'Alice' ]
];
session([ 'user' => $user ]);
# push()：把值加到陣列後面
# session user 的值是陣列 [ 'team' => [ 'Tom', 'Alice', 'John' ] ]
$request->session()->push('user.team', 'John');

# 新增 session，存取之後就刪除
# 功能和 $request->flashOnly('key') 一樣，但是用一般的 session 語法存取
# 無法用 $request->old('key') 存取
$request->session()->flash('key', 'value');
# 把所有 flash session 保存到下一次存取
# 當然，要在存取前執行，才能保留 session
$request->session()->reflash();
# 把 flash session key1 和 key2 保存到下一次存取
$request->session()->keep( 'key1', 'key2' );
$request->session()->keep( [ 'key1', 'key2' ] );
```

### 刪除 session 的值

``` php
<?php
# 刪除索引值爲 key 的 session
$request->session()->forget('key');
# 刪除索引值爲 key1 和 key2 的 session
$request->session()->forget( [ 'key1', 'key2' ] );
# 刪除所有 session
$request->session()->flush();
```

### 檢查 session

``` php
<?php
# 檢查有沒有索引值爲 key 的 session，而且值不是 null，則回傳 true，否則 false
if ($request->session()->has('key')) {
}
# 檢查有沒有索引值爲 key 的 session，值可以是 null，則回傳 true，否則 false
if ($request->session()->exists('key')) {
}
```

### 重新產生 Session ID

重新生成 Session ID 經常用於阻止惡意用戶對網站進行 [session fixation](https://en.wikipedia.org/wiki/Session_fixation) 攻擊。登入時如果使用內建的 `LoginController` 會自動更新 Session ID。使用 `regenerate()` 手動更新 Session ID
``` php
<?php
$request->session()->regenerate();
```

如果要在建構式中存取、新增 session，必須用全域函式 `session()`，因爲 Request 物件無法注入建構式