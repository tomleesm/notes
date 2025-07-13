---
hide:
  - toc
---

```php
// 獲取 Auth 對象，等同於 Auth Facade
auth();
// 判斷當前使用者是否已認證（是否已登錄）
Auth::check();
// 判斷當前使用者是否未登錄，與 check() 相反
Auth::guest();
// 自訂看守器 默認為 `web`
Auth::guard();
// 獲取當前的認證使用者
Auth::user();
// 獲取當前的認證使用者的 ID（未登錄情況下會報錯）
Auth::id();
// 通過給定的資訊來嘗試對使用者進行認證（成功後會自動啟動會話）
Auth::attempt(['email' => $email, 'password' => $password]);
// 通過 Auth::attempt() 傳入 true 值來開啟 '記住我' 功能
Auth::attempt($credentials, true);
// 註冊嘗試登錄的事件監聽器
Auth::attempting($callback);
// 只針對一次的請求來認證使用者
Auth::once($credentials);
// 使用 ID 登錄，無 Cookie 和會話登錄
Auth::onceUsingId($id);
// 登錄一個指定使用者到應用上
Auth::login(User::find(1), $remember = false);
// 檢測是否記住了登錄
Auth::viaRemember();
// 登錄指定使用者 ID 的使用者到應用上
Auth::loginUsingId(1, $remember = false);
// 使使用者退出登錄（清除會話）
Auth::logout();
// 清除當前使用者的其他會話
Auth::logoutOtherDevices('password', $attribute = 'password');
// 驗證使用者憑證
Auth::validate($credentials);
// 使用 HTTP 的基本認證方式來認證
Auth::basic('username');
// 執行「HTTP Basic」登錄嘗試，只認證一次
Auth::onceBasic();
// 傳送密碼重設提示給使用者
Password::remind($credentials, function($message, $user){});
使用者授權
// 定義權限
Gate::define('update-post', 'Class@method');
Gate::define('update-post', function ($user, $post) {...});
// 傳遞多個參數
Gate::define('delete-comment', function ($user, $post, $comment) {});
// 一次性的定義多個 Gate 方法
Gate::resource('posts',  'App\Policies\PostPolicy');
// 檢測權限是否被定義
Gate::has('update-post');

// 檢查權限
Gate::denies('update-post', $post);
Gate::allows('update-post', $post);
Gate::check('update-post', $post);
// 指定使用者進行檢查
Gate::forUser($user)->allows('update-post', $post);
// 在 User 模型下，使用 Authorizable trait
User::find(1)->can('update-post', $post);
User::find(1)->cannot('update-post', $post);
User::find(1)->cant('update-post', $post);

// 攔截所有檢查，返回 bool
Gate::before(function ($user, $ability) {});
// 設定每一次驗證的回呼
Gate::after(function ($user, $ability, $result, $arguments) {});

// Blade 範本語法
@can('update-post', $post)
@endcan
// 支援 else 表示式
@can('update-post', $post)
@else
@endcan
// 無權限判斷
@cannot
@endcannot

// 生成一個新的策略
php artisan make:policy PostPolicy
php artisan make:policy PostPolicy --model=Post
// `policy` 幫助函數
policy($post)->update($user, $post)

// 控製器授權
$this->authorize('update', $post);
// 指定使用者 $user 授權
$this->authorizeForUser($user, 'update', $post);
// 控製器的 __construct 中授權資源控製器
$this->authorizeResource(Post::class,  'post');

// AuthServiceProvider->boot() 裡修改策略自動發現的邏輯
Gate::guessPolicyNamesUsing(function ($modelClass) {
    // 返回模型對應的策略名稱，如：// 'App\Model\User' => 'App\Policies\UserPolicy',
    return 'App\Policies\\'.class_basename($modelClass).'Policy';
});

// 中介軟體指定模型實例
Route::put('/post/{post}',  function  (Post $post)  { ... })->middleware('can:update,post');
// 中介軟體未指定模型實例
Route::post('/post',  function  ()  { ... })->middleware('can:create,App\Post');
```
