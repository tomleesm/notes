# HTTP Session

## 簡介

由於 HTTP 驅動的應用程式是無狀態的，Session 提供了一種在多個請求之間儲存有關使用者資訊的方法，這類資訊一般都儲存在後續請求可以訪問的持久儲存 / 後端中。

Laravel 通過同一個可讀性強的 API 處理各種自帶的後台驅動程式。支援諸如比較熱門的[Memcached](https://memcached.org)、 [Redis](https://redis.io)和資料庫。

### 組態

Session 的組態檔案儲存在`config/session.php`檔案中。請務必查看此檔案中對於你而言可用的選項。默認情況下，Laravel 為絕大多數應用程式組態的 Session 驅動為`file` 驅動，它適用於大多數程序。如果你的應用程式需要在多個 Web 伺服器之間進行負載平衡，你應該選擇一個所有伺服器都可以訪問的集中式儲存，例如 Redis 或資料庫。

Session`driver`的組態預設了每個請求儲存 Session 資料的位置。Laravel 自帶了幾個不錯而且開箱即用的驅動：

- `file` - Sessions 儲存在`storage/framework/sessions`。
- `cookie` - Sessions 被儲存在安全加密的 cookie 中。
- `database` - Sessions 被儲存在關係型資料庫中。
- `memcached` / `redis` - Sessions 被儲存在基於快取記憶體的儲存系統中。
- `dynamodb` - Sessions 被儲存在 AWS DynamoDB 中。
- `array` - Sessions 儲存在 PHP 陣列中，但不會被持久化。

> **技巧**  
> 陣列驅動一般用於[測試](/docs/laravel/10.x/testing)並且防止儲存在 Session 中的資料被持久化。

### 驅動先決條件

#### 資料庫

使用`database`Session 驅動時，你需要建立一個記錄 Session 的表。下面是`Schema`的聲明示例：

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::create('sessions', function (Blueprint $table) {
        $table->string('id')->primary();
        $table->foreignId('user_id')->nullable()->index();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity')->index();
    });

你可以使用 Artisan 命令`session:table`生成這個遷移。瞭解更多資料庫遷移，請查看完整的文件[遷移文件](/docs/laravel/10.x/migrations):

```shell
php artisan session:table

php artisan migrate
```

#### Redis

在 Laravel 使用 Redis Session 驅動前，你需要安裝 PhpRedis PHP 擴展，可以通過 PECL 或者 通過 Composer 安裝這個`predis/predis`包 (~1.0)。更多關於 Redis 組態資訊，查詢 Laravel 的 [Redis 文件](/docs/laravel/10.x/redis#configuration).

> **技巧**  
> 在`session`組態檔案裡，`connection`選項可以用來設定 Session 使用 Redis 連接方式。

## 使用 Session

### 獲取資料

在 Laravel 中有兩種基本的 Session 使用方式：全域`session`助手函數和通過`Request`實例。首先看下通過`Request`實例訪問 Session , 它可以隱式繫結路由閉包或者 controller 方法。記住，Laravel 會自動注入 controller 方法的依賴。[服務容器](/docs/laravel/10.x/container)：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * 顯示指定使用者個人資料。
         */
        public function show(Request $request, string $id): View
        {
            $value = $request->session()->get('key');

            // ...

            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }



當你從 Session 獲取資料時，你也可以在`get`方法第二個參數里傳遞一個 default 預設值，如果 Session 裡不存在鍵值對 key 的資料結果，這個預設值就會返回。如果你傳遞給`get`方法一個閉包作為預設值，這個閉包會被執行並且返回結果：

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

#### 全域 Session 助手函數

你也可以在 Session 裡使用 PHP 全域`session`函數獲取和儲存資料。當這個`session`函數以一個單獨的字串形式被呼叫時，它將會返回這個 Session 鍵值對的結果。當函數以 key / value 陣列形式被呼叫時，這些值會被儲存在 Session 裡：

    Route::get('/home', function () {
        // 從 Session 獲取資料 ...
        $value = session('key');

        // 設定預設值...
        $value = session('key', 'default');

        // 在Session 裡儲存一段資料 ...
        session(['key' => 'value']);
    });

> **技巧**  
> 通過 HTTP 請求實例與通過`session`助手函數方式使用 Session 之間沒有實際區別。兩種方式都是[可的測試](/docs/laravel/10.x/testing)，你所有的測試用例中都可以通過 `assertSessionHas`方法進行斷言。

#### 獲取所有 Session 資料

如果你想要從 Session 裡獲取所有資料，你可以使用`all`方法：

    $data = $request->session()->all();

#### 判斷 Session 裡是否存在條目

判斷 Session 裡是否存在一個條目，你可以使用`has`方法。如果條目存在`has`，方法返回`true`不存在則返回`null`：

    if ($request->session()->has('users')) {
        // ...
    }

判斷 Session 裡是否存在一個即使結果值為`null`的條目，你可以使用`exists`方法：

    if ($request->session()->exists('users')) {
        // ...
    }

要確定某個條目是否在 session 中不存在，你可以使用 `missing`方法。如果條目不存在，`missing`方法返回`true`：

    if ($request->session()->missing('users')) {
        // ...
    }

### 儲存資料

Session 裡儲存資料，你通常將使用 Request 實例中的`put`方法或者`session`助手函數：

    // 通過 Request 實例儲存 ...
    $request->session()->put('key', 'value');

    // 通過全域 Session 助手函數儲存 ...
    session(['key' => 'value']);

#### Session 儲存陣列

`push`方法可以把一個新值推入到以陣列形式儲存的 session 值裡。例如：如果`user.teams`鍵值對有一個關於團隊名字的陣列，你可以推入一個新值到這個陣列裡：

    $request->session()->push('user.teams', 'developers');

#### 獲取 & 刪除條目

`pull`方法會從 Session 裡獲取並且刪除一個條目，只需要一步如下：

    $value = $request->session()->pull('key', 'default');

#### 遞增 / 遞減 session 值

如果你的 Session 資料裡有整形你希望進行加減操作，可以使用`increment`和`decrement`方法：

    $request->session()->increment('count');

    $request->session()->increment('count', $incrementBy = 2);

    $request->session()->decrement('count');

    $request->session()->decrement('count', $decrementBy = 2);

### 快閃記憶體資料

有時你可能想在 Session 裡為下次請求儲存一些條目。你可以使用`flash`方法。使用這個方法，儲存在 Session 的資料將立即可用並且會保留到下一個 HTTP 請求期間，之後會被刪除。快閃記憶體資料主要用於短期的狀態消息：

    $request->session()->flash('status', 'Task was successful!');

如果你需要為多次請求持久化快閃記憶體資料，可以使用`reflash`方法，它會為一個額外的請求保持住所有的快閃記憶體資料，如果你僅需要保持特定的快閃記憶體資料，可以使用`keep`方法：

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

如果你僅為了當前的請求持久化快閃記憶體資料，可以使用`now` 方法：

    $request->session()->now('status', 'Task was successful!');

### 刪除資料

`forget`方法會從 Session 刪除一些資料。如果你想刪除所有 Session 資料，可以使用`flush`方法：

    // 刪除一個單獨的鍵值對 ...
    $request->session()->forget('name');

    // 刪除多個 鍵值對 ...
    $request->session()->forget(['name', 'status']);

    $request->session()->flush();


### 重新生成 Session ID

重新生成 Session ID 經常被用來阻止惡意使用者使用 [session fixation](https://owasp.org/www-community/attacks/Session_fixation) 攻擊你的應用。

如果你正在使用[入門套件](/docs/laravel/10.x/starter-kits)或 [Laravel Fortify](/docs/laravel/10.x/fortify)中的任意一種， Laravel 會在認證階段自動生成 Session ID；然而如果你需要手動重新生成 Session ID ，可以使用`regenerate`方法：

    $request->session()->regenerate();

如果你需要重新生成 Session ID 並同時刪除所有 Session 裡的資料，可以使用`invalidate`方法：

    $request->session()->invalidate();

## Session 阻塞

> **注意**  
> 應用 Session 阻塞功能，你的應用必須使用一個支援[原子鎖 ](/docs/laravel/10.x/cache#atomic-locks)的快取驅動。目前，可用的快取驅動有`memcached`、 `dynamodb`、 `redis`和`database`等。另外，你可能不會使用`cookie` Session 驅動。

默認情況下，Laravel 允許使用同一 Session 的請求並行地執行，舉例來說，如果你使用一個 JavaScript HTTP 庫向你的應用執行兩次 HTTP 請求，它們將同時執行。對多數應用這不是問題，然而 在一小部分應用中可能出現 Session 資料丟失，這些應用會向兩個不同的應用端並行請求，並同時寫入資料到 Session。

為瞭解決這個問題，Laravel 允許你限制指定 Session 的並行請求。首先，你可以在路由定義時使用`block`鏈式方法。在這個示例中，一個到`/profile`的路由請求會拿到一把 Session 鎖。當它處在鎖定狀態時，任何使用相同 Session ID 的到`/profile`或`/order`的路由請求都必須等待，直到第一個請求處理完成後再繼續執行：

    Route::post('/profile', function () {
        // ...
    })->block($lockSeconds = 10, $waitSeconds = 10)

    Route::post('/order', function () {
        // ...
    })->block($lockSeconds = 10, $waitSeconds = 10)



`block`方法接受兩個可選參數。`block`方法接受的第一個參數是 Session 鎖釋放前應該持有的最大秒數。當然，如果請求在此時間之前完成執行，鎖將提前釋放。

`block`方法接受的第二個參數是請求在試圖獲得 Session 鎖時應該等待的秒數。如果請求在給定的秒數內無法獲得 session 鎖，將拋出`Illuminate\Contracts\Cache\LockTimeoutException`異常。

如果不傳參，那麼 Session 鎖默認鎖定最大時間是 10 秒，請求鎖最大的等待時間也是 10 秒：

    Route::post('/profile', function () {
        // ...
    })->block()

## 新增自訂 Session 驅動

#### 實現驅動

如果現存的 Session 驅動不能滿足你的需求，Laravel 允許你自訂 Session Handler。你的自訂驅動應實現 PHP 內建的`SessionHandlerInterface`。這個介面僅包含幾個方法。以下是 MongoDB 驅動實現的程式碼片段：

    <?php

    namespace App\Extensions;

    class MongoSessionHandler implements \SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> **技巧**  
> Laravel 沒有內建存放擴展的目錄，你可以放置在任意目錄下，這個示例裡，我們建立了一個`Extensions`目錄存放`MongoSessionHandler`。



由於這些方法的含義並非通俗易懂，因此我們快速瀏覽下每個方法：

<div class="content-list" markdown="1">

- `open`方法通常用於基於檔案的 Session 儲存系統。因為 Laravel 附帶了一個`file`  Session 驅動。你無須在裡面寫任何程式碼。可以簡單地忽略掉。
- `close`方法跟`open`方法很像，通常也可以忽略掉。對大多數驅動來說，它不是必須的。
- `read` 方法應返回與給定的`$sessionId`關聯的 Session 資料的字串格式。在你的驅動中獲取或儲存 Session 資料時，無須作任何序列化和編碼的操作，Laravel 會自動為你執行序列化。
- `write`方法將與`$sessionId`關聯的給定的`$data`字串寫入到一些持久化儲存系統，如 MongoDB 或者其他你選擇的儲存系統。再次，你無須進行任何序列化操作，Laravel 會自動為你處理。
- `destroy`方法應可以從持久化儲存中刪除與`$sessionId`相關聯的資料。
- `gc`方法應可以銷毀給定的`$lifetime`（UNIX 時間戳格式 ）之前的所有 Session 資料。對於像 Memcached 和 Redis 這類擁有過期機制的系統來說，本方法可以置空。

</div>

#### 註冊驅動

一旦你的驅動實現了，需要註冊到 Laravel 。在 Laravel 中新增額外的驅動到 Session 後端 ，你可以使用`Session` [Facade](/docs/laravel/10.x/facades) 提供的`extend`方法。你應該在[服務提供者](/docs/laravel/10.x/providers)中的`boot`方法中呼叫`extend`方法。可以通過已有的`App\Providers\AppServiceProvider`或建立一個全新的服務提供者執行此操作：

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionHandler;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * 註冊任意應用服務。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 啟動任意應用服務。
         */
        public function boot(): void
        {
            Session::extend('mongo', function (Application $app) {
                // 返回一個 SessionHandlerInterface 介面的實現 ...
                return new MongoSessionHandler;
            });
        }
    }



一旦 Session 驅動註冊完成，就可以在`config/session.php`組態檔案選擇使用`mongo` 驅動。

