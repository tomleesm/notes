``` php
// 獲取快取對象，約等於 Cache
cache()
// 注意 5.8 快取單位為「秒」，之前版本為「分」
Cache::put('key', 'value', $seconds);
// 未設定過期時間將永久有效
Cache::put('key',  'value'); 
Cache::add('key', 'value', $seconds);
Cache::forever('key', 'value');
Cache::sear('key', function(){ return 'value' });
Cache::remember('key', $seconds, function(){ return 'value' });
Cache::rememberForever('key', function(){ return 'value' });
Cache::forget('key');
Cache::has('key');
Cache::get('key');
Cache::get('key', 'default');
Cache::get('key', function(){ return 'default'; });
// 取到資料之後再刪除它
Cache::pull('key'); 
// 清空所有快取
Cache::flush();
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
Cache::tags('my-tag')->put('key','value', $seconds);
Cache::tags('my-tag')->has('key');
Cache::tags('my-tag')->get('key');
Cache::tags(['people',  'artists'])->put('John',  $john,  $seconds);
Cache::tags('my-tag')->forget('key');
Cache::tags('my-tag')->flush();
Cache::tags(['people',  'authors'])->flush();
Cache::section('group')->put('key', $value);
Cache::section('group')->get('key');
Cache::section('group')->flush();
Cache::tags(['people',  'artists'])->put('John',  $john,  $seconds);
// 輔助函數
cache('key');
cache(['key' => 'value'], $seconds);
cache(['key' => 'value'], now()->addMinutes(10));
cache()->remember('users',  $seconds,  function() { return  User::all(); });
// 指定快取儲存
Cache::store('file')->get('foo');
Cache::store('redis')->put('name',  'Jack',  600);  // 10 分鐘
// 事件
'Illuminate\Cache\Events\CacheHit' => ['App\Listeners\LogCacheHit',],
'Illuminate\Cache\Events\CacheMissed' => ['App\Listeners\LogCacheMissed',],
'Illuminate\Cache\Events\KeyForgotten' => ['App\Listeners\LogKeyForgotten',],
'Illuminate\Cache\Events\KeyWritten' => ['App\Listeners\LogKeyWritten',],
```
