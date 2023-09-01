## Log

``` php
// 記錄器提供了 7 種在 RFC 5424 標準內定義的記錄等級:
// debug, info, notice, warning, error, critical, and alert.
Log::info('info');
Log::info('info',array('context'=>'additional info'));
Log::error('error');
Log::warning('warning');
// 獲取 monolog 實例
Log::getMonolog();
// 新增監聽器
Log::listen(function($level, $message, $context) {});
```

## 記錄 SQL 查詢語句

``` php
// 開啟 log
DB::connection()->enableQueryLog();
// 獲取已執行的查詢陣列
DB::getQueryLog();
```
