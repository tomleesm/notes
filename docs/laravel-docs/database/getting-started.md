# 連線設定、原生 SQL 查詢、監聽、交易

## 資料庫連線設定

資料庫連線設定依照 config/database.php 而定，如下所示。`env(設定值, 預設值)` 回傳 `.env` 檔案的設定值，例如 `env('DB_FOREIGN_KEYS', true)` 回傳 `.env` 的 `DB_FOREIGN_KEYS` 的值，沒有的話回傳預設值 true

所以資料庫設定通常在 `.env` 只需要設定如下

```
DB_CONNECTION=mysql
DB_DATABASE=資料庫名稱
DB_USERNAME=使用者名稱
DB_PASSWORD=密碼
```

另外可以用另一種 `DATABASE_URL` 的語法設定，使用它的話會忽略其他的設定值：

`DATABASE_URL='driver://username:password@host:port/database?options'` 例如，

`DATABASE_URL='mysql://root:123456@127.0.0.1:3306/forge?charset=utf8mb4'`

``` php
<?php
return [
    # 設定使用哪個 connections 陣列的值，預設是 mysql
    'default' => env('DB_CONNECTION', 'mysql'),
    'connections' => [

        'sqlite' => [
            'driver' => 'sqlite',
            'url' => env('DATABASE_URL'),
            # DB_DATABASE = 完整的檔案路徑，例如 /home/tom/database.db，不是 ~/database.db
            # database_path('database.sqlite') 回傳網站根目錄加上 database/database.sqlite
            'database' => env('DB_DATABASE', database_path('database.sqlite')),
            'prefix' => '',
            'foreign_key_constraints' => env('DB_FOREIGN_KEYS', true),
        ]
    ]
];
```

### SQLite

#### 安裝
``` bash
sudo apt install sqlite3 php-sqlite3
```

#### 新增資料庫

用 touch 就能新增資料庫，一個檔案代表一個資料庫，另外副檔名可以隨你喜歡，省略也可以

``` bash
touch your/dir/database.sqlite3
```

### 讀寫分離

會了提高可靠度和降低主機負擔，需要在不同的主機上讀取和寫入資料庫，例如主機 A 的資料庫寫入(INSERT, UPDATE 等)，主機 B 上的資料庫讀取 (SELECT)。Laravel 的相關設定參考[Read & Write Connections](https://laravel.com/docs/6.x/database#read-and-write-connections)。雖然 write 是陣列，但是目前讀寫分離只支援一個寫入，讀取可以有兩個以上，而且讀取時會隨機分配連線

### 使用不同的資料庫連接

``` php
<?php
use Illuminate\Support\Facades\DB;
#  DB::connection() 的參數是 config/database.php connections 陣列的值
DB::connection('pgsql')->select('SELECT * FROM users WHERE id = ?', [ 1 ]);

# 如果 config/database.php 是這樣
'connections' => [
        'sqlite' => [
            'A' => [
                'database' => database_path('A.db')
            ],
            'B' => [
                'database' => database_path('B.db')
            ],
            'driver' => 'sqlite',
            'url' => env('DATABASE_URL'),
            'prefix' => '',
            'foreign_key_constraints' => env('DB_FOREIGN_KEYS', true),
        ]
# 讀取 database_path('A.db') 的資料庫
DB::connection('sqlite::A')->select('SELECT * FROM users WHERE id = ?', [ 1 ]);

# 使用 PDO 物件操作資料庫
$pdo = DB::connection('sqlite::A')->getPdo();
get_class($pdo); // return 'PDO'
```

## 原生 SQL 查詢

``` php
<?php
use Illuminate\Support\Facades\DB;

# SELECT
## DB::select(SQL, 參數陣列)，回傳值是陣列，每一個元素都是 stdClass 物件
## 用 ? 作爲參數的 placeholder
$result = DB::select('SELECT * FROM users WHERE id = ?', [ 1 ]);
## 或是命名綁定，注意參數陣列 id 沒有冒號
$result = DB::select('SELECT * FROM users WHERE id = :id', [ 'id' => 1 ]);
## $result 是陣列，每一個元素都是 stdClass 物件
$user = $result[0]; # $user->id 爲 1

# INSERT
## DB::insert(SQL, 參數陣列)，注意回傳值是 true，而不是 id 值
DB::insert('INSERT INTO users(name) VALUES (?)', [ 'Alice' ]);
DB::insert('INSERT INTO users(name) VALUES ( :name )', [ 'name' => $name ]);

# UPDATE
## DB::update(SQL, 參數陣列)，回傳更新的資料筆數
DB::update('UPDATE users SET name = ? WHERE id = ? ', [ 'Peter', 1 ]);
DB::update('UPDATE users SET name = :name WHERE id = :id ',
		    [ 'name' => 'Peter', 'id' => 1 ]);

# DELETE
## DB::delete(SQL, 參數陣列)，回傳刪除的資料筆數
DB::delete('DELETE FROM users WHERE id = ?', [ 1 ]);
DB::delete('DELETE FROM users WHERE id = :id', [ 'id' => 1 ]);

# 通用SQL：任何 SQL 都可以，但是一般用在 CREATE\DROP\ALTER TABLE
## DB::statement(SQL)，不管是什麼 SQL 都回傳 true
DB::statement('DROP TABLE users');
## 用在 SELECT，回傳值也是 true
DB::statement('SELECT * FROM users WHERE id = ?', [ 1 ]);
```

## 監聽

`DB::listen()` 監聽 SQL 的執行，方便用在 debug 和 log

``` php
<?php
use Illuminate\Support\Facades\DB;

class AppServiceProvider extends ServiceProvider
{
    public function boot() {
        DB::listen(function($query) {
            # $query->sql = SELECT * FROM users WHERE id = ?
            # $query->bindings = [ 0 => 1 ]
            # $query->time = 10 (單位是 ms)
            dd($query);
        });
    }
}
```

## 交易

### 自動

使用 `DB::transaction(Closure, 重試次數)` 把多個資料庫操作包起來。

- 執行成功，會自動 commit
- 執行失敗，也就是丟出 Exception，會自動 rollback

重試次數是指發生 deadlock 時的最大重試次數

``` php
<?php
use Illuminate\Support\Facades\DB;

DB::transaction(function() {
    DB::update('UPDATE users SET name = :name WHERE id = :id ',
		    [ 'name' => 'Peter', 'id' => 1 ]);
});
```

### 手動

``` php
<?php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

try {
    DB::beginTransaction();
    // 一些資料庫操作
    DB::commit();
} catch (Exception $e) {
    DB::rollBack();

    Log::error($e->getMessage());
}
```

注意：經實測，使用交易時，如果操作時有使用 `dd()`，例如在 AppServiceProvider 中的 `DB::listen()` 使用 `dd()`，交易會自動 rollback，只是 echo 的話沒有這個問題。所以最好在 `DB::commit()` 之後才使用 `dd()`