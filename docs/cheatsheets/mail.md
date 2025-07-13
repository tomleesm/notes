---
hide:
  - toc
---

## Mail

``` php
Mail::send('email.view', $data, function($message){});
Mail::send(array('html.view', 'text.view'), $data, $callback);
Mail::queue('email.view', $data, function($message){});
Mail::queueOn('queue-name', 'email.view', $data, $callback);
Mail::later(5, 'email.view', $data, function($message){});
// 臨時將傳送郵件請求寫入 log，方便測試
Mail::pretend();
```

## 消息

``` php
// 這些都能在 $message 實例中使用, 並可傳入到 Mail::send() 或 Mail::queue()
$message->from('email@example.com', 'Mr. Example');
$message->sender('email@example.com', 'Mr. Example');
$message->returnPath('email@example.com');
$message->to('email@example.com', 'Mr. Example');
$message->cc('email@example.com', 'Mr. Example');
$message->bcc('email@example.com', 'Mr. Example');
$message->replyTo('email@example.com', 'Mr. Example');
$message->subject('Welcome to the Jungle');
$message->priority(2);
$message->attach('foo\bar.txt', $options);
// 使用記憶體資料作為附件
$message->attachData('bar', 'Data Name', $options);
// 附帶檔案，並返回 CID
$message->embed('foo\bar.txt');
$message->embedData('foo', 'Data Name', $options);
// 獲取底層的 Swift Message 對象
$message->getSwiftMessage();
```
