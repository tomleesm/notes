# HTTP 響應

## 建立響應

#### 字串 & 陣列

所有路由和 controller 處理完業務邏輯之後都會返迴響應到使用者的瀏覽器，Laravel 提供了多種不同的響應方式，其中最基本就是從路由或 controller 返回一個簡單的字串，框架會自動將這個字串轉化為一個完整的 HTTP 響應：

    Route::get('/', function () {
        return 'Hello World';
    });

除了從路由和 controller 返回字串之外，你還可以返回陣列。 框架會自動將陣列轉換為 JSON 響應：

    Route::get('/', function () {
        return [1, 2, 3];
    });

> **技巧**  
> 你知道從路由或 controller 還可以返回 [Eloquent 集合](/docs/laravel/10.x/eloquent-collections)嗎？他們也會自動轉化為 JSON 響應！

#### Response 對象

通常情況下會只返回簡單的字串或陣列，大多數時候，需要返回一個完整的`Illuminate\Http\Response`實例或是[檢視](/docs/laravel/10.x/views).

返回一個完整的`Response` 實例允許你自訂返回的 HTTP 狀態碼和返回頭資訊。`Response`實例繼承自`Symfony\Component\HttpFoundation\Response`類，該類提供了各種建構 HTTP 響應的方法：

    Route::get('/home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });



#### Eloquent 模型 和 集合

你也可以直接從你的路由和 controller 返回 [Eloquent ORM](/docs/laravel/10.x/eloquent) 模型和集合。當你這樣做時，Laravel 將自動將模型和集合轉換為 JSON 響應，同時遵循模型的 [隱藏屬性](/docs/laravel/10.x/eloquent-serialization#hiding-attributes-from-json):

    use App\Models\User;

    Route::get('/user/{user}', function (User $user) {
        return $user;
    });

### 在響應中附加 Header 資訊

請記住，大多數響應方法都是可以鏈式呼叫的，它允許你流暢地建構響應實例。例如，在將響應傳送回使用者之前，可以使用 `header` 方法將一系列頭新增到響應中：

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

或者，你可以使用 `withHeaders` 方法指定要新增到響應的標頭陣列：

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

#### 快取控制中介軟體

Laravel 包含一個 `cache.headers` 中介軟體，可用於快速設定一組路由的 `Cache-Control` 標頭。指令應使用相應快取控制指令的 蛇形命名法 等效項提供，並應以分號分隔。如果在指令列表中指定了 `etag` ，則響應內容的 MD5 雜湊將自動設定為 ETag 識別碼：

    Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
        Route::get('/privacy', function () {
            // ...
        });

        Route::get('/terms', function () {
            // ...
        });
    });



### 在響應中附加 Cookie 資訊

可以使用 `cookie` 方法將 cookie 附加到傳出的 `illumize\Http\Response` 實例。你應將 cookie 的名稱、值和有效分鐘數傳遞給此方法：

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

`cookie` 方法還接受一些使用頻率較低的參數。通常，這些參數的目的和意義與 PHP 的原生 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 的參數相同

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

如果你希望確保 cookie 與傳出響應一起傳送，但你還沒有該響應的實例，則可以使用 `Cookie` facade 將 cookie 加入佇列，以便在傳送響應時附加到響應中。`queue` 方法接受建立 cookie 實例所需的參數。在傳送到瀏覽器之前，這些 cookies 將附加到傳出的響應中：

    use Illuminate\Support\Facades\Cookie;

    Cookie::queue('name', 'value', $minutes);

#### 生成 Cookie 實例

如果要生成一個 `Symfony\Component\HttpFoundation\Cookie` 實例，打算稍後附加到響應實例中，你可以使用全域 `cookie` 助手函數。此 cookie 將不會傳送回客戶端，除非它被附加到響應實例中：

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);



#### 提前過期 Cookies

你可以通過響應中的`withoutCookie`方法使 cookie 過期，用於刪除 cookie ：

    return response('Hello World')->withoutCookie('name');

如果尚未有建立響應的實例，則可以使用`Cookie` facade 中的`expire` 方法使 Cookie 過期：

    Cookie::expire('name');

### Cookies 和 加密

默認情況下，由 Laravel 生成的所有 cookie 都經過了加密和簽名，因此客戶端無法篡改或讀取它們。如果要對應用程式生成的部分 cookie 停用加密，可以使用`App\Http\Middleware\EncryptCookies`中介軟體的`$except`屬性，該屬性位於`app/Http/Middleware`目錄中：

    /**
     * 這個名字的 Cookie 將不會加密。
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

## 重新導向

重新導向響應是`Illuminate\Http\RedirectResponse` 類的實例，包含將使用者重新導向到另一個 URL 所需的適當 HTTP 頭。Laravel 有幾種方法可以生成`RedirectResponse`實例。最簡單的方法是使用全域`redirect`助手函數：

    Route::get('/dashboard', function () {
        return redirect('home/dashboard');
    });

有時你可能希望將使用者重新導向到以前的位置，例如當提交的表單無效時。你可以使用全域 back 助手函數來執行此操作。由於此功能使用 [session](/docs/laravel/10.x/session)，請確保呼叫`back` 函數的路由使用的是`web`中介軟體組：

    Route::post('/user/profile', function () {
        // 驗證請求參數

        return back()->withInput();
    });



### 重新導向到指定名稱的路由

當你在沒有傳遞參數的情況下呼叫 `redirect` 助手函數時，將返回 `Illuminate\Routing\Redirector` 的實例，允許你呼叫 `Redirector` 實例上的任何方法。例如，要對命名路由生成 `RedirectResponse` ，可以使用 `route` 方法：

    return redirect()->route('login');

如果路由中有參數，可以將其作為第二個參數傳遞給 `route` 方法：

    // 對於具有以下URI的路由: /profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### 通過 Eloquent 模型填充參數

如果你要重新導向到使用從 Eloquent 模型填充 「ID」 參數的路由，可以直接傳遞模型本身。ID 將會被自動提取：

    // 對於具有以下URI的路由: /profile/{id}

    return redirect()->route('profile', [$user]);

如果你想要自訂路由參數，你可以指定路由參數 (`/profile/{id:slug}`) 或者重寫 Eloquent 模型上的 `getRouteKey` 方法：

    /**
     * 獲取模型的路由鍵值。
     */
    public function getRouteKey(): mixed
    {
        return $this->slug;
    }

### 重新導向到 controller 行為

也可以生成重新導向到 [controller actions](/docs/laravel/10.x/controllers)。只要把 controller 和 action 的名稱傳遞給 `action` 方法：

    use App\Http\Controllers\UserController;

    return redirect()->action([UserController::class, 'index']);

如果 controller 路由有參數，可以將其作為第二個參數傳遞給 `action` 方法：

    return redirect()->action(
        [UserController::class, 'profile'], ['id' => 1]
    );



### 重新導向到外部域名

有時候你需要重新導向到應用外的域名。可以通過呼叫`away`方法，它會建立一個不帶有任何額外的 URL 編碼、有效性總和檢查碼檢查`RedirectResponse`實例：

    return redirect()->away('https://www.google.com');

### 重新導向並使用快閃記憶體的 Session 資料

重新導向到新的 URL 的同時[傳送資料給 seesion](/docs/laravel/10.x/session#flash-data) 是很常見的。 通常這是在你將消息傳送到 session 後成功執行操作後完成的。為了方便，你可以建立一個`RedirectResponse`實例並在鏈式方法呼叫中將資料傳送給 session：

    Route::post('/user/profile', function () {
        // ...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

在使用者重新導向後，你可以顯示 [session](/docs/laravel/10.x/session)。例如，你可以使用[ Blade 範本語法](/docs/laravel/10.x/blade)：

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

#### 使用輸入重新導向

你可以使用`RedirectResponse`實例提供的`withInput`方法將當前請求輸入的資料傳送到 session ，然後再將使用者重新導向到新位置。當使用者遇到驗證錯誤時，通常會執行此操作。每當輸入資料被傳送到 session , 你可以很簡單的在下一次重新提交的表單請求中[取回它](/docs/laravel/10.x/requests#retrieving-old-input)：

    return back()->withInput();



## 其他響應類型

`response` 助手可用於生成其他類型的響應實例。當不帶參數呼叫 `response` 助手時，會返回 `Illuminate\Contracts\Routing\ResponseFactory` [contract](/docs/laravel/10.x/contracts) 的實現。 該契約提供了幾種有用的方法來生成響應。

### 響應檢視

如果你需要控制響應的狀態和標頭，但還需要返回 [view](/docs/laravel/10.x/views) 作為響應的內容，你應該使用 `view` 方法：

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

當然，如果你不需要傳遞自訂 HTTP 狀態程式碼或自訂標頭，則可以使用全域 `view` 輔助函數。

### JSON Responses

`json` 方法會自動將 `Content-Type` 標頭設定為 `application/json`，並使用 `json_encode` PHP 函數將給定的陣列轉換為 JSON：

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA',
    ]);

如果你想建立一個 JSONP 響應，你可以結合使用 `json` 方法和 `withCallback` 方法：

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

### 檔案下載

`download` 方法可用於生成強制使用者瀏覽器在給定路徑下載檔案的響應。`download` 方法接受檔案名稱作為該方法的第二個參數，這將確定下載檔案的使用者看到的檔案名稱。 最後，你可以將一組 HTTP 標頭作為該方法的第三個參數傳遞：

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

> 注意：管理檔案下載的 Symfony HttpFoundation 要求正在下載的檔案具有 ASCII 檔案名稱。


#### 流式下載

有時你可能希望將給定操作的字串響應轉換為可下載的響應，而不必將操作的內容寫入磁碟。在這種情況下，你可以使用`streamDownload`方法。此方法接受回呼、檔案名稱和可選的標頭陣列作為其參數：

    use App\Services\GitHub;

    return response()->streamDownload(function () {
        echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents'];
    }, 'laravel-readme.md');

### 檔案響應

`file`方法可用於直接在使用者的瀏覽器中顯示檔案，例如圖像或 PDF，而不是啟動下載。這個方法接受檔案的路徑作為它的第一個參數和一個頭陣列作為它的第二個參數：

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

## 響應宏

如果你想定義一個可以在各種路由和 controller 中重複使用的自訂響應，你可以使用`Response` facade 上的`macro`方法。通常，你應該從應用程式的[服務提供者](/docs/laravel/10.x/providers)，如`App\Providers\AppServiceProvider`服務提供程序的`boot`方法呼叫此方法：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 啟動一個應用的服務
         */
        public function boot(): void
        {
            Response::macro('caps', function (string $value) {
                return Response::make(strtoupper($value));
            });
        }
    }

`macro`函數接受名稱作為其第一個參數，並接受閉包作為其第二個參數。當從`ResponseFactory`實現或`response`助手函數呼叫宏名稱時，將執行宏的閉包：

    return response()->caps('foo');

