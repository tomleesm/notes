# Redis

## 簡介

[Redis](https://redis.io) 是一個開放原始碼的, 高級鍵值對儲存資料庫。 保護的資料庫類型有

- [字串](https://redis.io/topics/data-types#strings)
- [hash](https://redis.io/topics/data-types#hashes)
- [列表](https://redis.io/topics/data-types#lists)
- [集合](https://redis.io/topics/data-types#sets)
- [有序集合](https://redis.io/topics/data-types#sorted-sets)

在將 Redis 與 Laravel 一起使用前，我們鼓勵你通過 PECL 安裝並使用 [PhpRedis](https://github.com/phpredis/phpredis) ，儘管擴展安裝起來更複雜，但對於大量使用 Redis 的應用程式可能會帶來更好的性能。如果你使用 [Laravel Sail](/docs/laravel/10.x/sail), 這個擴展已經事先在你的 Docker 容器中安裝完成。

如果你不能安裝 PHPRedis 擴展，你或許可以使用 composer 安裝 predis/predis 包。Predis 是一個完全用 PHP 編寫的 Redis 客戶端，不需要任何額外的擴展：

```shell
composer require predis/predis
```

## 組態

在你的應用中組態 Redis 資訊，你要在 `config/database.php` 檔案中進行組態。在該檔案中，你將看到一個 `Redis` 陣列包含了你的 Redis 組態資訊。

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

        'cache' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_CACHE_DB', 1),
        ],

    ],



在你的組態檔案裡定義的每個 Redis 伺服器，除了用 URL 來表示的 Redis 連接，都必需要指定名稱 、 host （主機）和 port （連接埠）欄位：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'url' => 'tcp://127.0.0.1:6379?database=0',
        ],

        'cache' => [
            'url' => 'tls://user:password@127.0.0.1:6380?database=1',
        ],

    ],

#### 組態連接方案

默認情況下，Redis 客戶端使用 `tcp` 方案連接 Redis 伺服器。另外，你也可以在你的 Redis 服務組態陣列中指定一個 `scheme` 組態項，來使用 TLS/SSL 加密：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'scheme' => 'tls',
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

    ],

### 叢集

如果你的應用使用 Redis 叢集，你應該在 Redis 組態檔案中用 `clusters` 鍵來定義叢集。這個組態鍵默認沒有，所以你需要在 `config/database.php` 組態檔案中手動建立：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD'),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

默認情況下，叢集可以在節點上實現客戶端分片，允許你實現節點池以及建立大量可用記憶體。這裡要注意，客戶端共享不會處理失敗的情況；因此，這個功能主要適用於從另一個主資料庫獲取的快取資料。
如果要使用 Redis 原生叢集，需要把 `config/database.php` 組態檔案下的 `options.cluster` 組態項的值設定為 `redis` ：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],

        'clusters' => [
            // ...
        ],

    ],



### Predis

要使用 Predis 擴展去連接 Redis， 請確保環境變數 `REDIS_CLIENT` 的值為 `predis` ：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'predis'),

        // ...
    ],

除默認的 `host` ，`port` ，`database` 和 `password` 這些服務組態選項外， Predis 還支援為每個 Redis 伺服器定義其它的 [連接參數](https://github.com/nrk/predis/wiki/Connection-Parameters)。如果要使用這些額外的組態項，可以在 config/database.php 組態檔案中將任意選項新增到 Redis 伺服器組態內：

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

#### Redis Facade 別名

Laravel 的 `config/app.php` 組態檔案包含了 `aliases` 陣列，該陣列可用於定義通過框架註冊的所有類別名。方便起見，Laravel 提供了一份包含了所有 facade 的別名入口；不過，`Redis` 別名不能在這裡使用，因為這與 phpredis 擴展提供的 `Redis` 類名衝突。如果正在使用 `Predis` 客戶端並確實想要用這個別名，你可以在 `config/app.php` 組態檔案中取消對此別名的註釋。

    'aliases' => Facade::defaultAliases()->merge([
        'Redis' => Illuminate\Support\Facades\Redis::class,
    ])->toArray(),

### phpredis

Laravel 默認使用 phpredis 擴展與 Redis 通訊。Laravel 用於與 Redis 通訊的客戶端由 `redis.client` 組態項決定，這個組態通常為環境變數 `REDIS_CLIENT` 的值：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        // 重設 Redis 組態項...
    ],



除默認的 `scheme` , `host` , `port `, `database` 和 `password` 的伺服器組態選項外，`phpredis` 還支援以下額外的連接參數：`name` , `persistent `, `persistent_id `, `prefix `, `read_timeout `, `retry_interval `, `timeout` 和 `context `。 你可以在 `config/database.php` 組態檔案中將任意選項新增到 Redis 伺服器組態內：

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
        'context' => [
            // 'auth' => ['username', 'secret'],
            // 'stream' => ['verify_peer' => false],
        ],
    ],

#### phpredis 序列化和壓縮

phpredis 擴展可以組態使用各種序列化和壓縮演算法。可以通過設定 Redis 組態中的 `options` 陣列進行組態：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'serializer' => Redis::SERIALIZER_MSGPACK,
            'compression' => Redis::COMPRESSION_LZ4,
        ],

        // 重設 Redis 組態項...
    ],

當前支援的序列化演算法包括： `Redis::SERIALIZER_NONE` （默認）, `Redis::SERIALIZER_PHP` , `Redis::SERIALIZER_JSON` , `Redis::SERIALIZER_IGBINARY` 和 `Redis::SERIALIZER_MSGPACK` 。

支援的壓縮演算法包括： `Redis::COMPRESSION_NONE` （默認）, `Redis::COMPRESSION_LZF` , `Redis::COMPRESSION_ZSTD` 和 `Redis::COMPRESSION_LZ4` 。

## 與 Redis 互動

你可以通過呼叫 `Redis` [facade](/docs/laravel/10.x/facades) 上的各種方法來與Redis進行互動。 `Redis` facade 支援動態方法，所以你可以在facade上呼叫各種 [Redis 命令](https://redis.io/commands) ,這些命令將直接傳遞給 Redis 。 在本例中，我們將呼叫 `Redis` facade 的 `get` 方法，來呼叫 Redis `GET` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * 顯示給定使用者的組態檔案
         */
        public function show(string $id): View
        {
            return view('user.profile', [
                'user' => Redis::get('user:profile:'.$id)
            ]);
        }
    }



如上所述，你可以在 `Redis` facade 上呼叫任意 Redis 命令。 Laravel 使用魔術方法將命令傳遞給 Redis 伺服器。如果一個 Redis 命令需要參數，則應將這些參數傳遞給 `Redis` facade 的相應方法：

    use Illuminate\Support\Facades\Redis;

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

或者，你也可以使用 `Redis` facade 上的 `command` 方法將命令傳遞給伺服器，它接受命令的名稱作為其第一個參數，並將值的陣列作為其第二個參數：

    $values = Redis::command('lrange', ['name', 5, 10]);

#### 使用多個 Redis 連接

你應用裡的 `config/database.php` 組態檔案允許你去定義多個 Redis 連接或者伺服器。你可以使用 `Redis` facade 上的 `connection` 方法獲得指定的 Redis 連接：

    $redis = Redis::connection('connection-name');

要獲取獲取一個默認的 Redis 連接，你可以呼叫 `connection` 方法時，不帶任何參數：

    $redis = Redis::connection();

### 事務

`Redis` facade 上的 `transaction` 方法對 Redis 原生的 `MULTI` 和 `EXEC` 命令進行了封裝。 `transaction` 方法接受一個閉包作為其唯一參數。這個閉包將接收一個 Redis 連接實例，並可能向這個實例發出想要的任何命令。閉包中發出的所有 Redis 命令都將在單個原子性事務中執行：

    use Redis;
    use Illuminate\Support\Facades;

    Facades\Redis::transaction(function (Redis $redis) {
        $redis->incr('user_visits', 1);
        $redis->incr('total_visits', 1);
    });

> **注意**  
> 定義一個 Redis 事務時，你不能從 Redis 連接中獲取任何值。請記住，事務是作為單個原子性操作執行的，在整個閉包執行完其命令之前，不會執行該操作。



#### Lua 指令碼

`eval` 方法提供了另外一種原子性執行多條 Redis 命令的方式。但是，`eval` 方法的好處是能夠在操作期間與 Redis 鍵值互動並檢查它們。 Redis 指令碼是用 [Lua 程式語言](https://www.lua.org/) 編寫的。

`eval` 方法一開始可能有點令人勸退，所以我們將用一個基本示例來明確它的使用方法。 `eval` 方法需要幾個參數。第一，在方法中傳遞一個 Lua 指令碼（作為一個字串）。第二，在方法中傳遞指令碼互動中用到的鍵的數量（作為一個整數）。第三，在方法中傳遞所有鍵名。最後，你可以傳遞一些指令碼中用到的其他參數。

在本例中，我們要對第一個計數器進行遞增，檢查它的新值，如果該計數器的值大於 5，那麼遞增第二個計數器。最終，我們將返回第一個計數器的值：

    $value = Redis::eval(<<<'LUA'
        local counter = redis.call("incr", KEYS[1])

        if counter > 5 then
            redis.call("incr", KEYS[2])
        end

        return counter
    LUA, 2, 'first-counter', 'second-counter');

> **注意**  
> 請參考 [Redis 文件](https://redis.io/commands/eval) 更多關於 Redis 指令碼的資訊。

### 管道命令

當你需要執行很多個 Redis 命令時，你可以使用 `pipeline` 方法一次性提交所有命令，而不需要每條命令都與 Redis 伺服器建立一次網路連線。 `pipeline` 方法只接受一個參數：接收一個 Redis 實例的閉包。你可以將所有命令發給這個 Redis 實例，它們將同時傳送到 Redis 伺服器，以減少到伺服器的網路訪問。這些命令仍然會按照發出的順序執行：

    use Redis;
    use Illuminate\Support\Facades;

    Facades\Redis::pipeline(function (Redis $pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });



## 發佈 / 訂閱

Laravel 為 Redis 的 `publish` 和 `subscribe` 命令提供了方便的介面。你可以用這些 Redis 命令監聽指定「頻道」上的消息。你也可以從一個應用程式發消息給另一個應用程式，哪怕它是用其它程式語言開發的，讓應用程式和處理程序之間能夠輕鬆進行通訊。

首先，用 `subscribe` 方法設定一個頻道監聽器。我們將這個方法呼叫放到一個 [Artisan 命令](/docs/laravel/10.x/artisan) 中，因為呼叫 `subscribe` 方法會啟動一個常駐處理程序：

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * 控制台命令的名稱和簽名
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * 控制台命令的描述
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * 執行控制台命令
         */
        public function handle(): void
        {
            Redis::subscribe(['test-channel'], function (string $message) {
                echo $message;
            });
        }
    }

現在我們可以使用 `publish` 方法將消息發佈到頻道：

    use Illuminate\Support\Facades\Redis;

    Route::get('/publish', function () {
        // ...

        Redis::publish('test-channel', json_encode([
            'name' => 'Adam Wathan'
        ]));
    });

#### 萬用字元訂閱

使用 `psubscribe` 方法，你可以訂閱一個萬用字元頻道，用來獲取所有頻道中的所有消息，頻道名稱將作為第二個參數傳遞給提供的回呼閉包：

    Redis::psubscribe(['*'], function (string $message, string $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function (string $message, string $channel) {
        echo $message;
    });

