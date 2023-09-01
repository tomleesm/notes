``` shell
composer create-project laravel/laravel folder_name
composer create-project laravel/laravel folder_name --prefer-dist "5.8.*"
composer install
composer install --prefer-dist
composer update
composer update package/name
composer dump-autoload [--optimize]
composer self-update
composer require [options] [--] [vendor/packages]...
// 全域安裝
composer require global vendor/packages
// 羅列所有擴展包括版本資訊
composer show
```
