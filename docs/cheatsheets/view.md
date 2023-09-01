``` php
View::make('path/to/view');
View::make('foo/bar')->with('key', 'value');
View::make('foo/bar')->withKey('value');
View::make('foo/bar', array('key' => 'value'));
View::exists('foo/bar');
// 跨檢視共享變數
View::share('key', 'value');
// 檢視巢狀
View::make('foo/bar')->nest('name', 'foo/baz', $data);
// 註冊一個檢視構造器
View::composer('viewname', function($view){});
// 註冊多個檢視到一個檢視構造器中
View::composer(array('view1', 'view2'), function($view){});
// 註冊一個檢視構造器類
View::composer('viewname', 'FooComposer');
View::creator('viewname', function($view){});
```
