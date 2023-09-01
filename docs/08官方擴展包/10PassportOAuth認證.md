# Passport OAuth 認證

## 簡介

[Laravel Passport](https://github.com/laravel/passport) 可以在幾分鐘之內為你的應用程式提供完整的 OAuth2 伺服器端實現。Passport 是基於由 Andy Millington 和 Simon Hamp 維護的 [League OAuth2 server](https://github.com/thephpleague/oauth2-server) 建立的。

> **注意**  
> 本文件假定你已熟悉 OAuth2 。如果你並不瞭解 OAuth2 ，閱讀之前請先熟悉下 OAuth2 的 [常用術語](https://oauth2.thephpleague.com/terminology/) 和特性。

### Passport 還是 Sanctum?

在開始之前，我們希望你先確認下是 Laravel Passport 還是 [Laravel Sanctum](/docs/laravel/10.x/sanctum) 能為你的應用提供更好的服務。如果你的應用確確實實需要支援 OAuth2，那沒疑問，你需要選用 Laravel Passport。

然而，如果你只是試圖要去認證一個單頁應用，或者手機應用，或者發佈 API 令牌，你應該選用 [Laravel Sanctum](/docs/laravel/10.x/sanctum)。 Laravel Sanctum 不支援 OAuth2，它提供了更為簡單的 API 授權開發體驗。

## 安裝

在開始使用之前，使用 Composer 包管理器安裝 Passport：

```shell
composer require laravel/passport
```

Passport 的 [服務提供器](/docs/laravel/10.x/providers) 註冊了自己的資料庫遷移指令碼目錄， 所以你應該在安裝軟體包完成後遷移你自己的資料庫。 Passport 的遷移指令碼將為你的應用建立用於儲存 OAuth2 客戶端和訪問令牌的資料表：

```shell
php artisan migrate
```

接下來，你需要執行 Artisan 命令 `passport:install`。這個命令將會建立一個用於生成安全訪問令牌的加密秘鑰。另外，這個命令也將建立用於生成訪問令牌的 「個人訪問」 客戶端和 「密碼授權」 客戶端 ：

```shell
php artisan passport:install
```

> **技巧**  
> 如果你想用使用 UUID 作為 Passport `Client` 模型的主鍵，代替默認的自動增長整形欄位，請在安裝 Passport 時使用 [uuids 參數](#client-uuids) 。

在執行 `passport:install` 命令後， 新增 `Laravel\Passport\HasApiTokens` trait 到你的 `App\Models\User` 模型中。 這個 trait 會提供一些幫助方法用於檢查已認證使用者的令牌和權限範圍。如果你的模型已經在使用 `Laravel\Sanctum\HasApiTokens` trait，你可以刪除該 trait：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;
    }



最後，在您的應用的 `config/auth.php` 組態檔案中，您應當定義一個 `api` 的授權看守器，並且將其 `driver` 選項設定為 `passport` 。這個調整將會讓您的應用程式使用 Passport 的 `TokenGuard` 來鑑權 API 介面請求：

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

#### 客戶端 UUID

您也可以在運行 `passport:install` 命令的時候使用 `--uuids` 選項。這個參數將會讓 Passport 使用 UUID 來替代默認的自增長形式的 Passport `Client` 模型主鍵。在您運行帶有 `--uuids` 參數的 `passport:install` 命令後，您將得到關於停用 Passport 默認遷移的相關指令說明：

```shell
php artisan passport:install --uuids
```

### 部署 Passport

在您第一次部署 Passport 到您的應用伺服器時，您需要執行 `passport:keys` 命令。該命令用於生成 Passport 用於生成 access token 的一個加密金鑰。生成的加密金鑰不應到新增到原始碼控制系統中：

```shell
php artisan passport:keys
```

如有必要，您可以定義 Passport 的金鑰應當載入的位置。您可以使用 `Passport:loadKeysFrom` 方法來實現。通常，這個方法應當在您的 `App\Providers\AuthServiceProvider` 類的 `boot` 方法中呼叫：

    /**
     * Register any authentication / authorization services.
     */
    public function boot(): void
    {
        Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
    }

#### 從環境中載入金鑰

此外，您可以使用 `vendor:publish` Artisan 命令來發佈您的 Passport 組態檔案：

```shell
php artisan vendor:publish --tag=passport-config
```

在發佈組態檔案之後，您可以將加密金鑰組態為環境變數，再載入它們：

```ini
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

### 自訂遷移

如果您不打算使用 Passport 的默認遷移，您應當在 `App\Providers\AppServiceProvider` 類的 `register` 方法中呼叫 `Passport::ignoreMigrations` 方法。您可以 使用 `vendor:publish` Artisan 命令來匯出默認的遷移檔案：

```shell
php artisan vendor:publish --tag=passport-migrations
```

### Passport 的升級

當升級到 Passport 的主要版本時，請務必查閱 [升級指南](https://github.com/laravel/passport/blob/master/UPGRADE.).

## 組態

### 客戶端金鑰的 Hash 加密

如果您希望客戶端金鑰在儲存到資料庫時使用 Hash 對其進行加密，您應當在 `App\Provider\AuthServiceProvider` 類的 `boot` 方法中呼叫 `Passport:hashClientSecrets` ：

    use Laravel\Passport\Passport;

    Passport::hashClientSecrets();

一旦啟用後，所有的客戶端金鑰都將只在建立的時候顯示。由於明文的客戶端金鑰沒有儲存到資料庫中，因此一旦其丟失後便無法恢復。

### Token 生命週期

默認情況下，Passport 會頒髮長達一年的長期 token 。如果您想要組態一個更長或更短的 token 生命週期，您可以在 `App\Provider\AuthServiceProvider` 類的 `boot` 方法中呼叫 `tokensExpiresIn` 、`refresgTokensExpireIn` 和 `personalAccessTokensExpireIn` 方法：

    /**
     * 註冊身份驗證/授權服務。
     */
    public function boot(): void
    {
        Passport::tokensExpireIn(now()->addDays(15));
        Passport::refreshTokensExpireIn(now()->addDays(30));
        Passport::personalAccessTokensExpireIn(now()->addMonths(6));
    }

> **注意**  
>  Passport 資料庫表中的 `expires_at` 列是唯讀的，僅僅用於顯示。在頒發 token 的時候，Passport 將過期資訊儲存在已簽名和加密的 token 中。如果你想讓 token 失效，你應當 [撤銷它](#revoking-tokens) 。



### 重寫 Passport 的默認模型

您可以通過定義自己的模型並繼承相應的 Passport 模型來實現自由自由擴展 Passport 內部使用的模型：

    use Laravel\Passport\Client as PassportClient;

    class Client extends PassportClient
    {
        // ...
    }

在定義您的模型之後，您可以在 `Laravel\Passport\Passport` 類中指定 Passport 使用您自訂的模型。一樣的，您應該在應用程式的 `App\Providers\AuthServiceProvider` 類中的 `boot` 方法中指定 Passport 使用您自訂的模型：

    use App\Models\Passport\AuthCode;
    use App\Models\Passport\Client;
    use App\Models\Passport\PersonalAccessClient;
    use App\Models\Passport\RefreshToken;
    use App\Models\Passport\Token;

    /**
     * 註冊任意認證/授權服務。
     */
    public function boot(): void
    {
        Passport::useTokenModel(Token::class);
        Passport::useRefreshTokenModel(RefreshToken::class);
        Passport::useAuthCodeModel(AuthCode::class);
        Passport::useClientModel(Client::class);
        Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
    }

### 重寫路由

您可能希望自訂 Passport 定義的路由。要實現這個功能，第一步，您需要在應用程式的 `AppServiceProvider` 中的 `register` 方法中新增 `Passport:ignoreRoutes` 語句，以忽略由 Passport 註冊的路由：

    use Laravel\Passport\Passport;

    /**
     * 註冊任意的應用程式服務。
     */
    public function register(): void
    {
        Passport::ignoreRoutes();
    }

然後，您可以複製 Passport [在自己的檔案中](https://github.com/laravel/passport/blob/11.x/routes/web.php) 定義的路由到應用程式的 `routes/web.php` 檔案中，並且將其修改為您喜歡的任何形式：

    Route::group([
        'as' => 'passport.',
        'prefix' => config('passport.path', 'oauth'),
        'namespace' => 'Laravel\Passport\Http\Controllers',
    ], function () {
        // Passport 路由……
    });

## 發佈訪問令牌

通過授權碼使用 OAuth2 是大多數開發人員熟悉的方式。使用授權碼方式時，客戶端應用程式會將使用者重新導向到你的伺服器，在那裡他們會批准或拒絕向客戶端發出訪問令牌的請求。



### 客戶端管理

首先，開發者如果想要搭建一個與你的伺服器端介面互動的應用端，需要在伺服器端這邊註冊一個「客戶端」。通常，這需要開發者提供應用程式的名稱和一個 URL，在應用軟體的使用者授權請求後，應用程式會被重新導向到該 URL。

#### `passport:client` 命令

使用 Artisan 命令 `passport:client` 是一種最簡單的建立客戶端的方式。 這個命令可以建立你自己私有的客戶端，用於 Oauth2 功能測試。 當你執行 `client` 命令後， Passport 將會給你更多關於客戶端的提示，以及生成的客戶端 ID

```shell
php artisan passport:client
```

**多重新導向 URL 地址的設定**

如果你想為你的客戶端提供多個重新導向 URL ，你可以在執行 `Passport:client` 命令出現提示輸入 URL 地址的時候，輸入用逗號分割的多個 URL 。任何包含逗號的 URL 都需要先執行 URL 轉碼：

```shell
http://example.com/callback,http://examplefoo.com/callback
```

#### JSON API

因為應用程式的開發者是無法使用 `client` 命令的，所以 Passport 提供了 JSON 格式的 API ，用於建立客戶端。 這解決了你還要去手動建立 controller 程式碼（程式碼用於新增，更新，刪除客戶端）的麻煩。

但是，你需要結合 Passport 的 JSON API 介面和你的前端面板管理頁面， 為你的使用者提供客戶端管理功能。接下里，我們會回顧所有用於管理客戶端的的 API 介面。方便起見，我們使用 [Axios](https://github.com/axios/axios) 模擬對端點的 HTTP 請求。



這些 JSON API 介面被 `web` 和 `auth` 兩個中介軟體保護著，因此，你只能從你的應用中呼叫。 外部來源的呼叫是被禁止的。

#### `GET /oauth/clients`

下面的路由將為授權使用者返回所有的客戶端。最主要的作用是列出所有的使用者客戶端，接下來就可以編輯或刪除它們了：

```js
axios.get('/oauth/clients')
    .then(response => {
        console.log(response.data);
    });
```

#### `POST /oauth/clients`

下面的路由用於建立新的客戶端。 它需要兩個參數： `客戶端名稱`和`重新導向URL` 地址。 `重新導向URL` 地址是使用者在授權或者拒絕授權後被重新導向到的地方。

客戶端被建立後，將會生成客戶端 ID 和客戶端秘鑰。 這對值用於從你的應用獲取訪問令牌。 呼叫下面的客戶端建立路由將建立新的客戶端實例：

```js
const data = {
    name: 'Client Name',
    redirect: 'http://example.com/callback'
};

axios.post('/oauth/clients', data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // 列出響應的錯誤...
    });
```

#### `PUT /oauth/clients/{client-id}`

下面的路由用來更新客戶端。它需要兩個參數： 客戶端名稱和重新導向 URL 地址。 重新導向 URL 地址是使用者在授權或者拒絕授權後被重新導向到的地方。路由將返回更新後的客戶端實例：

```js
const data = {
    name: 'New Client Name',
    redirect: 'http://example.com/callback'
};

axios.put('/oauth/clients/' + clientId, data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // 列出響應的錯誤...
    });
```



#### `DELETE /oauth/clients/{client-id}`

下面的路由用於刪除客戶端：

```js
axios.delete('/oauth/clients/' + clientId)
    .then(response => {
        // ...
    });
```

### 請求令牌

#### 授權重新導向

客戶端建立好後，開發者使用 client ID 和秘鑰向你的應用伺服器傳送請求，以便獲取授權碼和訪問令牌。 首先，接收到請求的業務端伺服器會重新導向到你應用的 `/oauth/authorize` 路由上，如下所示：

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

`prompt` 參數可用於指定 Passport 應用程式的認證行為。

如果 `prompt` 值為 `none`，如果使用者還沒有通過 Passport 應用程式的認證，Passport 將總是拋出一個認證錯誤。如果值是 `同意`，Passport 將總是顯示授權批准螢幕，即使所有的範疇以前都被授予消費應用程式。如果值是 `login`，Passport 應用程式將總是提示使用者重新登錄到應用程式，即使他們已經有一個現有的 session 。

如果沒有提供 `prompt` 值，只有當使用者以前沒有授權訪問所請求範圍的消費應用程式時，才會提示使用者進行授權。

> **技巧：**請記住，`/oauth/authorize` 路由默認已經在 `Passport::route` 方法中定義，你無需手動定義它。



#### 請求認證

當接收到一個請求後， Passport 會自動展示一個範本頁面給使用者，使用者可以選擇授權或者拒絕授權。如果請求被認證，使用者將被重新導向到之前業務伺服器設定的 `redirect_uri` 上去。 這個 `redirect_uri` 就是客戶端在建立時提供的重新導向地址參數。


如果你想自訂授權頁面，你可以先使用 Artisan 命令 `vendor:publish` 發佈 Passport 的檢視頁面。 被發佈的檢視頁面位於 `resources/views/vendor/passport` 路徑下：

```shell
php artisan vendor:publish --tag=passport-views
```

有時，你可能希望跳過授權提示，比如在授權第一梯隊客戶端的時候。你可以通過 [繼承 `Client` 模型](#overriding-default-models)並實現 `skipsAuthorization` 方法。如果 `skipsAuthorization` 方法返回 `true`， 客戶端就會直接被認證並立即重新導向到設定的重新導向地址：

    <?php

    namespace App\Models\Passport;

    use Laravel\Passport\Client as BaseClient;

    class Client extends BaseClient
    {
        /**
         * 確定客戶端是否應跳過授權提示。
         */
        public function skipsAuthorization(): bool
        {
            return $this->firstParty();
        }
    }

#### 授權碼到授權令牌的轉化

如果使用者授權了訪問，他們會被重新導向到業務伺服器端。首先，業務端服務需要檢查 `state` 參數是否和重新導向之前儲存的值一致。 如果 state 參數的值正確，業務端伺服器需要對你的應用發起獲取 access token 的 `POST` 請求。 請求需要攜帶有授權碼，授權碼就是之前使用者授權後由你的應用伺服器生成的碼：

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Http;

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'code' => $request->code,
        ]);

        return $response->json();
    });



呼叫路由 `/oauth/token` 將返回一串 json 字串，包含了 `access_token`, `refresh_token` 和 `expires_in` 屬性。`expires_in` 屬性的值是 access_token 剩餘的有效時間。

> **技巧：**就和 `/oauth/authorize` 路由一樣， `/oauth/token` 路由已經在 `Passport::routes` 方法中定義，你無需再自訂這個路由。

#### JSON API

Passport 同樣包含了一個 JSON API 介面用來管理授權訪問令牌。你可以使用該介面為使用者搭建一個管理訪問令牌的控制面板。方便來著，我們將使用 [Axios](https://github.com/mzabriskie/axios) 模擬 HTTP 對端點發起請求。由於 JSON API 被中介軟體 `web` 和 `auth` 保護著，我們只能在應用內部呼叫。

#### `GET /oauth/tokens`

下面的路由包含了授權使用者建立的所有授權訪問令牌。介面的主要作用是列出使用者所有可撤銷的令牌：

```js
axios.get('/oauth/tokens')
    .then(response => {
        console.log(response.data);
    });
```

#### `DELETE /oauth/tokens/{token-id}`

下面的路由用於撤銷授權訪問令牌以及相關的刷新令牌：

```js
axios.delete('/oauth/tokens/' + tokenId);
```

### 刷新令牌

如果你的應用發佈的是短生命週期訪問令牌，使用者需要使用刷新令牌來延長訪問令牌的生命週期，刷新令牌是在生成訪問令牌時同時生成的：

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ]);

    return $response->json();

呼叫路由 `/oauth/token` 將返回一串 json 字串，包含了 `access_token`, `refresh_token` 和 `expires_in` 屬性。`expires_in` 屬性的值是 access_token 剩餘的有效時間。



### 撤銷令牌

你可以使用 `Laravel\Passport\TokenRepository` 類的 `revokeAccessToken` 方法撤銷令牌。你可以使用 `Laravel\Passport\RefreshTokenRepository` 類的 `revokeRefreshTokensByAccessTokenId` 方法撤銷刷新令牌。這兩個類可以通過 Laravel 的[服務容器](/docs/laravel/10.x/container)得到：

    use Laravel\Passport\TokenRepository;
    use Laravel\Passport\RefreshTokenRepository;

    $tokenRepository = app(TokenRepository::class);
    $refreshTokenRepository = app(RefreshTokenRepository::class);

    // 撤銷一個訪問令牌...
    $tokenRepository->revokeAccessToken($tokenId);

    // 撤銷該令牌的所有刷新令牌...
    $refreshTokenRepository->revokeRefreshTokensByAccessTokenId($tokenId);

### 清除令牌

如果令牌已經被撤銷或者已經過期了，你可能希望把它們從資料庫中清理掉。Passport 提供了 Artisan 命令 `passport:purge` 幫助你實現這個操作:

```shell
# 清除已經撤銷或者過期的令牌以及授權碼...
php artisan passport:purge

# 只清理過期6小時的令牌以及授權碼...
php artisan passport:purge --hours=6

# 只清理撤銷的令牌以及授權碼...
php artisan passport:purge --revoked

# 只清理過期的令牌以及授權碼...
php artisan passport:purge --expired
```

你可以在應用的 `App\Console\Kernel` 類中組態一個[定時任務](/docs/laravel/10.x/scheduling)，每天自動的清理令牌：

    /**
     * Define the application's command schedule.
     */
    protected function schedule(Schedule $schedule): void
    {
        $schedule->command('passport:purge')->hourly();
    }

## 通過 PKCE 發佈授權碼

通過 PKCE 「 Proof Key for Code Exchange, 中文譯為 程式碼交換的證明金鑰」 發放授權碼是對單頁面應用或原生應用進行認證以便訪問 API 介面的安全方式。這種發放授權碼是用於不能保證客戶端密碼被安全儲存，或為降低攻擊者攔截授權碼的威脅。在這種模式下，當授權碼獲取令牌時，用 「驗證碼」( code verifier ) 和 「質疑碼」（ code challenge, challenge, 名詞可譯為：挑戰；異議；質疑等）的組合來交換客戶端訪問金鑰。



### 建立客戶端

在使用 PKCE 方式發佈令牌之前，你需要先建立一個啟用了 PKCE 的客戶端。你可以使用 Artisan 命令 `passport:client` 並帶上 `--public` 參數來完成該操作：

```shell
php artisan passport:client --public
```

### 請求令牌

#### 驗證碼（Code Verifier ）和質疑碼（Code Challenge）

這種授權方式不提供授權秘鑰，開發者需要建立一個驗證碼和質疑碼的組合來請求得到一個令牌。

驗證碼是一串包含 43 位到 128 位字元的隨機字串。可用字元包括字母，數字以及下面這些字元：`"-"`, `"."`, `"_"`, `"~"`，可參考 [RFC 7636 specification](https://tools.ietf.org/html/rfc7636) 定義。

質疑碼是一串 Base64 編碼包含 URL 和檔案名稱安全字元的字串，字串結尾的 `'='` 號需要刪除，並且不能包含分行符號，空白符或其他附加字元。

    $encoded = base64_encode(hash('sha256', $code_verifier, true));

    $codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');

#### 授權重新導向

客戶端建立完後，你可以使用客戶端 ID 以及生成的驗證碼，質疑碼從你的應用請求獲取授權碼和訪問令牌。首先，業務端應用需要向伺服器端路由 `/oauth/authorize` 發起重新導向請求：

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $request->session()->put(
            'code_verifier', $code_verifier = Str::random(128)
        );

        $codeChallenge = strtr(rtrim(
            base64_encode(hash('sha256', $code_verifier, true))
        , '='), '+/', '-_');

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            'code_challenge' => $codeChallenge,
            'code_challenge_method' => 'S256',
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });



#### 驗證碼到訪問令牌的轉換

使用者授權訪問後，將重新導向到業務端服務。正如標準授權定義那樣，業務端需要驗證回傳的 `state` 參數的值和在重新導向之前設定的值是否一致。

如果 state 的值驗證通過，業務接入端需要嚮應用端發起一個獲取訪問令牌的 `POST` 請求。請求的參數需要包括之前使用者授權通過後你的應用生成的授權碼，以及之前生成的驗證碼：

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Http;

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        $codeVerifier = $request->session()->pull('code_verifier');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'code_verifier' => $codeVerifier,
            'code' => $request->code,
        ]);

        return $response->json();
    });

## 密碼授權方式的令牌

> **注意**  
> 我們不再建議使用密碼授予令牌。相反，你應該選擇 [OAuth2 伺服器當前推薦的授權類型](https://oauth2.thephpleague.com/authorization-server/which-grant/) 。

OAuth2 的密碼授權方式允許你自己的客戶端（比如手機端應用），通過使用信箱 / 使用者名稱和密碼獲取訪問秘鑰。這樣你就可以安全的為自己發放令牌，而不需要完整地走 OAuth2 的重新導向授權訪問流程。



### 建立密碼授權方式客戶端

在你使用密碼授權方式發佈令牌前，你需要先建立密碼授權方式的客戶端。你可以通過 Artisan 命令 `passport:client` ， 並加上 `--password` 參數來建立這樣的客戶端。 **如果你已經運行過 `passport:install` 命令，則不需要再運行下面的命令:**

```shell
php artisan passport:client --password
```

### 請求令牌

密碼授權方式的客戶端建立好後，你就可以使用使用者信箱和密碼向 `/oauth/token` 路由發起 `POST` 請求，以獲取訪問令牌。請記住，該路由已經在 `Passport::routes` 方法中定義，你無需再手動實現它。如果請求成功，你將在返回 JSON 串中獲取到 `access_token` 和 `refresh_token` :

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '',
    ]);

    return $response->json();

> **技巧**  
> 請記住，默認情況下 access token 都是長生命週期的，但是如果有需要的話，你可以主動去 [設定 access token 的過期時間](#configuration) 。

### 請求所有的範疇

當使用密碼授權（password grant）或者客戶端認證授權（client credentials grant）方式時，你可能希望將應用所有的範疇範圍都授權給令牌。你可以通過設定 scope 參數為 `*` 來實現。 一旦你這樣設定了，所有的 `can` 方法都將返回 `true` 值。 此範圍只能在密碼授權 `password` 或客戶端認證授權 `client_credentials` 下使用：

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '*',
    ]);



### 自訂使用者提供者

如果你的應用程式使用多個 [使用者認證提供器](/docs/laravel/10.x/authentication#introduction)，你可以在建立客戶端通過 `artisan passport:client --password` 命令時使用 `--provider` 選項來指定提供器。 給定的提供器名稱應與應用程式的 `config/auth.php` 組態檔案中定義的有效提供器匹配。 然後，你可以 [使用中介軟體保護你的路由](#via-middleware) 以確保只有來自守衛指定提供器的使用者才被授權。

### 自訂使用者名稱欄位

當使用密碼授權進行身份驗證時，Passport 將使用可驗證模型的 `email` 屬性作為 「使用者名稱」 。 但是，你可以通過在模型上定義 `findForPassport` 方法來自訂此行為：

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;

        /**
         * 尋找給定使用者名稱的使用者實例。
         */
        public function findForPassport(string $username): User
        {
            return $this->where('username', $username)->first();
        }
    }

### 自訂密碼驗證

當使用密碼授權進行身份驗證時，Passport 將使用模型的 `password` 屬性來驗證給定的密碼。 如果你的模型沒有 `password` 屬性或者你希望自訂密碼驗證邏輯，你可以在模型上定義 `validateForPassportPasswordGrant` 方法：

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Support\Facades\Hash;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;

        /**
         * 驗證使用者的密碼以獲得 Passport 密碼授權。
         */
        public function validateForPassportPasswordGrant(string $password): bool
        {
            return Hash::check($password, $this->password);
        }
    }



## 隱式授權令牌

> **注意**  
> 我們不再推薦使用隱式授權令牌。 相反，你應該選擇 [ OAuth2 伺服器當前推薦的授權類型](https://oauth2.thephpleague.com/authorization-server/which-grant/) 。

隱式授權類似於授權碼授權； 但是，令牌會在不交換授權碼的情況下返回給客戶端。 此授權最常用於無法安全儲存客戶端憑據的 JavaScript 或移動應用程式。 要啟用授權，請在應用程式的 `App\Providers\AuthServiceProvider` 類的 `boot` 方法中呼叫 `enableImplicitGrant` 方法：

    /**
     * 註冊任何身份驗證/授權服務。
     */
    public function boot(): void
    {
        Passport::enableImplicitGrant();
    }

啟用授權後，開發人員可以使用他們的客戶端 ID 從你的應用程式請求訪問令牌。 消費應用程式應該嚮應用程序的 `/oauth/authorize` 路由發出重新導向請求，如下所示：

    use Illuminate\Http\Request;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'token',
            'scope' => '',
            'state' => $state,
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

> **技巧**  
> 請記住， `/oauth/authorize` 路由已經由 `Passport::routes` 方法定義。 你無需手動定義此路由。

## 客戶憑證授予令牌

客戶端憑據授予適用於機器對機器身份驗證。 例如，你可以在通過 API 執行維護任務的計畫作業中使用此授權。



要想讓應用程式可以通過客戶端憑據授權發佈令牌，首先，你需要建立一個客戶端憑據授權客戶端。你可以使用 `passport:client` Artisan 命令的 `--client` 選項來執行此操作：

```shell
php artisan passport:client --client
```

接下來，要使用這種授權，你首先需要在 `app/Http/Kernel.php` 的 `$routeMiddleware` 屬性中新增 `CheckClientCredentials` 中介軟體：

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;

    protected $middlewareAliases = [
        'client' => CheckClientCredentials::class,
    ];

之後，在路由上附加中介軟體：

    Route::get('/orders', function (Request $request) {
        ...
    })->middleware('client');

要將對路由的訪問限製為特定範圍，你可以在將 `client` 中介軟體附加到路由時提供所需範圍的逗號分隔列表：

    Route::get('/orders', function (Request $request) {
        ...
    })->middleware('client:check-status,your-scope');

### 檢索令牌

要使用此授權類型檢索令牌，請向 `oauth/token` 端點發出請求：

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'client_credentials',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => 'your-scope',
    ]);

    return $response->json()['access_token'];

## 個人訪問令牌

有時，你的使用者要在不經過傳統的授權碼重新導向流程的情況下向自己頒發訪問令牌。允許使用者通過應用程式使用者介面對自己發佈令牌，有助於使用者體驗你的 API，或者也可以將其作為一種更簡單的發佈訪問令牌的方式。

> **技巧**  
> 如果你的應用程式主要使用 Passport 來發佈個人訪問令牌，請考慮使用 Laravel 的輕量級第一方庫 [Laravel Sanctum](/docs/laravel/10.x/sanctum) 來發佈 API 訪問令牌。



### 建立個人訪問客戶端

在應用程式發出個人訪問令牌前，你需要在 `passport:client` 命令後帶上 `--personal` 參數來建立對應的客戶端。如果你已經運行了 `passport:install` 命令，則無需再運行此命令:

```shell
php artisan passport:client --personal
```

建立個人訪問客戶端後，將客戶端的 ID 和純文字金鑰放在應用程式的 `.env` 檔案中:

```ini
PASSPORT_PERSONAL_ACCESS_CLIENT_ID="client-id-value"
PASSPORT_PERSONAL_ACCESS_CLIENT_SECRET="unhashed-client-secret-value"
```

### 管理個人令牌

建立個人訪問客戶端後，你可以使用 `App\Models\User` 模型實例的 `createToken` 方法來為給定使用者發佈令牌。 `createToken` 方法接受令牌的名稱作為其第一個參數和可選的 [範疇](#token-scopes) 陣列作為其第二個參數:

    use App\Models\User;

    $user = User::find(1);

    // 建立沒有範疇的令牌...
    $token = $user->createToken('Token Name')->accessToken;

    // 建立具有範疇的令牌...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### JSON API

Passport 中還有一個用於管理個人訪問令牌的 JSON API。你可以將其與你自己的前端配對，為你的使用者提供一個用於管理個人訪問令牌的儀表板。下面，我們將回顧所有用於管理個人訪問令牌的 API 。為了方便起見，我們將使用 [Axios](https://github.com/mzabriskie/axios) 來演示向 API 發出 HTTP 請求。



JSON API 由 `web` 和 `auth` 這兩個中介軟體保護；因此，只能從你自己的應用程式中呼叫它。無法從外部源呼叫它。

#### `GET /oauth/scopes`

此路由會返回應用中定義的所有 [範疇](#token-scopes) 。你可以使用此路由列出使用者可以分配給個人訪問令牌的範圍:

```js
axios.get('/oauth/scopes')
    .then(response => {
        console.log(response.data);
    });
```

#### `GET /oauth/personal-access-tokens`

此路由返回認證使用者建立的所有個人訪問令牌。這主要用於列出使用者的所有令牌，以便他們可以編輯和撤銷它們:

```js
axios.get('/oauth/personal-access-tokens')
    .then(response => {
        console.log(response.data);
    });
```

#### `POST /oauth/personal-access-tokens`

此路由建立新的個人訪問令牌。它需要兩個資料：令牌的 `name` 和 `scopes` 。

```js
const data = {
    name: 'Token Name',
    scopes: []
};

axios.post('/oauth/personal-access-tokens', data)
    .then(response => {
        console.log(response.data.accessToken);
    })
    .catch (response => {
        // 列出響應的錯誤...
    });
```

#### `DELETE /oauth/personal-access-tokens/{token-id}`

此路由可用於撤銷個人訪問令牌：

```js
axios.delete('/oauth/personal-access-tokens/' + tokenId);
```

## 路由保護

### 通過中介軟體

Passport 包含一個 [驗證保護機制](/docs/laravel/10.x/authentication#adding-custom-guards) 驗證請求中傳入的訪問令牌。 若組態 `api` 的看守器使用 `passport` 驅動，你只要在需要有效訪問令牌的路由上指定 `auth:api` 中介軟體即可：

    Route::get('/user', function () {
        // ...
    })->middleware('auth:api');

> **注意**  
> 如果你正在使用 [客戶端授權令牌](#client-credentials-grant-tokens) ，你應該使用 [ `client` 中介軟體](#client-credentials-grant-tokens) 來保護你的路由，而不是使用 `auth:api` 中介軟體。



#### 多個身份驗證看守器

如果你的應用程式可能使用完全不同的 `Eloquent` 模型、不同類型的使用者進行身份驗證，則可能需要為應用程式中的每種使用者設定看守器。這使你可以保護特定看守器的請求。例如，在組態檔案 `config/auth.php` 中設定以下看守器：

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],

以下路由將使用 `customers` 使用者提供者的 `api-customers` 看守器來驗證傳入的請求：

    Route::get('/customer', function () {
        // ...
    })->middleware('auth:api-customers');

> **技巧**  
> 關於使用 Passport 的多個使用者提供器的更多資訊，請參考 [密碼認證文件](#customizing-the-user-provider) 。

### 傳遞訪問令牌

當呼叫 Passport 保護下的路由時，接入的 API 應用需要將訪問令牌作為 `Bearer` 令牌放在要求標頭 `Authorization` 中。例如，使用 Guzzle HTTP 庫時：

    use Illuminate\Support\Facades\Http;

    $response = Http::withHeaders([
        'Accept' => 'application/json',
        'Authorization' => 'Bearer '.$accessToken,
    ])->get('https://passport-app.test/api/user');

    return $response->json();

## 令牌範疇

範疇可以讓 API 客戶端在請求帳戶授權時請求特定的權限。例如，如果你正在建構電子商務應用程式，並不是所有接入的 API 應用都需要下訂單的功能。你可以讓接入的 API 應用只被允許授權訪問訂單發貨狀態。換句話說，範疇允許應用程式的使用者限制第三方應用程式執行的操作。



### 定義範疇

你可以在 `App\Providers\AuthServiceProvider` 的 `boot` 方法中使用 `Passport::tokensCan` 方法來定義 API 的範疇。`tokensCan` 方法接受一個包含範疇名稱和描述的陣列作為參數。範疇描述將會在授權確認頁中直接展示給使用者，你可以將其定義為任何你需要的內容：

    /**
     * 註冊身份驗證/授權服務。
     */
    public function boot(): void
    {
        Passport::tokensCan([
            'place-orders' => 'Place orders',
            'check-status' => 'Check order status',
        ]);
    }

### 默認範疇

如果客戶端沒有請求任何特定的範圍，你可以在 `App\Providers\AuthServiceProvider` 類的 `boot`   方法中使用 `setDefaultScope` 方法來定義默認的範疇。

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

    Passport::setDefaultScope([
        'check-status',
        'place-orders',
    ]);

> **技巧**
> Passport 的默認範疇不適用於由使用者生成的個人訪問令牌。

### 給令牌分配範疇

#### 請求授權碼

使用授權碼請求訪問令牌時，接入的應用需為 `scope` 參數指定所需範疇。 `scope` 參數包含多個範疇時，名稱之間使用空格分割：

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

#### 分發個人訪問令牌



使用 `App\Models\User` 模型的 `createToken` 方法發放個人訪問令牌時，可以將所需範疇的陣列作為第二個參數傳給此方法：

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

### 檢查範疇

Passport 包含兩個中介軟體，可用於驗證傳入的請求是否包含訪問指定範疇的令牌。使用之前，需要將下面的中介軟體新增到 `app/Http/Kernel.php` 檔案的 `$middlewareAliases` 屬性中：

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### 檢查所有範疇

路由可以使用 `scopes` 中介軟體來檢查當前請求是否擁有指定的所有範疇：

    Route::get('/orders', function () {
        // 訪問令牌具有 "check-status" 和 "place-orders" 範疇...
    })->middleware(['auth:api', 'scopes:check-status,place-orders']);

#### 檢查任意範疇

路由可以使用 `scope` 中介軟體來檢查當前請求是否擁有指定的 *任意* 範疇：

    Route::get('/orders', function () {
        // 訪問令牌具有 "check-status" 或 "place-orders" 範疇...
    })->middleware(['auth:api', 'scope:check-status,place-orders']);

#### 檢查令牌實例上的範疇

即使含有訪問令牌驗證的請求已經通過應用程式的驗證，你仍然可以使用當前授權 `App\Models\User` 實例上的 `tokenCan` 方法來驗證令牌是否擁有指定的範疇：

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            // ...
        }
    });



#### 附加範疇方法

`scopeIds` 方法將返回所有已定義 ID / 名稱的陣列：

    use Laravel\Passport\Passport;

    Passport::scopeIds();

`scopes` 方法將返回一個包含所有已定義範疇陣列的 `Laravel\Passport\Scope` 實例：

    Passport::scopes();

`scopesFor` 方法將返回與給定 ID / 名稱匹配的 `Laravel\Passport\Scope` 實例陣列：

    Passport::scopesFor(['place-orders', 'check-status']);

你可以使用 `hasScope` 方法確定是否已定義給定範疇：

    Passport::hasScope('place-orders');

## 使用 JavaScript 接入 API

在建構 API 時， 如果能通過 JavaScript 應用接入自己的 API 將會給開發過程帶來極大的便利。這種 API 開發方法允許你使用自己的應用程式的 API 和別人共享的 API 。你的 Web 應用程式、移動應用程式、第三方應用程式以及可能在各種軟體包管理器上發佈的任何 SDK 都可能會使用相同的 API 。

通常，如果要在 JavaScript 應用程式中使用 API ，需要手動嚮應用程序傳送訪問令牌，並將其傳遞給應用程式。但是， Passport 有一個可以處理這個問題的中介軟體。將 `CreateFreshApiToken` 中介軟體新增到 `app/Http/Kernel.php` 檔案中的 `web` 中介軟體組就可以了：

    'web' => [
        // 其他中介軟體...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

> **注意**  
> 你需要確保 `CreateFreshApiToken` 中介軟體是你的中介軟體堆疊中的最後一個中介軟體。

該中介軟體會將 `laravel_token` cookie 附加到你的響應中。該 cookie 將包含一個加密後的 JWT ， Passport 將用來驗證來自 JavaScript 應用程式的 API 請求。JWT 的生命週期等於你的 `session.lifetime` 組態值。至此，你可以在不明確傳遞訪問令牌的情況下嚮應用程序的 API 發出請求：

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });



#### 自訂 Cookie 名稱

如果需要，你可以在 `App\Providers\AuthServiceProvider` 類的 `boot` 方法中使用 `Passport::cookie` 方法來自訂 `laravel_token` cookie 的名稱：

    /**
     * 註冊認證 / 授權服務.
     */
    public function boot(): void
    {
        Passport::cookie('custom_name');
    }

#### CSRF 保護

當使用這種授權方法時，你需要確認請求中包含有效的 CSRF 令牌。默認的 Laravel JavaScript 腳手架會包含一個 Axios 實例，該實例是自動使用加密的 `XSRF-TOKEN` cookie 值在同源請求上傳送 `X-XSRF-TOKEN` 要求標頭。

> **技巧**  
> 如果你選擇傳送 `X-CSRF-TOKEN` 要求標頭而不是 `X-XSRF-TOKEN` ，則需要使用 `csrf_token()` 提供的未加密令牌。

## 事件

Passport 在發出訪問令牌和刷新令牌時引發事件。 你可以使用這些事件來修改或撤消資料庫中的其他訪問令牌。如果你願意，可以在應用程式的 `App\Providers\EventServiceProvider` 類中將監聽器註冊到這些事件：

    /**
     * 應用程式的事件監聽器對應
     *
     * @var array
     */
    protected $listen = [
        'Laravel\Passport\Events\AccessTokenCreated' => [
            'App\Listeners\RevokeOldTokens',
        ],

        'Laravel\Passport\Events\RefreshTokenCreated' => [
            'App\Listeners\PruneOldTokens',
        ],
    ];

## 測試

Passport 的 `actingAs` 方法可以指定當前已認證使用者及其範疇。`actingAs` 方法的第一個參數是使用者實例，第二個參數是使用者令牌範疇陣列：

    use App\Models\User;
    use Laravel\Passport\Passport;

    public function test_servers_can_be_created(): void
    {
        Passport::actingAs(
            User::factory()->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(201);
    }



Passport 的 `actingAsClient` 方法可以指定當前已認證使用者及其範疇。 `actingAsClient` 方法的第一個參數是使用者實例，第二個參數是使用者令牌範疇陣列：

    use Laravel\Passport\Client;
    use Laravel\Passport\Passport;

    public function test_orders_can_be_retrieved(): void
    {
        Passport::actingAsClient(
            Client::factory()->create(),
            ['check-status']
        );

        $response = $this->get('/api/orders');

        $response->assertStatus(200);
    }

