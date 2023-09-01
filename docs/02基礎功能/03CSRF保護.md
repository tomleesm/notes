# CSRF 保護

## 簡介

跨站點請求偽造是一種惡意利用，利用這種手段，代表經過身份驗證的使用者執行未經授權的命令。值得慶幸的是，Laravel 可以輕鬆保護您的應用程式免受[跨站點請求偽造](https://en.wikipedia.org/wiki/Cross-site_request_forgery)（CSRF）攻擊。

#### 漏洞的解釋

如果你不熟悉跨站點請求偽造，我們討論一個利用此漏洞的示例。假設您的應用程式有一個 `/user/email` 路由，它接受 POST 請求來更改經過身份驗證使用者的電子郵件地址。最有可能的情況是，此路由希望 `email` 輸入欄位包含使用者希望開始使用的電子郵件地址。

沒有 CSRF 保護，惡意網站可能會建立一個 HTML 表單，指向您的應用程式 `/user/email` 路由，並提交惡意使用者自己的電子郵件地址：

```html
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

 如果惡意網站在頁面載入時自動提交了表單，則惡意使用者只需要誘使您的應用程式的一個毫無戒心的使用者訪問他們的網站，他們的電子郵件地址就會在您的應用程式中更改。

為了防止這種漏洞，我們需要檢查每一個傳入的 `POST`，`PUT`，`PATCH` 或 `DELETE` 請求以獲取惡意應用程式無法訪問的秘密 session 值。

## 阻止 CSRF 請求

Laravel 為應用程式管理的每個活動 [使用者 session ](/docs/laravel/10.x/session) 自動生成 CSRF 「令牌」。此令牌用於驗證經過身份驗證的使用者是實際嚮應用程序發出請求的人。由於此令牌儲存在使用者的 session 中，並且每次重新生成 session 時都會更改，因此惡意應用程式將無法訪問它。

當前 session 的 CSRF 令牌可以通過請求的 session 或通過 `csrf_token` 輔助函數進行訪問：

    use Illuminate\Http\Request;

    Route::get('/token', function (Request $request) {
        $token = $request->session()->token();

        $token = csrf_token();

        // ...
    });

無論何時在應用程式中定義 `POST` 、`PUT` 、`PATCH` 或 `DELETE` HTML表單，都應在表單中包含隱藏的CSRF `_token` 欄位，以便CSRF保護中介軟體可以驗證請求。為方便起見，可以使用 `@csrf` Blade指令生成隱藏的令牌輸入欄位：

```html
<form method="POST" action="/profile">
    @csrf

    <!-- 相當於。。。 -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

默認情況下包含在 `web` 中介軟體組中的`App\Http\Middleware\VerifyCsrfToken` [中介軟體](/docs/laravel/10.x/middleware)將自動驗證請求輸入中的令牌是否與 session 中儲存的令牌匹配。當這兩個令牌匹配時，我們知道身份驗證的使用者是發起請求的使用者。

### CSRF Tokens & SPAs

如果你正在建構一個將 Laravel 用作 API 後端的 SPA，你應該查閱 [Laravel Sanctum 文件](/docs/laravel/10.x/sanctum)，以獲取有關使用 API 進行身份驗證和防範 CSRF 漏洞的資訊。

### 從 CSRF 保護中排除 URI

有時你可能希望從 CSRF 保護中排除一組 URIs。例如，如果你使用  [Stripe](https://stripe.com)  處理付款並使用他們的 webhook 系統，則需要將你的 Stripe webhook 處理程序路由從 CSRF 保護中排除，因為 Stripe 不會知道要向您的路由傳送什麼 CSRF 令牌。

通常，你應該將這些類型的路由放在 `App\Providers\RouteServiceProvider` 應用於 routes/web.php 檔案中的所有路由的 `web` 中介軟體組之外。但是，現在也可以通過將路由的 URIs 新增到 `VerifyCsrfToken` 中介軟體的 `$except` 屬性來排除路由：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * 從 CSRF 驗證中排除的 URIs。
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ];
    }

>技巧：為方便起見，[運行測試](/docs/laravel/10.x/testing).時自動停用所有路由的 CSRF 中介軟體。

## X-CSRF-TOKEN

除了檢查 CSRF 令牌作為 POST 參數外， `App\Http\Middleware\VerifyCsrfToken` 中介軟體還將檢查 `X-CSRF-TOKEN` 請求標頭。 例如，你可以將令牌儲存在 HTML 的  `meta` 標籤中：

```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```

然後，你可以指示 jQuery 之類的庫自動將令牌新增到所有請求標頭。 這為使用傳統 JavaScript 技術的基於 AJAX 的應用程式提供了簡單、方便的 CSRF 保護：

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

## X-XSRF-TOKEN

Laravel 將當前CSRF令牌儲存在加密的 `XSRF-TOKEN` cookie 中，該 cookie 包含在框架生成的每個響應中。您可以使用 cookie 值設定 `X-XSRF-TOKEN` 請求標頭。

由於一些 JavaScript 框架和庫（如 Angular 和 Axios ）會自動將其值放置在同一源請求的 `X-XSRF-TOKEN` 標頭中，因此傳送此 cookie 主要是為了方便開發人員。

> 技巧：默認情況下，`resources/js/bootstrap.js` 檔案包含 Axios HTTP 庫，它會自動為您傳送 `X-XSRF-TOKEN` 標頭。

