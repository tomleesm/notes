---
hide:
  - toc
---

## 安裝和運行

``` php
// 將其加入到 composer.json 並更新:
composer require "phpunit/phpunit:4.0.*"
// 運行測試 (在項目根目錄下運行)
./vendor/bin/phpunit
```

## 斷言

``` php
$this->assertTrue(true);
$this->assertEquals('foo', $bar);
$this->assertCount(1,$times);
$this->assertResponseOk();
$this->assertResponseStatus(403);
$this->assertRedirectedTo('foo');
$this->assertRedirectedToRoute('route.name');
$this->assertRedirectedToAction('Controller@method');
$this->assertViewHas('name');
$this->assertViewHas('age', $value);
$this->assertSessionHasErrors();
// 由單個 key 值來假定 session 有錯誤...
$this->assertSessionHasErrors('name');
// 由多個 key 值來假定 session 有錯誤...
$this->assertSessionHasErrors(array('name', 'age'));
$this->assertHasOldInput();
```

## 訪問路由

``` php
$response = $this->call($method, $uri, $parameters, $files, $server, $content);
$response = $this->callSecure('GET', 'foo/bar');
$this->session(['foo' => 'bar']);
$this->flushSession();
$this->seed();
$this->seed($connection);
```
