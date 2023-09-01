## Input

``` php
Input::get('key');
// 指定預設值
Input::get('key', 'default');
Input::has('key');
Input::all();
// 只取回 'foo' 和 'bar'，返回陣列
Input::only('foo', 'bar');
// 取除了 'foo' 的所有使用者輸入陣列
Input::except('foo');
Input::flush();
// 欄位 'key' 是否有值，返回 boolean
Input::filled('key');
```

## 會話週期內 Input

``` php
// 清除會話週期內的輸入
Input::flash();
// 清除會話週期內的指定輸入
Input::flashOnly('foo', 'bar');
// 清除會話週期內的除了指定的其他輸入
Input::flashExcept('foo', 'baz');
// 取回一個舊的輸入條目
Input::old('key','default_value');
```

## Files

``` php
// 使用一個已上傳的檔案
Input::file('filename');
// 判斷檔案是否已上傳
Input::hasFile('filename');
// 獲取檔案屬性
Input::file('name')->getRealPath();
Input::file('name')->getClientOriginalName();
Input::file('name')->getClientOriginalExtension();
Input::file('name')->getSize();
Input::file('name')->getMimeType();
// 移動一個已上傳的檔案
Input::file('name')->move($destinationPath);
// 移動一個已上傳的檔案，並設定新的名字
Input::file('name')->move($destinationPath, $fileName);
```
