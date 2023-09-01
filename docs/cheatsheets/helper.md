註：陣列和字串函數將在 5.9 全面廢棄，推薦使用 Arr 和 Str Facade

## 陣列 & 對象
``` php
// 如果給定的鍵不存在於該陣列，Arr::add 函數將給定的鍵值對加到陣列中
Arr::add(['name' => 'Desk'], 'price', 100); 
// >>> ['name' => 'Desk', 'price' => 100]
// 將陣列的每一個陣列折成單一陣列
Arr::collapse([[1, 2, 3], [4, 5, 6]]);  
// >>> [1, 2, 3, 4, 5, 6]
// 函數返回兩個陣列，一個包含原本陣列的鍵，另一個包含原本陣列的值
Arr::divide(['key1' => 'val1', 'key2' =>'val2'])
// >>> [["key1","key2"],["val1","val2"]]
// 把多維陣列扁平化成一維陣列，並用「點」式語法表示深度
Arr::dot($array);
// 從陣列移除給定的鍵值對
Arr::except($array, array('key'));
// 返回陣列中第一個通過真值測試的元素
Arr::first($array, function($value, $key){}, $default);
// 將多維陣列扁平化成一維
 // ['Joe', 'PHP', 'Ruby'];
Arr::flatten(['name' => 'Joe', 'languages' => ['PHP', 'Ruby']]);
// 以「點」式語法從深度巢狀陣列移除給定的鍵值對
Arr::forget($array, 'foo');
Arr::forget($array, 'foo.bar');
// 使用「點」式語法從深度巢狀陣列取回給定的值
Arr::get($array, 'foo', 'default');
Arr::get($array, 'foo.bar', 'default');
// 使用「點」式語法檢查給定的項目是否存在於陣列中
Arr::has($array, 'products.desk');
// 從陣列返回給定的鍵值對
Arr::only($array, array('key'));
// 從陣列拉出一列給定的鍵值對
Arr::pluck($array, 'key');
// 從陣列移除並返回給定的鍵值對
Arr::pull($array, 'key');
// 使用「點」式語法在深度巢狀陣列中寫入值
Arr::set($array, 'key', 'value');
Arr::set($array, 'key.subkey', 'value');
// 借由給定閉包結果排序陣列
Arr::sort($array, function(){});
// 使用 sort 函數遞迴排序陣列
Arr::sortRecursive();
// 使用給定的閉包過濾陣列
Arr::where();
// 陣列"洗牌"
Arr::shuffle($array,'I-AM-GROOT');
// 陣列包裹(如果不是陣列，就變成陣列，如果是空的，返回[],否則返回原資料）
Arr::wrap($array);
// 返回給定陣列的第一個元素
head($array);
// 返回給定陣列的最後一個元素
last($array);
```

## 路徑

``` php
// 取得 app 資料夾的完整路徑
app_path();
// 取得項目根目錄的完整路徑
base_path();
// 取得應用組態目錄的完整路徑
config_path();
// 取得應用資料庫目錄的完整路徑
database_path();
// 取得加上版本號的 Elixir 檔案路徑
elixir();
// 取得 public 目錄的完整路徑
public_path();
// 取得 storage 目錄的完整路徑
storage_path();
```

## 字串

``` php
// 將給定的字串轉換成 駝峰式命名
Str::camel($value);
// 返回不包含命名空間的類名稱
class_basename($class);
class_basename($object);
// 對給定字串運行 htmlentities
e('<html>');
// 判斷字串開頭是否為給定內容
Str::startsWith('Foo bar.', 'Foo');
// 判斷給定字串結尾是否為指定內容
Str::endsWith('Foo bar.', 'bar.');
// 將給定的字串轉換成 蛇形命名
Str::snake('fooBar');
// 將給定字串轉換成「首字大寫命名」: FooBar
Str::studly('foo_bar');
// 根據你的本地化檔案翻譯給定的語句
trans('foo.bar');
// 根據後綴變化翻譯給定的語句
trans_choice('foo.bar', $count);
URLs and Links
// 產生給定控製器行為網址
action('FooController@method', $parameters);
// 根據目前請求的協定（HTTP 或 HTTPS）產生資原始檔網址
asset('img/photo.jpg', $title, $attributes);
// 根據 HTTPS 產生資原始檔網址
secure_asset('img/photo.jpg', $title, $attributes);
// 產生給定路由名稱網址
route($route, $parameters, $absolute = true);
// 產生給定路徑的完整網址
url('path', $parameters = array(), $secure = null);
```

## 其他

``` php
// 返回一個認證器實例。你可以使用它取代 Auth facade
auth()->user();
// 產生一個重新導向回應讓使用者回到之前的位置
back();
// 使用 Bcrypt 雜湊給定的數值。你可以使用它替代 Hash facade
bcrypt('my-secret-password');
// 從給定的項目產生集合實例
collect(['taylor', 'abigail']);
// 取得設定選項的設定值
config('app.timezone', $default);
// 產生包含 CSRF 令牌內容的 HTML 表單隱藏欄位
{!! csrf_field() !!} 
// 5.7+用這個
@csrf
// 取得當前 CSRF 令牌的內容
$token = csrf_token();
// 輸出給定變數並結束指令碼運行
dd($value);
// var_dump縮寫（如果用dump-server,var_dump可能無效）
dump($value);
// 取得環境變數值或返回預設值
$env = env('APP_ENV');
$env = env('APP_ENV', 'production');
// 配送給定事件到所屬的偵聽器
event(new UserRegistered($user));
// 根據給定類、名稱以及總數產生模型工廠建構器
$user = factory(App\User::class)->make();
// 產生擬造 HTTP 表單動作內容的 HTML 表單隱藏欄位
{!! method_field('delete') !!}
// 5.7+
@method('delete')
// 取得快閃到 session 的舊有輸入數值
$value = old('value');
$value = old('value', 'default');
// 返回重新導向器實例以進行 重新導向
 return redirect('/home');
// 取得目前的請求實例或輸入的項目
$value = request('key', $default = null)
// 建立一個回應實例或獲取一個回應工廠實例
return response('Hello World', 200, $headers);
// 可被用於取得或設定單一 session 內容
$value = session('key');
// 在沒有傳遞參數時，將返回 session 實例
$value = session()->get('key');
session()->put('key', $value);
// 返回給定數值
value(function(){ return 'bar'; });
// 取得檢視 實例
return view('auth.login');
// 返回給定的數值
$value = with(new Foo)->work();
```
