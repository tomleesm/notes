``` php
return Redirect::to('foo/bar');
return Redirect::to('foo/bar')->with('key', 'value');
return Redirect::to('foo/bar')->withInput(Input::get());
return Redirect::to('foo/bar')->withInput(Input::except('password'));
return Redirect::to('foo/bar')->withErrors($validator);
// 重新導向到之前的請求
return Redirect::back();
// 重新導向到命名路由（根據命名路由算出 URL）
return Redirect::route('foobar');
return Redirect::route('foobar', array('value'));
return Redirect::route('foobar', array('key' => 'value'));
// 重新導向到控製器動作（根據控製器動作算出 URL）
return Redirect::action('FooController@index');
return Redirect::action('FooController@baz', array('value'));
return Redirect::action('FooController@baz', array('key' => 'value'));
// 跳轉到目的地址，如果沒有設定則使用預設值 foo/bar
return Redirect::intended('foo/bar');
```
