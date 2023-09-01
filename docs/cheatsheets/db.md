``` php
基本使用
DB::connection('connection_name');
// 運行資料庫查詢語句
$results = DB::select('select * from users where id = ?', [1]);
$results = DB::select('select * from users where id = :id', ['id' => 1]);
// 運行普通語句
DB::statement('drop table users');
// 監聽查詢事件
DB::listen(function($sql, $bindings, $time) { code_here; });
// 資料庫事務處理
DB::transaction(function() {
    DB::table('users')->update(['votes' => 1]);
    DB::table('posts')->delete();
});
DB::beginTransaction();
DB::rollBack();
DB::commit();

// 獲取表前綴
DB::getTablePrefix()
查詢語句構造器 
// 取得資料表的所有行
DB::table('name')->get();
// 取資料表的部分資料
DB::table('users')->chunk(100, function($users) {
  foreach ($users as $user) {
      //
  }
});
// 取回資料表的第一條資料
$user = DB::table('users')->where('name', 'John')->first();
DB::table('name')->first();
// 從單行中取出單列資料
$name = DB::table('users')->where('name', 'John')->pluck('name');
DB::table('name')->pluck('column');
// 取多行資料的「列資料」陣列
$roles = DB::table('roles')->lists('title');
$roles = DB::table('roles')->lists('title', 'name');
// 指定一個選擇欄位
$users = DB::table('users')->select('name', 'email')->get();
$users = DB::table('users')->distinct()->get();
$users = DB::table('users')->select('name as user_name')->get();
// 新增一個選擇欄位到一個已存在的查詢語句中
$query = DB::table('users')->select('name');
$users = $query->addSelect('age')->get();
// 使用 Where 運算子
$users = DB::table('users')->where('votes', '>', 100)->get();
$users = DB::table('users')
              ->where('votes', '>', 100)
              ->orWhere('name', 'John')
              ->get();
$users = DB::table('users')
      ->whereBetween('votes', [1, 100])->get();
$users = DB::table('users')
      ->whereNotBetween('votes', [1, 100])->get();
$users = DB::table('users')
      ->whereIn('id', [1, 2, 3])->get();
$users = DB::table('users')
      ->whereNotIn('id', [1, 2, 3])->get();
$users = DB::table('users')
      ->whereNull('updated_at')->get();
DB::table('name')->whereNotNull('column')->get();
// 動態的 Where 欄位
$admin = DB::table('users')->whereId(1)->first();
$john = DB::table('users')
      ->whereIdAndEmail(2, 'john@doe.com')
      ->first();
$jane = DB::table('users')
      ->whereNameOrAge('Jane', 22)
      ->first();
// Order By, Group By, 和 Having
$users = DB::table('users')
      ->orderBy('name', 'desc')
      ->groupBy('count')
      ->having('count', '>', 100)
      ->get();
DB::table('name')->orderBy('column')->get();
DB::table('name')->orderBy('column','desc')->get();
DB::table('name')->having('count', '>', 100)->get();
// 偏移 & 限制
$users = DB::table('users')->skip(10)->take(5)->get();
Joins 
// 基本的 Join 聲明語句
DB::table('users')
    ->join('contacts', 'users.id', '=', 'contacts.user_id')
    ->join('orders', 'users.id', '=', 'orders.user_id')
    ->select('users.id', 'contacts.phone', 'orders.price')
    ->get();
// Left Join 聲明語句
DB::table('users')
->leftJoin('posts', 'users.id', '=', 'posts.user_id')
->get();
// select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
DB::table('users')
    ->where('name', '=', 'John')
    ->orWhere(function($query) {
        $query->where('votes', '>', 100)
              ->where('title', '<>', 'Admin');
    })
    ->get();
聚合 
$users = DB::table('users')->count();
$price = DB::table('orders')->max('price');
$price = DB::table('orders')->min('price');
$price = DB::table('orders')->avg('price');
$total = DB::table('users')->sum('votes');
原始表達句
$users = DB::table('users')
                   ->select(DB::raw('count(*) as user_count, status'))
                   ->where('status', '<>', 1)
                   ->groupBy('status')
                   ->get();
// 返回行
DB::select('select * from users where id = ?', array('value'));
DB::insert('insert into foo set bar=2');
DB::update('update foo set bar=2');
DB::delete('delete from bar');
// 返回 void
DB::statement('update foo set bar=2');
// 在聲明語句中加入原始的表示式
DB::table('name')->select(DB::raw('count(*) as count, column2'))->get();
Inserts / Updates / Deletes / Unions / Pessimistic Locking
// 插入
DB::table('users')->insert(
  ['email' => 'john@example.com', 'votes' => 0]
);
$id = DB::table('users')->insertGetId(
  ['email' => 'john@example.com', 'votes' => 0]
);
DB::table('users')->insert([
  ['email' => 'taylor@example.com', 'votes' => 0],
  ['email' => 'dayle@example.com', 'votes' => 0]
]);
// 更新
DB::table('users')
          ->where('id', 1)
          ->update(['votes' => 1]);
DB::table('users')->increment('votes');
DB::table('users')->increment('votes', 5);
DB::table('users')->decrement('votes');
DB::table('users')->decrement('votes', 5);
DB::table('users')->increment('votes', 1, ['name' => 'John']);
// 刪除
DB::table('users')->where('votes', '<', 100)->delete();
DB::table('users')->delete();
DB::table('users')->truncate();
// 集合
 // unionAll() 方法也是可供使用的，呼叫方式與 union 相似
$first = DB::table('users')->whereNull('first_name');
$users = DB::table('users')->whereNull('last_name')->union($first)->get();
// 消極鎖
DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```
