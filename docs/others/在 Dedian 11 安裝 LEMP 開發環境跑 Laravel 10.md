# 在 Dedian 11 安裝 LEMP 開發環境跑 Laravel 10

原文在 2023-12-26 完成

## 摘要

這個是在 VirtualBox 6 新增虛擬機器，裝好 Debian 11 後，安裝 LEMP 的過程，最後安裝 Laravel 10 來寫網站後端

[下載 debian-11.8.0-amd64-netinst.iso](https://cdimage.debian.org/cdimage/archive/11.8.0/amd64/iso-cd/debian-11.8.0-amd64-netinst.iso)

## 安裝 PHP-FPM 8.3

``` bash
sudo apt -y install lsb-release apt-transport-https ca-certificates wget gnupg
sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
sudo apt update; sudo apt upgrade -y

PHP_VERSION='8.3'
sudo apt install -y php$PHP_VERSION-{cli,fpm,common,gd,mbstring,curl,bcmath,opcache}
```

## 設定 PHP-FPM

``` bash
# 備份 php-fpm 池設定
sudo cp /etc/php/$PHP_VERSION/fpm/pool.d/www.conf \
  /etc/php/$PHP_VERSION/fpm/pool.d/www.conf.sample
sudo vim /etc/php/$PHP_VERSION/fpm/pool.d/www.conf
```

改成以下的設定

``` bash
user = tom
group = tom
listen = 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1
slowlog = /home/tom/apps/logs/slow.log
request_slowlog_timeout = 5s
```

建立上述設定需要的資料夾和權限

``` bash
mkdir -p /home/tom/apps/logs/
sudo chown -R tom:tom /home/tom/apps/logs/
```

重新啟動 PHP-FPM

``` bash
sudo service php$PHP_VERSION-fpm restart
```

## nginx

### 安裝

``` bash
sudo apt install -y nginx
# 檢查 nginx 是否正常啟動。按 q 離開
sudo systemctl status nginx
```

瀏覽 [http://192.168.56.10](http://192.168.56.10/) 應該會看到  Welcome to nginx!

> 網址是 [http://192.168.56.10](http://192.168.56.10/) ，表示這個虛擬環境有設定 host-only 網路

### 設定

新增設定需要的目錄。在使用者 tom 的 `home` 目錄新增 `apps` 目錄

``` bash
mkdir -p /home/tom/apps/logs/localhost
mkdir -p /home/tom/apps/localhost/current/public
sudo chown tom:tom /home/tom/apps/logs/slow.log
```

編輯 nginx 設定

``` bash
sudo vim /etc/nginx/sites-available/default
```
以下設定取消註解

```
 location ~ \.php$ {
    include snippets/fastcgi-php.conf;

    fastcgi_pass 127.0.0.1:9000;
  }
```

啟用上述設定，重開 nginx，並查看 nginx 是否正常運作

``` bash
sudo service nginx restart
sudo systemctl status nginx
```

編輯 `info.php` 檔案
``` bash
sudo vim /var/www/html/info.php
```
複製貼上以下程式碼

``` php
<?php phpinfo();
```

瀏覽 [http://192.168.56.10/info.php](http://192.168.56.10/info.php) ，應該會看到 PHP Version 8.3 相關設定

刪除 `info.php` 檔案

``` bash
sudo rm /var/www/html/info.php
```
## 前端編譯用程式

### node.js

``` bash
sudo apt install -y curl build-essential
curl -sL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt install -y nodejs

node -v
npm -v
```

### yarn

``` bash
sudo npm install --global yarn
yarn --version
```

## 安裝 composer

複製貼上官網的指令

[https://getcomposer.org/download/](https://getcomposer.org/download/)  

移到系統資料夾，並檢查是否可以全域呼叫

``` bash
sudo mv composer.phar /usr/local/bin/composer
composer --version
```

## 安裝 laravel

``` bash
sudo apt install -y zip unzip php$PHP_VERSION-{zip,xml,bcmath}
composer global require "laravel/installer"

echo 'export PATH=~/.local/bin:~/.config/composer/vendor/bin:$PATH' >> ~/.bash_profile
source ~/.bash_profile
```

### 建立 Laravel 10 專案

``` bash
cd ~/apps

# 安裝最新版本，project_name 改成想要的目錄名稱
laravel new project_name
# 或者安裝指定版本 (10)，project_name 改成想要的目錄名稱
composer create-project laravel/laravel=10.* project_name --prefer-dist

cd project_name

# 安裝相依的 PHP 套件
composer install

# 安裝前端編譯需要的套件
yarn install
```

之前執行 `yarn dev` 遇到問題，產生錯誤訊息

> Error: ENOSPC: System limit for number of file watchers reached

解決方法是修改 `sysctl.conf` 的設定

``` bash
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
# 應該顯示 524288
cat /proc/sys/fs/inotify/max_user_watches
```
上述方法來自於 [React Native Error: ENOSPC: System limit for number of file watchers reached](https://stackoverflow.com/questions/55763428/react-native-error-enospc-system-limit-for-number-of-file-watchers-reached)

執行前端編譯與監視檔案變動

```
# 按 q 離開
yarn dev
```

### 設定 nginx 執行 Laravel

假設專案名稱是 project_name
``` bash
sudo vim /etc/nginx/sites-available/project_name.conf
```

複製貼上以下設定。注意 root 要改成 Laravel 專案的絕對路徑，listen 8111 表示設定 port 是 8111

```
server {
  listen 8111;
  server_name 192.168.56.10;
  index index.php index.html;
  client_max_body_size 50M;
  error_log /home/tom/apps/logs/localhost/error.log;
  access_log /home/tom/apps/logs/localhost/access.log;
  root /home/tom/apps/project_name/public;
  location / {
    try_files $uri $uri/ /index.php$is_args$args;
  }
  location ~ \.php {
    try_files $uri = 404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME $fastcgi_script_name;
    fastcgi_index index.php;
    fastcgi_pass 127.0.0.1:9000;
  }
}
```

啟用設定，並重新啟動 nginx

``` bash
sudo ln -s /etc/nginx/sites-available/project_name.conf \
           /etc/nginx/sites-enabled/project_name.conf
sudo systemctl restart nginx
```

瀏覽 [http://192.168.56.10:8111](http://192.168.56.10:8111/)

## 安裝 MariaDB

``` bash
sudo apt install -y mariadb-server php$PHP_VERSION-mysql
sudo mysql_secure_installation
```

接下來會回答一系列問題，下一行則是建議操作

`Enter current password for root (enter for none):`

按下 enter

`Switch to unix_socket authentication [Y/n]`

按下 enter

`Change the root password? [Y/n] `

按下 n

`Remove anonymous users? [Y/n]`

按下 enter

`Disallow root login remotely? [Y/n]`

按下 enter

`Remove test database and access to it? [Y/n]`

按下 enter

`Reload privilege tables now? [Y/n]`

按下 enter

### 新增資料庫

``` bash
sudo mariadb
```

執行以下 SQL 新增使用者 project_name 和資料庫 project_name(或換成想要的名稱)。記得密碼要修改（`IDENTIFIED BY '密碼'`）

``` sql
CREATE DATABASE project_name CHARACTER SET utf8mb4;
CREATE USER 'project_name'@'localhost' IDENTIFIED BY '密碼';
GRANT ALL PRIVILEGES ON project_name.* TO 'project_name'@'localhost';
FLUSH PRIVILEGES;
exit
```

用編輯器打開 `.env` 檔案，修改以下設定。`DB_DATABASE`、`DB_USERNAME` 和密碼要改成剛剛設定的

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=project_name
DB_USERNAME=project_name
DB_PASSWORD='密碼'
```

執行 migration 測試設定是否正確

``` bash
php artisan migrate
```