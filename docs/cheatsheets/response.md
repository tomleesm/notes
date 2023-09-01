``` php
return Response::make($contents);
return Response::make($contents, 200);
return Response::json(array('key' => 'value'));
return Response::json(array('key' => 'value'))
->setCallback(Input::get('callback'));
return Response::download($filepath);
return Response::download($filepath, $filename, $headers);
// 建立一個回應且修改其頭部資訊的值
$response = Response::make($contents, 200);
$response->header('Content-Type', 'application/json');
return $response;
// 為回應附加上 cookie
return Response::make($content)
->withCookie(Cookie::make('key', 'value'));
```
