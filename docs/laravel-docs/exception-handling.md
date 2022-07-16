# 例外處理

app/Exceptions/Handler.php 的 `report()` 定義如何報告例外，是要記錄到 log 檔？還是寄信通知工程師？所以這個 method 基本上都是 Log 章節的東西。預設是記錄到 storage/logs/laravel.log。

`render()` 定義例外產生時送回給 client 端的 HTTP response。如果只是要自訂頁面，建議使用「自定義 HTTP 錯誤頁面」的內容，直接修改 resources/views/errors 目錄下的檔案就好。

`report()`：如果 .env 的 APP_DEBUG = true 的話，丟出例外時整個網頁會充滿錯誤訊息。而 `report($exception)` 可以把從參數傳入的例外只放到 log 中

## Reportable & Renderable Exceptions

實測結果，完全依照官方文件，仍然沒有如想像中的運作，Google 了一下，好像的確是 bug ?

`abort()` 的第二個參數可以自訂 `403 | Forbidden`  | 右邊的 Forbidden 訊息，但是不是每種錯誤都能自訂，例如 401, 404 都不能

## 自定義 HTTP 錯誤頁面

執行這個 artisan 命令，會複製 Laravel 內建的錯誤頁面，然後就可以自己修改

``` bash
php artisan vendor:publish --tag=laravel-errors
```

新增 resources/views/errors/404.blade.php，可以自訂 404 錯誤頁面，觸發的 Exception 則會自動變成 view 的變數 `$exception`，於是 `$exception->getMessage()` 得到的就是第二個參數的字串值，所以上述 `abort(404)` 無法自訂訊息，是指 Laravel 內建的 404 頁面無法自訂訊息。(因爲是 hard code )