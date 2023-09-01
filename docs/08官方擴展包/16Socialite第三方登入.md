# Socialite 第三方登入

## 簡介

除了典型的基於表單的身份驗證之外，Laravel 還提供了一種使用 [Laravel Socialite](https://github.com/laravel/socialite)對 OAuth providers 進行身份驗證的簡單方便的方法。 Socialite 目前支援 Facebook，Twitter，LinkedIn，Google，GitHub，GitLab 和 Bitbucket 的身份驗證。

> 技巧：其他平台的驅動器可以在 [Socialite Providers](https://socialiteproviders.com/) 社區驅動網站尋找。

## 安裝

在開始使用 Socialite 之前，通過 Composer 軟體包管理器將軟體包新增到項目的依賴項中:

```shell
composer require laravel/socialite
```

## 升級

升級到 Socialite 的新主要版本時，請務必仔細查看 [the upgrade guide](https://github.com/laravel/socialite/blob/master/UPGRADE.).

## 組態

在使用 Socialite 之前，你需要為應用程式使用的OAuth提供程序新增憑據。通常，可以通過在要驗證的服務的儀表板中建立“開發人員應用程式”來檢索這些憑據。

這些憑證應該放在你的 `config/services.php` 組態檔案中， 並且應該使用 `facebook`, `twitter` (OAuth 1.0), `twitter-oauth-2` (OAuth 2.0), `linkedin`, `google`, `github`, `gitlab`, 或者 `bitbucket`, 取決於應用程式所需的提供商：

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => 'http://example.com/callback-url',
    ],

> 技巧：如果 `redirect` 項的值包含一個相對路徑，它將會自動解析為全稱 URL。

## 認證

### 路由

要使用 OAuth 提供程序對使用者進行身份驗證，你需要兩個路由：一個用於將使用者重新導向到 OAuth provider，另一個用於在身份驗證後接收來自 provider 的回呼。下面的示例 controller 演示了這兩個路由的實現：

    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/redirect', function () {
        return Socialite::driver('github')->redirect();
    });

    Route::get('/auth/callback', function () {
        $user = Socialite::driver('github')->user();

        // $user->token
    });

`redirect` 提供的方法 `Socialite` facade 負責將使用者重新導向到 OAuth provider，而該 user 方法將讀取傳入的請求並在身份驗證後從提供程序中檢索使用者的資訊。

### 身份驗證和儲存

從 OAuth 提供程序檢索到使用者後，你可以確定該使用者是否存在於應用程式的資料庫中並[驗證使用者](/docs/laravel/10.x/authentication#authenticate-a-user-instance)。如果使用者在應用程式的資料庫中不存在，通常會在資料庫中建立一條新記錄來代表該使用者：

    use App\Models\User;
    use Illuminate\Support\Facades\Auth;
    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/callback', function () {
        $githubUser = Socialite::driver('github')->user();

        $user = User::updateOrCreate([
            'github_id' => $githubUser->id,
        ], [
            'name' => $githubUser->name,
            'email' => $githubUser->email,
            'github_token' => $githubUser->token,
            'github_refresh_token' => $githubUser->refreshToken,
        ]);

        Auth::login($user);

        return redirect('/dashboard');
    });

> 技巧：有關特定 OAuth 提供商提供哪些使用者資訊的更多資訊，請參閱有關 [檢索使用者詳細資訊](#retrieving-user-details) 的文件。

### 訪問範疇

在重新導向使用者之前，你還可以使用 `scopes` 方法在請求中新增其他「範疇」。此方法會將所有現有範疇與你提供的範疇合併：

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();

你可以使用 `setScopes` 方法覆蓋所有現有範圍：

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();

### 可選參數

許多 OAuth providers 支援重新導向請求中的可選參數。 要在請求中包含任何可選參數，請使用關聯陣列呼叫 `with` 方法：

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('google')
        ->with(['hd' => 'example.com'])
        ->redirect();

> 注意：使用  `with` 方法時, 注意不要傳遞任何保留的關鍵字，例如 `state` 或 `response_type`。

## 檢索使用者詳細資訊

在將使用者重新導向回你的身份驗證回呼路由之後，你可以使用 Socialite 的 `user` 方法檢索使用者的詳細資訊。`user` 方法為返回的使用者對象提供了各種屬性和方法，你可以使用這些屬性和方法在你自己的資料庫中儲存有關該使用者的資訊。

你可以使用不同的屬性和方法這取決於要進行身份驗證的 OAuth 提供程序是否支援 OAuth 1.0 或 OAuth 2.0：

    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/callback', function () {
        $user = Socialite::driver('github')->user();

        // OAuth 2.0 providers...
        $token = $user->token;
        $refreshToken = $user->refreshToken;
        $expiresIn = $user->expiresIn;

        // OAuth 1.0 providers...
        $token = $user->token;
        $tokenSecret = $user->tokenSecret;

        // All providers...
        $user->getId();
        $user->getNickname();
        $user->getName();
        $user->getEmail();
        $user->getAvatar();
    });



#### 從令牌中檢索使用者詳細資訊 (OAuth2)

如果你已經有了一個使用者的有效訪問令牌，你可以使用 Socialite 的 `userFromToken` 方法檢索其詳細資訊：

    use Laravel\Socialite\Facades\Socialite;

    $user = Socialite::driver('github')->userFromToken($token);

#### 從令牌中檢索使用者詳細資訊 (OAuth2)

如果你已經有了一對有效的使用者令牌/秘鑰，你可以使用 Socialite 的 `userFromTokenAndSecret` 方法檢索他們的詳細資訊：

    use Laravel\Socialite\Facades\Socialite;

    $user = Socialite::driver('twitter')->userFromTokenAndSecret($token, $secret);

#### 無認證狀態

`stateless` 方法可用於停用 session 狀態驗證。 這在向 API 新增社交身份驗證時非常有用：

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('google')->stateless()->user();

> 注意：Twitter 驅動程式不支援無狀態身份驗證，它使用 OAuth 1.0 進行身份驗證

