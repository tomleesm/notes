``` php
$environment = app()->environment();
$environment = App::environment();
// 判斷當環境是否為 local
if (app()->environment('local')){}
// 判斷當環境是否為 local 或 staging...
if (app()->environment(['local', 'staging'])){}
```
