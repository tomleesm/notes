# HTTP 請求

## 路由
請求是 GET http://www.example.com/users/123?name=tom&message=hello

``` php
<?php
# 回傳字串 users/123
$request->path();
# 回傳字串 http://www.example.com/users/123
$request->url();
# 回傳字串 http://www.example.com/users/123?name=tom&message=hello
$request->fullUrl();
# 回傳字串 http://www.example.com/users/123
url('/users/123');
# 目前的 URL，不包含 query，回傳字串 http://www.example.com/users/123
url()->current();
# 目前的 URL，包含 query
# 回傳字串 http://www.example.com/users/123?name=tom&message=hello
url()->full();
# 上一個請求的 URL，包含 query
# 回傳字串 http://www.example.com/users/123?name=tom&message=hello
url()->previous();
# 上述的 url() 都可以改成 URL Facades
use Illuminate\Support\Facades\URL;
URL::current();
URL::full();
URL::previous();
# 回傳字串 GET
$request->method();
# HTTP 方法是否爲 GET。實測參數是 'get', 'GeT' 都可以
$request->isMethod('GET')
# 回傳所有輸入值的陣列：[ 'name' => 'tom', 'message' => 'hello' ]
# 不包含路由參數 123
$request->all()
# URI 爲 users/123 則回傳 true
$request->is('users/*')
```

## 抓取輸入

### 欄位值

``` php
<?php
# 回傳陣列 [ '欄位名稱' => 欄位值 ]，所有表單欄位值和網址 query
$request->all();
$request->input();
# 表單欄位或網址 query 的值
$request->input('表單欄位名稱', '預設值');
# ?products[][name]=A&products[][name]=B
/**
[
  0 => [ 'name' => 'A' ],
  1 => [ 'name' => 'B' ],
]
**/
$request->input('products');
$request->input('products.*');
/**
  [ 'name' => 'A' ]
**/
$request->input('products.0');
/**
  'A'
**/
$request->input('products.0.name');
/**
  [
    0 => 'A',
    1 => 'B'
  ]
**/
$request->input('products.*.name');
# 網址 query。語法和 input() 一樣
$request->query();
$request->query('欄位名稱', '預設值');

# 回傳表單欄位名稱 name 的值，或路由參數 {name} 的值
$request->name;
```

### JSON

``` php
<?php
# 如果 HTTP 請求的 Content-Type: application/json
/**
  {
    "products": [
      "name": "A",
      "name": "B"
    ]
  }
  上述的 json
**/
# 回傳 'A'
$request->input('products.0.name');
```

## 檢查欄位

``` php
<?php
# 只要指定的欄位。如果沒有這個欄位，則直接略過，不會回傳錯誤訊息
$input = $request->only( [ '欄位名稱A', '欄位名稱B' ] );
$input = $request->only('products.0.name', 'products.1.name');
# 除了指定的欄位，其它都要。如果沒有這個欄位，則直接略過，不會回傳錯誤訊息
$input = $request->except( [ 'credit_card' ] );
$input = $request->except('credit_card');
# 檢查有沒有指定的欄位，空值也回傳 true
$request->has('欄位名稱')
# 檢查有沒有欄位 A 和 B
$request->has('A', 'B')
$request->has( [ 'A', 'B' ] )
# 檢查有沒有指定的欄位，並且不是空值
$request->filled('欄位名稱')
# 檢查有沒有欄位 A 和 B，並且都不是空值
$request->filled('A', 'B')
$request->filled( [ 'A', 'B' ] )
```

## 上一次請求的輸入

``` php
<?php
# 把輸入都儲存到拋棄式 session
$request->flash();
# 把欄位 A 和 B 的值儲存到拋棄式 session
$request->flashOnly('A', 'B');
$request->flashOnly( [ 'A', 'B' ] );
# 把欄位 A 和 B 以外的值儲存到拋棄式 session
$request->flashExcept('A', 'B');
$request->flashExcept( [ 'A', 'B' ] );
# 把輸入都儲存到拋棄式 session 後重新導向
# withInput($request->only('name')) 可以用 $request->only() 和 $request->except()
# 指定或排除欄位，only() 和 except() 的參數語法都可用
# 但是不能用 withInput('name') 和 withInput(['name'])
redirect()->route('users.create')->withInput();

# 抓取儲存到拋棄式 session 的欄位值
# 實測過，用 session('欄位名稱') 抓不到
# 所以拋棄式 session 和一般的 session 是分開來的
$request->old('欄位名稱');
# 回傳陣列，含有所有拋棄式 session 的值
$request->old();
```