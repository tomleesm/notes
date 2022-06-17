# CSRF 保護

對於會修改資料的 HTTP 請求，(POST、PUT、PATCH、DELETE)，Laravel 會自動提供 CSRF 保護。Laravel 會自動產生一個字串，類似 XjPG16nkhyTaeG0dTx2zkFhosWynfsOcI79kuSna，儲存在 session 中，然後必須在 HTTP 請求中包含同樣的字串（隱藏欄位、header X-CSRF-TOKEN、cookie），Laravel 會用 middleware VerifyCsrfToken 檢查 session 和請求中的字串兩者是否相同，是的話繼續執行，否則產生狀態碼 419 錯誤。

在 app/Http/Kernel.php 的 `$middlewareGroups` 設定 web 有套用 VerifyCsrfToken，所以 routes/web.php 會自動給予 CSRF 保護，routes/api.php 則沒有。

## 產生 CSRF token

``` html
<form method="POST" action="/profile">
    <!--
    以下這兩個都是產生 <input type="hidden" name="_token" value="ozrUaeXZUx0riFNOkn7J2uGZ2OnjLSfHmga4Riw6">
    -->
    @csrf
    {{ csrf_field() }}
</form>
```