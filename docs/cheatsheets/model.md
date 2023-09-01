## 基礎使用

``` php
// 定義一個 Eloquent 模型
class User extends Model {}
// 生成一個 Eloquent 模型
php artisan make:model User
// 生成一個 Eloquent 模型的時候，順便生成遷移檔案
php artisan make:model User --migration OR -m
// 生成一個 Eloquent 模型的時候，順便生成遷移檔案、控製器（或資源控製器）
php artisan make:model User -mc[r]
// 指定一個自訂的資料表名稱
class User extends Model {
  protected $table = 'my_users';
}
```

## More

``` php
//新增一條新資料
Model::create(array('key' => 'value'));
// 通過屬性找到第一條相匹配的資料或創造一條新資料
Model::firstOrCreate(array('key' => 'value'));
// 通過屬性找到第一條相匹配的資料或實例化一條新資料
Model::firstOrNew(array('key' => 'value'));
// 通過屬性找到相匹配的資料並更新，如果不存在即建立
Model::updateOrCreate(array('search_key' => 'search_value'), array('key' => 'value'));
// 使用屬性的陣列來填充一個模型, 用的時候要小心「Mass Assignment」安全問題 !
Model::fill($attributes);
Model::destroy(1);
Model::all();
Model::find(1);
// 使用雙主鍵進行尋找
Model::find(array('first', 'last'));
// 尋找失敗時拋出異常
Model::findOrFail(1);
// 使用雙主鍵進行尋找, 失敗時拋出異常
Model::findOrFail(array('first', 'last'));
Model::where('foo', '=', 'bar')->get();
Model::where('foo', '=', 'bar')->first();
Model::where('foo', '=', 'bar')->exists();
// 動態屬性尋找
Model::whereFoo('bar')->first();
// 尋找失敗時拋出異常
Model::where('foo', '=', 'bar')->firstOrFail();
Model::where('foo', '=', 'bar')->count();
Model::where('foo', '=', 'bar')->delete();
// 輸出原始的查詢語句
Model::where('foo', '=', 'bar')->toSql();
Model::whereRaw('foo = bar and cars = 2', array(20))->get();
Model::on('connection-name')->find(1);
Model::with('relation')->get();
Model::all()->take(10);
Model::all()->skip(10);
// 默認的 Eloquent 排序是上升排序
Model::all()->sortBy('column');
Model::all()->sortDesc('column');

// 查詢 json 資料
Model::where('options->language', 'en')->get(); # 欄位是字串
Model::whereJsonContains('options->languages', 'en')->get(); # 欄位是陣列
Model::whereJsonLength('options->languages', 0)->get(); # 欄位長度為 0
Model::whereJsonDoesntContain('options->languages', 'en')->get(); # 欄位是陣列, 不包含
```

## 軟刪除

``` php
Model::withTrashed()->where('cars', 2)->get();
// 在查詢結果中包括帶被軟刪除的模型
Model::withTrashed()->where('cars', 2)->restore();
Model::where('cars', 2)->forceDelete();
// 尋找只帶有軟刪除的模型
Model::onlyTrashed()->where('cars', 2)->get();
```

## 模型關聯

``` php
// 一對一 - User::phone()
return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
// 一對一 - Phone::user(), 定義相對的關聯
return $this->belongsTo('App\User', 'foreign_key', 'other_key');

// 一對多 - Post::comments()
return $this->hasMany('App\Comment', 'foreign_key', 'local_key');
//  一對多 - Comment::post()
return $this->belongsTo('App\Post', 'foreign_key', 'other_key');

// 多對多 - User::roles();
return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'role_id');
// 多對多 - Role::users();
return $this->belongsToMany('App\User');
// 多對多 - Retrieving Intermediate Table Columns
$role->pivot->created_at;
// 多對多 - 中介表欄位
return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');
// 多對多 - 自動維護 created_at 和 updated_at 時間戳
return $this->belongsToMany('App\Role')->withTimestamps();

// 遠層一對多 - Country::posts(), 一個 Country 模型可能通過中介的 Users
// 模型關聯到多個 Posts 模型(User::country_id)
return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');

// 多型關聯 - Photo::imageable()
return $this->morphTo();
// 多型關聯 - Staff::photos()
return $this->morphMany('App\Photo', 'imageable');
// 多型關聯 - Product::photos()
return $this->morphMany('App\Photo', 'imageable');
// 多型關聯 - 在 AppServiceProvider 中註冊你的「多型對照表」
Relation::morphMap([
    'Post' => App\Post::class,
    'Comment' => App\Comment::class,
]);

// 多型多對多關聯 - 涉及資料庫表: posts,videos,tags,taggables
// Post::tags()
return $this->morphToMany('App\Tag', 'taggable');
// Video::tags()
return $this->morphToMany('App\Tag', 'taggable');
// Tag::posts()
return $this->morphedByMany('App\Post', 'taggable');
// Tag::videos()
return $this->morphedByMany('App\Video', 'taggable');

// 尋找關聯
$user->posts()->where('active', 1)->get();
// 獲取所有至少有一篇評論的文章...
$posts = App\Post::has('comments')->get();
// 獲取所有至少有三篇評論的文章...
$posts = Post::has('comments', '>=', 3)->get();
// 獲取所有至少有一篇評論被評分的文章...
$posts = Post::has('comments.votes')->get();
// 獲取所有至少有一篇評論相似於 foo% 的文章
$posts = Post::whereHas('comments', function ($query) {
    $query->where('content', 'like', 'foo%');
})->get();

// 預載入
$books = App\Book::with('author')->get();
$books = App\Book::with('author', 'publisher')->get();
$books = App\Book::with('author.contacts')->get();

// 延遲預載入
$books->load('author', 'publisher');

// 寫入關聯模型
$comment = new App\Comment(['message' => 'A new comment.']);
$post->comments()->save($comment);
// Save 與多對多關聯
$post->comments()->saveMany([
    new App\Comment(['message' => 'A new comment.']),
    new App\Comment(['message' => 'Another comment.']),
]);
$post->comments()->create(['message' => 'A new comment.']);

// 更新「從屬」關聯
$user->account()->associate($account);
$user->save();
$user->account()->dissociate();
$user->save();

// 附加多對多關係
$user->roles()->attach($roleId);
$user->roles()->attach($roleId, ['expires' => $expires]);
// 從使用者上移除單一身份...
$user->roles()->detach($roleId);
// 從使用者上移除所有身份...
$user->roles()->detach();
$user->roles()->detach([1, 2, 3]);
$user->roles()->attach([1 => ['expires' => $expires], 2, 3]);

// 任何不在給定陣列中的 IDs 將會從中介表中被刪除。
$user->roles()->sync([1, 2, 3]);
// 你也可以傳遞中介表上該 IDs 額外的值：
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

## 事件

``` php
Model::retrieved(function($model){});
Model::creating(function($model){});
Model::created(function($model){});
Model::updating(function($model){});
Model::updated(function($model){});
Model::saving(function($model){});
Model::saved(function($model){});
Model::deleting(function($model){});
Model::deleted(function($model){});
Model::restoring(function($model){});
Model::restored(function($model){});
Model::observe(new FooObserver);
```

## Eloquent 組態資訊

``` php
// 關閉模型插入或更新操作引發的 「mass assignment」異常
Eloquent::unguard();
// 重新開啟「mass assignment」異常拋出功能
Eloquent::reguard();
```
