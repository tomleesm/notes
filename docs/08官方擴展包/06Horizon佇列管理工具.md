# Horizon 佇列管理工具

## 介紹

> **提示**  
> 在深入瞭解 Laravel Horizon 之前，您應該熟悉 Laravel 的基礎 [佇列服務](/docs/laravel/10.x/queues)。 Horizon 為 Laravel 的佇列增加了額外的功能，如果你還不熟悉 Laravel 提供的基本佇列功能，這些功能可能會讓人感到困惑。

[Laravel Horizon](https://github.com/laravel/horizon) 為你的 Laravel [Redis queues](/docs/laravel/10.x/queues).提供了一個美觀的儀表盤和程式碼驅動的組態。它可以方便的監控佇列系統的關鍵指標：任務吞吐量、執行階段間、作業失敗情況。

在使用 Horizon 時，所有佇列的 worker 組態都儲存在一個簡單的組態檔案中。通過在受版本控制的檔案中定義應用程式的 worker 組態，你可以在部署應用程式時輕鬆擴展或修改應用程式的佇列 worker。

## 安裝

> **注意**
> Laravel Horizon 要求你使用 [Redis](https://redis.io) 來為你的佇列服務。因此，你應該確保在應用程式的 `config/queue.php` 組態檔案中將佇列連接設定為 `redis`。

你可以使用 Composer 將 Horizon 安裝到你的 Laravel 項目裡：

```shell
composer require laravel/horizon
```

Horizon 安裝之後，使用 `horizon:install` Artisan 命令發佈資源：

```shell
php artisan horizon:install
```

### 組態

Horizon 資源發佈之後，其主要組態檔案會被分配到 `config/horizon.php` 檔案。可以用這個組態檔案組態工作選項，每個組態選項包含一個用途描述，請務必仔細研究這個檔案。


>**注意：**Horizon 在內部使用名為 `horizon` 的 Redis 連接。此 Redis 連接名稱是保留的，不應分配給 `database.php` 組態檔案中的另一個 Redis 連接或作為 `horizon.php` 組態檔案中的 `use` 選項的值。

#### 環境組態

安裝後，你需要熟悉的重點 Horizon 組態選項是 `environments` 組態選項。此組態選項定義了你的應用程式運行的一系列環境，並為每個環境定義了工作處理程序選項。默認情況下，此條目包含　`生產 (production)`　和 `本地 (local)`環境。簡而言之，你可以根據自己的需要自由新增更多環境：

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
            ],
        ],

        'local' => [
            'supervisor-1' => [
                'maxProcesses' => 3,
            ],
        ],
    ],

當你啟動 Horizon 時，它將使用指定應用程式運行環境所組態的 worker 處理程序選項。通常，環境組態由 `APP_ENV` [環境變數](/docs/laravel/10.x/configuration#determining-the-current-environment) 的值確定。例如，默認的 `local` Horizon 環境組態為啟動三個工作處理程序，並自動平衡分配給每個佇列的工作處理程序數量。默認的「生產」環境組態為最多啟動 10 個 worker 處理程序，並自動平衡分配給每個佇列的 worker 處理程序數量。

> **注意：**你應該確保你的 `horizon` 組態檔案的 `environments` 部分包含計畫在其上運行 Horizon 的每個 [環境](/docs/laravel/10.x/configuration#environment-configuration) 的組態。

#### Supervisors

正如你在 Horizon 的默認組態檔案中看到的那樣。每個環境可以包含一個或多個 Supervisor 組態。默認情況下，組態檔案將這個Supervisor 定義為 `supervisor-1`；但是，你可以隨意命名你的 Supervisor。每個 Supervisor 負責監督一組 worker，並負責平衡佇列之間的 worker。

如果你想定義一組在指定環境中運行的新 worker，可以向相應的環境新增額外的 Supervisor。如果你想為應用程式使用的特定佇列定義不同的平衡策略或 worker 數量，也可以選擇這樣做。

#### 預設值

在 Horizon 的默認組態檔案中，你會注意到一個 `defaults` 組態選項。這個組態選項指定應用程式的 [supervisors](#supervisors) 的預設值。Supervisor 的默認組態值將合併到每個環境的 Supervisor  組態中，讓你在定義 Supervisor 時避免不必要的重複工作。

### 均衡策略

與 Laravel 的默認佇列系統不同，Horizon 允許你從三個平衡策略中進行選擇：`simple`， `auto`， 和 `false`。`simple` 策略是組態檔案的默認選項，它會在處理程序之間平均分配進入的任務：

    'balance' => 'simple',

`auto` 策略根據佇列的當前工作負載來調整每個佇列的工作處理程序數量。舉個例子，如果你的 `notifications` 佇列有 1000 個等待的任務，而你的 `render` 佇列是空的，那麼 Horizon 將為 `notifications` 佇列分配更多的工作執行緒，直到佇列為空。

當使用 `auto` 策略時，你可以定義 `minProcesses` 和 `maxProcesses` 的組態選項來控制 Horizon  擴展處理程序的最小和最大數量：

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['default'],
                'balance' => 'auto',
                'autoScalingStrategy' => 'time',
                'minProcesses' => 1,
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
                'tries' => 3,
            ],
        ],
    ],

`autoScalingStrategy` 組態值決定了 Horizon 是根據清除佇列所需的總時間（`time` 策略）還是根據佇列上的作業總數（`size` 策略）來為佇列分配更多的Worker 處理程序。


`balanceMaxShift` 和 `balanceCooldown` 組態項可以確定 Horizon 將以多快的速度擴展處理程序，在上面的示例中，每 3 秒鐘最多建立或銷毀一個新處理程序，你可以根據應用程式的需要隨意調整這些值。

當 `balance` 選項設定為 `false` 時，將使用默認的 Laravel 行為，它按照佇列在組態中列出的順序處理佇列。

### 控制面板授權

Horizon 在 `/horizon` 上顯示了一個控制面板。默認情況下，你只能在 `local` 環境中訪問這個面板。在你的 `app/Providers/HorizonServiceProvider.php` 檔案中，有一個 [授權攔截器（Gates）](/docs/laravel/10.x/authorization#gates) 的方法定義，該攔截器用於控制在**非本地**環境中對 Horizon 的訪問。末可以根據需要修改此方法，來限制對 Horizon 的訪問：

    /**
     * 註冊 Horizon 授權
     *
     * 此方法決定了誰可以在非本地環境中訪問 Horizon
     */
    protected function gate(): void
    {
        Gate::define('viewHorizon', function (User $user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

#### 可替代的身份驗證策略

需要留意的是，Laravel 會自動將經過認證的使用者注入到攔截器（Gate）閉包中。如果你的應用程式通過其他方法（例如 IP 限制）提供 Horizon 安全性保障，那麼你訪問 Horizon 使用者可能不需要實現這個「登錄」動作。因此，你需要將上面的 `function ($user)` 更改為 `function ($user = null)` 以強制 Laravel 跳過身份驗證。

### 靜默作業

有時，你可能對查看某些由你的應用程式或第三方軟體包發出的工作不感興趣。與其讓這些作業在你的「已完成作業」列表中佔用空間，你可以讓它們靜默。要開始的話，在你的應用程式的 `horizon` 組態檔案中的 `silenced` 組態選項中新增作業的類名。

    'silenced' => [
        App\Jobs\ProcessPodcast::class,
    ],

或者，你希望靜默的作業可以實現 `Laravel\Horizon\Contracts\Silenced` 介面。如果一個作業實現了這個介面，它將自動被靜默，即使它不在 `silenced` 組態陣列中。

    use Laravel\Horizon\Contracts\Silenced;

    class ProcessPodcast implements ShouldQueue, Silenced
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        // ...
    }

## 升級 Horizon

當你升級到 Horizon 的一個新的主要版本時，你需要仔細閱讀 [升級指南](https://github.com/laravel/horizon/blob/master/UPGRADE.)。

此外，升級到新的 Horizon 版本時，你應該重新發佈 Horizon 資源：

```shell
php artisan horizon:publish
```

為了使資原始檔保持最新並避免以後的更新中出現問題，你可以將以下  `horizon:publish`  命令新增到 `composer.json` 檔案中的 `post-update-cmd` 指令碼中：

```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ]
    }
}
```

## 運行 Horizon

在 `config/horizon.php` 中組態了你的 workers 之後，你可以使用 `horizon` Artisan 命令啟動 Horizon。只需這一個命令你就可以啟動你的所有已組態的 workers：

```shell
php artisan horizon
```

你可以暫停 Horizon 處理程序，並使用 `horizon:pause` 和 `horizon:continue` Artisan 命令指示它繼續處理任務：

```shell
php artisan horizon:pause

php artisan horizon:continue
```

你還可以使用 `horizon:pause-supervisor` 和 `horizon:continue-supervisor` Artisan 命令暫停和繼續指定的 Horizon [supervisors](#supervisors)：

```shell
php artisan horizon:pause-supervisor supervisor-1

php artisan horizon:continue-supervisor supervisor-1
```

你可以使用 `horizon:status` Artisan 命令檢查 Horizon 處理程序的當前狀態：

```shell
php artisan horizon:status
```

你可以使用 `horizon:terminate` Artisan 命令優雅地終止機器上的主 Horizon 處理程序。Horizon 會等當前正在處理的所有任務都完成後退出：

```shell
php artisan horizon:terminate
```

### 部署 Horizon

如果要將 Horizon 部署到一個正在運行的伺服器上，應該組態一個處理程序監視器來監視 `php artisan horizon` 命令，並在它意外退出時重新啟動它。

在將新程式碼部署到伺服器時，你需要終止 Horizon 主處理程序，以便處理程序監視器重新啟動它並接收程式碼的更改。

```shell
php artisan horizon:terminate
```

#### 安裝 Supervisor

Supervisor 是一個用於 Linux 作業系統的處理程序監視器。如果 `Horizon` 處理程序被退出或終止，Supervisor 將自動重啟你的 `Horizon` 處理程序。如果要在 Ubuntu 上安裝 Supervisor，你可以使用以下命令。如果你不使用 Ubuntu，也可以使用作業系統的包管理器安裝 Supervisor：

```shell
sudo apt-get install supervisor
```

> **技巧：**如果你覺得自己組態 Supervisor 難如登天，可以考慮使用 [Laravel Forge](https://forge.laravel.com)，它將自動為你的 Laravel 項目安裝和組態 Supervisor。

#### Supervisor 組態

Supervisor 組態檔案通常儲存在 `/etc/supervisor/conf.d` 目錄下。在此目錄中，你可以建立任意數量的組態檔案，這些組態檔案會告訴 supervisor 如何監視你的處理程序。例如，讓我們建立一個 `horizon.conf` 檔案，它啟動並監視一個 `horizon` 處理程序：

```ini
[program:horizon]
process_name=%(program_name)s
command=php /home/forge/example.com/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/horizon.log
stopwaitsecs=3600
```

在定義 Supervisor 組態時，你應該確保 `stopwaitsecs` 的值大於最長運行作業所消耗的秒數。否則，Supervisor 可能會在作業處理完之前就將其殺死。

> **注意：**雖然上面的例子對基於Ubuntu的伺服器有效，但其他伺服器作業系統對監督員組態檔案的位置和副檔名可能有所不同。請查閱你的伺服器的文件以瞭解更多資訊。

#### 啟動 Supervisor

建立了組態檔案後，可以使用以下命令更新 Supervisor 組態並啟動處理程序：

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start horizon
```

> **技巧：**關於 Supervisor 的更多資訊，可以查閱 [Supervisor 文件](http://supervisord.org/index.html)。

## 標記 (Tags)

Horizon 允許你將 `tags` 分配給任務，包括郵件、事件廣播、通知和排隊的事件監聽器。實際上，Horizon 會根據附加到作業上的有 Eloquent 模型，智能地、自動地標記大多數任務。例如，看看下面的任務：

    <?php

    namespace App\Jobs;

    use App\Models\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;

    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * 建立一個新的任務實例
         */
        public function __construct(
            public Video $video,
        ) {}

        /**
         * 執行任務
         */
        public function handle(): void
        {
            // ...
        }
    }

如果此任務與 `App\Models\Video` 實例一起排隊，且該實例的 `id` 為 `1`，則該作業將自動接收 `App\Models\Video:1` 標記。這是因為 Horizon 將為任何有 Eloquent 的模型檢查任務的屬性。如果找到了有 Eloquent 的模型，Horizon 將智能地使用模型的類名和主鍵標記任務：

    use App\Jobs\RenderVideo;
    use App\Models\Video;

    $video = Video::find(1);

    RenderVideo::dispatch($video);

#### 手動標記作業

如果你想手動定義你的一個佇列對象的標籤，你可以在類上定義一個 `tags` 方法：

    class RenderVideo implements ShouldQueue
    {
        /**
         * 獲取應該分配給任務的標記
         *
         * @return array<int, string>
         */
        public function tags(): array
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

## 通知

> **注意：** 當組態 Horizon 傳送 Slack 或 SMS 通知時，你應該查看 [相關通知驅動程式的先決條件](/docs/laravel/10.x/notifications)。

如果你希望在一個佇列有較長的等待時間時得到通知，你可以使用 `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, 和 `Horizon::routeSmsNotificationsTo` 方法。你可以通過應用程式的 `App\Providers\HorizonServiceProvider` 中的 `boot` 方法來呼叫這些方法：


    /**
     * 服務引導
     */
    public function boot(): void
    {
        parent::boot();

        Horizon::routeSmsNotificationsTo('15556667777');
        Horizon::routeMailNotificationsTo('example@example.com');
        Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    }

#### 組態通知等待時間閾值

你可以在 `config/horizon.php` 的組態檔案中組態多少秒算是「長等待」。你可以用該檔案中的 `waits` 組態選項控制每個 連接 / 佇列 組合的長等待閾值：

    'waits' => [
        'redis:default' => 60,
        'redis:critical,high' => 90,
    ],

## 指標

Horizon 有一個指標控制面板，它提供了任務和佇列的等待時間和吞吐量等資訊。要讓這些資訊顯示在這個控制面板上，你應該組態 Horizon 的 `snapshot` Artisan 命令，通過你的應用程式的 [調度器](/docs/laravel/10.x/scheduling) 每五分鐘運行一次：

    /**
     * 定義應用程式的命令調度
     */
    protected function schedule(Schedule $schedule): void
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }

## 刪除失敗的作業

如果你想刪除失敗的作業，可以使用 `horizon:forget` 命令。 `horizon:forget` 命令接受失敗作業的 ID 或 UUID 作為其唯一參數：

```shell
php artisan horizon:forget 5
```

## 從佇列中清除作業

如果你想從應用程式的默認佇列中刪除所有作業，你可以使用 `horizon:clear` Artisan 命令執行此操作：

```shell
php artisan horizon:clear
```

你可以設定 `queue` 選項來從特定佇列中刪除作業：

```shell
php artisan horizon:clear --queue=emails
```

