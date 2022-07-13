# Log

直接看官方文件會看不太懂，最好先看完 Monolog 官方文件的 [Using Monolog](https://github.com/Seldaek/monolog/blob/2.x/doc/01-usage.md) 這一篇文章

``` php
<?php

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\ErrorLogHandler;

$logger = new Logger('my_logger');

$logger->pushHandler(new StreamHandler(__DIR__.'/my_app.log', Logger::DEBUG));
$logger->pushHandler(new ErrorLogHandler());

$logger->info('My logger is now ready');
```

需要先建立一個 Logger，Monolog 稱呼 logger 爲頻道，然後繫結多個 handler，上面的例子，StreamHandler 設定好會把記錄新增到同一個目錄下的檔案 my_app.log，以及 ErrorLogHandler，會把記錄新增到 php 設定的 error_log 位置。

依照 [RFC 5424](https://datatracker.ietf.org/doc/html/rfc5424) ，等級有八種：debug, info, notice, warning, error, critical, alert, emergency，使用和上述等級同名的方法新增記錄，所以 `$logger->info()` 新增一筆 info 等級的記錄，然後頻道會把記錄交給 handler 處理

首先 handler 會依照自身設定的嚴重等級決定是否處理，如果記錄的等級大於等於 handler 設定的，則 handler 會處理記錄。StreamHandler 設定嚴重等級是 debug（第二個參數），記錄是 info，大於 debug，所以輪到 StreamHandler 時會處理這個記錄。ErrorLogHandler 則是預設嚴重等級爲 error，所以 `$logger->error()` 時才會處理

一些 handler 例如 MailHandler 會有 bubble 選項，和 JavaScript 一樣的概念，設定處理完記錄後， 是否把記錄傳給下一個 handler 處理，如果是 `$bubble = true` 則會傳給下一個。畢竟如果是 error 等級的記錄，寄信通知後不一定要同時記錄在檔案系統中

-----------------------------------------

Laravel 的 config/logging.php，stack 可以說是上述 Monolog 的頻道，然後其他的「頻道」，例如 single, slack 其實是 Monolog 的 handler

Single 和 Daily 的 bubble 選項即是上述 Monolog 的 bubble 選項。但是 permission 選項實測時會把權限弄成奇怪的結果，所以不要使用它，用預設的 644 即可

syslog 頻道是寫入到檔案 /var/log/syslog（要有 sudo 才能看到內容）

設定 errorlog 頻道：執行 `phpinfo()`，找到 php.ini 的位置。修改 error_log = /your/dir/errors.log，然後重啓 php 使其生效

修改 config/app.php 的 timezone 爲 Asia/Taipei，log 才會記錄正確的時間

`Log::info('User failed to login.', ['id' => 123]);` 會產生 log 類似 `[2022-07-11 17:32:51] v6.INFO: User failed to login. {"id":123}` ，陣列會轉成 JSON 字串

## 自訂 Monolog channel

tap 元素是一個陣列，裡面是類別清單。當執行 single 頻道時就執行這些類別

``` php
<?php
'single' => [
    'driver' => 'single',
    'tap' => [ App\Logging\CustomizeFormatter::class ],
    'path' => storage_path('logs/laravel.log'),
    'level' => 'debug',
],
```

php 在 new 一個類別時會自動呼叫 `__invoke()`，所以以下就直接用這個 method。Laravel 透過 `__invoke()` 的參數自動注入一個 Monolog 物件 $logger。所以其他的程式碼基本上和官方文件  [Using Monolog](https://github.com/Seldaek/monolog/blob/2.x/doc/01-usage.md) 一樣。 

``` php
<?php
// app/Logging/CustomizeFormatter.php
namespace App\Logging;

use Monolog\Formatter\LineFormatter;

class CustomizeFormatter
{
    /**
     * Customize the given logger instance.
     *
     * @param  \Illuminate\Log\Logger  $logger
     * @return void
     */
    public function __invoke($logger)
    {
        $dateFormat = 'Y n j, g:i a';
        $output = '%datetime% > %level_name% > %message% %context% %extra%"' . PHP_EOL;
        foreach($logger->getHandlers() as $handler)
        {
            $handler->setFormatter(new LineFormatter($output, $dateFormat));
        }
    }
}
```

## Creating Monolog Handler Channels

依照官方文件，實際上沒有試成功，自訂 channel 也是