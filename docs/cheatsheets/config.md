---
hide:
  - toc
---

``` php
// config/app.php 裡的 timezone 組態項
Config::get('app.timezone')
Config::get('app.timezone', 'default');
Config::set('database.default', 'sqlite');
// config() 等同於 Config Facade
config()->get('app.timezone');
config('app.timezone', 'default');   // get
config(['database.default' => 'sqlite']); // set
```
