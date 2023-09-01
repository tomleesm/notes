# Fortify 與前端無關的身份認證後端實現

## 介紹

[Laravel Fortify](https://github.com/laravel/fortify) 是一個在Laravel中與前端無關的身份認證後端實現。 Fortify 註冊了所有實現 Laravel 身份驗證功能所需的路由和 controller , 包括登錄、註冊、重設密碼、郵件驗證等。安裝 Fortify 後，你可以運行 Artisan 命令 `route:list` 來查看 Fortify 已註冊的路由。

由於 Fortify 不提供其自己的使用者介面，因此它應該與你自己的使用者介面配對，該使用者介面向其註冊的路由發出請求。在本文件的其餘部分中，我們將進一步討論如何向這些路由發出請求。

> **提示**  
> 請記住， Fortify 是一個軟體包，旨在使你能夠快速開始實現 Laravel 的身份驗證功能。 **你並非一定要使用它。**  你始終可以按照以下說明中提供的文件，自由地與 Laravel 的身份認證服務進行互動， [使用者認證](/docs/laravel/10.x/authentication)， [重設密碼](/docs/laravel/10.x/passwords) 和  [信箱認證](/docs/laravel/10.x/verification) 文件。

### Fortify 是什麼？

如上所述，Laravel Fortify 是一個與前端無關的身份認證後端實現，Fortify 註冊了所有實現 Laravel 身份驗證功能所需的路由和 controller ，包括登錄，註冊，重設密碼，郵件認證等。

**你不必使用 Fortify，也可以使用 Laravel 的身份認證功能。** 你始終可以按照 [使用者認證](/docs/laravel/10.x/authentication)，[重設密碼](/docs/laravel/10.x/passwords) 和 [信箱認證](/docs/laravel/10.x/verification) 文件中提供的文件來手動與 Laravel 的身份驗證服務進行互動。

如果你是一名新手，在使用 Laravel Fortify 之前不妨嘗試使用 [Laravel Breeze](/docs/laravel/10.x/starter-kits) 應用入門套件。Laravel Breeze 為你的應用提供身份認證支架，其中包括使用 [Tailwind CSS](https://tailwindcss.com)。與 Fortify 不同，Breeze 將其路由和 controller 直接發佈到你的應用程式中。這使你可以學習並熟悉 Laravel 的身份認證功能，然後再允許 Laravel Fortify 為你實現這些功能。

Laravel Fortify 本質上是採用了 Laravel Breeze 的路由和 controller ，且提供了不包含使用者介面的擴展。這樣，你可以快速搭建應用程式身份認證層的後端實現，而不必依賴於任何特定的前端實現。

### 何時使用 Fortify？

你可能想知道何時使用 Laravel Fortify。首先，如果你正在使用 Laravel 的 [應用入門套件](/docs/laravel/10.x/starter-kits)，你不需要安裝 Laravel Fortify，因為它已經提供了完整的身份認證實現。

如果你不使用應用入門套件，並且你的應用需要身份認證功能，則有兩個選擇：手動實現應用的身份認證功能或使用由 Laravel Fortify 提供這些功能的後端實現。

如果你選擇安裝 Fortify，你的使用者介面將向 Fortify 的身份驗證路由發出請求，本文件中對此進行了詳細介紹，以便對使用者進行身份認證和註冊。

如果你選擇手動與 Laravel 的身份認證服務進行互動而不是使用 Fortify，可以按照 [使用者認證](/docs/laravel/10.x/authentication)，[重設密碼](/docs/laravel/10.x/passwords) 和 [信箱認證](/docs/laravel/10.x/verification) 文件中提供的說明進行操作。

#### Laravel Fortify & Laravel Sanctum

一些開發人員對 [Laravel Sanctum](/docs/laravel/10.x/sanctum) 和 Laravel Fortify 兩者之間的區別感到困惑。由於這兩個軟體包解決了兩個不同但相關的問題，因此 Laravel Fortify 和 Laravel Sanctum 並非互斥或競爭的軟體包。

Laravel Sanctum 只關心管理 API 令牌和使用 session  cookie 或令牌來認證現有使用者。Sanctum 不提供任何處理使用者註冊，重設密碼等相關的路由。

如果你嘗試為提供 API 或用作單頁應用的後端的應用手動建構身份認證層，那麼完全有可能同時使用 Laravel Fortify（用於使用者註冊，重設密碼等）和 Laravel Sanctum（API 令牌管理， session 身份認證）。

## 安裝

首先，使用 Composer 軟體包管理器安裝 Fortify：

```shell
composer require laravel/fortify
```


下一步，使用 `vendor:publish` 命令來發佈 Fortify 的資源：

```shell
php artisan vendor:publish --provider="Laravel\Fortify\FortifyServiceProvider"
```

該命令會將 Fortify 的行為類發佈到你的 `app/Actions` 目錄，如果該目錄不存在，則會建立該目錄。此外，還將發佈 Fortify 的組態（`FortifyServiceProvider`）和遷移檔案。

下一步，你應該遷移資料庫：

```shell
php artisan migrate
```

### Fortify 服務提供商

上面討論的 `vendor:publish` 命令還將發佈 `App\Providers\FortifyServiceProvider` 類。你應該確保該類已在應用程式的 `config/app.php` 組態檔案的 `providers` 陣列中註冊。

Fortify 服務提供商註冊了 Fortify 所發佈的行為類，並指導 Fortify 在執行各自的任務時使用它們。

### Fortify 包含的功能

該 `fortify` 組態檔案包含一個 `features` 組態陣列。該陣列默路定義了 Fortify 的路由和功能。如果你不打算將 Fortify 與 [Laravel Jetstream](https://jetstream.laravel.com) 配合使用，我們建議你僅啟用以下功能，這是大多數 Laravel 應用提供的基本身份認證功能：

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

### 停用檢視

默認情況下，Fortify 定義用於返回檢視的路由，例如登錄或註冊。但是，如果要建構 JavaScript 驅動的單頁應用，那麼可能不需要這些路由。因此，你可以通過將 `config/fortify.php` 組態檔案中的  `views` 組態值設為 `false` 來停用這些路由：

```php
'views' => false,
```

#### 停用檢視 & 重設密碼

如果你選擇停用 Fortify 的檢視，並且將為你的應用實現重設密碼功能，這時你仍然需要定義一個名為 `password.reset` 的路由，該路由負責顯示應用的「重設密碼」檢視。這是必要的，因為 Laravel 的 `Illuminate\Auth\Notifications\ResetPassword` 通知將通過名為 `password.reset` 的路由生成重設密碼 URL。

## 身份認證

首先，我們需要指導 Fortify 如何返回「登錄」檢視。記住，Fortify 是一個無頭認證擴展。如果你想要一個已經為你完成的 Laravel 身份認證功能的前端實現，你應該使用 [應用入門套件](/docs/laravel/9.x/starter-kits)。

所有的身份認證檢視邏輯，都可以使用 `Laravel\Fortify\Fortify` 類提供的方法來自訂。通常，你應該從應用的 `App\Providers\FortifyServiceProvider` 的 `boot` 方法中呼叫此方法。Fortify 將負責定義返回此檢視的 `/login` 路由：

    use Laravel\Fortify\Fortify;

    /**
     * 引導任何應用服務。
     */
    public function boot(): void
    {
        Fortify::loginView(function () {
            return view('auth.login');
        });

        // ...
    }

你的登錄範本應包括一個向 `/login` 發出 POST 請求的表單。 `/login` 表單需要一個 `email` / `username` 和 `password`。 `email` / `username` 欄位與 `config/fortify.php` 組態檔案中的 `username` 值相匹配。另外，可以提供布林值 `remember` 欄位來指導使用者想要使用 Laravel 提供的「記住我」功能。

如果登錄嘗試成功，Fortify 會將你重新導向到通過應用程式 `fortify` 組態檔案中的 `home` 組態選項組態的 URI。如果登錄請求是 XHR 請求，將返回 200 HTTP 響應。

如果請求不成功，使用者將被重新導向回登錄頁，驗證錯誤將通過共享的 `$errors` [Blade 範本變數](/docs/laravel/10.x/validation#quick-displaying-the-validation-errors) 提供給你。或者，在 XHR 請求的情況下，驗證錯誤將與 422 HTTP 響應一起返回。

### 自訂使用者認證

Fortify 將根據提供的憑據和為你的應用程式組態的身份驗證保護自動檢索和驗證使用者。但是，你有時可能希望對登錄憑據的身份驗證和使用者的檢索方式進行完全自訂。幸運的是，Fortify 允許你使用 `Fortify::authenticateUsing` 方法輕鬆完成此操作。

此方法接受接收傳入 HTTP 請求的閉包。閉包負責驗證附加到請求的登錄憑據並返回關聯的使用者實例。如果憑據無效或找不到使用者，則閉包應返回 `null` 或 `false` 。通常，這個方法應該從你的 `FortifyServiceProvider` 的 `boot` 方法中呼叫：

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Laravel\Fortify\Fortify;

/**
 * 引導應用服務
 */
public function boot(): void
{
    Fortify::authenticateUsing(function (Request $request) {
        $user = User::where('email', $request->email)->first();

        if ($user &&
            Hash::check($request->password, $user->password)) {
            return $user;
        }
    });

    // ...
}
```

#### 身份驗證看守器

你可以在應用程式的 `fortify` 檔案中自訂 Fortify 使用的身份驗證看守器。但是，你應該確保組態的看守器是 `Illuminate\Contracts\Auth\StatefulGuard` 的實現。如果你嘗試使用 Laravel Fortify 對 SPA 進行身份驗證，你應該將 Laravel 的默認 `web` 防護與 [Laravel Sanctum](https://laravel.com/docs/sanctum) 結合使用。

### 自訂身份驗證管道

Laravel Fortify 通過可呼叫類的管道對登錄請求進行身份驗證。如果你願意，你可以定義一個自訂的類管道，登錄請求應該通過管道傳輸。每個類都應該有一個 `__invoke` 方法，該方法接收傳入 `Illuminate\Http\Request` 實例的方法，並且像 [中介軟體](/docs/laravel/10.x/middleware) 一樣，呼叫一個 `$next` 變數，以便將請求傳遞給管道中的下一個類。

要定義自訂管道，可以使用 `Fortify::authenticateThrough` 方法。此方法接受一個閉包，該閉包應返回類陣列，以通過管道傳遞登錄請求。通常，應該從 `App\Providers\FortifyServiceProvider` 的 `boot` 方法呼叫此方法。

下面的示例包含默認管道定義，你可以在自己進行修改時將其用作開始：

```php
use Laravel\Fortify\Actions\AttemptToAuthenticate;
use Laravel\Fortify\Actions\EnsureLoginIsNotThrottled;
use Laravel\Fortify\Actions\PrepareAuthenticatedSession;
use Laravel\Fortify\Actions\RedirectIfTwoFactorAuthenticatable;
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

Fortify::authenticateThrough(function (Request $request) {
    return array_filter([
            config('fortify.limiters.login') ? null : EnsureLoginIsNotThrottled::class,
            Features::enabled(Features::twoFactorAuthentication()) ? RedirectIfTwoFactorAuthenticatable::class : null,
            AttemptToAuthenticate::class,
            PrepareAuthenticatedSession::class,
    ]);
});
```

### 自訂跳轉

如果登錄嘗試成功，Fortify 會將你重新導向到你應用程式 `Fortify` 的組態檔案中的 `home` 組態選項的 URI 值。如果登錄請求是 XHR 請求，將返回 200 HTTP 響應。使用者註銷應用程式後，該使用者將被重新導向到 `/` 地址。

如果需要對這種行為進行高級定製，可以將 `LoginResponse` 和 `LogoutResponse` 契約的實現繫結到 Laravel [服務容器](/docs/laravel/10.x/container) 。通常，這應該在你應用程式的 `App\Providers\FortifyServiceProvider` 類的 `register` 方法中完成：

```php
use Laravel\Fortify\Contracts\LogoutResponse;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

/**
 * 註冊任何應用程式服務。
 */
public function register(): void
{
    $this->app->instance(LogoutResponse::class, new class implements LogoutResponse {
        public function toResponse(Request $request): RedirectResponse
        {
            return redirect('/');
        }
    });
}
```

## 雙因素認證

當 Fortify 的雙因素身份驗證功能啟用時，使用者需要在身份驗證過程中輸入一個六位數的數字令牌。該令牌使用基於時間的一次性密碼（TOTP）生成，該密碼可以從任何與 TOTP 相容的移動認證應用程式（如 Google Authenticator）中檢索。

在開始之前，你應該首先確保應用程式的 `App\Models\User` 模型使用 `Laravel\Fortify\TwoFactorAuthenticatable` trait：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Fortify\TwoFactorAuthenticatable;

class User extends Authenticatable
{
    use Notifiable, TwoFactorAuthenticatable;
}
 ```

接下來，你應該在應用程式中建構一個頁面，使用者可以在其中管理他們的雙因素身份驗證設定。該頁面應允許使用者啟用和停用雙因素身份驗證，以及重新生成雙因素身份驗證恢復的程式碼。

> 默認情況下， `fortify` 組態檔案的 `features` 陣列管理著 Fortify 的雙因素身份驗證設定在修改前需要密碼確認。因此，在使用之前，你的應用程式應該實現 Fortify 的 [密碼確認](#password-confirmation) 功能。

### 啟用雙因素身份驗證

要啟用雙重身份驗證，你的應用程式應向 Fortify 定義的 `/user/two-factor-authentication` 發出 POST 請求。如果請求成功，使用者將被重新導向回之前的 URL，並且 `status` session 變數將設定為 `two-factor-authentication-enabled`。你可以在範本中檢測這個 `status` session 變數以顯示適當的成功消息。如果請求是 XHR 請求，將返回  `200` HTTP 響應：

在選擇啟用雙因素認證後，使用者仍然必須通過提供一個有效的雙因素認證程式碼來「確認」他們的雙因素認證組態。因此，你的「成功」消息應該指示使用者，雙因素認證的確認仍然是必需的。

```html
@if (session('status') == 'two-factor-authentication-enabled')
    <div class="mb-4 font-medium text-sm">
        Please finish configuring two factor authentication below.
    </div>
@endif
```

接下來，你應該顯示雙重身份驗證二維碼，供使用者掃描到他們的身份驗證器應用程式中。如果你使用 Blade 呈現應用程式的前端，則可以使用使用者實例上可用的 `twoFactorQrCodeSvg` 方法檢索二維碼 SVG：

```php
$request->user()->twoFactorQrCodeSvg();
```

如果你正在建構由 JavaScript 驅動的前端，你可以向 `/user/two-factor-qr-code` 發出 XHR GET 請求以檢索使用者的雙重身份驗證二維碼。將返回一個包含 `svg` 鍵的 JSON 對象。

#### 確認雙因素認證

除了顯示使用者的雙因素認證 QR 碼，你應該提供一個文字輸入，使用者可以提供一個有效的認證碼來「確認」他們的雙因素認證組態。這個程式碼應該通過 POST 請求提供到 `/user/confirmed-two-factor-authentication`，由 Fortify 來進行確認。

If the request is successful, the user will be redirected back to the previous URL and the `status` session variable will be set to `two-factor-authentication-confirmed`:

如果請求成功，使用者將被重新導向到之前的URL，`status`  session 變數將被設定為 `two-factor-authentication-confirmed'。

```html
@if (session('status') == 'two-factor-authentication-confirmed')
    <div class="mb-4 font-medium text-sm">
        Two factor authentication confirmed and enabled successfully.
    </div>
@endif
```

如果對雙因素認證確認端點的請求是通過 XHR 請求進行的，將返回一個 `200` HTTP響應。

#### 顯示恢復程式碼

你還應該顯示使用者的兩個因素恢復程式碼。這些恢復程式碼允許使用者在無法訪問其移動裝置時進行身份驗證。如果你使用 Blade 來渲染應用程式的前端，你可以通過經過身份驗證的使用者實例訪問恢復程式碼：

```php
(array) $request->user()->recoveryCodes()
```

如果你正在建構一個 JavaScript 驅動的前端，你可以向 `/user/two-factor-recovery-codes` 端點發出 XHR GET 請求。此端點將返回一個包含使用者恢復程式碼的 JSON 陣列。

要重新生成使用者的恢復程式碼，你的應用程式應向 `/user/two-factor-recovery-codes` 端點發出 POST 請求。

### 使用雙因素身份驗證進行身份驗證

在身份驗證過程中，Fortify 將自動將使用者重新導向到你的應用程式的雙因素身份驗證檢查頁面。但是，如果你的應用程式正在發出 XHR 登錄請求，則在成功進行身份驗證嘗試後返回的 JSON 響應將包含一個具有 `two_factor` 布林值屬性的 JSON 對象。你應該檢查此值以瞭解是否應該重新導向到應用程式的雙因素身份驗證檢查頁面。

要開始實現兩因素身份驗證功能，我們需要指示 Fortify 如何返回我們的雙因素身份驗證檢查頁面。Fortify 的所有身份驗證檢視渲染邏輯都可以使用通過 `Laravel\Fortify\Fortify` 類提供的適當方法進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法呼叫此方法：

```php
use Laravel\Fortify\Fortify;

/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Fortify::twoFactorChallengeView(function () {
        return view('auth.two-factor-challenge');
    });

    // ...
}
```

Fortify 將負責定義返回此檢視的 `/two-factor-challenge` 路由。你的 `two-factor-challenge` 範本應包含一個向 `/two-factor-challenge` 端點發出 POST 請求的表單。 `/two-factor-challenge` 操作需要包含有效 TOTP 令牌的 `code` 欄位或包含使用者恢復程式碼之一的 `recovery_code` 欄位。

如果登錄嘗試成功，Fortify 會將使用者重新導向到通過應用程式的 `fortify` 組態檔案中的 `home` 組態選項組態的 URI。如果登錄請求是 XHR 請求，將返回 204 HTTP 響應。

如果請求不成功，使用者將被重新導向回兩因素挑戰螢幕，驗證錯誤將通過共享的 `$errors` [Blade 範本變數](/docs/laravel/10.x/驗證#快速顯示驗證錯誤)。或者，在 XHR 請求的情況下，驗證錯誤將返回 422 HTTP 響應。

### 停用兩因素身份驗證

要停用雙因素身份驗證，你的應用程式應向 `/user/two-factor-authentication` 端點發出 DELETE 請求。請記住，Fortify 的兩個因素身份驗證端點在被呼叫之前需要 [密碼確認](#password-confirmation)。

## 註冊

要開始實現我們應用程式的註冊功能，我們需要指示 Fortify 如何返回我們的“註冊”檢視。請記住，Fortify 是一個無頭身份驗證庫。如果你想要一個已經為你完成的 Laravel 身份驗證功能的前端實現，你應該使用 [application starter kit](/docs/laravel/10.x/starter-kits)。

Fortify 的所有檢視渲染邏輯都可以使用通過 `Laravel\Fortify\Fortify` 類提供的適當方法進行自訂。通常，你應該從 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法呼叫此方法：

```php
use Laravel\Fortify\Fortify;

/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Fortify::registerView(function () {
        return view('auth.register');
    });

    // ...
}
```

Fortify 將負責定義返回此檢視的 `/register` 路由。你的 `register` 範本應包含一個向 Fortify 定義的 `/register` 端點發出 POST 請求的表單。

`/register` 端點需要一個字串 `name`、字串電子郵件地址/使用者名稱、`password` 和 `password_confirmation` 欄位。電子郵件/使用者名稱欄位的名稱應與應用程式的 `fortify` 組態檔案中定義的 `username` 組態值匹配。

如果註冊嘗試成功，Fortify 會將使用者重新導向到通過應用程式的 `fortify` 組態檔案中的 `home` 組態選項組態的 URI。如果登錄請求是 XHR 請求，將返回 201 HTTP 響應。

如果請求不成功，使用者將被重新導向回註冊螢幕，驗證錯誤將通過共享的 `$errors` [Blade 範本變數](/docs/laravel/10.x/validation#快速顯示驗證錯誤)。或者，在 XHR 請求的情況下，驗證錯誤將返回 422 HTTP 響應。

### 定製註冊

可以通過修改安裝 Laravel Fortify 時生成的 `App\Actions\Fortify\CreateNewUser` 操作來自訂使用者驗證和建立過程。

## 重設密碼

### 請求密碼重設連結

要開始實現我們應用程式的密碼重設功能，我們需要指示 Fortify 如何返回我們的“忘記密碼”檢視。請記住，Fortify 是一個無頭身份驗證庫。如果你想要一個已經為你完成的 Laravel 身份驗證功能的前端實現，你應該使用 [application starter kit](/docs/laravel/10.x/starter-kits)。

Fortify 的所有檢視渲染邏輯都可以使用通過 `Laravel\Fortify\Fortify` 類提供的適當方法進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法呼叫此方法：

```php
use Laravel\Fortify\Fortify;

/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Fortify::requestPasswordResetLinkView(function () {
        return view('auth.forgot-password');
    });

    // ...
}
```

Fortify 將負責定義返回此檢視的 `/forgot-password` 端點。你的 `forgot-password` 範本應該包含一個向 `/forgot-password` 端點發出 POST 請求的表單。

`/forgot-password` 端點需要一個字串 `email` 欄位。此欄位/資料庫列的名稱應與應用程式的 `fortify` 組態檔案中的 `email` 組態值匹配。

#### 處理密碼重設連結請求響應

如果密碼重設連結請求成功，Fortify 會將使用者重新導向回 `/forgot-password` 端點，並向使用者傳送一封電子郵件，其中包含可用於重設密碼的安全連結。如果請求是 XHR 請求，將返回 200 HTTP 響應。

在請求成功後被重新導向到 `/forgot-password` 端點，`status`  session 變數可用於顯示密碼重設連結請求的狀態。

在成功請求後被重新導向回 `/forgot-password` 端點後，`status`  session 變數可用於顯示密碼重設連結請求嘗試的狀態。此 session 變數的值將匹配應用程式的 `password` [語言檔案](/docs/laravel/10.x/localization) 中定義的翻譯字串之一：

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

如果請求不成功，使用者將被重新導向回請求密碼重設連結螢幕，驗證錯誤將通過共享的 `$errors`  [Blade 範本變數](/docs/laravel/10.x/validation#quick-displaying-the-validation-errors) 提供給你。或者，在 XHR 請求的情況下，驗證錯誤將返回 422 HTTP 響應。

### 重設密碼

為了完成應用程式的密碼重設功能，我們需要指示 Fortify 如何返回我們的「重設密碼」檢視。

Fortify 的所有檢視渲染邏輯都可以使用通過 `Laravel\Fortify\Fortify` 類提供的適當方法進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法呼叫此方法：

```php
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Fortify::resetPasswordView(function (Request $request) {
        return view('auth.reset-password', ['request' => $request]);
    });

    // ...
}
```

Fortify 將負責定義顯示此檢視的路線。你的 `reset-password` 範本應該包含一個向 `/reset-password` 發出 POST 請求的表單。

`/reset-password` 端點需要一個字串 `email` 欄位、一個 `password` 欄位、一個 `password_confirmation` 欄位和一個名為 `token` 的隱藏欄位，其中包含`request()->route('token')`。 `email` 欄位/資料庫列的名稱應與應用程式的 `fortify` 組態檔案中定義的 `email` 組態值匹配。

#### 處理密碼重設響應

如果密碼重設請求成功，Fortify 將重新導向回 `/login` 路由，以便使用者可以使用新密碼登錄。此外，還將設定一個 `status`  session 變數，以便你可以在登錄螢幕上顯示重設的成功狀態：

```blade
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

如果請求是 XHR 請求，將返回 200 HTTP 響應。

如果請求不成功，使用者將被重新導向回重設密碼螢幕，驗證錯誤將通過共享的 `$errors` [Blade 範本變數](/docs/laravel/10.x/validation#快速顯示驗證錯誤)。或者，在 XHR 請求的情況下，驗證錯誤將返回 422 HTTP 響應。

### 自訂密碼重設

可以通過修改安裝 Laravel Fortify 時生成的 `App\Actions\ResetUserPassword` 操作來自訂密碼重設過程。

## 電子郵件驗證

註冊後，你可能希望使用者在繼續訪問你的應用程式之前驗證他們的電子郵件地址。要開始使用，請確保在 `fortify` 組態檔案的 `features` 陣列中啟用了 `emailVerification` 功能。接下來，你應該確保你的 `App\Models\User` 類實現了 `Illuminate\Contracts\Auth\MustVerifyEmail` 介面。

完成這兩個設定步驟後，新註冊的使用者將收到一封電子郵件，提示他們驗證其電子郵件地址的所有權。但是，我們需要通知 Fortify 如何顯示電子郵件驗證螢幕，通知使用者他們需要點選電子郵件中的驗證連結。

Fortify 的所有檢視的渲染邏輯都可以使用通過 `Laravel\Fortify\Fortify` 類提供的適當方法進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法呼叫此方法：

```php
use Laravel\Fortify\Fortify;

/**
 * 引導所有應用程式服務。
 */
public function boot(): void
{
    Fortify::verifyEmailView(function () {
        return view('auth.verify-email');
    });

    // ...
}
```

當使用者被 Laravel 內建的 `verified` 中介軟體重新導向到 `/email/verify` 端點時，Fortify 將負責定義顯示此檢視的路由。

你的 `verify-email` 範本應包含一條資訊性消息，指示使用者點選傳送到其電子郵件地址的電子郵件驗證連結。

#### 重新傳送電子郵件驗證連結

如果你願意，你可以在應用程式的 `verify-email` 範本中新增一個按鈕，該按鈕會觸發對 `/email/verification-notification` 端點的 POST 請求。當此端點收到請求時，將通過電子郵件將新的驗證電子郵件連結傳送給使用者，如果先前的驗證連結被意外刪除或丟失，則允許使用者獲取新的驗證連結。

如果重新傳送驗證連結電子郵件的請求成功，Fortify 將使用 `status`  session 變數將使用者重新導向回 `/email/verify` 端點，允許你向使用者顯示資訊性消息，通知他們操作已完成成功的。如果請求是 XHR 請求，將返回 202 HTTP 響應：

```blade
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        A new email verification link has been emailed to you!
    </div>
@endif
```

### 保護路由

要指定一個路由或一組路由要求使用者驗證他們的電子郵件地址，你應該將 Laravel 的內建 `verified` 中介軟體附加到該路由。該中介軟體在你的應用程式的 `App\Http\Kernel` 類中註冊：

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

## 確認密碼

在建構應用程式時，你可能偶爾會有一些操作需要使用者在執行操作之前確認其密碼。通常，這些路由受到 Laravel 內建的 `password.confirm` 中介軟體的保護。

要開始實現密碼確認功能，我們需要指示 Fortify 如何返回應用程式的「密碼確認」檢視。請記住，Fortify 是一個無頭身份驗證庫。如果你想要一個已經為你完成的 Laravel 身份驗證功能的前端實現，你應該使用 [application starter kit](/docs/laravel/10.x/starter-kits)。

Fortify 的所有檢視渲染邏輯都可以使用通過 `Laravel\Fortify\Fortify` 類提供的適當方法進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法呼叫此方法：

```php
use Laravel\Fortify\Fortify;

/**
 * 引導所有應用程式服務。
 */
public function boot(): void
{
    Fortify::confirmPasswordView(function () {
        return view('auth.confirm-password');
    });

    // ...
}
```

Fortify 將負責定義返回此檢視的 `/user/confirm-password` 端點。你的 `confirm-password` 範本應包含一個表單，該表單向 `/user/confirm-password` 端點發出 POST 請求。 `/user/confirm-password` 端點需要一個包含使用者當前密碼的 `password` 欄位。

如果密碼與使用者的當前密碼匹配，Fortify 會將使用者重新導向到他們嘗試訪問的路由。如果請求是 XHR 請求，將返回 201 HTTP 響應。

如果請求不成功，使用者將被重新導向回確認密碼螢幕，驗證錯誤將通過共享的 `$errors` Blade 範本變數提供給你。或者，在 XHR 請求的情況下，驗證錯誤將返回 422 HTTP 響應。

