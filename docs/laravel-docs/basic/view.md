# View

## 建立 view

``` php
<?php
# view 放在 resources/views/admin/ 的 profile.blade.php 或 profile.php
# 傳遞 $name = 'Tom' 給 view
return view('admin.profile', [ 'name' => 'Tom' ]);
return view('admin.profile')->with( 'name', 'Tom' );
# first() 第一個參數是 view 陣列，回傳第一個存在的 view。所以如果不存在 A，則回傳 B
# 適合用在 A 是使用者自訂 view，B 是系統預設 view
return view()->first( [ 'A', 'B' ], [ 'name' => 'Tom' ]);

# 所有 view 共享變數
# 在 AppServiceProvider 或新增的 Service Provider
use View;
public function boot() {
  # 所有的 view 都能使用 $name = 'Tom'
  View::share('name', 'Tom');
  # 或 View::share( [ 'name' => 'Tom' ] );
}
```

## View Composer

新增 ComposerServiceProvider

``` bash
php artisan make:provider ComposerServiceProvider
```

註冊 Service Provider

``` php
<?php
# config/app.php
'providers' => [
  App\Providers\ComposerServiceProvider::class,
],
```

### Closure 方式

``` php
<?php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use View;

class ComposerServiceProvider extends ServiceProvider
{
    public function boot()
    {
        # return view('dashboard') 時執行 Closure
        # 繫結變數 $title = 'Dashboard' 到 view dashboard
        # Closure 的 \Illuminate\View\View 可省略，只是爲了表明兩個 View 是不同的類別
        View::composer('dashboard', function (\Illuminate\View\View $view) {
          # 和 return view('dashboard')->with( [ 'title' => 'Dashboard' ] ) 一樣
          $view->with( [ 'title' => 'Dashboard' ] );
        });
    }
}
```

### class method 方式

``` php
<?php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use View;

class ComposerServiceProvider extends ServiceProvider
{
    public function boot()
    {
        # return view('dashboard') 時執行
        # app/Http/ViewComposers/DashboardComposer.php 的 compose()
        View::composer(
            'dashboard', 'App\Http\ViewComposers\DashboardComposer'
        );
    }
}

# app/Http/ViewComposers/DashboardComposer.php
<?php
namespace App\Http\ViewComposers;

use Illuminate\View\View;

class DashboardComposer
{
    public function compose(View $view)
    {
        # 和 return view('dashboard')->with( [ 'title' => 'Dashboard' ] ) 一樣
        $view->with( [ 'title' => 'Dashboard' ] );
    }
}
```

### 添加 Composer 到多個 view

``` php
<?php
# 把 Composer 添加到 view A 和 B
View::composer([ 'A', 'B' ], ...);
# 把一個 Composer 添加到 resources/views/admin/ 的所有 view
View::composer('admin/*', ...);
# 把一個 Composer 添加到所有 view
View::composer('*', ...);
```

### View::creator()

`View::creator()` 和 `View::composer()` 很類似，只是前者在 view 實例化之後立即失效而不是等到view 即將 render。說實在不懂有什麼差別

## 檢查 view 是否存在

``` php
<?php
use Illuminate\Support\Facades\View;
# 檢查 resources/views/emails/ 的 customer.blade.php 或 customer.php 是否存在
if (View::exists('emails.customer')) {
  // do something
}
```