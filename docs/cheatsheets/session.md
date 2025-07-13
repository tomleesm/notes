---
hide:
  - toc
---

``` php
Session::get('key');
// 從會話中讀取一個條目
 Session::get('key', 'default');
Session::get('key', function(){ return 'default'; });
// 獲取 session 的 ID
Session::getId();
// 增加一個會話鍵值資料
Session::put('key', 'value');
// 將一個值加入到 session 的陣列中
Session::push('foo.bar','value');
// 返回 session 的所有條目
Session::all();
// 檢查 session 裡是否有此條目
Session::has('key');
// 從 session 中移除一個條目
Session::forget('key');
// 從 session 中移除所有條目
Session::flush();
// 生成一個新的 session 識別碼
Session::regenerate();
// 把一條資料暫存到 session 中
Session::flash('key', 'value');
// 清空所有的暫存資料
Session::reflash();
// 重新暫存當前暫存資料的子集
Session::keep(array('key1', 'key2'));
```
