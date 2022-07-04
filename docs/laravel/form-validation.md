# 表單驗證

## 摘要：

- 用於傳統多頁網站：`$request->validate()`、`$this->validate()`、form request
   - 如果驗證通過，則繼續向下執行
   - 驗證失敗，則自動記住目前的輸入，跳轉回上一頁
- 用於 AJAX：`\Validator::make()`
   - 驗證通過（`fails()` return false），則繼續向下執行
   - 驗證失敗（`fails()` return true），則手動回傳錯誤訊息 JSON 回應

注意，`validate()` 無法用於 AJAX。官方文件寫錯了！！！它不會回傳狀態碼 422 ，也不會回傳錯誤訊息 JSON 回應，而是回傳跳轉回上一頁的 HTML，附帶錯誤訊息

路由設定

``` php
<?php
# routes/web.php
Route::get('posts/create', 'PostController@create');
Route::post('posts', 'PostController@store');
```

在 `PostController@create` 這樣設定
``` php
<?php
public function create()
{
    return view('posts.create');
}
```

所以 resources/views/posts/create.blade.php 如下：

``` html
<form method="post" action="/posts">
    @csrf
    <p>
        <label>title:</label>
        <input type="text" name="title">
    </p>
    <p>
        <label>publish_date:</label>
        <input type="text" name="publish_date">
    </p>
    <p>
        <button type="submit">Submit</button>
    </p>
</form>
```

## `validate(array 驗證規則)`

- `$request->validate()`：`Request` 物件的 `validate()` 
- `$this->validate()`：來自於 app/Http/Controller `ValidatesRequests` trait

``` php
<?php
public function store(Request $request) {
    // 用 | 分隔驗證規則
    $validation = $request->validate([
        'title' => 'required|min:3',
        'publish_date' => 'nullable|date',
    ]);
    // 也可以用陣列元素
    $validation = $request->validate([
        'title' => [ 'required', 'min:3' ],
        'publish_date' => [ 'nullable', 'date' ],
    ]);
}
```

## form request

新增 form request 到目錄 app/Http/Requests/

``` bash
php artisan make:request MyFormRequest
```

在 Controller 注入 form request

``` php
<?php
use App\Http\Requests\MyFormRequest;

public function store(MyFormRequest $request) {
  // 會在 MyFormRequest 中執行表單驗證，
  // 所以 controller 就沒有 $request->validate()，
  // 可以專心做其它事
}
```

form request 設定，因爲 form request 繼承 Request 類別，所以可以用 `$this` 呼叫 Request 本身的方法，例如 `$request->user()` 回傳目前登入的使用者 User 物件，你可以用 `$this->user()` 做一樣的事。參數和建構式可以注入類別，容器會自動生成物件

``` php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class MyFormRequest extends FormRequest
{
    /**
     * 能否使用這個 form request 的權限控制。
     * 如果 return false，預設會顯示頁面 403 This action is unauthorized.
     * 如果不想在這裡做權限控制，則 return true 即可
     */
    public function authorize()
    {
        return true;
    }

    /**
     * 驗證規則
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|min:3',
            'publish_date' => 'nullable|date'
        ];
    }
}
```

## `\Validator\make()`

手動新增 `Validator` 物件

``` php
<?php
public function store(Request $request) {
    /**
        \Validator::make(array 要驗證的表單欄位,
                         array 驗證規則,
                         array 自訂驗證失敗訊息的規則)
    **/
    $validator = \Validator::make($request->all(), [
        'title' => 'required|min:3',
        'publish_date' => 'nullable|date',
    ], [ 'title.required' => '標題必須填寫' ]);

    # 驗證失敗，不會自動跳轉頁面，必須手動建立回應
    if ( $validator->fails() ) {
        # 回傳 JSON，內含錯誤訊息
        # { "errors" => [ '錯誤訊息A', '錯誤訊息B', ... ] }
        return response()->json([
            'errors' => $validator->errors()->all()
         ]);
    }

    # 驗證通過，回傳 JSON，內含成功訊息
    return response()->json([
        'success' => 'Record is successfully added'
    ]);
}
```

## 驗證失敗訊息

### 顯示訊息

以下的 `all()` 都回傳陣列，類似 `[ '標題必須填寫', '標題最少 3 個字元', ... ]`

#### 傳統多頁網站

訊息儲存在 session errors，並自動更新到 view 的變數 `$errors`

所有欄位的驗證失敗訊息：

``` html
@if ($errors->any())
    <ul>
        <!-- $errors->all() 回傳陣列 -->
        @foreach ($errors->all() as $error)
            <li>{{ $error }}</li>
        @endforeach
    </ul>
@endif
```

指定欄位的驗證失敗訊息：

``` html
<!-- @error('title')：表單欄位 title 是否有驗證錯誤訊息 -->
@error('title')
    <!-- $message：表單欄位 title 的驗證錯誤訊息 -->
    <div>{{ $message }}</div>
@enderror
```

#### AJAX

所有欄位的驗證失敗訊息：

``` php
<?php
// all() 回傳字串陣列
foreach( $validator->errors()->all() as $message ) {
  // ...
}
```

指定欄位的驗證失敗訊息：

``` php
<?php
// 所有欄位的第一個訊息
$validator->errors()->first()
// 指定欄位名稱的第一個訊息字串，例如 title 有 2 個規則，所以最多有 2 個訊息
$validator->errors()->first('欄位名稱')
// 指定欄位名稱的所有訊息陣列，例如 title 有 2 個規則，所以最多有 2 個訊息，
// 則回傳類似 [ '標題必須填寫', '標題最少 3 個字元' ]
$validator->errors()->get('欄位名稱')
// 如果 <input name="author[]"> author 欄位是陣列
// 則使用 get('author.*') 回傳所有訊息陣列
$validator->errors()->get('author.*')
// 是否有表單欄位 title 的驗證失敗訊息
if ( $validator->errors()->has('title') ) {
  // ...
}
```

### 自訂訊息

#### form request

``` php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class MyFormRequest extends FormRequest
{
    // 自訂驗證失敗的訊息
    // [ '欄位.規則' => '訊息' ]
    // 注意規則不包含冒號之後的值，所以 title.min 不能加上 :3
    // 否則規則會被忽略
    public function messages()
    {
        return [
            'title.required' => '這個標題需要填寫',
            'title.min' => '標題至少要 3 個字元',
            'publish_date.date' => '這不是日期！'
        ];
    }

    // 把錯誤訊息中的 :title 和 :publish_date 換成自訂文字
    // 實測結果是 messages() 會完全蓋掉 attributes() 的設定
    // 它只會跳換 resources/lang/en/validation.php 的 :title
    // 所以實務上建議一律在 resources/lang/en/validation.php 設定錯誤訊息
    // 不要使用 attributes() 和 messages()
    public function attributes()
    {
        return [
            'title' => '標題',
            'publish_date' => '出版日期'
        ];
    }
}
```

#### `\Validator::make()`

``` php
<?php
$messages = [
    'title.required' => '這個標題需要填寫',
    'title.min' => '標題至少要 3 個字元',
    'publish_date.date' => '這不是日期！',
    // 只替換佔位符似乎不容易套用
    'size'    => 'The :attribute must be exactly :size.'
];
# \Validator::make() 第三個參數是自訂驗證失敗的訊息
$validator = \Validator::make($request->all(), [
        'title' => 'required|min:3',
        'publish_date' => 'nullable|date',
    ], $messages);
```

#### 在語言文件中設定

config/app.php 的 `'locale' => 'zh-tw'`，設定使用繁體中文語系，則在 resources/lang/zh-tw/validation.php 中：

``` php
<?php
return [
    # 自訂訊息
    'custom' => [
        'title' => [
            'required' => '這個標題需要填寫',
            'min'      => '標題至少要 3 個字元'
        ]
    ],
    # 自訂 :attribute。似乎不容易成功，請注意
    'attributes' => [
        'publish_date' => '出版日期'
    ]
];
```

## 驗證規則

### 如果表單欄位不是必填

表單欄位如果留空，Laravel 會用 middleware ConvertEmptyStringsToNull 把空字串轉成 null，然後因爲 null 無法比較，所以以下的驗證規則不通過，雖然規則沒有 required。如果表單欄位不是必填，驗證規則第一個應該是 nullable

``` php
<?php
[
    'publish_date' => 'date'
]
```

### 驗證失敗後停止後續規則驗證

規則 `A|B|C`，預設是規則 A 沒有通過，會繼續檢查 B ，然後是 C，所以錯誤訊息列出最多 3 個；如果最前面加上 `bail`，則只要有規則沒有通過，後續的不會繼續檢查，所以錯誤訊息最多只有 1 個。欄位 title 驗證通過與否，不影響欄位 publish_date

``` php
<?php
[
    'title' => 'bail|required|unique:posts|max:255',
    'publish_date' => 'nullable|date',
]
```

### 表單欄位名稱

如果表單 HTML 如下

``` html
<form method="POST" action="{{ route('posts.store') }}">
    {{csrf_field()}}
    <input type="text" name="title"/>
    <input type="text" name="author[name]"/>
    <input type="text" name="author[description]"/>
    <input type="text" name="author[]"/>
    <textarea cols="20" rows="5" name="body"></textarea>
    <button type="submit">submit</button>
</form>
```

則驗證規則中的欄位名稱使用 . 分隔，用中括號反而不行

``` php
<?php
[
    'title'              => 'required|unique:posts|max:255',
    'author.name'        => 'required',
    'author.description' => 'required',
    'body'               => 'required',
    // 欄位名稱爲數字索引的，實測結果如下
    'author.0'           => 'required'
]
```

驗證規則爲 required

- O：驗證結果正常，該 pass 和不應該 pass 的都正常
- X：驗證結果不正常，分成 all pass 和 all not pass
    - all pass：欄位未輸入和欄位輸入 abc 都 pass
    - all not pass：欄位未輸入或欄位輸入 abc 都不會 pass

| HTML \\ 驗證規則 | author | author.0 | author.1 | author.* |
| ----------------- | ------- | -------- | -------- | -------- |
| author | O | X (all not pass) | X (all not pass) | X (all pass) |
| `author[]` | X (all pass) | O | X (all not pass) | O |
| `author[0]` | X (all pass) | O | X (all not pass) | O |
| `author[1]` | X (all pass) | X (all not pass) | O | O |

結論是驗證規則請使用 `athor.數字`，不要用 `author.*`。`[ 'author.*' => 'required' ]` 會代表 HTML 表單欄位 `author[0]` 和 `author[1]`，並在兩個欄位都有輸入時才通過，但最好還是分開成兩個規則比較好

## 驗證規則索引

## 頁面上有多個表單

``` php
<?php
// validateWithBag(錯誤訊息物件的名稱, 驗證規則)
public function store($request) {
    // 在 view 中改用 $errors->blog->all() 抓取所有的錯誤，
    // 而不是 $errors->all()，用於頁面上有多個表單
    $validation = $request->validateWithBag('blog', [
        'title' => 'required|min:3',
        'publish_date' => 'nullable|date',
    ]);
}
```
