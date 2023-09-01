``` php
// 1. EventServiceProvider 類裡的 $listen 屬性
protected $listen =['App\Events\OrderShipped' => ['App\Listeners\SendShipmentNotification']];
// 2. 生成監聽類
php artisan event:generate

// 觸發命令
Event::dispatch($event, $payload = [], $halt = false);
event($event, $payload = [], $halt = false);
// 觸發命令並等待
Event::until($event, $payload = []);
// 註冊一個事件監聽器.
// void listen(string|array $events, mixed $listener, int $priority)
Event::listen('App\Events\UserSignup', function($bar){});
Event::listen('event.*', function($bar){}); // 萬用字元監聽器
Event::listen('foo.bar', 'FooHandler', 10);
Event::listen('foo.bar', 'BarHandler', 5);
// 你可以直接在處理邏輯中返回 false 來停止一個事件的傳播.
Event::listen('foor.bar', function($event){ return false; });
Event::subscribe('UserEventHandler');
// 獲取所有監聽者
Event::getListeners($eventName);
// 移除事件及其對應的監聽者
Event::forget($event);
// 將事件推入堆疊中等待執行
Event::push($event, $payload = []);
// 移除指定的堆疊事件
Event::flush($event);
// 移除所有堆疊中的事件
Event::forgetPushed();
```
