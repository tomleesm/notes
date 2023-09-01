# Mocking

## 介紹

在 Laravel 應用程式測試中，你可能希望「模擬」應用程式的某些功能的行為，從而避免該部分在測試中真正執行。例如：在 controller 執行過程中會觸發事件，您可能希望模擬事件監聽器，從而避免該事件在測試時真正執行。這允許你在僅測試 controller  HTTP 響應的情況時，而不必擔心觸發事件，因為事件偵聽器可以在它們自己的測試用例中進行測試。

Laravel 針對事件、任務和 Facades 的模擬，提供了開箱即用的輔助函數。這些函數基於 `Mockery` 封裝而成，使用非常方便，無需手動呼叫複雜的 `Mockery` 函數。

## 模擬對象

當模擬一個對象將通過 Laravel 的 [服務容器](/docs/laravel/10.x/container) 注入到應用中時，你將需要將模擬實例作為 `instance` 繫結到容器中。這將告訴容器使用對象的模擬實例，而不是構造對象的本身：

    use App\Service;
    use Mockery;
    use Mockery\MockInterface;

    public function test_something_can_be_mocked(): void
    {
        $this->instance(
            Service::class,
            Mockery::mock(Service::class, function (MockInterface $mock) {
                $mock->shouldReceive('process')->once();
            })
        );
    }

為了讓以上過程更便捷，你可以使用 Laravel 的基本測試用例類提供的 `mock` 方法。例如，下面的例子跟上面的例子的執行效果是一樣的：

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->mock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });



當你只需要模擬對象的幾個方法時，可以使用 `partialMock` 方法。 未被模擬的方法將在呼叫時正常執行：

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->partialMock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });

同樣，如果你想 [監控](http://docs.mockery.io/en/latest/reference/spies.html) 一個對象，Laravel 的基本測試用例類提供了一個便捷的 `spy` 方法作為 `Mockery::spy` 的替代方法。`spies` 與模擬類似。但是，`spies` 會記錄 `spy` 與被測試程式碼之間的所有互動，從而允許您在執行程式碼後做出斷言：

    use App\Service;

    $spy = $this->spy(Service::class);

    // ...

    $spy->shouldHaveReceived('process');

## Facades 模擬

與傳統靜態方法呼叫不同的是，[facades](/docs/laravel/10.x/facades) (包含的 [real-time facades](/docs/laravel/10.x/facades#real-time-facades)) 也可以被模擬。相較傳統的靜態方法而言，它具有很大的優勢，同時提供了與傳統依賴注入相同的可測試性。在測試中，你可能想在 controller 中模擬對 Laravel Facade 的呼叫。比如下面 controller 中的行為：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * 顯示該應用程式的所有使用者的列表。
         */
        public function index(): array
        {
            $value = Cache::get('key');

            return [
                // ...
            ];
        }
    }

我們可以使用 `shouldReceive` 方法模擬對 `Cache` Facade 的呼叫，該方法將返回一個 [Mockery](https://github.com/padraic/mockery) 模擬的實例。由於 Facades 實際上是由 Laravel [服務容器](/docs/laravel/10.x/container) 解析和管理的，因此它們比傳統的靜態類具有更好的可測試性。例如，讓我們模擬對 `Cache` Facade 的 `get` 方法的呼叫：

    <?php

    namespace Tests\Feature;

    use Illuminate\Support\Facades\Cache;
    use Tests\TestCase;

    class UserControllerTest extends TestCase
    {
        public function test_get_index(): void
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $response = $this->get('/users');

            // ...
        }
    }

> **注意**
> 你不應該模擬 `Request` facade。相反，在運行測試時將你想要的輸入傳遞到 [HTTP 測試方法](/docs/laravel/10.x/http-tests) 中，例如 `get` 和 `post` 方法。同樣也不要模擬 `Config` facade，而是在測試中呼叫 `Config::set` 方法。


### Facade Spies

如果你想 [監控](http://docs.mockery.io/en/latest/reference/spies.html) 一個 facade，你可以在相應的 facade 上呼叫 `spy` 方法。`spies` 類似於 `mocks`；但是，`spies` 會記錄 `spy` 和被測試程式碼之間的所有互動，允許你在程式碼執行後做出斷言：

    use Illuminate\Support\Facades\Cache;

    public function test_values_are_be_stored_in_cache(): void
    {
        Cache::spy();

        $response = $this->get('/');

        $response->assertStatus(200);

        Cache::shouldHaveReceived('put')->once()->with('name', 'Taylor', 10);
    }

## 設定時間

當我們測試時，有時可能需要修改諸如 `now` 或 `Illuminate\Support\Carbon::now()` 之類的助手函數返回的時間。 Laravel 的基本功能測試類包中包含了一些助手函數，可以讓你設定當前時間：

    use Illuminate\Support\Carbon;

    public function test_time_can_be_manipulated(): void
    {
        // 設定未來的時間...
        $this->travel(5)->milliseconds();
        $this->travel(5)->seconds();
        $this->travel(5)->minutes();
        $this->travel(5)->hours();
        $this->travel(5)->days();
        $this->travel(5)->weeks();
        $this->travel(5)->years();

        // 設定過去的時間...
        $this->travel(-5)->hours();

        // 設定一個確切的時間...
        $this->travelTo(now()->subHours(6));

        // 返回現在的時間...
        $this->travelBack();
    }

你還可以為各種設定時間方法寫一個閉包。閉包將在指定的時間被凍結呼叫。一旦閉包執行完畢，時間將恢復正常:

    $this->travel(5)->days(function () {
        // 在5天之後測試...
    });
    
    $this->travelTo(now()->subDays(10), function () {
        // 在指定的時間測試...
    });



`freezeTime` 方法可用於凍結當前時間。與之類似地，`freezeSecond` 方法也可以秒為單位凍結當前時間：

    use Illuminate\Support\Carbon;

    // 凍結時間並在完成後恢復正常時間...
    $this->freezeTime(function (Carbon $time) {
        // ...
    });

    // 凍結以秒為單位的時間並在完成後恢復正常時間...
    $this->freezeSecond(function (Carbon $time) {
        // ...
    })

正如你期望的一樣，上面討論的所有方法都主要用於測試對時間敏感的應用程式的行為，比如鎖定論壇上不活躍的帖子:

    use App\Models\Thread;
    
    public function test_forum_threads_lock_after_one_week_of_inactivity()
    {
        $thread = Thread::factory()->create();
        
        $this->travel(1)->week();
        
        $this->assertTrue($thread->isLockedByInactivity());
    }

