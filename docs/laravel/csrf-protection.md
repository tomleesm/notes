# CSRF 保護

對於會修改資料的 HTTP 請求，(POST、PUT、PATCH、DELETE)，Laravel 會自動提供 CSRF 保護。Laravel 會自動產生一個字串，類似 XjPG16nkhyTaeG0dTx2zkFhosWynfsOcI79kuSna，儲存在 session 中，然後必須在 HTTP 請求中包含同樣的字串（隱藏欄位、header X-CSRF-TOKEN、cookie），Laravel 會用 middleware VerifyCsrfToken 檢查 session 和請求中的字串兩者是否相同，是的話繼續執行，否則產生狀態碼 419 錯誤。

在 app/Http/Kernel.php 的 `$middlewareGroups` 設定 web 有套用 VerifyCsrfToken，所以 routes/web.php 會自動給予 CSRF 保護，routes/api.php 則沒有。

## 產生 CSRF token

``` html
<form method="POST" action="/profile">
    <!--
    以下這兩個都是產生 <input type="hidden" name="_token" value="ozrUaeXZUx0riFNOkn7J2uGZ2OnjLSfHmga4Riw6">
    -->
    @csrf
    {{ csrf_field() }}
</form>
```

## X-CSRF-TOKEN

在 `<meta>` 設定 CSRF token
``` html
<meta name="csrf-token" content="{{ csrf_token() }}">
```

然後設定 header X-CSRF-TOKEN

### jQuery

```javascript
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});

// 路由設定 POST /ajax 回傳字串 ajax
$.ajax({
  method: "POST",
  url: "/ajax",
  data: { name: "John", location: "Boston" }
}).done(function( msg ) {
// 載入頁面時會用 alert 顯示 Data Saved: ajax
  alert( "Data Saved: " + msg );
});
```

### Fetch

路由設定
``` php
<?php
use Illuminate\Http\Request;
Route::post('/ajax', function(Request $request) {
  return [
    'message' => $request->input('name')
  ];
});
```

javascript 設定
```javascript
const token = document.querySelector('meta[name="csrf-token"]').getAttribute('content');

// 要設定 Content-Type: application/json
// body 才能以 JSON 格式送入 server
fetch('/ajax', {
    method: 'POST',
    headers: new Headers({
        'X-CSRF-TOKEN': token,
        'Content-Type': 'application/json'
    }),
    body: JSON.stringify({
        "name": "Alice"
    })
}).then(function( response ) {
   return response.json();
}).then(function ( response ) {
// 載入頁面時會用 alert 顯示 Data Saved: Alice
    alert( "Fetch Data Saved: " + response.message );
});
```

## X-XSRF-TOKEN

把 CSRF token 儲存在名爲 `XSRF-TOKEN` 的 Cookie 中，可以用它來設定 HTTP request 的 header X-XSRF-TOKEN，也能有 CSRF 保護，如下所示。

但是實測結果只能用一次，之後就會產生 419 錯誤，所以不推薦使用

``` javascript
function getCookie(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    if (parts.length === 2) return parts.pop().split(';').shift();
}
// jQuery
$.ajaxSetup({
    headers: {
        'X-XSRF-TOKEN': getCookie('XSRF-TOKEN')
    }
});
```

## 排除 CSRF 保護
`routes/api.php` 自動排除在 CSRF 保護中，或設定 `app/Http/middleware/VerifyCsrfToken.php` 的屬性 `$except` ，把路由加到字串陣列中，就能排除 CSRF 保護

``` php
protected $except = [
  'pay/*',
  'http://example.com/foo/bar',
  'http://example.com/foo/*',
];
```