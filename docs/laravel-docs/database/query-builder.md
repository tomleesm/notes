# Query Builder

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