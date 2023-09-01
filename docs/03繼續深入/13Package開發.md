#  package 開發

## 介紹

 package 是向 Laravel 新增功能的主要方式。 package 可能是處理日期的好方法，例如 [Carbon](https://github.com/briannesbitt/Carbon)，也可能是允許您將檔案與 Eloquent 模型相關聯的 package ，例如 Spatie 的 [Laravel 媒體櫃](https://github.com/spatie/laravel-medialibrary)。

 package 有不同類型。有些 package 是獨立的，這意味著它們可以與任何 PHP 框架一起使用。 Carbon 和 PHPUnit 是獨立 package 的示例。這種 package 可以通過 `composer.json` 檔案引入，在 Laravel 中使用。

此外，還有一些 package 是專門用在 Laravel 中。這些 package 可能 package 含路由、 controller 、檢視和組態，專門用於增強 Laravel 應用。本教學主要涵蓋的就是這些專用於 Laravel 的 package 的開發。

### 關於 Facades

編寫 Laravel 應用時，通常使用契約（Contracts）還是門面（Facades）並不重要，因為兩者都提供了基本相同的可測試性等級。但是，在編寫 package 時， package 通常是無法使用 Laravel 的所有測試輔助函數。如果您希望能夠像將 package 安裝在典型的 Laravel 應用程式中一樣編寫 package 測試，您可以使用 [Orchestral Testbench](https://github.com/orchestral/testbench)  package 。


##  package 發現

在 Laravel 應用程式的 `config/app.php` 組態檔案中，providers 選項定義了 Laravel 應該載入的服務提供者列表。當有人安裝您的軟體 package 時，您通常希望您的服務提供者也 package 含在此列表中。 您可以在 package 的 `composer.json` 檔案的 `extra` 部分中定義提供者，而不是要求使用者手動將您的服務提供者新增到列表中。除了服務提供者外，您還可以列出您想註冊的任何 [facades](/docs/laravel/10.x/facades)：

```json
"extra": {
    "laravel": {
        "providers": [
            "Barryvdh\\Debugbar\\ServiceProvider"
        ],
        "aliases": {
            "Debugbar": "Barryvdh\\Debugbar\\Facade"
        }
    }
},
```

當你的 package 組態了 package 發現後，Laravel 會在安裝該 package 時自動註冊服務提供者及 Facades，這樣就為你的 package 使用者創造一個便利的安裝體驗。

### 退出 package 發現

如果你是 package 消費者，要停用 package 發現功能，你可以在應用的 `composer.json` 檔案的 `extra` 區域列出 package 名：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
},
```

你可以在應用的 `dont-discover` 指令中使用 `*` 字元，停用所有 package 的 package 發現功能：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
},
```

## 服務提供者

[服務提供者](/docs/laravel/10.x/providers)是你的 package 和 Laravel 之間的連接點。服務提供者負責將事物繫結到 Laravel 的[服務容器](/docs/laravel/10.x/container)並告知 Laravel 到哪裡去載入 package 資源，比如檢視、組態及語言檔案。



服務提供者擴展了 `Illuminate/Support/ServiceProvider` 類， package 含兩個方法： `register` 和 `boot`。基本的 `ServiceProvider` 類位於 `illuminate/support` Composer  package 中，你應該把它新增到你自己 package 的依賴項中。要瞭解更多關於服務提供者的結構和目的，請查看 [服務提供者](/docs/laravel/10.x/providers).

## 資源

### 組態

通常情況下，你需要將你的 package 的組態檔案發佈到應用程式的 `config` 目錄下。這將允許在使用 package 時覆蓋擴展 package 中的默認組態選項。發佈組態檔案，需要在服務提供者的 `boot` 方法中呼叫 `publishes` 方法:

    /**
     * 引導 package 服務
     */
    public function boot(): void
    {
        $this->publishes([
            __DIR__.'/../config/courier.php' => config_path('courier.php'),
        ]);
    }

使用擴展 package 的時候執行 Laravel 的 `vendor:publish` 命令, 你的檔案將被覆制到指定的發佈位置。 一旦你的組態被發佈, 它的值可以像其他的組態檔案一樣被訪問:

    $value = config('courier.option');

> **Warning**  
> 你不應該在你的組態檔案中定義閉 package 。當使用者執行 `config:cache` Artisan 命令時，它們不能被正確序列化。

#### 默認的 package 組態

你也可以將你自己的 package 的組態檔案與應用程式的發佈副本合併。這將允許你的使用者在組態檔案的發佈副本中只定義他們真正想要覆蓋的選項。要合併組態檔案的值，請使用你的服務提供者的 `register` 方法中的 `mergeConfigFrom` 方法。



`mergeConfigFrom` 方法的第一個參數為你的 package 的組態檔案的路徑，第二個參數為應用程式的組態檔案副本的名稱：

    /**
     * 註冊應用程式服務
     */
    public function register(): void
    {
        $this->mergeConfigFrom(
            __DIR__.'/../config/courier.php', 'courier'
        );
    }

> **Warning**  
> 這個方法只合併了組態陣列的第一層。如果你的使用者部分地定義了一個多維的組態陣列，缺少的選項將不會被合併。

### 路由

如果你的軟體 package  package 含路由，你可以使用 `loadRoutesFrom` 方法載入它們。這個方法會自動判斷應用程式的路由是否被快取，如果路由已經被快取，則不會載入你的路由檔案：

    /**
     * 引導 package 服務
     */
    public function boot(): void
    {
        $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
    }

### 遷移

如果你的軟體 package  package 含了 [資料庫遷移](/docs/laravel/10.x/migrations) , 你可以使用 `loadMigrationsFrom` 方法來載入它們。`loadMigrationsFrom` 方法的參數為軟體 package 遷移檔案的路徑。

    /**
     * 引導 package 服務
     */
    public function boot(): void
    {
        $this->loadMigrationsFrom(__DIR__.'/../database/migrations');
    }

一旦你的軟體 package 的遷移被註冊，當 `php artisan migrate` 命令被執行時，它們將自動被運行。你不需要把它們匯出到應用程式的 `database/migrations` 目錄中。

### 語言檔案

如果你的軟體 package  package 含 [語言檔案](/docs/laravel/10.x/localization) , 你可以使用 `loadTranslationsFrom` 方法來載入它們。 例如, 如果你的 package 被命名為 `courier` , 你應該在你的服務提供者的 `boot` 方法中加入以下內容:

    /**
     * 引導 package 服務
     */
    public function boot(): void
    {
        $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');
    }



 package 的翻譯行是使用 `package::file.line` 的語法慣例來引用的。因此，你可以這樣從 `messages` 檔案中載入 `courier`  package 的 `welcome` 行：

    echo trans('courier::messages.welcome');

#### 發佈語言檔案

如果你想把 package 的語言檔案發佈到應用程式的 `lang/vendor` 目錄，可以使用服務提供者的 `publishes` 方法。`publishes` 方法接受一個軟體 package 路徑和它們所需的發佈位置的陣列。例如，要發佈 `courier`  package 的語言檔案，你可以做以下工作：

    /**
     * 引導 package 服務
     */
    public function boot(): void
    {
        $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');

        $this->publishes([
            __DIR__.'/../lang' => $this->app->langPath('vendor/courier'),
        ]);
    }

當你的軟體 package 的使用者執行Laravel的 `vendor:publish` Artisan 命令時, 你的軟體 package 的語言檔案會被發佈到指定的發佈位置。

### 檢視

要在 Laravel 註冊你的 package 的 [檢視](/docs/laravel/10.x/views) , 你需要告訴 Laravel 這些檢視的位置. 你可以使用服務提供者的 `loadViewsFrom` 方法來完成。`loadViewsFrom` 方法接受兩個參數: 檢視範本的路徑和 package 的名稱。 例如，如果你的 package 的名字是 `courier`，你可以在服務提供者的 `boot` 方法中加入以下內容：

    /**
     * 引導 package 服務
     */
    public function boot(): void
    {
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');
    }

 package 的檢視是使用 `package::view` 的語法慣例來引用的。因此，一旦你的檢視路徑在服務提供者中註冊，你可以像這樣從 `courier`  package 中載入 `dashboard` 檢視。

    Route::get('/dashboard', function () {
        return view('courier::dashboard');
    });



#### 覆蓋 package 的檢視

當你使用 `loadViewsFrom` 方法時, Laravel 實際上為你的檢視註冊了兩個位置: 應用程式的 `resources/views/vendor` 目錄和你指定的目錄。 所以, 以 `courier`  package 為例, Laravel 首先會檢查檢視的自訂版本是否已經被開發者放在 `resources/views/vendor/courier` 目錄中。 然後, 如果檢視沒有被定製, Laravel 會搜尋你在呼叫 `loadViewsFrom` 時指定的 package 的檢視目錄. 這使得 package 的使用者可以很容易地定製/覆蓋你的 package 的檢視。

#### 發佈檢視

如果你想讓你的檢視可以發佈到應用程式的 `resources/views/vendor` 目錄下，你可以使用服務提供者的 `publishes` 方法。`publishes` 方法接受一個陣列的 package 檢視路徑和它們所需的發佈位置：

    /**
     * 引導 package 服務
     */
    public function boot(): void
    {
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');

        $this->publishes([
            __DIR__.'/../resources/views' => resource_path('views/vendor/courier'),
        ]);
    }

當你的 package 的使用者執行 Laravel 的 `vendor:publish` Artisan 命令時, 你的 package 的檢視將被覆制到指定的發佈位置。

### 檢視元件

如果你正在建立一個用 Blade 元件的 package ，或者將元件放在非傳統的目錄中，你將需要手動註冊你的元件類和它的 HTML 標籤別名，以便 Laravel 知道在哪裡可以找到這個元件。你通常應該在你的 package 的服務提供者的 `boot` 方法中註冊你的元件:

    use Illuminate\Support\Facades\Blade;
    use VendorPackage\View\Components\AlertComponent;

    /**
     * 引導你的 package 的服務
     */
    public function boot(): void
    {
        Blade::component('package-alert', AlertComponent::class);
    }



當元件註冊成功後，你就可以使用標籤別名對其進行渲染：

```blade
<x-package-alert/>
```

#### 自動載入 package 元件

此外，你可以使用 `compoentNamespace` 方法依照規範自動載入元件類。比如，`Nightshade`  package 中可能有 `Calendar` 和 `ColorPicker` 元件，存在於 `Nightshade\Views\Components` 命名空間中：

    use Illuminate\Support\Facades\Blade;

    /**
     * 啟動 package 服務
     */
    public function boot(): void
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

我們可以使用 `package-name::` 語法，通過 package 提供商的命名空間呼叫 package 元件：

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade 會通過元件名自動檢測連結到該元件的類。子目錄也支援使用'點'語法。

#### 匿名元件

如果 package 中有匿名元件，則必須將它們放在 package 的檢視目錄(由[`loadViewsFrom` 方法](#views)指定)的 `components` 資料夾下。然後，你就可以通過在元件名的前面加上 package 檢視的命名空間來對其進行渲染了：

```blade
<x-courier::alert />
```

### "About" Artisan 命令

Laravel 內建的 `about` Artisan 命令提供了應用環境和組態的摘要資訊。 package 可以通過 `AboutCommand` 類為該命令輸出新增附加資訊。一般而言，這些資訊可以在 package 服務提供者的 `boot` 方法中新增：

    use Illuminate\Foundation\Console\AboutCommand;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        AboutCommand::add('My Package', fn () => ['Version' => '1.0.0']);
    }

## 命令

要在 Laravel 中註冊你的 package 的 Artisan 命令，你可以使用 `commands` 方法。 此方法需要一個命令類名稱陣列。 註冊命令後，您可以使用 [Artisan CLI](https://learnku.com/docs/laravel/9.x/artisan) 執行它們：

    use Courier\Console\Commands\InstallCommand;
    use Courier\Console\Commands\NetworkCommand;

    /**
     * Bootstrap any package services.
     */
    public function boot(): void
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                InstallCommand::class,
                NetworkCommand::class,
            ]);
        }
    }



## 公共資源

你的 package 可能有諸如 JavaScript 、CSS 和圖片等資源。要發佈這些資源到應用程式的 `public` 目錄，請使用服務提供者的 `publishes` 方法。在下面例子中，我們還將新增一個 `public` 資源組標籤，它可以用來輕鬆發佈相關資源組：

    /**
     * 引導 package 服務
     */
    public function boot(): void
    {
        $this->publishes([
            __DIR__.'/../public' => public_path('vendor/courier'),
        ], 'public');
    }

當你的軟體 package 的使用者執行 `vendor:publish` 命令時，你的資源將被覆制到指定的發佈位置。通常使用者需要在每次更新 package 的時候都要覆蓋資源，你可以使用 `--force` 標誌。

```shell
php artisan vendor:publish --tag=public --force
```

## 發佈檔案組

你可能想單獨發佈軟體 package 的資源和資源組。例如，你可能想讓你的使用者發佈你的 package 的組態檔案，而不被強迫發佈你的 package 的資源。你可以通過在呼叫 package 的服務提供者的 `publishes` 方法時對它們進行 `tagging` 來做到這一點。例如，讓我們使用標籤在軟體 package 服務提供者的 `boot` 方法中為 `courier` 軟體 package 定義兩個發佈組（ `courier-config` 和 `courier-migrations` ）。

    /**
     * 引導 package 服務
     */
    public function boot(): void
    {
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php')
        ], 'courier-config');

        $this->publishes([
            __DIR__.'/../database/migrations/' => database_path('migrations')
        ], 'courier-migrations');
    }

現在你的使用者可以在執行 `vendor:publish` 命令時引用他們的標籤來單獨發佈這些組。

```shell
php artisan vendor:publish --tag=courier-config
```

