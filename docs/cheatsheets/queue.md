---
hide:
  - toc
---

``` php
Queue::push('SendMail', array('message' => $message));
Queue::push('SendEmail@send', array('message' => $message));
Queue::push(function($job) use $id {});
// 在多個 workers 中使用相同的負載
 Queue::bulk(array('SendEmail', 'NotifyUser'), $payload);
// 開啟佇列監聽器
php artisan queue:listen
php artisan queue:listen connection
php artisan queue:listen --timeout=60
// 只處理第一個佇列任務
php artisan queue:work
// 在後台模式啟動一個佇列 worker
php artisan queue:work --daemon
// 為失敗的任務建立 migration 檔案
php artisan queue:failed-table
// 監聽失敗任務
php artisan queue:failed
// 通過 id 刪除失敗的任務
php artisan queue:forget 5
// 刪除所有失敗任務
php artisan queue:flush
// 因為佇列不會默認使用 PHP memory 參數，需要在佇列指定(單位默認mb)
php artisan queue:work --memory=50
```
