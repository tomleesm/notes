``` php
// 建立指定資料表
 Schema::create('table', function($table)
{
  $table->increments('id');
});
// 指定一個連接
 Schema::connection('foo')->create('table', function($table){});
// 通過給定的名稱來重新命名資料表
 Schema::rename($from, $to);
// 移除指定資料表
 Schema::drop('table');
// 當資料表存在時, 將指定資料表移除
 Schema::dropIfExists('table');
// 判斷資料表是否存在
 Schema::hasTable('table');
// 判斷資料表是否有該列
 Schema::hasColumn('table', 'column');
// 獲取資料表欄位列表
 Schema::getColumnListing('table');
// 更新一個已存在的資料表
 Schema::table('table', function($table){});
// 重新命名資料表的列
$table->renameColumn('from', 'to');
// 移除指定的資料表列
$table->dropColumn(string|array);
// 指定資料表使用的儲存引擎
$table->engine = 'InnoDB';
// 欄位順序，只能在 MySQL 中才能用
$table->string('name')->after('email');
```

## 索引

``` php
$table->string('column')->unique();
$table->primary('column');
// 建立一個雙主鍵
$table->primary(array('first', 'last'));
$table->unique('column');
$table->unique('column', 'key_name');
// 建立一個雙唯一性索引
$table->unique(array('first', 'last'));
$table->unique(array('first', 'last'), 'key_name');
$table->index('column');
$table->index('column', 'key_name');
// 建立一個雙索引
$table->index(array('first', 'last'));
$table->index(array('first', 'last'), 'key_name');
$table->dropPrimary(array('column'));
$table->dropPrimary('table_column_primary');
$table->dropUnique(array('column'));
$table->dropUnique('table_column_unique');
$table->dropIndex(array('column'));
$table->dropIndex('table_column_index');
```

## 外部索引鍵

``` php
$table->foreign('user_id')->references('id')->on('users');
$table->foreign('user_id')->references('id')->on('users')->onDelete('cascade'|'restrict'|'set null'|'no action');
$table->foreign('user_id')->references('id')->on('users')->onUpdate('cascade'|'restrict'|'set null'|'no action');
$table->dropForeign(array('user_id'));
$table->dropForeign('posts_user_id_foreign');
```

## 欄位類型

``` php
// 自增
$table->increments('id');
$table->bigIncrements('id');

// 數字
$table->integer('votes');
$table->tinyInteger('votes');
$table->smallInteger('votes');
$table->mediumInteger('votes');
$table->bigInteger('votes');
$table->float('amount');
$table->double('column', 15, 8);
$table->decimal('amount', 5, 2);

// 字串和文字
$table->char('name', 4);
$table->string('email');
$table->string('name', 100);
$table->text('description');
$table->mediumText('description');
$table->longText('description');

// 日期和時間
$table->date('created_at');
$table->dateTime('created_at');
$table->time('sunrise');
$table->timestamp('added_on');
// Adds created_at and updated_at columns
 // 新增 created_at 和 updated_at 行
$table->timestamps();
$table->nullableTimestamps();

// 其它類型
$table->binary('data');
$table->boolean('confirmed');
// 為軟刪除新增 deleted_at 欄位
$table->softDeletes();
$table->enum('choices', array('foo', 'bar'));
// 新增 remember_token 為 VARCHAR(100) NULL
$table->rememberToken();
// 新增整型的 parent_id 和字串類型的 parent_type
$table->morphs('parent');
->nullable()
->default($value)
->unsigned()
->comment()
```
