# Controllers

新增一般的 controller

``` bash
php artisan make:controller Namespace/Controller名稱
```

## 相依性注入

``` php
<?php  
use App\Repositories\UserRepository;

class UserController extends Controller
{
    protected $user;

    # 透過建構式注入
    public function __construct(UserRepository $user)
    {
        $this->user = $user;
    }

    # 透過 method 參數注入
    public function show(UserRepository $user)
    {
        $user->doSomething();
    }
}

// UserRepository 只是一般的 class
namespace App\Repositories;

class UserRepository
{
    public function doSomething()
    {
        return 'doSomething()';
    }
}
```