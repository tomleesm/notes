``` php
App::setLocale('en');
Lang::get('messages.welcome');
Lang::get('messages.welcome', array('foo' => 'Bar'));
Lang::has('messages.welcome');
Lang::choice('messages.apples', 10);
// Lang::get 的別名
trans('messages.welcome');
// Lang::choice 的別名
trans_choice('messages.apples',  10)
// 輔助函數
__('messages.welcome')
```
