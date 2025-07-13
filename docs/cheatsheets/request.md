---
hide:
  - toc
---

``` php
//獲取請求參數 form-data 與 raw 請求類型
request()->input();
// url: http://xx.com/aa/bb
Request::url();
// 路徑: /aa/bb
Request::path();
// 獲取請求 Uri: /aa/bb/?c=d
Request::getRequestUri();
// 返回使用者的 IP
Request::ip();
// 獲取 Uri: http://xx.com/aa/bb/?c=d
Request::getUri();
// 獲取查詢字串: c=d
Request::getQueryString();
// 獲取請求連接埠 (例如 80, 443 等等)
Request::getPort();
// 判斷當前請求的 URI 是否可被匹配
Request::is('foo/*');
// 獲取 URI 的分段值 (索引從 1 開始)
Request::segment(1);
// 從請求中取回頭部資訊
Request::header('Content-Type');
// 從請求中取回伺服器變數
Request::server('PATH_INFO');
// 判斷請求是否是 AJAX 請求
Request::ajax();
// 判斷請求是否使用 HTTPS
Request::secure();
// 獲取請求方法
Request::method();
// 判斷請求方法是否是指定類型的
Request::isMethod('post');
// 獲取原始的 POST 資料
Request::instance()->getContent();
// 獲取請求要求返回的格式
Request::format();
// 判斷 HTTP Content-Type 頭部資訊是否包含 */json
Request::isJson();
// 判斷 HTTP Accept 頭部資訊是否為 application/json
Request::wantsJson();
```
