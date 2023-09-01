# Sanctum API 授權

## 介紹

[Laravel Sanctum](https://github.com/laravel/sanctum) 提供了一個輕量級的認證系統，可用於 SPA（單頁應用程式）、移動應用程式和基於簡單令牌的 API。Sanctum 允許的應用程式中的每個使用者為他們的帳戶生成多個 API 令牌。這些令牌可以被授予權限/範圍，以指定令牌允許執行哪些操作。

### 工作原理

Laravel Sanctum 旨在解決兩個不同的問題。在深入探討該庫之前，讓我們先討論一下每個問題。

#### API 令牌

首先，Sanctum 是一個簡單的包，你可以使用它向你的使用者發出 API 令牌，而無需 OAuth 的複雜性。這個功能受到 GitHub 和其他應用程式發出「訪問令牌」的啟發。例如，假如你的應用程式的「帳戶設定」有一個介面，使用者可以在其中為他們的帳戶生成 API 令牌。你可以使用 Sanctum 生成和管理這些令牌。這些令牌通常具有非常長的過期時間（以年計），但使用者可以隨時手動撤銷它們。

Laravel Sanctum 通過將使用者 API 令牌儲存在單個資料庫表中，並通過應該包含有效 API 令牌的 `Authorization` 標頭對傳入的 HTTP 請求進行身份驗證來提供此功能。

#### SPA 認證

第二個功能，Sanctum 存在的目的是為需要與 Laravel 支援的 API 通訊的單頁應用程式 (SPAs) 提供一種簡單的身份驗證方式。這些 SPAs 可能存在於與 Laravel 應用程式相同的儲存庫中，也可能是一個完全獨立的儲存庫，例如使用 Vue CLI 建立的 SPA 或 Next.js 應用程式。

對於此功能，Sanctum 不使用任何類型的令牌。相反，Sanctum 使用 Laravel 內建基於 cookie 的 session 身份驗證服務。通常，Sanctum 使用 Laravel 的 `web` 認證保護方式實現這一點。這提供了 CSRF 保護、 session 身份驗證以及防止通過 XSS 洩漏身份驗證憑據的好處。

只有在傳入請求來自你自己的 SPA 前端時，Sanctum 才會嘗試使用 cookies 進行身份驗證。當 Sanctum 檢查傳入的 HTTP 請求時，它首先會檢查身份驗證 cookie，如果不存在，則 Sanctum 會檢查 `Authorization` 標頭是否包含有效的 API 令牌。

> **注意**
> 完全可以只使用 Sanctum 進行 API 令牌身份驗證或只使用 Sanctuary 進行 SPA 身份驗證。僅因為你使用 Sanctum 並不意味著你必須使用它提供的兩個功能。

## 安裝

> **注意**
> 最近的 Laravel 版本已經包括 Laravel Sanctum。但如果你的應用程式的 `composer.json` 檔案不包括 `laravel/sanctum`，你可以遵循下面的安裝說明。

你可以通過 Composer 包管理器安裝 Laravel Sanctum：

```shell
composer require laravel/sanctum
```

接下來，你應該使用 `vendor:publish` Artisan 命令發佈 Sanctum 組態檔案和遷移檔案。`sanctum` 組態檔案將被放置在你的應用程式的 `config` 目錄中：

```shell
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
```

最後，你應該運行資料庫遷移。Sanctum 會建立一個資料庫表來儲存 API 令牌：

```shell
php artisan migrate
```

接下來，如果你打算使用 Sanctum 來對 SPA 單頁應用程式進行認證，則應該將 Sanctum 的中介軟體新增到你的應用程式的 `app/Http/Kernel.php` 檔案中的 `api` 中介軟體組中：

```
'api' => [
\Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
   \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
   \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```

#### 自訂遷移

如果你不打算使用 Sanctum 的默認遷移檔案，則應該在 `App\Providers\AppServiceProvider` 類的 `register` 方法中呼叫 `Sanctum::ignoreMigrations` 方法。你可以通過執行以下命令匯出默認的遷移檔案：`php artisan vendor:publish --tag=sanctum-migrations`


## 組態

### 覆蓋默認模型

雖然通常不需要，但你可以自由擴展 Sanctum 內部使用的 `PersonalAccessToken` 模型:

```
use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;

class PersonalAccessToken extends SanctumPersonalAccessToken
{
    // ...
}
```

然後，你可以通過 Sanctum 提供的 `usePersonalAccessTokenModel` 方法來指示 Sanctum 使用你的自訂模型。通常，你應該在一個應用程式的服務提供者的 `boot` 方法中呼叫此方法：

```
use App\Models\Sanctum\PersonalAccessToken;
use Laravel\Sanctum\Sanctum;

/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
}
```

## API 令牌認證

> **注意**
> 你不應該使用 API 令牌來認證你自己的第一方單頁應用程式。而應該使用 Sanctum 內建的 [SPA 身份驗證功能](#spa-authentication)。

### 發行 API 令牌

Sanctum 允許你發行 API 令牌/個人訪問令牌，可用於對你的應用程式的 API 請求進行身份驗證。使用 API 令牌發出請求時，應將令牌作為 `Bearer` 令牌包括在 `Authorization` 頭中。

要開始為使用者發行令牌，你的使用者模型應該使用 `Laravel\Sanctum\HasApiTokens` trait：

```
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

要發行令牌，你可以使用 `createToken` 方法。`createToken` 方法會返回一個 `Laravel\Sanctum\NewAccessToken` 實例。在將 API 令牌儲存到資料庫之前，令牌將使用 SHA-256 雜湊進行雜湊處理，但是你可以通過 `NewAccessToken` 實例的 `plainTextToken` 屬性訪問令牌的明文值。你應該在令牌被建立後立即將其值顯示給使用者：

```
use Illuminate\Http\Request;

Route::post('/tokens/create', function (Request $request) {
    $token = $request->user()->createToken($request->token_name);

    return ['token' => $token->plainTextToken];
});

```
你可以使用 `HasApiTokens` trait 提供的 `tokens` Eloquent 關聯來訪問使用者的所有令牌：

```
foreach ($user->tokens as $token) {
    // ...
}
```

### 令牌能力

Sanctum 允許你為令牌分配「能力」 。能力的作用類似於 OAuth 的「Scope」 。你可以將一個字串能力陣列作為 `createToken` 方法的第二個參數傳遞：

```
return $user->createToken('token-name', ['server:update'])->plainTextToken;
```

當處理由 Sanctum 驗證的入站請求時，你可以使用 `tokenCan` 方法確定令牌是否具有給定的能力：

```
if ($user->tokenCan('server:update')) {
    // ...
}
```

#### 令牌能力中介軟體

Sanctum 還包括兩個中介軟體，可用於驗證傳入的請求是否使用授予了給定能力的令牌進行了身份驗證。首先，請將以下中介軟體新增到應用程式的 `app/Http/Kernel.php` 檔案的 `$middlewareAliases` 屬性中：

```
'abilities' => \Laravel\Sanctum\Http\Middleware\CheckAbilities::class,
'ability' => \Laravel\Sanctum\Http\Middleware\CheckForAnyAbility::class,
```

可以將 `abilities` 中介軟體分配給路由，以驗證傳入請求的令牌是否具有所有列出的能力：

```
Route::get('/orders', function () {
    // 令牌具有「check-status」和「place-orders」能力...
})->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);

```

可以將 `ability` 中介軟體分配給路由，以驗證傳入請求的令牌是否至少具有一個列出的能力：

```
Route::get('/orders', function () {
    // 令牌具有「check-status」或「place-orders」能力...
})->middleware(['auth:sanctum', 'ability:check-status,place-orders']);

```

#### 第一方 UI 啟動的請求

為了方便起見，如果入站身份驗證請求來自你的第一方 SPA ，並且你正在使用 Sanctum 內建的 [SPA 認證](#spa-authentication)，`tokenCan` 方法將始終返回 `true`。

然而，這並不一定意味著你的應用程式必須允許使用者執行該操作。通常，你的應用程式的[授權策略](/docs/laravel/10.x/authorization#creating-policies) 將確定是否已授予令牌執行能力的權限，並檢查使用者實例本身是否允許執行該操作。

例如，如果我們想像一個管理伺服器的應用程式，這可能意味著檢查令牌是否被授權更新伺服器**並且**伺服器屬於使用者：

```php
return $request->user()->id === $server->user_id &&
       $request->user()->tokenCan('server:update')
```

首先允許 `tokenCan` 方法被呼叫並始終為第一方 UI 啟動的請求返回 `true` 可能看起來很奇怪。然而，能夠始終假設 API 令牌可用並可通過 `tokenCan` 方法進行檢查非常方便。通過採用這種方法，你可以始終在應用程式的授權策略中呼叫 `tokenCan` 方法，而不用再擔心請求是從應用程式的 UI 觸發還是由 API 的第三方使用者發起的。


### 保護路由

為了保護路由，使所有入站請求必須進行身份驗證，你應該在你的 `routes/web.php` 和 `routes/api.php` 路由檔案中，將 `sanctum` 認證守衛附加到受保護的路由上。如果該請求來自第三方，該守衛將確保傳入的請求經過身份驗證，要麼是具有狀態的 Cookie 身份驗證請求，要麼是包含有效的 API 令牌標頭的請求。

你可能想知道我們為什麼建議你使用 `sanctum` 守衛在應用程式的 `routes/web.php` 檔案中對路由進行身份驗證。請記住，Sanctum 首先將嘗試使用 Laravel 的典型 session 身份驗證 cookie 對傳入請求進行身份驗證。如果該 cookie 不存在，則 Sanctum 將嘗試使用請求的 `Authorization` 標頭中的令牌來驗證請求。此外，使用 Sanctum 對所有請求進行身份驗證，確保我們可以始終在當前經過身份驗證的使用者實例上呼叫 `tokenCan` 方法：

```
use Illuminate\Http\Request;

Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

### 撤銷令牌

你可以通過使用 `Laravel\Sanctum\HasApiTokens` trait 提供的 `tokens` 關係，從資料庫中刪除它們來達到「撤銷」令牌的目的：

```
// 撤銷所有令牌...
$user->tokens()->delete();

// 撤銷用於驗證當前請求的令牌...
$request->user()->currentAccessToken()->delete();

// 撤銷特定的令牌...
$user->tokens()->where('id', $tokenId)->delete();

```

### 令牌有效期

默認情況下，Sanctum 令牌永不過期，並且只能通過[撤銷令牌](#revoking-tokens)進行無效化。但是，如果你想為你的應用程式 API 令牌組態過期時間，可以通過在應用程式的 `sanctum` 組態檔案中定義的 `expiration` 組態選項進行組態。此組態選項定義發放的令牌被視為過期之前的分鐘數：

```php
// 365天後過期
'expiration' => 525600,
```

如果你已為應用程式組態了令牌過期時間，你可能還希望[任務調度](/docs/laravel/10.x/scheduling)來刪除應用程式過期的令牌。幸運的是，Sanctum 包括一個 `sanctum:prune-expired` Artisan 命令，你可以使用它來完成此操作。例如，你可以組態工作排程來刪除所有過期至少24小時的令牌資料庫記錄：

```php
$schedule->command('sanctum:prune-expired --hours=24')->daily();
```

## SPA 身份驗證

Sanctum 還提供一種簡單的方法來驗證需要與 Laravel API 通訊的單頁面應用程式（SPA）。這些 SPA 可能存在於與你的 Laravel 應用程式相同的儲存庫中，也可能是一個完全獨立的儲存庫。

對於此功能，Sanctum 不使用任何類型的令牌。相反，Sanctum 使用 Laravel 內建的基於 cookie 的 session 身份驗證服務。此身份驗證方法提供了 CSRF 保護、session 身份驗證以及防止身份驗證憑據通過 XSS 洩漏的好處。

> **警告**
> 為了進行身份驗證，你的 SPA 和 API 必須共享相同的頂級域。但是，它們可以放置在不同的子域中。此外，你應該確保你的請求中傳送 `Accept: application/json` 標頭檔。


### 組態


#### 組態你的第一個域

首先，你應該通過 `sanctum` 組態檔案中的 `stateful` 組態選項來組態你的 SPA 將從哪些域發出請求。此組態設定確定哪些域將在向你的 API 傳送請求時使用 Laravel session cookie 維護「有狀態的」身份驗證。

> **警告**
> 如果你通過包含連接埠的 URL（`127.0.0.1:8000`）訪問應用程式，你應該確保在域名中包括連接埠號。


#### Sanctum 中介軟體

接下來，你應該將 Sanctum 中介軟體新增到你的 `app/Http/Kernel.php` 檔案中的 `api` 中介軟體組中。此中介軟體負責確保來自你的 SPA 的傳入請求可以使用 Laravel  session  cookie 進行身份驗證，同時仍允許來自第三方或移動應用程式使用 API 令牌進行身份驗證：

```
'api' => [ \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
   \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
   \Illuminate\Routing\Middleware\SubstituteBindings::class,
],

```

#### CORS 和 Cookies

如果你無法從執行在單獨子域上的 SPA 中進行應用程式身份驗證的話，你可能已錯誤組態了 CORS（跨域資源共享）或 session  cookie 設定。

你應該確保你的應用程式的 CORS 組態返回的 `Access-Control-Allow-Credentials` 要求標頭的值為 `True` 。這可以通過在應用程式的 `config/cors.php` 組態檔案中設定 `supports_credentials` 選項為 `true` 來完成。

此外，你應該在應用程式的全域 `axios` 實例中啟用 `withCredentials` 選項。通常，這應該在你的 `resources/js/bootstrap.js` 檔案中進行。如果你沒有使用 Axios 從前端進行 HTTP 請求，你應該使用自己的 HTTP 客戶端進行等效組態：

```js
axios.defaults.withCredentials = true;
```

最後，你應該確保應用程式的 session  cookie 域組態支援根域的任何子域。你可以通過在應用程式的 `config/session.php` 組態檔案中使用前導 `.` 作為域的前綴來實現此目的：

```
'domain' => '.domain.com',
```

### 身份驗證

#### CSRF 保護

要驗證你的 SPA，你的 SPA 的「登錄」頁面應首先向 `/sanctum/csrf-cookie` 發出請求以初始化應用程式的 CSRF 保護：

```js
axios.get('/sanctum/csrf-cookie').then(response => {
    // Login...
});
```

在此請求期間，Laravel 將設定一個包含當前 CSRF 令牌的 `XSRF-TOKEN` cookie。然後，此令牌應在隨後的請求中通過 `X-XSRF-TOKEN` 標頭傳遞，其中某些 HTTP 客戶端庫（如 Axios 和 Angular HttpClient）將自動為你執行此操作。如果你的 JavaScript HTTP 庫沒有為你設定值，你將需要手動設定 `X-XSRF-TOKEN` 要求標頭以匹配此路由設定的  `XSRF-TOKEN` cookie 的值。


#### 登錄

一旦已經初始化了 CSRF 保護，你應該向 Laravel 應用程式的 `/login` 路由發出 `POST` 請求。這個 `/login` 路由可以通過[手動實現](/docs/laravel/10.x/authentication#authenticating-users)或使用像 [Laravel Fortify](/docs/laravel/10.x/fortify) 這樣的無要求標頭身份驗證包來實現。

如果登錄請求成功，你將被驗證，隨後對應用程式路由的後續請求將通過 Laravel 應用程式發出的 session  cookie 自動進行身份驗證。此外，由於你的應用程式已經發出了對 `/sanctum/csrf-cookie` 路由的請求，因此只要你的 JavaScript HTTP 客戶端在 `X-XSRF-TOKEN` 標頭中傳送了 `XSRF-TOKEN` cookie 的值，後續的請求應該自動接受 CSRF 保護。

當然，如果你的使用者 session 因缺乏活動而過期，那麼對 Laravel 應用程式的後續請求可能會收到 401 或 419 HTTP 錯誤響應。在這種情況下，你應該將使用者重新導向到你 SPA 的登錄頁面。

> **警告**
> 你可以自己編寫 `/login` 端點；但是，你應該確保使用 Laravel 提供的標準基於[ session 的身份驗證服務](/docs/laravel/10.x/authentication#authenticating-users)來驗證使用者。通常，這意味著使用 `web` 身份驗證 Guard。

### 保護路由

為了保護路由，以便所有傳入的請求必須進行身份驗證，你應該將 `sanctum` 身份驗證 guard 附加到 `routes/api.php` 檔案中的 API 路由上。這個 guard 將確保傳入的請求被驗證為來自你的 SPA 的有狀態身份驗證請求，或者如果請求來自第三方，則包含有效的 API 令牌標頭：

```
use Illuminate\Http\Request;

Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

### 授權私有廣播頻道

如果你的 SPA 需要對[私有/存在 broadcast 頻道進行身份驗證](/docs/laravel/10.x/broadcasting#authorizing-channels)，你應該在 `routes/api.php` 檔案中呼叫 `Broadcast::routes` 方法：

```
Broadcast::routes(['middleware' => ['auth:sanctum']]);

```

接下來，為了讓 Pusher 的授權請求成功，你需要在初始化 [Laravel Echo](/docs/laravel/10.x/broadcasting#client-side-installation) 時提供自訂的 Pusher `authorizer`。這允許你的應用程式組態 Pusher 以使用[為跨域請求正確組態的](#cors-and-cookies) `axios` 實例：

```js
window.Echo = new Echo({
    broadcaster: "pusher",
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    encrypted: true,
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    authorizer: (channel, options) => {
        return {
            authorize: (socketId, callback) => {
                axios.post('/api/broadcasting/auth', {
                    socket_id: socketId,
                    channel_name: channel.name
                })
                .then(response => {
                    callback(false, response.data);
                })
                .catch(error => {
                    callback(true, error);
                });
            }
        };
    },
})
```

## 移動應用程式身份驗證

你也可以使用 Sanctum 令牌來驗證你的移動應用程式對 API 的請求。驗證移動應用程式請求的過程類似於驗證第三方 API 請求；但是，你將發佈 API 令牌的方式有所不同。

### 發佈 API 令牌

首先，請建立一個路由，該路由接受使用者的電子郵件/使用者名稱、密碼和裝置名稱，然後將這些憑據交換為新的 Sanctum 令牌。給此端點提供「裝置名稱」的目的是為了記錄資訊，僅供參考。通常來說，裝置名稱值應該是使用者能夠識別的名稱，例如「Nuno’s iPhone 12」。

通常，你將從你的移動應用程式的「登錄」頁面向令牌端點發出請求。此端點將返回純文字的 API 令牌，可以儲存在移動裝置上，並用於進行額外的 API 請求：

```
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

Route::post('/sanctum/token', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }

    return $user->createToken($request->device_name)->plainTextToken;
});

```

當移動應用程式使用令牌向你的應用程式發出 API 請求時，它應該將令牌作為 `Bearer` 令牌放在 `Authorization` 標頭中傳遞。

> **注意**
> 當為移動應用程式發佈令牌時，你可以自由指定[令牌權限](#token-abilities)。

### 路由保護

如之前所述，你可以通過使用 `sanctum` 認證守衛附加到路由上來保護路由，以便所有傳入請求都必須進行身份驗證：

```
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});

```


### 撤銷令牌

為了允許使用者撤銷發放給移動裝置的 API 令牌，你可以在 Web 應用程式 UI 的 「帳戶設定」部分中按名稱列出它們，並提供一個「撤銷」按鈕。當使用者點選「撤銷」按鈕時，你可以從資料庫中刪除令牌。請記住，你可以通過 `Laravel\Sanctum\HasApiTokens` 特性提供的 `tokens` 關係訪問使用者的 API 令牌：

```
// 撤銷所有令牌...
$user->tokens()->delete();

// 撤銷特定令牌...
$user->tokens()->where('id', $tokenId)->delete();
```

## 測試

在測試時，`Sanctum::actingAs` 方法可用於驗證使用者並指定為其令牌授予哪些能力：

    use App\Models\User;
    use Laravel\Sanctum\Sanctum;

    public function test_task_list_can_be_retrieved(): void
    {
        Sanctum::actingAs(
            User::factory()->create(),
            ['view-tasks']
        );

        $response = $this->get('/api/task');

        $response->assertOk();
    }

如果你想授予令牌所有的能力，你應該在提供給 `actingAs` 方法的能力列表中包含 `*` ：

    Sanctum::actingAs(
        User::factory()->create(),
        ['*']
    );

