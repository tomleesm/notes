# Octane 加速引擎

## 簡介

[Laravel Octane](https://github.com/laravel/octane) 通過使用高性能應用程式伺服器為您的應用程式提供服務來增強您的應用程式的性能，包括 [Open Swoole](https://openswoole.com/)，[Swoole](https://github.com/swoole/swoole-src)，和 [RoadRunner](https://roadrunner.dev)。Octane 啟動您的應用程式一次，將其保存在記憶體中，然後以極快的速度向它提供請求。

## 安裝

Octane 可以通過 Composer 包管理器安裝：

```shell
composer require laravel/octane
```

安裝 Octane 後，您可以執行 `octane:install` 命令，該命令會將 Octane 的組態檔案安裝到您的應用程式中：

```shell
php artisan octane:install
```

## 伺服器先決條件

> **注意**
> Laravel Octane 需要 [PHP 8.1+](https://php.net/releases/).

### RoadRunner

[RoadRunner](https://roadrunner.dev) 由使用 Go 建構的 RoadRunner 二進制檔案提供支援。當您第一次啟動基於 RoadRunner 的 Octane 伺服器時，Octane 將為您提供下載和安裝 RoadRunner 二進制檔案。

#### 通過 Laravel Sail 使用 RoadRunner

如果你打算使用 [Laravel Sail](/docs/laravel/10.x/sail) 開發應用，你應該運行如下命令安裝 Octane 和 RoadRunner:

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane spiral/roadrunner
```

接下來，你應該啟動一個 Sail Shell，並運行 `rr` 可執行檔案檢索基於 Linux 的最新版 RoadRunner 二進制檔案：

```shell
./vendor/bin/sail shell

# Within the Sail shell...
./vendor/bin/rr get-binary
```

安裝完 RoadRunner 二進制檔案後，你可以退出 Sail Shell  session 。然後，需要調整 Sail 用來保持應用運行的 `supervisor.conf` 檔案。首先，請執行 `sail:publish` Artisan 命令：

```shell
./vendor/bin/sail artisan sail:publish
```

接著，更新應用 `docker/supervisord.confd` 檔案中的 `command` 指令，這樣 Sail 就可以使用 Octane 作為伺服器，而非 PHP 開發伺服器，運行服務了：

```ini
command=/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=roadrunner --host=0.0.0.0 --rpc-port=6001 --port=80
```

最後，請確保 `rr` 二進制檔案是可執行的並重新建構 Sail 鏡像：

```shell
chmod +x ./rr

./vendor/bin/sail build --no-cache
```

### Swoole

如果你打算使用 Swoole 伺服器來運行 Laravel Octane 應用，你必須安裝 Swoole PHP 元件。通常可以通過 PECL 安裝：

```shell
pecl install swoole
```

#### Open Swoole

如果你想要使用 Open Swoole 伺服器運行 Laravel Octane 應用，你必須安裝 Open Swoole PHP 擴展。通常可以通過 PECL 完成安裝：

```shell
pecl install openswoole
```

通過 Open Swoole 使用 Laravel Octane，可以獲得 Swoole 提供的相同功能，如並行任務，計時和間隔。

#### 通過 Laravel Sail 使用 Swoole

> **注意**
> 在通過 Sail 提供 Octane 應用程式之前，請確保你使用的是最新版本的 Laravel Sail 並在應用程式的根目錄中執行 `./vendor/bin/sail build --no-cache`。

你可以使用 Laravel 的官方 Docker 開發環境 [Laravel Sail](/docs/laravel/10.x/sail) 開發基於 Swoole 的 Octane 應用程式。 Laravel Sail 默認包含 Swoole 擴展。但是，你仍然需要調整 Sail 使用的 `supervisor.conf` 件以保持應用運行。首先，執行 `sail:publish` Artisan 命令：

```shell
./vendor/bin/sail artisan sail:publish
```

接下來，更新應用程式的 `docker/supervisord.conf` 檔案的 `command` 指令，使得 Sail 使用 Octane 替代 PHP 開發伺服器：

```ini
command=/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port=80
```

最後，建構你的 Sail 鏡像：

```shell
./vendor/bin/sail build --no-cache
```

#### Swoole 組態

Swoole 支援一些額外的組態選項，如果需要，你可以將它們新增到你的 `octane` 組態檔案中。因為它們很少需要修改，所以這些選項不包含在默認組態檔案中：

```php
'swoole' => [
    'options' => [
        'log_file' => storage_path('logs/swoole_http.log'),
        'package_max_length' => 10 * 1024 * 1024,
    ],
],
```

## 為應用程式提供服務

Octane 伺服器可以通過 `octane:start`  Artisan 命令啟動。此命令將使用由應用程式的 `octane` 組態檔案的 `server` 組態選項指定的伺服器：

```shell
php artisan octane:start
```

默認情況下，Octane 將在 8000 連接埠上啟動伺服器（可組態），因此你可以在 Web 瀏覽器中通過 `http://localhost:8000` 訪問你的應用程式。

### 通過 HTTPS 為應用程式提供服務

默認情況下，通過 Octane 運行的應用程式會生成以 `http://` 為前綴的連結。當使用 HTTPS 時，可將在應用的 `config/octane.php` 組態檔案中使用的 `OCTANE_HTTPS` 環境變數設定為 `true`。當此組態值設定為 `true` 時，Octane 將指示 Laravel 在所有生成的連結前加上 `https://`：

```php
'https' => env('OCTANE_HTTPS', false),
```

### 通過 Nginx 為應用提供服務

> **提示**
> 如果你還沒有準備好管理自己的伺服器組態，或者不習慣組態運行健壯的 Laravel Octane 應用所需的所有各種服務，請查看 [Laravel Forge](https://forge.laravel.com)。

在生產環境中，你應該在傳統 Web 伺服器（例如 Nginx 或 Apache）之後為 Octane 應用提供服務。 這樣做將允許 Web 伺服器為你的靜態資源（例如圖片和樣式表）提供服務，並管理 SSL 證書。

在下面的 Nginx 組態示例檔案中，Nginx 將向在連接埠 8000 上運行的 Octane 伺服器提供站點的靜態資源和代理請求：

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    listen [::]:80;
    server_name domain.com;
    server_tokens off;
    root /home/forge/domain.com/public;

    index index.php;

    charset utf-8;

    location /index.php {
        try_files /not_exists @octane;
    }

    location / {
        try_files $uri $uri/ @octane;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;

    error_page 404 /index.php;

    location @octane {
        set $suffix "";

        if ($uri = /index.php) {
            set $suffix ?$query_string;
        }

        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_pass http://127.0.0.1:8000$suffix;
    }
}
```

### 監視檔案更改

由於 Octane 伺服器啟動時應用程式被載入到記憶體中一次，因此對應用程式檔案的任何更改都不會在您刷新瀏覽器時反映出來。例如，新增到 `routes/web.php` 檔案的路由定義在伺服器重新啟動之前不會反映出來。為了方便起見，你可以使用 `--watch` 標誌指示 Octane 在應用程式中的任何檔案更改時自動重新啟動伺服器：

```shell
php artisan octane:start --watch
```

在使用此功能之前，您應該確保在本地開發環境中安裝了 [Node](https://nodejs.org/)。此外，你還應該在項目中安裝 [Chokidar](https://github.com/paulmillr/chokidar) 檔案監視庫：

```shell
npm install --save-dev chokidar
```

你可以使用應用程式的 `config/octane.php` 組態檔案中的 `watch` 組態選項來組態應該被監視的目錄和檔案。

### 指定工作處理程序數

默認情況下，Octane 會為機器提供的每個 CPU 核心啟動一個應用程式請求工作處理程序。這些工作處理程序將用於在進入應用程式時服務傳入的 HTTP 請求。你可以使用 `--workers` 選項手動指定要啟動的工作處理程序數量，當呼叫 `octane:start` 命令時：

```shell
php artisan octane:start --workers=4
```

如果你使用 Swoole 應用程式伺服器，則還可以指定要啟動的任務工作處理程序數量：

```shell
php artisan octane:start --workers=4 --task-workers=6
```

### 指定最大請求數量

為了防止記憶體洩漏，Octane 在處理完 500 個請求後會優雅地重新啟動任何 worker。要調整這個數字，你可以使用 `--max-requests` 選項：

```shell
php artisan octane:start --max-requests=250
```

### 多載 Workers

你可以使用 `octane:reload` 命令優雅地重新啟動 Octane 伺服器的應用 workers。通常，這應該在部署後完成，以便將新部署的程式碼載入到記憶體中並用於為後續請求提供服務：

```shell
php artisan octane:reload
```

### 停止伺服器

你可以使用  `octane:stop` Artisan 命令停止 Octane 伺服器：

```shell
php artisan octane:stop
```

#### 檢查伺服器狀態

你可以使用 `octane:status` Artisan 命令檢查 Octane 伺服器的當前狀態：

```shell
php artisan octane:status
```

## 依賴注入和 Octane

由於 Octane 只啟動你的應用程式一次，並在服務請求時將其保留在記憶體中，所以在建構你的應用程式時，你應該考慮一些注意事項。例如，你的應用程式的服務提供者的 `register` 和 `boot` 方法將只在 request worker 最初啟動時執行一次。在隨後的請求中，將重用相同的應用程式實例。

鑑於這個機制，在將應用服務容器或請求注入任何對象的建構函式時應特別小心。這樣一來，該對像在隨後的請求中就可能有一個穩定版本的容器或請求。

Octane 會在兩次請求之間自動處理重設任何第一方框架的狀態。然而，Octane 並不總是知道如何重設由你的應用程式建立的全域狀態。因此，你應該知道如何以一種對 Octane 友好的方式來建構你的應用程式。下面，我們將討論在使用 Octane 時可能引起問題的最常見情況。

### 容器注入

通常來說，你應該避免將應用服務容器或 HTTP 請求實例注入到其他對象的建構函式中。例如，下面的繫結將整個應用服務容器注入到繫結為單例的對象中：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app);
    });
}
```

在這個例子中，如果在應用程式引導過程中解析 `Service` 實例，容器將被注入到該服務中，並且該容器將在後續的請求中保留。這對於你的特定應用程式**可能**不是一個問題，但是它可能會導致容器意外地缺少後來在引導過程中新增的繫結或後續請求中新增的繫結。

為瞭解決這個問題，你可以停止將繫結註冊為單例，或者你可以將一個容器解析器閉包注入到服務中，該閉包總是解析當前的容器實例：

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app);
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance());
});
```

全域的 `app` 輔助函數和 `Container::getInstance()` 方法將始終返回應用程式容器的最新版本。

### 請求注入

通常來說，你應該避免將應用服務容器或 HTTP 請求實例注入到其他對象的建構函式中。例如，下面的繫結將整個請求實例注入到繫結為單例的對象中：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app['request']);
    });
}
```

在這個例子中，如果在應用程式啟動過程中解析 `Service` 實例，則會將 HTTP 請求注入到服務中，並且相同的請求將由 `Service` 實例保持在後續請求中。因此，所有標頭、輸入和查詢字串資料以及所有其他請求資料都將不正確。

為瞭解決這個問題，你可以停止將繫結註冊為單例，或者你可以將請求解析器閉包注入到服務中，該閉包始終解析當前請求實例。或者，最推薦的方法是在執行階段將對象所需的特定請求資訊傳遞給對象的方法之一：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app['request']);
});

$this->app->singleton(Service::class, function (Application $app) {
    return new Service(fn () => $app['request']);
});

// Or...

$service->method($request->input('name'));
```

全域的 `request` 幫助函數將始終返回應用程式當前處理的請求，因此可以在應用程式中安全使用它。

> **警告**
> 在 controller 方法和路由閉包中類型提示 Illuminate\Http\Request 實例是可以接受的。

### 組態庫注入

一般來說，你應該避免將組態庫實例注入到其他對象的建構函式中。例如，以下繫結將組態庫注入到繫結為單例的對象中：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app->make('config'));
    });
}
```

在這個示例中，如果在請求之間的組態值更改了，那麼這個服務將無法訪問新的值，因為它依賴於原始儲存庫實例。

作為解決方法，你可以停止將繫結註冊為單例，或者將組態儲存庫解析器閉包注入到類中：

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app->make('config'));
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance()->make('config'));
});
```

全域 `config` 將始終返回組態儲存庫的最新版本，因此在應用程式中使用是安全的。

### 管理記憶體洩漏

請記住，Octane 在請求之間保留應用程式，因此將資料新增到靜態維護的陣列中將導致記憶體洩漏。例如，以下 controller 具有記憶體洩漏，因為對應用程式的每個請求將繼續向靜態的 `$data` 陣列新增資料：

```php
use App\Service;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

/**
 * 處理傳入的請求。
 */
public function index(Request $request): array
{
    Service::$data[] = Str::random(10);

    return [
        // ...
    ];
}
```

在建構應用程式時，你應特別注意避免建立此類記憶體洩漏。建議在本地開發期間監視應用程式的記憶體使用情況，以確保您不會在應用程式中引入新的記憶體洩漏。

## 並行任務

> **警告**
> 此功能需要 [Swoole](#swoole)。



當使用 Swoole 時，你可以通過輕量級的後台任務並行執行操作。你可以使用 Octane 的 `concurrently` 方法實現此目的。你可以將此方法與 PHP 陣列解構結合使用，以檢索每個操作的結果：

```php
use App\User;
use App\Server;
use Laravel\Octane\Facades\Octane;

[$users, $servers] = Octane::concurrently([
    fn () => User::all(),
    fn () => Server::all(),
]);
```

由 Octane 處理的並行任務利用 Swoole 的 「task workers」 並在與傳入請求完全不同的處理程序中執行。可用於處理並行任務的工作程序的數量由 `octane:start` 命令的 `--task-workers` 指令確定：

```shell
php artisan octane:start --workers=4 --task-workers=6
```

在呼叫 `concurrently` 方法時，你應該不要提供超過 1024 個任務，因為 Swoole 任務系統強制執行此限制。

## 刻度和間隔

> **警告**
> 此功能需要 [Swoole](#swoole).

當使用 Swoole 時，你可以註冊定期執行的 「tick」 操作。你可以通過 `tick` 方法註冊 「tick」 回呼函數。提供給 `tick` 方法的第一個參數應該是一個字串，表示定時器的名稱。第二個參數應該是在指定間隔內呼叫的可呼叫對象。

在此示例中，我們將註冊一個閉包，每 10 秒呼叫一次。通常，`tick` 方法應該在你應用程式的任何服務提供程序的 `boot` 方法中呼叫：

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
        ->seconds(10);
```

使用 `immediate` 方法，你可以指示 Octane 在 Octane 伺服器初始啟動時立即呼叫 tick 回呼，並在 N 秒後每次呼叫：

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
        ->seconds(10)
        ->immediate();
```

## Octane 快取

> **警告**
> 此功能需要 [Swoole](#swoole).

使用 Swoole 時，你可以利用 Octane 快取驅動程式，該驅動程式提供每秒高達 200 萬次的讀寫速度。因此，這個快取驅動程式是需要從快取層中獲得極高讀寫速度的應用程式的絕佳選擇。

該快取驅動程式由 [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table) 驅動。快取中的所有資料可供伺服器上的所有工作處理程序訪問。但是，當伺服器重新啟動時，快取資料將被清除：

```php
Cache::store('octane')->put('framework', 'Laravel', 30);
```

> **注意**
> Octane 快取中允許的最大條目數可以在您的應用程式的 octane 組態檔案中定義。

### 快取間隔

除了 Laravel 快取系統提供的典型方法外，Octane 快取驅動程式還提供了基於間隔的快取。這些快取會在指定的間隔自動刷新，並應在一個應用程式服務提供程序的 `boot` 方法中註冊。例如，以下快取將每五秒刷新一次：

```php
use Illuminate\Support\Str;

Cache::store('octane')->interval('random', function () {
    return Str::random(10);
}, seconds: 5);
```

## 表格

> **警告**
> 此功能需要 [Swoole](#swoole).

使用 Swoole 時，你可以定義和與自己的任意 [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table) 進行互動。Swoole tables 提供極高的性能吞吐量，並且可以通過伺服器上的所有工作處理程序訪問其中的資料。但是，當它們內部的資料在伺服器重新啟動時將被丟失。


表在應用 `octane` 組態檔案 `tables` 陣列組態中設定。最大運行 1000 行的示例表已經組態。像下面這樣，字串行支援的最大長度在列類型後面設定：

```php
'tables' => [
    'example:1000' => [
        'name' => 'string:1000',
        'votes' => 'int',
    ],
],
```

通過 `Octane::table` 方法訪問表：

```php
use Laravel\Octane\Facades\Octane;

Octane::table('example')->set('uuid', [
    'name' => 'Nuno Maduro',
    'votes' => 1000,
]);

return Octane::table('example')->get('uuid');
```

> **注意**
> Swoole table 支援的列類型有： `string` ，`int` 和 `float` 。

