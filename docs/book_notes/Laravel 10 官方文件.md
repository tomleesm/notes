# 基礎功能

## 表單驗證

新增路由、Controller 和 View 以便進行以下的說明

`routes/web.php`

``` php
use App\Http\Controllers\PostController;

# 顯示新增文章的表單
Route::get('/post/create', [PostController::class, 'create']);
# 新增文章
Route::post('/post', [PostController::class, 'store']);
```

新增 `PostController` 如下

``` bash
php artisan make:controller PostController
```

``` php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\View\View;

class PostController extends Controller
{
    public function create(): View
    {
        return view('post/create');
    }

    public function store(Request $request): View
    {
    }
}
```

新增 view post.create 和 post.show

``` bash
php artisan make:view post.create
php artisan make:view post.show
```

post.create

``` html
<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- CSRF Token -->
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>新增文章</title>

</head>
<body>
    <h1>新增文章</h1>

    <form method="post" action="/post">
        @csrf

        <div>
            <label>標題</label>
            <input type="text" name="title" />
        </div>
        <div>
            <label>內容</label>
            <textarea name="body" cols="50" rows="20"></textarea>
        </div>
        <button type="submit">新增文章</button>
    </form>
</body>
    <!-- Scripts -->
    @vite(['resources/js/app.js'])

    <script type="module">
    </script>
</html>
```

post.show

``` html
<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
    <h1>成功新增</h1>
    <p><a href="/post/create">回到上一頁</a></p>
</body>
</html>
```

### 編寫驗證邏輯

在 PostController，如果驗證失敗，會丟出異常，表示會停在 Request 物件 `validate()`，不會繼續執行，然後：

- 如果傳入的請求是 XHR，則異常會觸發 Laravel 去回傳 JSON 物件給前端，所以不用自己寫 `return` 回傳錯誤訊息
- 如果請求是傳統的多頁式，則異常會觸發 `redirect()` 回到上一頁，錯誤訊息儲存到 session，用 `@error` 或 `@errors` 存取

``` php
<?php
public function store(Request $request)
{
        # 表單驗證
        $validated = $request->validate([
            'title' => 'required|min:3'
        ]);

        # 表單驗證正確，才會執行以下的 return
        return [
            'status' => 'success create new post',
            'data' => [
                'post' => [
                    'id' => 1,
                    'title' => $request->title,
                    'body' => $request->body,
                    'created_at' => now()
                ]
            ]
        ];
}
```

表單驗證的 AJAX 客戶端程式碼範例（後端 Laravel）

HTML 表單

``` html
<form method="post" action="/post">
        @csrf

        <div>
            <label>標題</label>
            <input type="text" name="title" />
        </div>
        <div>
            <label>內容</label>
            <textarea name="body" cols="50" rows="20"></textarea>
        </div>
        <button type="submit">新增文章</button>
    </form>
```

新增

JavaScript

``` js
$('form').on('submit', function(event) {
    // 不要送出表單
    event.preventDefault();

    $.ajaxSetup({
        headers: {
            // 抓取 <meta name="csrf-token" content="{{ csrf_token() }}"> 的值
            // 否則會丟出 419 錯誤訊息
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });
    $.ajax({
        method: 'POST',
        // 在 VirtualBox VM 中使用和 host 相同的網址，否則
        // POST http://127.0.0.1:7000/post net::ERR_CONNECTION_REFUSED
        url: 'http://192.168.56.10:7000/post',
        // jQuery 預設的請求 contentType，如要用 json 則設定為 application/json
        contentType: 'application/x-www-form-urlencoded; charset=UTF-8',
        // $(this).serialize() 其值如下，如果 title 輸入 abc，body 留空，_token 是 CSRF token，因為表單有 @csrf
        // _token=oKq5SyYeHZFfqgw0KgdhiS62w9zuVbYYMiuGpRoQ&title=abc&body=
        data: $(this).serialize(),
        // 設定回應的資料類型為 json，則在以下的 success 等 function 參數會自動轉換參數為 json 物件，不用自己 JSON.parse()
        dataType: 'json',
        // status code 200 之類時，呼叫這個 function，response 已經自動轉成 json 物件
        success: function(response) {
            // 標示呼叫的是這裡
            console.log('success');
            console.log(response);
       },
        // Laravel 驗證失敗時會自動回傳 status code 422，所以會呼叫 function error
        // jqxhr 物件的屬性 responseJSON 和 responseText 即為你需要的驗證錯誤訊息
        error: function(jqxhr, statusMessage, extra) {
            // 標示呼叫的是這裡
            console.log(statusMessage);
            console.log(jqxhr);
        }
    });
});
```

jqxhr 驗證的錯誤訊息類似這樣

``` js
{
  responseText: "{\"message\":\"The title field is required.\",\"errors\",
  responseJSON: {
    message: "The title field is required.",
    errors: {
      title: [
        "The title field is required."
      ]
    }
  }
}
```

#### 巢狀欄位

如有以下的巢狀欄位

``` html
<input type="text" name="author[name]" />
<input type="text" name="v1.0" />
```

欄位名稱可以用 `author[name]` 或 `author.name`。名稱如果包含點（`v1.0`），實測不需要如下的 escape 也可以，但是官方文件說可以用反斜線 escape 防止解釋為 `.` 語法

``` php
<?php
$validationResult = $request->validate([
  'author[name]' => 'required',
  'author.name' => 'required',
  'v1\.0' => 'required'
]);
```

`validateWithBag('名稱')` 只適用 `@error`，在 XHR 回傳的 JSON 沒有任何改變，也不適用 `@errors`

``` php
<?php
$validationResult = $request->validateWithBag('post', [
  'title' => 'required|min:3'
]);
```
