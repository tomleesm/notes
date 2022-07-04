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

### `$this->validate()`

在 `PostController@store` 中，`Request` 物件的 `validate()` 只需要傳入驗證規則陣列即可

``` php
<?php
public function store($request) {
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

## 驗證規則

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
