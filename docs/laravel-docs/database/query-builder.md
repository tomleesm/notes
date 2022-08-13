# Query Builder

## 獲得資料

``` php
<?php
use Illuminate\Support\Facades\DB;

# 資料表 users 的所有資料，回傳 Illuminate\Support\Collection，每一個元素都是 stdClass 物件
# select * from "users"
DB::table('users')->get();

# 資料表 users 的一筆資料，回傳 stdClass 物件
# select * from "users" limit 1
DB::table('users')->first();

# 資料表 users 的一筆資料的欄位 name，回傳字串
# select "name" from "users" limit 1
DB::table('users')->value('name');

# 資料表 users 的 id = 1 資料，回傳 stdClass
# select * from "users" where "id" = 1 limit 1
DB::table('users')->find(1);

# 資料表 users 的欄位 name 值，回傳 Illuminate\Support\Collection，每一個元素都是 stdClass 物件
# select "name" from "users"
DB::table('users')->pluck('name');

# 資料表 users 的兩個欄位，第一個欄位是值，第二個是 key，回傳 Illuminate\Support\Collection
# 注意回傳值是陣列如下，而且 pluck() 最多兩個參數，多的會忽略
/**
[
  // id => name
  1 => 'Tom',
  2 => 'Peter'
]
**/
# select "name", "id" from "users"
DB::table('users')->pluck('name', 'id');
```

### chunk 一次抓一小塊

常見的錯誤是初期階段網站資料很少，所以執行 `SELECT * FROM users` 依然很快。等到網站發展到後面，一個資料表有幾千萬筆資料時，同樣的資料庫查詢造成效能問題。

`chunk(一次抓幾筆資料, Closure)`

``` php
<?php
use Illuminate\Support\Facades\DB;

# 資料表 users 一次抓 100 筆，變成 Closure 的參數 $users 進一步處理
# 使用 chunk() 時一定要有 orderBy()，否則會丟出 RuntimeException
/**
select * from "users" order by "id" asc limit 100 offset 0
select * from "users" order by "id" asc limit 100 offset 100
select * from "users" order by "id" asc limit 100 offset 200
...
**/
DB::table('users')->orderBy('id')->chunk(100, function($users) {
  foreach($users as $user) {
    // do something...

    # return false 會中斷 chunk() 執行
    if($user->id === 1230)
      return false;
  }
});

# 和上面的 chunk() 做的事一樣，但是 SQL 不一樣。因爲使用 WHERE 子句過濾，所以速度較快
/**
select * from "users" order by "id" asc limit 100
select * from "users" where "id" > 100 order by "id" asc limit 100
select * from "users" where "id" > 200 order by "id" asc limit 100
...
**/
DB::table('users')->chunkById(100, function($users) {
  # 注意：在 Closure 內修改資料，最好使用 chunkById() 或如上的 orderBy('id')
  # 如果 orderBy('name')，然後 Closure 內執行的是 UPDATE users SET "name" = 
  # 造成有些資料被遺漏了。所以改成依照 id 這種不會修改的欄位來排序
  foreach($users as $user) {
    // do something...

    # return false 會中斷 chunk() 執行
    if($user->id === 1230)
      return false;
  }
});
```