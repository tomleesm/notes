---
hide:
  - toc
---

``` php
// 等於 Cookie
cookie();
request()->cookie('name');
Cookie::get('key');
Cookie::get('key', 'default');
// 建立一個永久有效的 cookie
Cookie::forever('key', 'value');
// 建立一個 N 分鐘有效的 cookie
Cookie::make('key', 'value', 'minutes');
cookie('key', 'value', 'minutes');
// 在回應之前先積累 cookie，回應時統一返回
Cookie::queue('key', 'value', 'minutes');
// 移除 Cookie
Cookie::forget('key');
// 從 response 傳送一個 cookie
$response = Response::make('Hello World');
$response->withCookie(Cookie::make('name', 'value', $minutes));
// 設定未加密 Cookie app/Http/Middleware/EncryptCookies.php
EncryptCookies->except = ['cookie_name_1'];
```
