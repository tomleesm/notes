#  controller 

## 介紹

你可能希望使用「controller」類來組織此行為，而不是將所有請求處理邏輯定義為路由檔案中的閉包。 controller 可以將相關的請求處理邏輯分組到一個類中。 例如，一個 `UserController` 類可能會處理所有與使用者相關的傳入請求，包括顯示、建立、更新和刪除使用者。 默認情況下， controller 儲存在 `app/Http/Controllers` 目錄中。

## 編寫 controller 

### 基本 controller 

如果要快速生成新 controller ，可以使用 `make:controller` Artisan 命令。默認情況下，應用程式的所有 controller 都儲存在`app/Http/Controllers` 目錄中：

```shell
php artisan make:controller UserController
```

讓我們來看一個基本 controller 的示例。 controller 可以有任意數量的公共方法來響應傳入的HTTP請求：

    <?php

    namespace App\Http\Controllers;
    
    use App\Models\User;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * 顯示給定使用者的組態檔案。
         */
        public function show(string $id): View
        {
            return view('user.profile', [
                'user' => User::findOrFail($id)
            ]);
        }
    }



編寫 controller 類和方法後，可以定義到 controller 方法的路由，如下所示：

    use App\Http\Controllers\UserController;

    Route::get('/user/{id}', [UserController::class, 'show']);

當傳入的請求與指定的路由 URI 匹配時，將呼叫 `App\Http\Controllers\UserController` 類的 `show` 方法，並將路由參數傳遞給該方法。

>技巧： controller 並不是 **必需** 繼承基礎類。如果 controller 沒有繼承基礎類，你將無法使用一些便捷的功能，比如 `middleware` 和 `authorize` 方法。

### 單動作 controller 

如果 controller 動作特別複雜，你可能會發現將整個 controller 類專用於該單個動作很方便。為此，您可以在 controller 中定義一個 `__invoke` 方法：

    <?php

    namespace App\Http\Controllers;
    
    use App\Models\User;
    use Illuminate\Http\Response;

    class ProvisionServer extends Controller
    {
        /**
         * 設定新的web伺服器。
         */
        public function __invoke()
        {
            // ...
        }
    }

為單動作 controller 註冊路由時，不需要指定 controller 方法。相反，你可以簡單地將 controller 的名稱傳遞給路由器：

    use App\Http\Controllers\ProvisionServer;

    Route::post('/server', ProvisionServer::class);

你可以使用 `make:controller` Artisan 命令的 `--invokable` 選項生成可呼叫 controller ：

```shell
php artisan make:controller ProvisionServer --invokable
```

>技巧：可以使用 [stub 定製](/docs/laravel/10.x/artisan#stub-customization) 自訂 controller 範本。

##  controller 中介軟體

[中介軟體](/docs/laravel/10.x/middleware) 可以在你的路由檔案中分配給 controller 的路由：

    Route::get('profile', [UserController::class, 'show'])->middleware('auth');



或者，你可能會發現在 controller 的建構函式中指定中介軟體很方便。使用 controller 建構函式中的 `middleware` 方法，你可以將中介軟體分配給 controller 的操作：

    class UserController extends Controller
    {
        /**
         * Instantiate a new controller instance.
         */
        public function __construct()
        {
            $this->middleware('auth');
            $this->middleware('log')->only('index');
            $this->middleware('subscribed')->except('store');
        }
    }

 controller 還允許你使用閉包註冊中介軟體。這提供了一種方便的方法來為單個 controller 定義內聯中介軟體，而無需定義整個中介軟體類：

    use Closure;
    use Illuminate\Http\Request;

    $this->middleware(function (Request $request, Closure $next) {
        return $next($request);
    });

## 資源型 controller 

如果你將應用程式中的每個 Eloquent 模型都視為資源，那麼通常對應用程式中的每個資源都執行相同的操作。例如，假設你的應用程式中包含一個 `Photo` 模型和一個 `Movie` 模型。使用者可能可以建立，讀取，更新或者刪除這些資源。

Laravel 的資源路由通過單行程式碼即可將典型的增刪改查（“CURD”）路由分配給 controller 。首先，我們可以使用 Artisan 命令 `make:controller` 的 `--resource` 選項來快速建立一個 controller :

```shell
php artisan make:controller PhotoController --resource
```

這個命令將會生成一個 controller  `app/Http/Controllers/PhotoController.php`。其中包括每個可用資源操作的方法。接下來，你可以給 controller 註冊一個資源路由：

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class);



這個單一的路由聲明建立了多個路由來處理資源上的各種行為。生成的 controller 為每個行為保留了方法，而且你可以通過運行 Artisan 命令 `route:list` 來快速瞭解你的應用程式。

你可以通過將陣列傳參到 `resources` 方法中的方式來一次性的建立多個資源 controller ：

    Route::resources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

#### 資源 controller 操作處理

|請求方式      | 請求URI                    | 行為       | 路由名稱
----------|------------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

#### 自訂缺失模型行為

通常，如果未找到隱式繫結的資源模型，則會生成狀態碼為 404 的 HTTP 響應。 但是，你可以通過在定義資源路由時呼叫 `missing` 的方法來自訂該行為。`missing` 方法接受一個閉包，如果對於任何資源的路由都找不到隱式繫結模型，則將呼叫該閉包：

    use App\Http\Controllers\PhotoController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::resource('photos', PhotoController::class)
            ->missing(function (Request $request) {
                return Redirect::route('photos.index');
            });

#### 軟刪除模型

通常情況下，隱式模型繫結將不會檢索已經進行了 [軟刪除](/docs/laravel/10.x/eloquent#soft-deleting) 的模型，並且會返回一個 404 HTTP 響應。但是，你可以在定義資源路由時呼叫 `withTrashed` 方法來告訴框架允許軟刪除的模型：

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->withTrashed();



當不傳遞參數呼叫 `withTrashed` 時，將在 `show`、`edit` 和 `update` 資源路由中允許軟刪除的模型。你可以通過一個陣列指定這些路由的子集傳遞給 `withTrashed` 方法：

    Route::resource('photos', PhotoController::class)->withTrashed(['show']);

#### 指定資源模型

如果你使用了路由模型的繫結 [路由模型繫結](/docs/laravel/10.x/routing#route-model-binding) 並且想在資源 controller 的方法中使用類型提示，你可以在生成 controller 的時候使用 `--model` 選項：

```shell
php artisan make:controller PhotoController --model=Photo --resource
```

#### 生成表單請求

你可以在生成資源 controller 時提供 `--requests`  選項來讓 Artisan 為 controller 的 storage 和 update 方法生成 [表單請求類](/docs/laravel/10.x/validation#form-request-validation)：

```shell
php artisan make:controller PhotoController --model=Photo --resource --requests
```

### 部分資源路由

當聲明資源路由時，你可以指定 controller 處理的部分行為，而不是所有默認的行為：

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->only([
        'index', 'show'
    ]);

    Route::resource('photos', PhotoController::class)->except([
        'create', 'store', 'update', 'destroy'
    ]);

#### API 資源路由

當聲明用於 API 的資源路由時，通常需要排除顯示 HTML 範本的路由，例如 `create` 和 `edit`。為了方便，你可以使用 `apiResource` 方法來排除這兩個路由：

    use App\Http\Controllers\PhotoController;

    Route::apiResource('photos', PhotoController::class);



你也可以傳遞一個陣列給 `apiResources` 方法來同時註冊多個 API 資源 controller ：

    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\PostController;

    Route::apiResources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

要快速生成不包含 `create` 或 `edit` 方法的 API 資源 controller ，你可以在執行 `make:controller` 命令時使用 `--api` 參數：

```shell
php artisan make:controller PhotoController --api
```

### 巢狀資源

有時可能需要定義一個巢狀的資源型路由。例如，照片資源可能被新增了多個評論。那麼可以在路由中使用 `.` 符號來聲明資源型 controller ：

    use App\Http\Controllers\PhotoCommentController;

    Route::resource('photos.comments', PhotoCommentController::class);

該路由會註冊一個巢狀資源，可以使用如下 URI 訪問：

    /photos/{photo}/comments/{comment}

#### 限定巢狀資源的範圍

Laravel 的 [隱式模型繫結](/docs/laravel/10.x/routing#implicit-model-binding-scoping) 特性可以自動限定巢狀繫結的範圍，以便確認已解析的子模型會自動屬於父模型。定義巢狀路由時，使用 scoped 方法，可以開啟自動範圍限定，也可以指定 Laravel 應該按照哪個欄位檢索子模型資源，有關如何完成此操作的更多資訊，請參見有關 [範圍資源路由](#restful-scoping-resource-routes) 的文件。

#### 淺層巢狀

通常，並不是在所有情況下都需要在 URI 中同時擁有父 ID 和子 ID，因為子 ID 已經是唯一的識別碼。當使用唯一識別碼（如自動遞增的主鍵）來標識 URL 中的模型時，可以選擇使用「淺巢狀」的方式定義路由：

    use App\Http\Controllers\CommentController;

    Route::resource('photos.comments', CommentController::class)->shallow();



上面的路由定義方式會定義以下路由：

|請求方式       | 請求URI                               | 行為       | 路由名稱
----------|-----------------------------------|--------------|---------------------
GET       | `/photos/{photo}/comments`        | index        | photos.comments.index
GET       | `/photos/{photo}/comments/create` | create       | photos.comments.create
POST      | `/photos/{photo}/comments`        | store        | photos.comments.store
GET       | `/comments/{comment}`             | show         | comments.show
GET       | `/comments/{comment}/edit`        | edit         | comments.edit
PUT/PATCH | `/comments/{comment}`             | update       | comments.update
DELETE    | `/comments/{comment}`             | destroy      | comments.destroy

### 命名資源路由

默認情況下，所有的資源 controller 行為都有一個路由名稱。你可以傳入 `names` 陣列來覆蓋這些名稱：

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->names([
        'create' => 'photos.build'
    ]);

### 命名資源路由參數

默認情況下，`Route::resource` 會根據資源名稱的「單數」形式建立資源路由的路由參數。你可以使用 `parameters` 方法來輕鬆地覆蓋資源路由名稱。傳入 `parameters` 方法應該是資源名稱和參數名稱的關聯陣列：

    use App\Http\Controllers\AdminUserController;

    Route::resource('users', AdminUserController::class)->parameters([
        'users' => 'admin_user'
    ]);

上面的示例將會為資源的 `show` 路由生成以下的 URL：

    /users/{admin_user}

### 限定範圍的資源路由

Laravel 的 [範疇隱式模型繫結](/docs/laravel/10.x/routing#implicit-model-binding-scoping) 功能可以自動確定巢狀繫結的範圍，以便確認已解析的子模型屬於父模型。通過在定義巢狀資源時使用 `scoped` 方法，你可以啟用自動範圍界定，並指示 Laravel 應該通過以下方式來檢索子資源的哪個欄位：

    use App\Http\Controllers\PhotoCommentController;

    Route::resource('photos.comments', PhotoCommentController::class)->scoped([
        'comment' => 'slug',
    ]);

此路由將註冊一個有範圍的巢狀資源，該資源可以通過以下 URI 進行訪問：

    /photos/{photo}/comments/{comment:slug}

當使用一個自訂鍵的隱式繫結作為巢狀路由參數時，Laravel 會自動限定查詢範圍，按照約定的命名方式去父類中尋找關聯方法，然後檢索到對應的巢狀模型。在這種情況下，將假定 `Photo` 模型有一個叫 `comments` （路由參數名的複數）的關聯方法，通過這個方法可以檢索到 `Comment` 模型。

### 本地化資源 URIs

默認情況下，`Route::resource` 將會用英文動詞建立資源 URIs。如果需要自訂 `create` 和 `edit` 行為的動名詞，你可以在 `App\Providers\RouteServiceProvider` 的 `boot` 方法中使用 `Route::resourceVerbs` 方法實現：

    /**
     * 定義你的路由模型繫結，模式過濾器等
     */
    public function boot(): void
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);

        // ...
    }

Laravel 的複數器支援[組態幾種不同的語言](/docs/laravel/10.x/localization#pluralization-language)。自訂動詞和複數語言後，諸如 `Route::resource('publicacion', PublicacionController::class)` 之類的資源路由註冊將生成以下URI：

    /publicacion/crear

    /publicacion/{publicaciones}/editar

### 補充資源 controller 

如果你需要向資源 controller 新增超出默認資源路由集的其他路由，則應在呼叫 `Route::resource` 方法之前定義這些路由；否則，由 `resource` 方法定義的路由可能會無意中優先於您的補充路由：
單例資源
    use App\Http\Controller\PhotoController;

    Route::get('/photos/popular', [PhotoController::class, 'popular']);
    Route::resource('photos', PhotoController::class);

> 技巧：請記住讓你的 controller 保持集中。如果你發現自己經常需要典型資源操作集之外的方法，請考慮將 controller 拆分為兩個更小的 controller 。

### 單例資源 controller 

有時候，應用中的資源可能只有一個實例。比如，使用者「個人資料」可被編輯或更新，但是一個使用者只會有一份「個人資料」。同樣，一張圖片也只有一個「縮圖」。這些資源就是所謂「單例資源」，這意味著該資源有且只能有一個實例存在。這種情況下，你可以註冊成單例(signleton)資源 controller :

```php
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::singleton('profile', ProfileController::class);
```

上例中定義的單例資源會註冊如下所示的路由。如你所見，單例資源中「新建」路由沒有被註冊；並且註冊的路由不接收路由參數，因為該資源中只有一個實例存在：

|請求方式      | 請求 URI                               | 行為       | 路由名稱
----------|-----------------------------------|--------------|---------------------
GET       | `/profile`                        | show         | profile.show
GET       | `/profile/edit`                   | edit         | profile.edit
PUT/PATCH | `/profile`                        | update       | profile.update

單例資源也可以在標準資源內巢狀使用：

```php
Route::singleton('photos.thumbnail', ThumbnailController::class);
```

上例中， `photo` 資源將接收所有的[標準資源路由](#actions-handled-by-resource-controller)；不過，`thumbnail` 資源將會是個單例資源，它的路由如下所示：

| 請求方式      | 請求 URI                              | 行為  | 路由名稱               |
|-----------|----------------------------------|---------|--------------------------|
| GET       | `/photos/{photo}/thumbnail`      | show    | photos.thumbnail.show    |
| GET       | `/photos/{photo}/thumbnail/edit` | edit    | photos.thumbnail.edit    |
| PUT/PATCH | `/photos/{photo}/thumbnail`      | update  | photos.thumbnail.update  |

#### Creatable 單例資源

有時，你可能需要為單例資源定義 create 和 storage 路由。要實現這一功能，你可以在註冊單例資源路由時，呼叫 `creatable` 方法：

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();
```

如下所示，將註冊以下路由。還為可建立的單例資源註冊 `DELETE` 路由：

| Verb      | URI                                | Action  | Route Name               |
|-----------|------------------------------------|---------|--------------------------|
| GET       | `/photos/{photo}/thumbnail/create` | create  | photos.thumbnail.create  |
| POST      | `/photos/{photo}/thumbnail`        | store   | photos.thumbnail.store   |
| GET       | `/photos/{photo}/thumbnail`        | show    | photos.thumbnail.show    |
| GET       | `/photos/{photo}/thumbnail/edit`   | edit    | photos.thumbnail.edit    |
| PUT/PATCH | `/photos/{photo}/thumbnail`        | update  | photos.thumbnail.update  |
| DELETE    | `/photos/{photo}/thumbnail`        | destroy | photos.thumbnail.destroy |

如果希望 Laravel 為單個資源註冊 `DELETE` 路由，但不註冊建立或儲存路由，則可以使用 `destroyable` 方法：

```php
Route::singleton(...)->destroyable();
```

#### API 單例資源

`apiSingleton` 方法可用於註冊將通過API操作的單例資源，從而不需要 `create` 和 `edit`  路由：

```php
Route::apiSingleton('profile', ProfileController::class);
```

當然， API 單例資源也可以是 `可建立的` ，它將註冊 `store` 和 `destroy` 資源路由：

```php
Route::apiSingleton('photos.thumbnail', ProfileController::class)->creatable();
```

## 依賴注入和 controller 

#### 建構函式注入

Laravel [服務容器](/docs/laravel/10.x/container) 用於解析所有 Laravel  controller 。因此，可以在其建構函式中對 controller 可能需要的任何依賴項進行類型提示。聲明的依賴項將自動解析並注入到 controller 實例中：

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * 建立新 controller 實例。
         */
        public function __construct(
            protected UserRepository $users,
        ) {}
    }



#### 方法注入

除了建構函式注入，還可以在 controller 的方法上鍵入提示依賴項。方法注入的一個常見用例是將 `Illuminate\Http\Request` 實例注入到 controller 方法中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 儲存新使用者。
         */
        public function store(Request $request): RedirectResponse
        {
            $name = $request->name;

            // 儲存使用者。。。

            return redirect('/users');
        }
    }

如果 controller 方法也需要路由參數，那就在其他依賴項之後列出路由參數。例如，路由是這樣定義的：

    use App\Http\Controllers\UserController;

    Route::put('/user/{id}', [UserController::class, 'update']);

如下所示，你依然可以類型提示 `Illuminate\Http\Request` 並通過定義您的 controller 方法訪問 `id` 參數：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 更新給定使用者。
         */
        public function update(Request $request, string $id): RedirectResponse
        {
            // 更新使用者。。。

            return redirect('/users');
        }
    }

