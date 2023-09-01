# Telescope 偵錯工具

## 簡介

[Laravel Telescope](https://github.com/laravel/telescope) 是 Laravel 本地開發環境的絕佳伴侶。Telescope 可以洞察你的應用程式的請求、異常、日誌條目、資料庫查詢、排隊的作業、郵件、消息通知、快取操作、定時工作排程、變數列印等。

## 安裝

你可以使用 Composer 將 Telescope 安裝到 Laravel 項目中：

```shell
composer require laravel/telescope
```

安裝 Telescope 後，你應使用 `telescope:install` 命令來發佈其公共資源，然後運行 `migrate` 命令執行資料庫變更來建立和保存 Telescope 需要的資料：

```shell
php artisan telescope:install

php artisan migrate
```

#### 自訂遷移

如果不打算使用 Telescope 的默認遷移，則應在應用程式的 `App\Providers\AppServiceProvider` 類的 `register` 方法中呼叫 `Telescope::ignoreMigrations` 方法。你可以使用以下命令匯出默認遷移：`php artisan vendor:publish --tag=telescope-migrations`

### 僅本地安裝

如果你僅打算使用 Telescope 來幫助你的本地開發，你可以使用 `--dev` 標記安裝 Telescope：

```shell
composer require laravel/telescope --dev

php artisan telescope:install

php artisan migrate
```

運行 `telescope:install` 後，應該從應用程式的 `config/app.php` 組態檔案中刪除 `TelescopeServiceProvider` 服務提供者註冊。手動在 `App\Providers\AppServiceProvider` 類的 `register` 方法中註冊 telescope 的服務提供者來替代。在註冊提供者之前，我們會確保當前環境是 `local`：

    /**
     * 註冊應用服務。
     */
    public function register(): void
    {
        if ($this->app->environment('local')) {
            $this->app->register(\Laravel\Telescope\TelescopeServiceProvider::class);
            $this->app->register(TelescopeServiceProvider::class);
        }
    }

最後，你還應該將以下內容新增到你的 `composer.json` 檔案中來防止 Telescope 擴展包被 [自動發現](/docs/laravel/10.x/packages#package-discovery)：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "laravel/telescope"
        ]
    }
},
```

### 組態

發佈 Telescope 的資原始檔後，其主要組態檔案將位於 `config/telescope.php`。此組態檔案允許你組態監聽 [觀察者選項](#available-watchers)，每個組態選項都包含其用途說明，因此請務必徹底瀏覽此檔案。

如果需要，你可以使用 `enabled` 組態選項完全停用 Telescope 的資料收集：

    'enabled' => env('TELESCOPE_ENABLED', true),

### 資料修改

有了資料修改， `telescope_entries` 表可以非常快速地累積記錄。 為了緩解這個問題，你應該使用 [調度](/docs/laravel/10.x/scheduling) 每天運行 `telescope:prune` 命令：

    $schedule->command('telescope:prune')->daily();

默認情況下，將獲取超過 24 小時的所有資料。在呼叫命令時可以使用 `hours` 選項來確定保留 `Telescope` 資料的時間。例如，以下命令將刪除 48 小時前建立的所有記錄：

    $schedule->command('telescope:prune --hours=48')->daily();

### 儀表板授權

訪問 `/telescope` 即可顯示儀表盤。默認情況下，你只能在 `local` 環境中訪問此儀表板。 在 `app/Providers/TelescopeServiceProvider.php` 檔案中，有一個 [gate 授權](/docs/laravel/10.x/authorization#gates) 。此授權能控制在 **非本地** 環境中對 Telescope 的訪問。你可以根據需要隨意修改此權限以限制對 Telescope 安裝和訪問：

    use App\Models\User;

    /**
     * 註冊 Telescope gate。
     *
     * 該 gate 確定誰可以在非本地環境中訪問 Telescope
     */
    protected function gate(): void
    {
        Gate::define('viewTelescope', function (User $user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

> 注意：你應該確保在生產環境中將 `APP_ENV` 環境變數更改為 `Production`。 否則，你的 Telescope 偵錯工具將公開可用。

## 更新 Telescope

升級到 Telescope 的新主要版本時，務必仔細閱讀 [升級指南](https://github.com/laravel/telescope/blob/master/UPGRADE.).

此外，升級到任何新的 Telescope 版本時，你都應該重建 Telescope 實例：

```shell
php artisan telescope:publish
```

為了使實例保持最新狀態並避免將來的更新中出現問題，可以在應用程式的 `composer.json` 檔案中的 `post-update-cmd` 指令碼新增 `telescope:publish` 命令：

```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ]
    }
}
```

## 過濾

### 單項過濾

你可以通過在 `App\Providers\TelescopeServiceProvider` 類中定義的 `filter` 閉包來過濾 Telescope 記錄的資料。 默認情況下，此回呼會記錄 `local` 環境中的所有資料以及異常、失敗任務、工作排程和帶有受監控標記的資料：

    use Laravel\Telescope\IncomingEntry;
    use Laravel\Telescope\Telescope;

    /**
     * 註冊應用服務
     */
    public function register(): void
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filter(function (IncomingEntry $entry) {
            if ($this->app->environment('local')) {
                return true;
            }

            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->isSlowQuery() ||
                $entry->hasMonitoredTag();
        });
    }

### 批次過濾

`filter` 閉包過濾單個條目的資料， 你也可以使用 `filterBatch` 方法註冊一個閉包，該閉包過濾給定請求或控制台命令的所有資料。如果閉包返回 `true`，則所有資料都由 Telescope 記錄：

    use Illuminate\Support\Collection;
    use Laravel\Telescope\IncomingEntry;
    use Laravel\Telescope\Telescope;

    /**
     *  註冊應用服務
     */
    public function register(): void
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filterBatch(function (Collection $entries) {
            if ($this->app->environment('local')) {
                return true;
            }

            return $entries->contains(function (IncomingEntry $entry) {
                return $entry->isReportableException() ||
                    $entry->isFailedJob() ||
                    $entry->isScheduledTask() ||
                    $entry->isSlowQuery() ||
                    $entry->hasMonitoredTag();
                });
        });
    }



## 標籤

Telescope 允許你通過 「tag」 搜尋條目。通常，標籤是 Eloquent 模型的類名或經過身份驗證的使用者 ID， 這些標籤會自動新增到條目中。有時，你可能希望將自己的自訂標籤附加到條目中。 你可以使用 `Telescope::tag` 方法。 `tag` 方法接受一個閉包，該閉包應返回一個標籤陣列。返回的標籤將與 Telescope 自動附加到條目的所有標籤合併。你應該在 `App\Providers\TelescopeServiceProvider` 類中的 `register` 方法呼叫 `tag` 方法：

    use Laravel\Telescope\IncomingEntry;
    use Laravel\Telescope\Telescope;

    /**
     * 註冊應用服務
     */
    public function register(): void
    {
        $this->hideSensitiveRequestDetails();

        Telescope::tag(function (IncomingEntry $entry) {
            return $entry->type === 'request'
                        ? ['status:'.$entry->content['response_status']]
                        : [];
        });
     }

## 可用的觀察者

Telescope 「觀察者」 在執行請求或控制台命令時收集應用資料。你可以在 `config/telescope.php` 組態檔案中自訂啟用的觀察者列表：

    'watchers' => [
        Watchers\CacheWatcher::class => true,
        Watchers\CommandWatcher::class => true,
        ...
    ],

一些監視器還允許你提供額外的自訂選項：

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 100,
        ],
        ...
    ],

### 批次監視器

批次監視器記錄佇列 [批次任務](/docs/laravel/10.x/queues#job-batching) 的資訊，包括任務和連接資訊。

### 快取監視器

當快取鍵被命中、未命中、更新和刪除時，快取監視器會記錄資料。

### 命令監視器

只要執行 Artisan 命令，命令監視器就會記錄參數、選項、退出碼和輸出。如果你想排除監視器記錄的某些命令，你可以在 `config/telescope.php` 檔案的 `ignore` 選項中指定命令：

    'watchers' => [
        Watchers\CommandWatcher::class => [
            'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
            'ignore' => ['key:generate'],
        ],
        ...
    ],

### 輸出監視器

輸出監視器在 Telescope 中記錄並顯示你的變數輸出。使用 Laravel 時，可以使用全域 `dump` 函數輸出變數。必須在瀏覽器中打開資料監視器選項卡，才能進行輸出變數，否則監視器將忽略此次輸出。

### 事件監視器

事件監視器記錄應用分發的所有 [事件](/docs/laravel/10.x/events) 的有效負載、監聽器和廣播資料。事件監視器忽略了 Laravel 框架的內部事件。

### 異常監視器

異常監視器記錄應用拋出的任何可報告異常的資料和堆疊跟蹤。

### Gate（攔截）監視器

Gate 監視器記錄你的應用的 [gate 和策略](/docs/laravel/10.x/authorization) 檢查的資料和結果。如果你希望將某些屬性排除在監視器的記錄之外，你可 `config/telescope.php` 檔案的 `ignore_abilities` 選項中指定它們：

    'watchers' => [
        Watchers\GateWatcher::class => [
            'enabled' => env('TELESCOPE_GATE_WATCHER', true),
            'ignore_abilities' => ['viewNova'],
        ],
        ...
    ],

### HTTP 客戶端監視器

HTTP 客戶端監視器記錄你的應用程式發出的傳出 [HTTP 客戶端請求](/docs/laravel/10.x/http-client)。

### 任務監視器

任務監視器記錄應用程式分發的任何 [任務](/docs/laravel/10.x/queues) 的資料和狀態。

### 日誌監視器

日誌監視器記錄應用程式寫入的任何日誌的 [日誌資料](/docs/laravel/10.x/logging)。

默認情況下，Telescope 將只記錄 [錯誤] 等級及以上的日誌。但是，你可以修改應用程式的 `config/tescope.php` 組態檔案中的 `level` 選項來修改此行為：

    'watchers' => [
        Watchers\LogWatcher::class => [
            'enabled' => env('TELESCOPE_LOG_WATCHER', true),
            'level' => 'debug',
        ],

        // ...
    ],

### 郵件監視器

郵件監視器允許你查看應用傳送的 [郵件](/docs/laravel/10.x/mail) 及其相關資料的瀏覽器內預覽。你也可以將該電子郵件下載為 `.eml` 檔案。

### 模型監視器

每當調度 Eloquent 的 [模型事件](/docs/laravel/10.x/eloquent#events) 時，模型監視器就會記錄模型更改。你可以通過監視器的 `events` 選項指定應記錄哪些模型事件：

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
        ],
        ...
    ],

如果你想記錄在給定請求期間融合的模型數量，請啟用 `hydrations` 選項：

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
            'hydrations' => true,
        ],
        ...
    ],

### 消息通知監視器

消息通知監聽器記錄你的應用程式傳送的所有 [消息通知](/docs/laravel/10.x/notifications) 。如果通知觸發了電子郵件並且你啟用了郵件監聽器，則電子郵件也可以在郵件監視器螢幕上進行預覽。

### 資料查詢監視器

資料查詢監視器記錄應用程式執行的所有查詢的原始 SQL、繫結和執行時間。監視器還將任何慢於 100 毫秒的查詢標記為 `slow`。你可以使用監視器的 `slow` 選項自訂慢查詢閾值：

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 50,
        ],
        ...
    ],

### Redis 監視器

Redis 監視器記錄你的應用程式執行的所有 [Redis](/docs/laravel/10.x/redis) 命令。如果你使用 Redis 進行快取，Redis 監視器也會記錄快取命令。

### 請求監視器

請求監視器記錄與應用程式處理的任何請求相關聯的請求、要求標頭、 session 和響應資料。你可以通過 `size_limit`（以 KB 為單位）選項限制記錄的響應資料：

    'watchers' => [
        Watchers\RequestWatcher::class => [
            'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
            'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
        ],
        ...
    ],

### 定時任務監視器

定時任務監視器記錄應用程式運行的任何 [工作排程](/docs/laravel/10.x/scheduling) 的命令和輸出。

### 檢視監視器

檢視監視器記錄渲染檢視時使用的 [檢視](/docs/laravel/10.x/views) 名稱、路徑、資料和「composer」元件。

## 顯示使用者頭像

Telescope 儀表盤顯示保存給定條目時會有登錄使用者的使用者頭像。 默認情況下，Telescope 將使用 Gravatar Web 服務檢索頭像。 但是，你可以通過在 `App\Providers\TelescopeServiceProvider` 類中註冊一個回呼來自訂頭像 URL。 回呼將收到使用者的 ID 和電子郵件地址，並應返回使用者的頭像 URL：

    use App\Models\User;
    use Laravel\Telescope\Telescope;

    /**
     * 註冊應用服務
     */
    public function register(): void
    {
        // ...

        Telescope::avatar(function (string $id, string $email) {
            return '/avatars/'.User::find($id)->avatar_path;
        });
    }

