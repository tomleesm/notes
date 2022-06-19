# 路由

Laravel 會由上到下匹配路由規則，匹配到就執行。`routes/api.php` 用來定義 API，URI 開頭自動加上 /api，定義在 `app/Providers/RouteServiceProvider.php` 的 `mapApiRoutes()`

``` php
<?php
# GET /hello，回傳文字 Hello World
Route::get('hello', function () {
    return 'Hello World';
});

# GET /user，執行 UserController 的方法 index()
Route::get('/user', 'UserController@index');

# 可用的 HTTP 動詞
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
Route::match(['get', 'post'], 'foo', function () {
    return 'GET /foo 或 POST /foo';
});
Route::any('bar', function () {
    return '任何 HTTP 動詞都匹配 /bar';
});
```

## redirect() 和 view()

``` php
<?php
# GET /here，回傳一個 HTML 要求瀏覽器重新導向 /there。routes/api.php 也一樣
# HTML 片段：<meta http-equiv="refresh" content="0;url='/there'" />
# 第三個參數是 HTTP 狀態碼，預設是 302
Route::redirect('/here', '/there');
# 等於 Route::redirect('/here', '/there', 301);
Route::permanentRedirect('/here', '/there');

# GET /welcome，回傳 resources/views/ 的 welcome.blade.php 或 welcome.php 產生的 HTML
# 第三個參數傳送 $name = 'Tom' 給 view
Route::view('/welcome', 'welcome', ['name' => 'Tom']);
```

## 路由參數 {id}

``` php
<?php
# 路由參數：{id} 不能是空值
# GET /posts/123/comments/456，則 $postId = 123，$commentId = 456
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    return $postId . '-' . $commentId;
});

# 可選參數：{id} 可以是空值
# GET /user，則 $name 使用設定的預設值 John
Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

## 正規約束

以下的正規表示法使用 PHP 的正規表示法，例如 `/[0-9]+/`，但是不用斜線 `/` 包起兩側

``` php
<?php
# where(路由參數名稱, 正規表示法) 或 where(關聯陣列)
Route::get('user/{name}', function ($name) {
    # $name 必須是英文字母且不能為空值
})->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
    # $id 必須是數字
})->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    # 同時指定 id 和 name 的格式
})->where( ['id' => '[0-9]+', 'name' => '[a-z]+'] );

# 在 RouteServiceProvider 使用 Route::pattern() 設定所有 {id} 都必須是數字
public function boot()
{
    Route::pattern('id', '[0-9]+');
    // parent::boot() 必須在最後呼叫，否則上面的 Route::pattern() 不會運作
    parent::boot();
}

# 設定 {search} 可以使用斜線 /，預設是不能使用
# 經實測，GET /search//abc 回傳 /abc，但是 GET /search/abc/ 回傳 abc
Route::get('search/{search}', function ($search) {
    return $search;
})->where('search', '.*');
# 經實測，GET //abc/search 回傳 abc，GET /abc//search 回傳 abc/
Route::get('{search}/search', function ($search) {
    return $search;
})->where('search', '.*');
```

## 命名路由

``` php
<?php
# GET /user/123/profile 回傳 my url: http://localhost:8114/user/123/profile
# 因爲 .env 的 APP_URL 設定爲 http://localhost
Route::get('user/{id}/profile', function ($id) {
    // route(路由名稱, 變數關聯陣列)：通過路由名稱生成 URL
    // 第二個參數傳送變數 $id = 123 給 {id}
    return 'my url: ' . route('profile', ['id' => $id]);
})->name('profile');
```

## 路由群組

``` php
<?php
# 等於 Route::get('user/{id}', function($id) {});
Route::prefix('user')->group(function() {
  Route::get('{id}', function($id) {});
});

# 套用 middleware。先執行 A，再執行 B
Route::get(...)->middlewawre('A', 'B');
Route::get(...)->middlewawre(['A', 'B']);
# Route::middleware('A', 'B') 或 Route::middleware(['A', 'B']) 
Route::middleware('A', 'B')->group(function() {
  Route::get(...);
});
# 在 controller 建構式套用 middleware
class UserController extends Controller
{
    public function __construct()
    {
        $this->middleware('A');
        $this->middleware( [ 'A', 'B' ] );
        # 特別注意！兩個 middleware 以上只能用陣列
        # 所以 $this->middleware('A', 'B') 會產生錯誤訊息
        # only() 只套用在指定的 method
        $this->middleware('A')->only('methodA', 'methodB');
        $this->middleware('A')->only( [ 'methodA', 'methodB' ] );
        # except() 除了指定的 method 外都套用
        $this->middleware('A')->except('methodA', 'methodB');
        $this->middleware('A')->except( [ 'methodA', 'methodB' ] );
        # 臨時註冊一個 middleware
        $this->middleware(function (Request $request, $next) {
            return $next($request);
        });
    }
}

# 會呼叫 app/Http/Controller/A/B/UserController.php
Route::namespace('A\B')->group(function() {
  # 等於 Route::get('user', 'A\B\UserController@show');
  Route::get('user', 'UserController@show');
});
# 網址 tom.blog.dev，則 $account = 'tom'
Route::domain('{account}.blog.dev')->group(function() {
  Route::get('/', function($account) {});
});
# 等於 name('user.get')。注意 user 結尾是一個點
Route::name('user.')->group(function() {
  Route::get(...)->name('get');
});

# 也可以用 Routw::group() 一次套用多種
$routeProperties = [
  'prefix' => 'user',
  'middleware' => 'auth',
  'namespace' => 'A\B',
  'domain' => '{account}.blog.dev',
  'name' => 'user.'
];
Route::group($routeProperties, function() {
  Route::get('{id}', function($account, $id) {})->name('get');
  Route::get('{id}/edit', 'UserController@edit')->name('edit');
});
```

## 路由模型綁定

分爲隱性綁定和顯性綁定：

``` php
<?php
# 隱性綁定
# GET /users/123 會自動執行 $user = \App\User::findOrFail(123)
Route::get('users/{user}', function (\App\User $user) {
    return $user->email;
});
# 在 app/User.php
# 則會執行 $user = \App\User::where('name', 123)->first();
public function getRouteKeyName() {
    return 'name';
}

# 顯性綁定
# 在 RouteServiceProvider 定義 Route::model()
public function boot()
{
    parent::boot();
    Route::model('user_model', \App\User::class);
}
# GET /users/123 會自動執行 $user = \App\User::findOrFail(123)
Route::get('users/{user_model}', function(\App\User $user) {
    return $user->email;
});
# 如果要改成查詢資料表欄位 name，則如下設定
public function boot()
{
    parent::boot();
    Route::bind('user_model', function($value) {
        return \App\User::where('name', $value)->first() ?? abort(404);
    });
}

# 或者覆寫 \App\User 繼承的 resolveRouteBinding() 也可以。
# 隱性或顯性都適用
# 如果同時定義有上面的 Route::bind()，則以 Route::bind() 優先
# 
/**
 * Retrieve the model for a bound value.
 *
 * @param  mixed  $value
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value)
{
    return $this->where('name', $value)->first() ?? abort(404);
}
```

## 匹配不到任何路由時，自訂頁面

``` php
<?php
# 由上到下匹配路由規則，匹配到就執行，所以這個要放在 routes/web.php 的最底下
Route::fallback('Fallback@show');
```

## 頻率限制

``` php
<?php
# 每 1 分鐘查詢此路由最多 60 次，超過會丟出 429 Too many requests
Route::middleware('throttle:60,1')->group(function () {
});
# 在 \App\User 中定義屬性 public $rate_limit = 30，則每分鐘最多 30 次
# 需配合 auth middleware 才行
# 未登入的訪客則爲 10 次
Route::middleware('auth:api', 'throttle:10|rate_limit,1')->group(function () {
});
```

## 一般的 controller

``` bash
php artisan make:controller Namespace/Controller名稱
```

``` php
<?php
Route::get('路由規則', 'Namespace\Controller名稱@方法名稱');
```

## Single action controller

``` bash
## 如果已經定義路由設定是 single action controller
## 無論有沒有 --invokable，都會產生錯誤訊息
php artisan make:controller Namespace/Controller名稱 --invokable
```

``` php
<?php
# single action controller，只有一個方法 __invoke()
Route::get('路由規則', 'Namespace\Controller名稱');
```

## Resource controller

| Method     | URI               | Name          | Action                 | 說明         |
| ---------- | ----------------  | ------------- | ---------------------- | ----------- |
| GET, HEAD  | posts             | posts.index   | PostController@index   | 文章列表     |
| POST       | posts             | posts.store   | PostController@store   | 新增文章     |
| GET, HEAD  | posts/{post}      | posts.show    | PostController@show    | 檢視文章     |
| GET, HEAD  | posts/create      | posts.create  | PostController@create  | 新增文章頁面  |
| GET, HEAD  | posts/{post}/edit | posts.edit    | PostController@edit    | 編輯文章頁面  |
| PUT, PATCH | posts/{post}      | posts.update  | PostController@update  | 更新文章     |
| DELETE     | posts/{post}      | posts.destroy | PostController@destroy | 刪除文章     |

``` bash
# 顯示路由設定
php artisan route:list
# 產生以上表格的 PostController 和 method
# 可再加上 --api 或把 --resource 換成 --api，產生的 method 會排除 create 和 edit
php artisan make:controller PostController --resource
# 可再加上 --model=Post 新增 model Post
# 並在 method 參數注入 Post 物件
php artisan make:controller PostController --resource --model=Post
```

``` php
<?php
# 產生以上表格的路由設定
Route::resource('posts', 'PostController');
# 產生以上表格的路由設定，但是排除 create 和 edit
Route::apiResource('posts', 'PostController');
# Route::resource() 和 Route::apiResource()
# 功能相同，只差排除 create 和 edit，所以以下就不重複列出了

# 等於這兩行
# Route::resource('posts', 'PostController');
# Route::resource('photos', 'PhotoController');
# 注意 Route::resources() 結尾有 s
Route::resources([
    'posts' => 'PostController',
    'photos' => 'PhotoController'
]);

# only 陣列：只產生指定路由
Route::resource('post', 'PostController',
  [
      'only' => ['index', 'show']
  ]
);
# except 陣列：除了指定路由外都產生
Route::resource('post', 'PostController',
  [
    'except' => ['create', 'store', 'update', 'destroy']
  ]
);

Route::resource('posts', 'PostController',
  [
    # 把路由名稱從 posts.create 改成 articles.build
    # 注意 names 結尾有 s
    'names' => ['create' => 'articles.build'],
    # /posts/{post} 改成 /posts/{article}
    # 注意 parameters 結尾有 s
    'parameters' => ['post' => 'article']
  ]
);
```

## Nested resource

| Method     | URI               | Name          | Action                 |
| ---------- | ----------------  | ------------- | ---------------------- |
| GET, HEAD  | posts/{post}/comments | posts.comments.index | PostController@index |
| POST       | posts/{post}/comments | posts.comments.store | PostController@store |
| GET, HEAD  | posts/{post}/comments/{comment} | posts.comments.show | PostController@show |
| GET, HEAD  | posts/{post}/comments/create | posts.comments.create | PostController@create  |
| GET, HEAD  | posts/{post}/comments/{comment}/edit | posts.comments.edit | PostController@edit |
| PUT, PATCH | posts/{post}/comments/{comment} | posts.comments.update | PostController@update |
| DELETE     | posts/{post}/comments/{comment} | posts.comments.destroy| PostController@destroy |

``` php
<?php
# 產生上面表格的路由規則
Route::resource('posts.comments', 'PostController');
```

## Route cache

路由必須都是指向 controller 才能用 route cache，因爲 php 序列化無法處理 Closure

``` bash
# 更新 route cache
php artisan route:cache
# 刪除 route cache
php artisan route:clear
```