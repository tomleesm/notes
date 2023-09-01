# 生成 URL

## 簡介

Laravel 提供了幾個輔助函數來為應用程式生成 URL。主要用於在範本和 API 響應中建構 URL 或者在應用程式的其它部分生成重新導向響應。

## 基礎

### 生成基礎 URLs

輔助函數 `url` 可以用於應用的任何一個 URL。生成的 URL 將自動使用當前請求中的方案 (HTTP 或 HTTPS) 和主機：

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

### 訪問當前 URL

如果沒有給輔助函數 `url` 提供路徑，則會返回一個 `Illuminate\Routing\UrlGenerator` 實例，來允許你訪問有關當前 URL 的資訊：

    // 獲取當前 URL 沒有 query string...
    echo url()->current();

    // 獲取當前 URL 包括 query string...
    echo url()->full();

    // 獲取上個請求 URL
    echo url()->previous();

上面的這些方法都可以通過 `URL` [facade](/docs/laravel/10.x/facades) 訪問:

    use Illuminate\Support\Facades\URL;

    echo URL::current();

## 命名路由的 URLs

輔助函數 `route` 可以用於生成指定 [命名路由](/docs/laravel/10.x/routing#named-routes) 的URLs。 命名路由生成的 URLs 不與路由上定義的 URL 相耦合。因此，就算路由的 URL 有任何改變，都不需要對 `route` 函數呼叫進行任何更改。例如，假設你的應用程式包含以下路由：

    Route::get('/post/{post}', function (Post $post) {
        // ...
    })->name('post.show');


要生成此路由的 URL ，可以像這樣使用輔助函數 `route` ：

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

當然，輔助函數 `route` 也可以用於為具有多個參數的路由生成 URL：

    Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
        // ...
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

任何與路由定義參數對應不上的附加陣列元素都將新增到 URL 的查詢字串中：

    echo route('post.show', ['post' => 1, 'search' => 'rocket']);

    // http://example.com/post/1?search=rocket

#### Eloquent Models

你通常使用 [Eloquent 模型](/docs/laravel/10.x/eloquent) 的主鍵生成 URL。因此，您可以將 Eloquent 模型作為參數值傳遞。 `route` 輔助函數將自動提取模型的主鍵：

    echo route('post.show', ['post' => $post]);

### 簽名 URLs

Laravel 允許你輕鬆地為命名路徑建立「簽名」 URLs，這些 URLs 在查詢字串後附加了「簽名」雜湊，允許 Laravel 驗證 URL 自建立以來未被修改過。 簽名 URLs 對於可公開訪問但需要一層防止 URL 操作的路由特別有用。

例如，你可以使用簽名 URLs 來實現通過電子郵件傳送給客戶的公共「取消訂閱」連結。要建立指向路徑的簽名 URL ，請使用  `URL` facade 的 `signedRoute` 方法：

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

如果要生成具有有效期的臨時簽名路由 URL，可以使用以下 `temporarySignedRoute` 方法，當 Laravel 驗證一個臨時的簽名路由 URL 時，它會確保編碼到簽名 URL 中的過期時間戳沒有過期：

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

#### 驗證簽名路由請求

要驗證傳入請求是否具有有效簽名，你應該對傳入的 `Illuminate\Http\Request` 實例中呼叫 `hasValidSignature` 方法：

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

有時，你可能需要允許你的應用程式前端將資料附加到簽名 URL，例如在執行客戶端分頁時。因此，你可以指定在使用 `hasValidSignatureWhileIgnoring` 方法驗證簽名 URL 時應忽略的請求查詢參數。請記住，忽略參數將允許任何人根據請求修改這些參數：

    if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
        abort(401);
    }

或者，你可以將 `Illuminate\Routing\Middleware\ValidateSignature` [中介軟體](/docs/laravel/10.x/middleware) 分配給路由。如果它不存在，則應該在 HTTP 核心的 `$middlewareAliases` 陣列中為此中介軟體分配一個鍵：

    /**
     * The application's middleware aliases.
     *
     * Aliases may be used to conveniently assign middleware to routes and groups.
     *
     * @var array<string, class-string|string>
     */
    protected $middlewareAliases = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

一旦在核心中註冊了中介軟體，就可以將其附加到路由。如果傳入的請求沒有有效的簽名，中介軟體將自動返回 `403` HTTP 響應：

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');



#### 響應無效的簽名路由

當有人訪問已過期的簽名 URL 時，他們將收到一個通用的錯誤頁面，顯示 `403` HTTP 狀態程式碼。然而，你可以通過在異常處理程序中為 `InvalidSignatureException` 異常定義自訂 “可渲染” 閉包來自訂此行為。這個閉包應該返回一個 HTTP 響應：

    use Illuminate\Routing\Exceptions\InvalidSignatureException;

    /**
     * 為應用程式註冊異常處理回呼
     */
    public function register(): void
    {
        $this->renderable(function (InvalidSignatureException $e) {
            return response()->view('error.link-expired', [], 403);
        });
    }

##  controller 行為的 URL

`action` 功能可以為給定的 controller 行為生成 URL。

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

如果 controller 方法接收路由參數，你可以通過第二個參數傳遞：

    $url = action([UserController::class, 'profile'], ['id' => 1]);

## 預設值

對於某些應用程式，你可能希望為某些 URL 參數的請求範圍指定預設值。例如，假設有些路由定義了 `{locale}` 參數：

    Route::get('/{locale}/posts', function () {
        // ...
    })->name('post.index');

每次都通過 `locale` 來呼叫輔助函數 `route` 也是一件很麻煩的事情。因此，使用 `URL::defaults` 方法定義這個參數的預設值，可以讓該參數始終存在當前請求中。然後就能從 [路由中介軟體](/docs/laravel/10.x/middleware#assigning-middleware-to-routes) 呼叫此方法來訪問當前請求：

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\URL;
    use Symfony\Component\HttpFoundation\Response;

    class SetDefaultLocaleForUrls
    {
        /**
         * 處理傳入的請求
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }



一旦設定了 `locale` 參數的預設值，你就不再需要通過輔助函數 `route` 生成 URL 時傳遞它的值。

#### 默認 URL & 中介軟體優先順序

設定 URL 的預設值會影響 Laravel 對隱式模型繫結的處理。因此，你應該通過[設定中介軟體優先順序](/docs/laravel/10.x/middleware#sorting-middleware)來確保在 Laravel 自己的 `SubstituteBindings` 中介軟體執行之前設定 URL 的預設值。你可以通過在你的應用的 HTTP kernel 檔案中的 `$middlewarePriority` 屬性裡把你的中介軟體放在 `SubstituteBindings` 中介軟體之前。

`$middlewarePriority` 這個屬性在 `Illuminate\Foundation\Http\Kernel` 這個基類裡。你可以複製一份到你的應用程式的 HTTP kernel 檔案中以便做修改:

    /**
     * 根據優先順序排序的中介軟體列表
     *
     * 這將保證非全域中介軟體按照既定順序排序
     *
     * @var array
     */
    protected $middlewarePriority = [
        // ...
         \App\Http\Middleware\SetDefaultLocaleForUrls::class,
         \Illuminate\Routing\Middleware\SubstituteBindings::class,
         // ...
    ];

