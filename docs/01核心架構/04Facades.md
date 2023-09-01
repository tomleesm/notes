# Facades

## 簡介

在整個 Laravel 文件中，你將看到通過 Facades 與 Laravel 特性互動的程式碼示例。Facades 為應用程式的[服務容器](/docs/laravel/10.x/container)中可用的類提供了「靜態代理」。在 Laravel 這艘船上有許多 Facades，提供了幾乎所有 Laravel 的特徵。

Laravel Facades 充當服務容器中底層類的「靜態代理」，提供簡潔、富有表現力的好處，同時保持比傳統靜態方法更多的可測試性和靈活性。如果你不完全理解引擎蓋下的 Facades 是如何工作的，那也沒問題，跟著流程走，繼續學習 Laravel。

Laravel 的所有 Facades 都在`Illuminate\Support\Facades`命名空間中定義。因此，我們可以很容易地訪問這樣一個 Facades ：

    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\Facades\Route;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

在整個 Laravel 文件中，許多示例將使用 Facades 來演示框架的各種特性。

#### 輔助函數

為了補充 Facades，Laravel 提供了各種全域 「助手函數」，使它更容易與常見的 Laravel 功能進行互動。可以與之互動的一些常用助手函數有`view`, `response`, `url`, `config`等。Laravel 提供的每個助手函數都有相應的特性；但是，在專用的[輔助函數文件](/docs/laravel/10.x/helpers)中有一個完整的列表。

例如，我們可以使用 `response` 函數而不是 `Illuminate\Support\Facades\Response` Facade 生成 JSON 響應。由於「助手函數」是全域可用的，因此無需匯入任何類即可使用它們：

```php
use Illuminate\Support\Facades\Response;

Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});

Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

## 何時使用 Facades

Facades 有很多好處。它們提供了簡潔、易記的語法，讓你可以使用 Laravel 的功能而不必記住必須手動注入或組態的長類名。此外，由於它們獨特地使用了 PHP 的動態方法，因此它們易於測試。

然而，在使用 Facades 時必須小心。Facades 的主要危險是類的「範疇洩漏」。由於 Facades 如此易於使用並且不需要注入，因此讓你的類繼續增長並在單個類中使用許多 Facades 可能很容易。使用依賴注入，這種潛在問題通過建構函式變得明顯，告訴你的類過於龐大。因此，在使用 Facades 時，需要特別關注類的大小，以便它的責任範圍保持狹窄。如果你的類變得太大，請考慮將它拆分成多個較小的類。

### Facades 與 依賴注入

依賴注入的主要好處之一是能夠替換注入類的實現。這在測試期間很有用，因為你可以注入一個模擬或存根並斷言各種方法是否在存根上呼叫了。

通常，真正的靜態方法是不可能 mock 或 stub 的。無論如何，由於 Facades 使用動態方法對服務容器中解析出來的對象方法的呼叫進行了代理， 我們也可以像測試注入類實例一樣測試 Facades。比如，像下面的路由：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

使用 Laravel 的 Facade 測試方法，我們可以編寫以下測試用例來驗證是否 Cache::get 使用我們期望的參數呼叫了該方法：

    use Illuminate\Support\Facades\Cache;

    /**
     *  一個基礎功能的測試用例
     */
    public function test_basic_example(): void
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }

### Facades Vs 助手函數

除了 Facades，Laravel 還包含各種「輔助函數」來實現這些常用功能，比如生成檢視、觸發事件、任務調度或者傳送 HTTP 響應。許多輔助函數都有與之對應的 Facade。例如，下面這個 Facades 和輔助函數的作用是一樣的：

    return Illuminate\Support\Facades\View::make('profile');

    return view('profile');

Facades 和輔助函數之間沒有實際的區別。 當你使用輔助函數時，你可以像測試相應的 Facade 那樣進行測試。例如，下面的路由：

    Route::get('/cache', function () {
        return cache('key');
    });

在底層實現，輔助函數 cache 實際是呼叫 Cache 這個 Facade 的 get 方法。因此，儘管我們使用的是輔助函數，我們依然可以帶上我們期望的參數編寫下面的測試程式碼來驗證該方法：

    use Illuminate\Support\Facades\Cache;

    /**
     * 一個基礎功能的測試用例
     */
    public function test_basic_example(): void
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }



## Facades 工作原理

在 Laravel 應用程式中，Facades 是一個提供從容器訪問對象的類。完成這項工作的部分屬於 `Facade` 類。Laravel 的 Facade、以及你建立的任何自訂 Facade，都繼承自 `Illuminate\Support\Facades\Facade` 類。

`Facade` 基類使用 `__callStatic()` 魔術方法將來自 Facade 的呼叫推遲到從容器解析出對象後。在下面的示例中，呼叫了 Laravel 快取系統。看一眼這段程式碼，人們可能會假設靜態的 `get` 方法正在 `Cache` 類上被呼叫：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         */
        public function showProfile(string $id): View
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

請注意，在檔案頂部附近，我們正在「匯入」`Cache` Facade。這個 Facade 作為訪問 `Illuminate\Contracts\Cache\Factory` 介面底層實現的代理。我們使用 Facade 進行的任何呼叫都將傳遞給 Laravel 快取服務的底層實例。

如果我們查看 `Illuminate\Support\Facades\Cache` 類，你會發現沒有靜態方法 `get`：

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         */
        protected static function getFacadeAccessor(): string
        {
            return 'cache';
        }
    }

相反，`Cache` Facade 繼承了 `Facade` 基類並定義了 `getFacadeAccessor()` 方法。此方法的工作是返回服務容器繫結的名稱。當使用者引用 `Cache` Facade 上的任何靜態方法時，Laravel 會從 [服務容器](/docs/laravel/10.x/container) 中解析 `cache` 繫結並運行該對象請求的方法（在這個例子中就是 `get` 方法）

## 即時 Facades

使用即時 Facade, 你可以將應用程式中的任何類視為 Facade。為了說明這是如何使用的， 讓我們首先看一下一些不使用即時 Facade 的程式碼。例如，假設我們的 `Podcast` 模型有一個 `publish 方法`。 但是，為了發佈 `Podcast`，我們需要注入一個 `Publisher` 實例：

    <?php

    namespace App\Models;

    use App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         */
        public function publish(Publisher $publisher): void
        {
            $this->update(['publishing' => now()]);

            $publisher->publish($this);
        }
    }

將 publisher 的實現注入到該方法中，我們可以輕鬆地測試這種方法，因為我們可以模擬注入的 publisher 。但是，它要求我們每次呼叫 `publish` 方法時始終傳遞一個 publisher 實例。 使用即時的 Facades, 我們可以保持同樣的可測試性，而不需要顯式地通過 `Publisher` 實例。要生成即時 Facade，請在匯入類的名稱空間中加上 `Facades`：

    <?php

    namespace App\Models;

    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         */
        public function publish(): void
        {
            $this->update(['publishing' => now()]);

            Publisher::publish($this);
        }
    }

當使用即時 Facade 時， publisher 實現將通過使用 `Facades` 前綴後出現的介面或類名的部分來解決服務容器的問題。在測試時，我們可以使用 Laravel 的內建 Facade 測試輔助函數來模擬這種方法呼叫：

    <?php

    namespace Tests\Feature;

    use App\Models\Podcast;
    use Facades\App\Contracts\Publisher;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Tests\TestCase;

    class PodcastTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A test example.
         */
        public function test_podcast_can_be_published(): void
        {
            $podcast = Podcast::factory()->create();

            Publisher::shouldReceive('publish')->once()->with($podcast);

            $podcast->publish();
        }
    }

