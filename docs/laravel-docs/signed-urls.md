# Signed URLs

命名路由 `route('unsubscribe')` 表示 `GET /unsubscribe/users/{user}`，可以用 `URL::signedRoute('unsubscribe', [ 'user' => 1])` 產生網址 http://www.example.com/unsubscribe/users/1?signature=ebedccaa0ef1fbfbe81bbda05338d6634e9c2f7eb65cb58b8b435da4e491f30d

它是把 /unsubscribe/users/1 的 checksum 加到後面的 signature，網址如果被修改就不會有效。可以用在E-mail 驗證、電子報退訂等。

``` php
use Illuminate\Support\Facades\URL;
# 回傳字串 http://www.example.com/unsubscribe/users/1
# ?signature=ebedccaa0ef1fbfbe81bbda05338d6634e9c2f7eb65cb58b8b435da4e491f
URL::signedRoute('unsubscribe', ['user' => 1]);
# 30 分鐘後過期無效的 signed url
# 回傳字串 http://www.example.com/unsubscribe/users/1
# ?expires=1656198463&signature=553e2ffc1f996d4ffed9ec4259d12cc08259daf9214
URL::temporarySignedRoute('unsubscribe', now()->addMinutes(30), ['user' => 1]);

# 如果網址是上述的 signed URL，則顯示 OK，如果網址被改了，顯示 Error
Route::get('unsubscribe/users/{user}', function(Request $request, $id) {
  if($request->hasValidSignature()) {
    return 'OK';
  } else {
    return 'Error !';
  }
})->name('unsubscribe');
# 用內建的 middleware signed 也是做一樣的事情
# 如果網址是上述的 signed URL，則顯示 OK，如果網址被改了，丟出 403 Invalid signature. 頁面
Route::get('unsubscribe/users/{user}', function(Request $request, $id) {
  return 'OK';
})->name('unsubscribe')->middleware('signed');
```

## 自訂 403 Invalid signature 頁面

如果想要自訂 403 Invalid signature. 頁面，要修改 `apps/Exceptions/Handler.php` 如下：

``` php
<?php
## 新增兩個 use
use Illuminate\Routing\Exceptions\InvalidSignatureException;
use Illuminate\Http\Response;

public function render($request, Throwable $exception) {
    # 新增以下的 if 片段，還有新增 errors/invalid_signature.blade.php
    if ($exception instanceof InvalidSignatureException) {
        return response()->view('errors.invalid_signature')
                         ->setStatusCode(Response::HTTP_FORBIDDEN);
    }
  
    return parent::render($request, $exception);
  }
```