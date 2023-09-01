# Email 認證

## 簡介

很多 Web 應用會要求使用者在使用之前進行 Email 地址驗證。Laravel 不會強迫你在每個應用中重複實現它，而是提供了便捷的方法來傳送和校驗電子郵件的驗證請求。

> **技巧**
> 想快速上手嗎？你可以在全新的應用中安裝 [Laravel 應用入門套件](/docs/laravel/10.x/starter-kits) 。入門套件將幫助你搭建整個身份驗證系統，包括電子郵件驗證支援。

### 準備模型

在開始之前，需要檢查你的 `App\Models\User` 模型是否實現了 `Illuminate\Contracts\Auth\MustVerifyEmail` 契約：

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        use Notifiable;

        // ...
    }

一旦這一介面被新增到模型中，新註冊的使用者將自動收到一封包含電子郵件驗證連結的電子郵件。檢查你的 `App\Providers\EventServiceProvider` 可以看到，Laravel 已經為 `Illuminate\Auth\Events\Registered` 事件註冊了一個 `SendEmailVerificationNotification` [監聽器](/docs/laravel/10.x/events) 。這個事件監聽器會通過郵件傳送驗證連結給使用者。
如果在應用中你沒有使用 [入門套件](/docs/laravel/10.x/starter-kits) 而是手動實現的註冊，你需要確保在使用者註冊成功後手動分發 `Illuminate\Auth\Events\Registered` 事件：

    use Illuminate\Auth\Events\Registered;

    event(new Registered($user));



### 資料庫準備

接下來，你的 `users` 表必須有一個 `email_verified_at` 欄位，用來儲存使用者信箱驗證的日期和時間。Laravel 框架自帶的 `users` 表以及默認包含了該欄位。因此，你只需運行資料庫遷移即可：

```shell
php artisan migrate
```

## 路由

為了實現完整的電子郵件驗證流程，你將需要定義三個路由。首先，需要定義一個路由向使用者顯示通知，告訴使用者應該點選註冊之後， Laravel 向他們傳送的驗證郵件中的連結。

其次，需要一個路由來處理使用者點選郵件中驗證連結時發來的請求。

第三，如果使用者沒有收到驗證郵件，則需要一路由來重新傳送驗證郵件。

### 信箱驗證通知

如上所述，應該定義一條路由，該路由將返回一個檢視，引導使用者點選註冊後 Laravel 傳送給他們郵件中的驗證連結。當使用者嘗試存取網站的其它頁面而沒有先完成信箱驗證時，將向使用者顯示此檢視。請注意，只要您的 `App\Models\User` 模型實現了 `MustVerifyEmail` 介面，就會自動將該連結發郵件給使用者：

    Route::get('/email/verify', function () {
        return view('auth.verify-email');
    })->middleware('auth')->name('verification.notice');

顯示信箱驗證的路由，應該命名為 `verification.notice`。組態這個命名路由很重要，因為如果使用者信箱驗證未通過，Laravel 自帶的[`verified` 中介軟體](#protecting-routes) 將會自動重新導向到該命名路由上。

> **注意**  
> 手動實現信箱驗證過程時，你需要自己定義驗證通知檢視。如果你希望包含所有必要的身份驗證和驗證檢視，請查看 [Laravel 應用入門套件](/docs/laravel/10.x/starter-kits)



### Email 認證處理

接下來，我們需要定義一個路由，該路由將處理當使用者點選驗證連結時傳送的請求。該路由應命名為 `verification.verify` ，並新增了 `auth` 和 `signed` 中介軟體

    use Illuminate\Foundation\Auth\EmailVerificationRequest;

    Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
        $request->fulfill();

        return redirect('/home');
    })->middleware(['auth', 'signed'])->name('verification.verify');

在繼續之前，讓我們仔細看一下這個路由。首先，您會注意到我們使用的是 `EmailVerificationRequest` 請求類型，而不是通常的 `Illuminate\Http\Request` 實例。 `EmailVerificationRequest` 是 Laravel 中包含的 [表單請求](/docs/laravel/10.x/validation#form-request-validation)。此請求將自動處理驗證請求的 id 和 hash 參數。

接下來，我們可以直接在請求上呼叫 `fulfill` 方法。該方法將在經過身份驗證的使用者上呼叫 `markEmailAsVerified` 方法，並會觸發 `Illuminate\Auth\Events\Verified` 事件。通過 `Illuminate\Foundation\Auth\User` 基類，`markEmailAsVerified` 方法可用於默認的 `App\Models\User` 模型。驗證使用者的電子郵件地址後，您可以將其重新導向到任意位置。

### 重新傳送 Email 認證郵件

有時候，使用者可能輸錯了電子郵件地址或者不小心刪除了驗證郵件。為瞭解決這種問題，您可能會想定義一個路由實現使用者重新傳送驗證郵件。您可以通過在 [驗證通知檢視](#the-email-verification-notice) 中放置一個簡單的表單來實現此功能。

    use Illuminate\Http\Request;

    Route::post('/email/verification-notification', function (Request $request) {
        $request->user()->sendEmailVerificationNotification();

        return back()->with('message', 'Verification link sent!');
    })->middleware(['auth', 'throttle:6,1'])->name('verification.send');



### 保護路由

[路由中介軟體](/docs/laravel/10.x/middleware)可用於僅允許經過驗證的使用者訪問給定路由。Laravel 附帶了一個 `verified` 中介軟體別名，它是 `Illuminate\Auth\Middleware\EnsureEmailIsVerified` 類的別名。由於該中介軟體已經在你的應用程式的 HTTP 核心中註冊，所以你只需要將中介軟體附加到路由定義即可。通常，此中介軟體與 `auth` 中介軟體配對使用。

    Route::get('/profile', function () {
        // 僅經過驗證的使用者可以訪問此路由。。。
    })->middleware(['auth', 'verified']);

如果未經驗證的使用者嘗試訪問已被分配了此中介軟體的路由，他們將自動重新導向到`verification.notice` [命名路由](/docs/laravel/10.x/routing#named-routes)。

## 自訂

#### 驗證郵件自訂

雖然默認的電子郵件驗證通知應該能夠滿足大多數應用程式的要求，但 Laravel 允許你自訂如何建構電子郵件驗證郵件消息。

要開始自訂郵件驗證消息，你需要將一個閉包傳遞給 `Illuminate\Auth\Notifications\VerifyEmail` 通知提供的 `toMailUsing` 方法。該閉包將接收到通知的可通知模型實例以及使用者必須訪問以驗證其電子郵件地址的已簽名電子郵件驗證 URL。該閉包應返回 `Illuminate\Notifications\Messages\MailMessage` 的實例。通常，你應該從應用程式的 `App\Providers\AuthServiceProvider` 類的 `boot` 方法中呼叫 `toMailUsing` 方法：

    use Illuminate\Auth\Notifications\VerifyEmail;
    use Illuminate\Notifications\Messages\MailMessage;

    /**
     * 註冊任何身份驗證/授權服務。
     */
    public function boot(): void
    {
        // ...

        VerifyEmail::toMailUsing(function (object $notifiable, string $url) {
            return (new MailMessage)
                ->subject('Verify Email Address')
                ->line('Click the button below to verify your email address.')
                ->action('Verify Email Address', $url);
        });
    }

> 技巧：要瞭解更多有關郵件通知的資訊，請參閱 [
郵件通知文件](/docs/laravel/10.x/notifications#mail-notifications)。



## 事件

如果你是使用 [Laravel 應用入門套件](/docs/laravel/10.x/starter-kits) 的話，Laravel 在電子郵件驗證通過後會派發 [事件](/docs/laravel/10.x/events) 。如果你想接收到這個事件並進行手動處理的話，你應該在 `EventServiceProvider` 中註冊監聽器：

    use App\Listeners\LogVerifiedUser;
    use Illuminate\Auth\Events\Verified;
    
    /**
     * 應用的事件監聽器
     *
     * @var array
     */
    protected $listen = [
        Verified::class => [
            LogVerifiedUser::class,
        ],
    ];

