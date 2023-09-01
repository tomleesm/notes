# Pennant 測試新功能

## 介紹

[Laravel Pennant](https://github.com/laravel/pennant) 是一個簡單輕量的特性標誌包，沒有臃腫。特性標誌使你可以有信心地逐步推出新的應用程式功能，測試新的介面設計，支援基幹開發策略等等。


## 安裝

首先，使用 Composer 包管理器將 Pennant 安裝到你的項目中：

```shell
composer require laravel/pennant
```

接下來，你應該使用 `vendor:publish` Artisan 命令發佈 Pennant 組態和遷移檔案： `vendor:publish` Artisan command:

```shell
php artisan vendor:publish --provider="Laravel\Pennant\PennantServiceProvider"
```

最後，你應該運行應用程式的資料庫遷移。這將建立一個 `features` 表，`Pennant` 使用它來驅動其 `database` 驅動程式：

```shell
php artisan migrate
```

## 組態

在發佈 Pennant 資源之後，組態檔案將位於 `config/pennant.php`。此組態檔案允許你指定 Pennant 用於儲存已解析的特性標誌值的默認儲存機制。

Pennant 支援使用 `array` 驅動程式在記憶體陣列中儲存已解析的特性標誌值。或者，Pennant 可以使用 `database` 驅動程式在關聯式資料庫中持久儲存已解析的特性標誌值，這是 Pennant 使用的默認儲存機制。


## 定義特性

要定義特性，你可以使用 `Feature` 門面提供的 `define` 方法。你需要為該特性提供一個名稱以及一個閉包，用於解析該特性的初始值。

通常，特性是在服務提供程序中使用 `Feature` 門面定義的。閉包將接收特性檢查的“範疇”。最常見的是，範疇是當前已認證的使用者。在此示例中，我們將定義一個功能，用於逐步嚮應用程序使用者推出新的 API：

```php
<?php

namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Lottery;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::define('new-api', fn (User $user) => match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        });
    }
}
```

正如你所看到的，我們對我們的特性有以下規則：

- 所有內部團隊成員應使用新 API。
- 任何高流量客戶不應使用新 API。
- 否則，該特性應在具有 1/100 機率啟動的使用者中隨機分配。

首次檢查給定使用者的 `new-api`特性時，儲存驅動程式將儲存閉包的結果。下一次針對相同使用者檢查特性時，將從儲存中檢索該值，不會呼叫閉包。

為方便起見，如果特性定義僅返回一個 Lottery，你可以完全省略閉包：

    Feature::define('site-redesign', Lottery::odds(1, 1000));


### 基於類的特性

Pennant 還允許你定義基於類的特性。不像基於閉包的特性定義，不需要在服務提供者中註冊基於類的特性。為了建立一個基於類的特性，你可以呼叫 `pennant:feature` Artisan 命令。默認情況下，特性類將被放置在你的應用程式的 `app/Features` 目錄中：

```shell
php artisan pennant:feature NewApi
```

在編寫特性類時，你只需要定義一個 `resolve` 方法，用於為給定的範圍解析特性的初始值。同樣，範圍通常是當前經過身份驗證的使用者：

```php
<?php

namespace App\Features;

use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * 解析特性的初始值.
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

> **注** 特性類是通過[容器](/docs/laravel/10.x/container),解析的，因此在需要時可以在特性類的建構函式中注入依賴項。


## 檢查特性

要確定一個特性是否處於活動狀態，你可以在 `Feature` 門面上使用 `active` 方法。默認情況下，特性針對當前已認證的使用者進行檢查：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * 顯示資源的列表.
     */
    public function index(Request $request): Response
    {
        return Feature::active('new-api')
                ? $this->resolveNewApiResponse($request)
                : $this->resolveLegacyApiResponse($request);
    }
    // ...
}
```

為了方便起見，如果一個特徵定義只返回一個抽獎結果，你可以完全省略閉包:

    Feature::define('site-redesign', Lottery::odds(1, 1000));


### 基於類的特徵

Pennant 還允許你定義基於類的特徵。與基於閉包的特徵定義不同，無需在服務提供者中註冊基於類的特徵。要建立基於類的特徵，你可以呼叫 pennant:feature Artisan 命令。默認情況下，特徵類將被放置在你的應用程式的 app/Features 目錄中。:

```php
php artisan pennant:feature NewApi
```

編寫特徵類時，你只需要定義一個 `resolve` 方法，該方法將被呼叫以解析給定範疇的特徵的初始值。同樣，該範疇通常是當前已驗證的使用者。:

```php
<?php

namespace App\Features;

use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * 解析特徵的初始值.
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

> **注意** 特徵類通過 [容器](/docs/laravel/10.x/container), 解析，因此在需要時，你可以將依賴項注入到特徵類的建構函式中.


## Checking Features

要確定特徵是否處於活動狀態，你可以在 Feature 門面上使用 `active` 方法。默認情況下，特徵將針對當前已驗證的使用者進行檢查。:

```php

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     *顯示資源的列表.
     */
    public function index(Request $request): Response
    {
        return Feature::active('new-api')
                ? $this->resolveNewApiResponse($request)
                : $this->resolveLegacyApiResponse($request);
    }
    // ...
}
```

雖然默認情況下特性針對當前已認證的使用者進行檢查，但你可以輕鬆地針對其他使用者或範圍檢查特性。為此，使用 `Feature` 門面提供的 `for` 方法：

```php
return Feature::for($user)->active('new-api')
        ? $this->resolveNewApiResponse($request)
        : $this->resolveLegacyApiResponse($request);
```

Pennant 還提供了一些額外的方便方法，在確定特性是否活動或不活動時可能非常有用：

```php
//確定所有給定的特性是否都活動...
Feature::allAreActive(['new-api', 'site-redesign']);

// 確定任何給定的特性是否都活動...
Feature::someAreActive(['new-api', 'site-redesign']);

// 確定特性是否處於非活動狀態...
Feature::inactive('new-api');

// 確定所有給定的特性是否都處於非活動狀態...
Feature::allAreInactive(['new-api', 'site-redesign']);

// 確定任何給定的特性是否都處於非活動狀態...
Feature::someAreInactive(['new-api', 'site-redesign']);
```

> **注**

>當在 HTTP 上下文之外使用 Pennant（例如在 Artisan 命令或排隊作業中）時，你通常應[明確指定特性的範疇](#specifying-the-scope)。或者，你可以定義一個[默認範疇](#default-scope)，該範疇考慮到已認證的 HTTP 上下文和未經身份驗證的上下文。


#### 檢查基於類的特性

對於基於類的特性，應該在檢查特性時提供類名：

```php
<?php

namespace App\Http\Controllers;

use App\Features\NewApi;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{

    /**
     * 顯示資源的列表.
     */
    public function index(Request $request): Response
    {
        return Feature::active(NewApi::class)
                ? $this->resolveNewApiResponse($request)
                : $this->resolveLegacyApiResponse($request);
    }
    // ...
}
```


### 條件執行

`when` 方法可用於在特性啟動時流暢地執行給定的閉包。此外，可以提供第二個閉包，如果特性未啟動，則將執行它：

```php
<?php

namespace App\Http\Controllers;

use App\Features\NewApi;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * 顯示資源的列表.
     */
    public function index(Request $request): Response
    {
        return Feature::when(NewApi::class,
            fn () => $this->resolveNewApiResponse($request),
            fn () => $this->resolveLegacyApiResponse($request),
        );
    }
    // ...
}
```

`unless` 方法是 `when` 方法的相反，如果特性未啟動，則執行第一個閉包：

    return Feature::unless(NewApi::class,

        fn () => $this->resolveLegacyApiResponse($request),

        fn () => $this->resolveNewApiResponse($request),

    );


### HasFeatures Trait

Pennant 的 `HasFeatures` Trait 可以新增到你的應用的 `User` 模型（或其他具有特性的模型）中，以提供一種流暢、方便的方式從模型直接檢查特性：

```php

<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Pennant\Concerns\HasFeatures;

class User extends Authenticatable
{
    use HasFeatures;
    // ...
}
```

一旦將 HasFeatures Trait 新增到你的模型中，你可以通過呼叫 `features` 方法輕鬆檢查特性：

```php
if ($user->features()->active('new-api')) {
    // ...
}
```

當然，`features` 方法提供了許多其他方便的方法來與特性互動：

```php
// 值...
$value = $user->features()->value('purchase-button')
$values = $user->features()->values(['new-api', 'purchase-button']);

// 狀態...
$user->features()->active('new-api');
$user->features()->allAreActive(['new-api', 'server-api']);
$user->features()->someAreActive(['new-api', 'server-api']);
$user->features()->inactive('new-api');
$user->features()->allAreInactive(['new-api', 'server-api']);
$user->features()->someAreInactive(['new-api', 'server-api']);

// 條件執行...
$user->features()->when('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);

$user->features()->unless('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);
```


### Blade 指令

為了使在 Blade 中檢查特性的體驗更加流暢，Pennant提供了一個 `@feature` 指令：

```blade
@feature('site-redesign')
<!-- 'site-redesign' 活躍中 -->
@else
<!-- 'site-redesign' 不活躍-->
@endfeature
```


### 中介軟體

Pennant 還包括一個[中介軟體](/docs/laravel/10.x/middleware)，它可以在路由呼叫之前驗證當前認證使用者是否有訪問功能的權限。首先，你應該將 `EnsureFeaturesAreActive` 中介軟體的別名新增到你的應用程式的 `app/Http/Kernel.php` 檔案中：

```php

use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

protected $middlewareAliases = [
    // ...
 'features' => EnsureFeaturesAreActive::class,
];
```

接下來，你可以將中介軟體分配給一個路由並指定需要訪問該路由的功能。如果當前認證使用者的任何指定功能未啟動，則路由將返回 `400 Bad Request` HTTP 響應。可以使用逗號分隔的列表指定多個功能：

```php
Route::get('/api/servers', function () {
    // ...
})->middleware(['features:new-api,servers-api']);
```


#### 自訂響應

如果你希望在未啟動列表中的任何一個功能時自訂中介軟體返回的響應，可以使用 `EnsureFeaturesAreActive` 中介軟體提供的 `whenInactive` 方法。通常，這個方法應該在應用程式的服務提供者的 boot 方法中呼叫：

```php

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

/**
 * 載入服務.
 */
public function boot(): void
{
    EnsureFeaturesAreActive::whenInactive(
        function (Request $request, array $features) {
            return new Response(status: 403);
        }
    );
    // ...
}
```


### 記憶體快取

當檢查特性時，Pennant 將建立一個記憶體快取以儲存結果。如果你使用的是 `database` 驅動程式，則在單個請求中重新檢查相同的功能標誌將不會觸發額外的資料庫查詢。這也確保了該功能在請求的持續時間內具有一致的結果。

如果你需要手動刷新記憶體快取，可以使用 `Feature` 門面提供的 `flushCache` 方法：

    Feature::flushCache();


## 範疇


### 指定範疇

如前所述，特性通常會針對當前已驗證的使用者進行檢查。但這可能並不總是適合你的需求。因此，你可以通過 `Feature` 門面的 `for` 方法來指定要針對哪個範疇檢查給定的特性：

```php
return Feature::for($user)->active('new-api')
        ? $this->resolveNewApiResponse($request)
        : $this->resolveLegacyApiResponse($request);
```

當然，特性範疇不限於“使用者”。假設你建構了一個新的結算體驗，你要將其推出給整個團隊而不是單個使用者。也許你希望年齡最大的團隊的推出速度比年輕的團隊慢。你的特性解析閉包可能如下所示：

```php

use App\Models\Team;
use Carbon\Carbon;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('billing-v2', function (Team $team) {
    if ($team->created_at->isAfter(new Carbon('1st Jan, 2023'))) {
        return true;
    }

    if ($team->created_at->isAfter(new Carbon('1st Jan, 2019'))) {
        return Lottery::odds(1 / 100);
    }

    return Lottery::odds(1 / 1000);
});
```

你會注意到，我們定義的閉包不需要 `User`，而是需要一個 `Team` 模型。要確定該特性是否對使用者的團隊可用，你應該將團隊傳遞給 `Feature` 門面提供的 `for` 方法：

```php
if (Feature::for($user->team)->active('billing-v2')) {
    return redirect()->to('/billing/v2');
}
// ...
```


### 默認範疇

還可以自訂 Pennant 用於檢查特性的默認範疇。例如，你可能希望所有特性都針對當前認證使用者的團隊進行檢查，而不是針對使用者。你可以在應用程式的服務提供程序中指定此範疇。通常，應該在一個應用程式的服務提供程序中完成這個過程:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 載入程序服務.
     */
    public function boot(): void
    {
        Feature::resolveScopeUsing(fn ($driver) => Auth::user()?->team);
        // ...
    }
}
```

如果沒有通過 `for` 方法顯式提供範疇，則特性檢查將使用當前認證使用者的團隊作為默認範疇：

```php
Feature::active('billing-v2');

// 目前等價於...
Feature::for($user->team)->active('billing-v2');
```


### 空範疇

如果你檢查特性時提供的範疇範圍為 `null`，且特性定義中不支援 `null`（即不是 nullable type 或者沒有在 union type 中包含`null`），那麼 Pennant 將自動返回 `false` 作為特性的結果值。

因此，如果你傳遞給特性的範疇可能為 null 並且你想要特性值的解析器被呼叫，你應該在特性定義邏輯中處理 `null` 範圍值。在一個 Artisan 命令、排隊作業或未經身份驗證的路由中檢查特性可能會出現 `null` 範疇。因為在這些情況下通常沒有經過身份驗證的使用者，所以默認的範疇將為 `null`。

如果你不總是[明確指定特性範疇](#specifying-the-scope)，則應確保範圍類型為"nullable"，並在特性定義邏輯中處理 `null` 範圍值:

```php
use App\Models\User;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('new-api', fn (User $user) => match (true) {// [tl! remove]
Feature::define('new-api', fn (User|null $user) => match (true) {// [tl! add]
    $user === null => true,// [tl! add]
    $user->isInternalTeamMember() => true,
    $user->isHighTrafficCustomer() => false,
    default => Lottery::odds(1 / 100),
});
```


### 標識範疇

Pennant 的內建 `array` 和 `database` 儲存驅動程式可以正確地儲存所有 PHP 資料類型以及 Eloquent 模型的範疇識別碼。但是，如果你的應用程式使用第三方的 Pennant 驅動程式，該驅動程式可能不知道如何正確地儲存 Eloquent 模型或應用程式中其他自訂類型的識別碼。

因此，Pennant 允許你通過在應用程式中用作 Pennant 範疇的對象上實現 `FeatureScopeable` 協議來格式化儲存範圍值。

例如，假設你在單個應用程式中使用了兩個不同的特性驅動程式：內建 `database` 驅動程式和第三方的“Flag Rocket”驅動程式。 "Flag Rocket"驅動程式不知道如何正確地儲存 Eloquent 模型。相反，它需要一個`FlagRocketUser` 實例。通過實現 `FeatureScopeable` 協議中的 `toFeatureIdentifier` 方法，我們可以自訂提供給應用程式中每個驅動程式的可儲存範圍值：

```php
<?php

namespace App\Models;

use FlagRocket\FlagRocketUser;
use Illuminate\Database\Eloquent\Model;
use Laravel\Pennant\Contracts\FeatureScopeable;

class User extends Model implements FeatureScopeable
{
    /**
     * 將對象強制轉換為給定驅動程式的功能範圍識別碼.
     */
    public function toFeatureIdentifier(string $driver): mixed
    {
        return match($driver) {
 		'database' => $this,
 		'flag-rocket' => FlagRocketUser::fromId($this->flag_rocket_id),
        };
    }
}
```


## 豐富的特徵值

到目前為止，我們主要展示了特性的二進制狀態，即它們是「活動的」還是「非活動的」，但是 Pennant 也允許你儲存豐富的值。

例如，假設你正在測試應用程式的「立即購買」按鈕的三種新顏色。你可以從特性定義中返回一個字串，而不是 `true` 或 `false`：

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn (User $user) => Arr::random([
 'blue-sapphire',
 'seafoam-green',
 'tart-orange',
]));

```

你可以使用 `value` 方法檢索 `purchase-button` 特性的值：

```php
$color = Feature::value('purchase-button');
```

Pennant 提供的 Blade 指令也使得根據特性的當前值條件性地呈現內容變得容易：

```blade
@feature('purchase-button', 'blue-sapphire')
<!-- 'blue-sapphire' is active -->
@elsefeature('purchase-button', 'seafoam-green')
<!-- 'seafoam-green' is active -->
@elsefeature('purchase-button', 'tart-orange')
<!-- 'tart-orange' is active -->
@endfeature
```

> 使用豐富值時，重要的是要知道，只要特性具有除 `false` 以外的任何值，它就被視為「活動」。

在呼叫[條件](#conditional-execution) `when` 方法時，特性的豐富值將提供給第一個閉包：

    Feature::when('purchase-button',
        fn ($color) => /* ... */,
        fn () => /* ... */,
    );

同樣，當呼叫條件 `unless` 方法時，特性的豐富值將提供給可選的第二個閉包：

    Feature::unless('purchase-button',
        fn () => /* ... */,
        fn ($color) => /* ... */,
    );


## 獲取多個特性

`values` 方法允許檢索給定範疇的多個特徵：

```php
Feature::values(['billing-v2', 'purchase-button']);

// [
// 'billing-v2' => false,
// 'purchase-button' => 'blue-sapphire',
// ]
```

或者，你可以使用 `all` 方法檢索給定範圍內所有已定義功能的值：

```php
Feature::all();

// [
// 'billing-v2' => false,
// 'purchase-button' => 'blue-sapphire',
// 'site-redesign' => true,
// ]
```

但是，基於類的功能是動態註冊的，直到它們被顯式檢查之前，Pennant並不知道它們的存在。這意味著，如果在當前請求期間尚未檢查過應用程式的基於類的功能，則這些功能可能不會出現在 `all` 方法返回的結果中。

如果你想確保使用 `all` 方法時始終包括功能類，你可以使用Pennant的功能發現功能。要開始使用，請在你的應用程式的任何服務提供程序之一中呼叫 `discover` 方法：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Laravel\Pennant\Feature;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Feature::discover();
            // ...
        }
    }

`discover` 方法將註冊應用程式 `app/Features` 目錄中的所有功能類。`all` 方法現在將在其結果中包括這些類，無論它們是否在當前請求期間進行了檢查：

```php
Feature::all();

// [
// 'App\Features\NewApi' => true,
// 'billing-v2' => false,
// 'purchase-button' => 'blue-sapphire',
// 'site-redesign' => true,
// ]
```


## 預載入

儘管 Pennant 在單個請求中保留了所有已解析功能的記憶體快取，但仍可能遇到性能問題。為了緩解這種情況，Pennant 提供了預載入功能。

為了說明這一點，想像一下我們正在循環中檢查功能是否處於活動狀態：

```php
use Laravel\Pennant\Feature;

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

假設我們正在使用資料庫驅動程式，此程式碼將為循環中的每個使用者執行資料庫查詢-執行潛在的數百個查詢。但是，使用 Pennant 的 `load` 方法，我們可以通過預載入一組使用者或範疇的功能值來消除這種潛在的性能瓶頸：

```php
Feature::for($users)->load(['notifications-beta']);

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

為了僅在尚未載入功能值時載入它們，你可以使用 `loadMissing` 方法：

```php
Feature::for($users)->loadMissing([
 'new-api',
 'purchase-button',
 'notifications-beta',
]);
```


## 更新值

當首次解析功能的值時，底層驅動程式將把結果儲存在儲存中。這通常是為了確保在請求之間為你的使用者提供一致的體驗。但是，有時你可能想手動更新功能的儲存值。

為了實現這一點，你可以使用 `activate` 和 `deactivate` 方法來切換功能的 「打開」或「關閉」狀態：

```php
use Laravel\Pennant\Feature;

// 啟動默認範疇的功能...
Feature::activate('new-api');

// 在給定的範圍中停用功能...
Feature::for($user->team)->deactivate('billing-v2');
```

還可以通過向 `activate` 方法提供第二個參數來手動設定功能的豐富值：

```php
Feature::activate('purchase-button', 'seafoam-green');
```

要指示 Pennant 忘記功能的儲存值，你可以使用 `forget` 方法。當再次檢查功能時，Pennant 將從其功能定義中解析功能的值：

```php
Feature::forget('purchase-button');
```


### 批次更新

要批次更新儲存的功能值，你可以使用 `activateForEveryone` 和 `deactivateForEveryone` 方法。

例如，假設你現在對 `new-api` 功能的穩定性有信心，並為結帳流程找到了最佳的「purchase-button」顏色-你可以相應地更新所有使用者的儲存值：

```php
use Laravel\Pennant\Feature;

Feature::activateForEveryone('new-api');
Feature::activateForEveryone('purchase-button', 'seafoam-green');
```

或者，你可以停用所有使用者的該功能：

```php
Feature::deactivateForEveryone('new-api');
```

> **注意：**這將僅更新已由 Pennant 儲存驅動程式儲存的已解析功能值。你還需要更新應用程式中的功能定義。


### 清除功能

有時，清除儲存中的整個功能可以非常有用。如果你已從應用程式中刪除了功能或已對功能的定義進行了調整，並希望將其部署到所有使用者，則通常需要這樣做。

你可以使用 `purge` 方法刪除功能的所有儲存值：

```php
// 清除單個功能...
Feature::purge('new-api');

// 清除多個功能...
Feature::purge(['new-api', 'purchase-button']);
```

如果你想從儲存中清除所有功能，則可以呼叫 `purge` 方法而不帶任何參數：

```php
Feature::purge();
```

由於在應用程式的部署流程中清除功能可能非常有用，因此 Pennant 包括一個`pennant:purge` Artisan命令：

```sh
php artisan pennant:purge new-api
php artisan pennant:purge new-api purchase-button
```


## 測試

當測試與功能標誌互動的程式碼時，控制測試中返回的功能標誌的最簡單方法是簡單地重新定義該功能。例如，假設你在應用程式的一個服務提供程序中定義了以下功能：

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn () => Arr::random([
 'blue-sapphire',
 'seafoam-green',
 'tart-orange',
]));
```

要在測試中修改功能的返回值，你可以在測試開始時重新定義該功能。以下測試將始終通過，即使 `Arr::random()` 實現仍然存在於服務提供程序中：

```php
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define('purchase-button', 'seafoam-green');
    $this->assertSame('seafoam-green', Feature::value('purchase-button'));
}
```

相同的方法也可以用於基於類的功能：

```php
use App\Features\NewApi;
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define(NewApi::class, true);
    $this->assertTrue(Feature::value(NewApi::class));
}
```

如果你的功能返回一個 `Lottery` 實例，那麼有一些有用的[測試輔助函數可用](/docs/laravel/10.x/helpers#testing-lotteries)。


#### 儲存組態

你可以通過在應用程式的 `phpunit.xml` 檔案中定義 `PENNANT_STORE` 環境變數來組態 Pennant 在測試期間使用的儲存：

```xml
<?xml version="1.0"  encoding="UTF-8"?>
<phpunit colors="true">
    <!-- ... -->
    <php>
<env name="PENNANT_STORE"  value="array"/>
        <!-- ... -->
    </php>
</phpunit>
```


## 新增自訂Pennant驅動程式


#### 實現驅動程式

如果 Pennant 現有的儲存驅動程式都不符合你的應用程式需求，則可以編寫自己的儲存驅動程式。你的自訂驅動程式應實現 `Laravel\Pennant\Contracts\Driver` 介面：

```php
<?php

namespace App\Extensions;

use Laravel\Pennant\Contracts\Driver;

class RedisFeatureDriver implements Driver
{
    public function define(string $feature, callable $resolver): void {}
    public function defined(): array {}
    public function getAll(array $features): array {}
    public function get(string $feature, mixed $scope): mixed {}
    public function set(string $feature, mixed $scope, mixed $value): void {}
    public function setForAllScopes(string $feature, mixed $value): void {}
    public function delete(string $feature, mixed $scope): void {}
    public function purge(array|null $features): void {}
}
```

現在，我們只需要使用 Redis 連接實現這些方法。可以在 [Pennant](https://github.com/laravel/pennant/blob/1.x/src/Drivers/DatabaseDriver.php) 原始碼中查看如何實現這些方法的示例。

> **注意**

> Laravel 不附帶包含擴展的目錄。你可以自由地將它們放在任何你喜歡的位置。在這個示例中，我們建立了一個 `Extensions` 目錄來存放 `RedisFeatureDriver`。


#### 註冊驅動

一旦你的驅動程式被實現，就可以將其註冊到 Laravel 中。要向 Pennant 新增其他驅動程式，可以使用 `Feature` 門面提供的 `extend` 方法。應該在應用程式的 [服務提供者](/docs/laravel/10.x/providers) 的 `boot` 方法中呼叫 `extend` 方法：

```php
<?php

namespace App\Providers;

use App\Extensions\RedisFeatureDriver;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        Feature::extend('redis', function (Application $app) {
            return new RedisFeatureDriver($app->make('redis'), $app->make('events'), []);
        });
    }
}
```

一旦驅動程式被註冊，就可以在應用程式的 `config/pennant.php` 組態檔案中使用 `redis` 驅動程式：

```php
'stores' => [
	'redis' => [
		'driver' => 'redis',
		'connection' => null,
	],
    // ...
],
```

## 事件

Pennant 分發了各種事件，這些事件在跟蹤應用程式中的特性標誌時非常有用。

### `Laravel\Pennant\Events\RetrievingKnownFeature`

該事件在請求特定範疇的已知特徵值第一次被檢索時被觸發。此事件可用於建立和跟蹤應用程式中使用的特徵標記的度量標準。

### `Laravel\Pennant\Events\RetrievingUnknownFeature`

當在請求特定範疇的情況下第一次檢索未知特性時，將分派此事件。如果你打算從應用程式中刪除功能標誌，但可能在整個應用程式中留下了某些零散的引用，此事件可能會有用。你可能會發現有用的是監聽此事件並在其發生時 `report` 或拋出異常：

例如，你可能會發現在監聽到此事件並出現此情況時，使用 `report` 或引發異常會很有用：

```php
<?php

namespace App\Providers;

use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Event;
use Laravel\Pennant\Events\RetrievingUnknownFeature;

class EventServiceProvider extends ServiceProvider
{
    /**
     * Register any other events for your application.
     */
    public function boot(): void
    {
        Event::listen(function (RetrievingUnknownFeature $event) {
            report("Resolving unknown feature [{$event->feature}].");
        });
    }
}
```

### `Laravel\Pennant\Events\DynamicallyDefiningFeature`

當在請求期間首次動態檢查基於類的特性時，將分派此事件。
