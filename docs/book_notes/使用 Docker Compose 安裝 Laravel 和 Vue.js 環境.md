# 使用 Docker Compose 安裝 Laravel 和 Vue.js 環境

想要用 Docker 建立 Laravel 環境，於是 [Google "Docker LEMP"](https://www.google.com/search?q=Docker+LEMP&oq=Docker+LEMP)，發現一篇很棒的文章[Docker for local web development, introduction: why should you care?](https://tech.osteel.me/posts/docker-for-local-web-development-introduction-why-should-you-care)。因為不想每次忘記指令都回去看這一系列文章（英文沒那麼好），於是寫筆記。

## 安裝 Docker

### 安裝指令

首先安裝 Docker，依照官方文件安裝即可，預設安裝 Docker 和 Docker Compose

- Windows：參考 [Install Docker Desktop on Windows](https://docs.docker.com/desktop/windows/install/)
- Linux：參考 [Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/) 或選擇對應的發行套件

如果是安裝在 Linux Mint，則用 [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)，在 Add the repository to Apt sources：

``` bash
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

`$(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")` 在 Linux Mint 是無效的，所以上述的指令改成

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  jammy stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

對應到 [Linux Mint Releases](https://linuxmint.com/download_all.php) 的 Package base

### WSL

如果在 WSL 安裝 Docker，可能會出現錯誤訊息：

> [sudo service docker start](docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.)

原因是 docker 服務沒有啟動，所以：

``` bash
# 啟動 docker
sudo service docker start
# 檢查 docker 已啟動
sudo service docker status
# 開機時啟動 docker
sudo systemctl enable docker
```

### 不用 sudo

docker 預設使用 root 執行，其他使用者必須用 sudo，所以用以下指令讓一般使用者也能用 docker

新增 docker 群組

``` bash
sudo groupadd docker
```

把目前登入的使用者（`$USER`）加入 docker 群組

``` bash
sudo usermod -aG docker $USER
```

重新登入，使得上面的指令生效，或者執行以下指令

``` bash
newgrp docker
```

如果上述方法無效，就重新啟動電腦

## 使用的指令簡介

依照 `docker-compose.yaml` 的設定新增並啟動容器，`-d` 參數表示在背景執行

``` bash
docker compose up -d
```

停止並刪除 docker compose 產生的容器和 volume（參數 `-v`）

``` bash
docker compose down -v
```

停止並刪除 docker compose 產生的容器、volume 和 image，自動略過仍被使用的 image

``` bash
docker compose down -v --rmi all --remove-orphans
```

顯示 `docker-compose.yaml` 相關的容器狀態(運作中、停止中)，`-a` 參數表示顯示包含停止中的容器

``` bash
docker compose ps -a
```

停止所有容器運作

``` bash
docker compose stop
```

要求 docker compose 在 php 容器上執行 bash，用來登入容器，登入後預設目錄是 docker 設定的 working_directory。指令 `exit` 離開容器

``` bash
docker compose exec php bash
```

顯示 nginx 容器的紀錄，如果省略 nginx 則顯示所有容器的紀錄。`-f` 代表持續監聽紀錄，按下 `Ctrl + C` 停止監聽

``` bash
docker compose logs -f nginx
```

新增 backend 容器並執行指令 `php -m` 後，刪除容器。`php -m` 指令顯示目前安裝的 php extension

``` bash
docker compose run --rm backend php -m
```
## 基本的 LEMP 環境

### Nginx

在專案目錄新增 `docker-compose.yml`，填入以下的設定

``` yaml
# Services
services:

  # Nginx Service
  nginx:
    # 使用 nginx 1.21 alpine 版本
    image: nginx:1.21-alpine
    # 本機的 port : 容器的 port
    ports:
      - 80:80
```

執行 `docker compose up -d` 後，等待下載並新增容器後，用瀏覽器打開 http://localhost ，應該會看到 nginx 歡迎畫面

### PHP

`docker-compose.yaml` 更新如下

``` yaml
# Services
services:

  # Nginx Service
  nginx:
    image: nginx:1.21-alpine
    ports:
      - 80:80
    volumes:
      - ./src:/var/www/php
      - ./.docker/nginx/conf.d:/etc/nginx/conf.d
    depends_on:
      - php

  # PHP Service
  php:
    image: php:8.1-fpm
    working_dir: /var/www/php
    volumes:
      - ./src:/var/www/php
```

`volumes` 用來同步本機和容器內的目錄與檔案，所以 `./src:/var/www/php` 表示本機端的 `./src` 目錄同步資料到容器內的 `/var/www/php` 目錄，src 目錄慣例用來放置應用程式原始碼

`depends_on` 確保 php-fpm 容器先成功新增，再來新增 nginx 容器，以免 nginx 容器回報找不到設定檔

新增目錄 `src` 和 `.docker/nginx/conf.d`。一般把 docker 設定放在 `.docker` 目錄內

``` bash
mkdir -p src .docker/nginx/conf.d
```

nginx 預設自動啟用 `/etc/nginx/conf.d` 目錄下所有副檔名為 conf 的設定檔，所以在本機端的 `.docker/nginx/conf.d` 目錄新增 `php.conf` 檔案，並填入 nginx 虛擬主機設定如下，讓 nginx 把 php 檔的請求轉給 php-fpm 處理。

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root   /var/www/php;
    index  index.php;

    location ~* \.php$ {
        fastcgi_pass   php:9000;
        include        fastcgi_params;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param  SCRIPT_NAME     $fastcgi_script_name;
    }
}
```

注意以上的 `fastcgi_pass   php:9000;` 設定，docker compose 自動建立內部網路並設定所有容器的 `/etc/hosts` 檔案，所以可以用 php 代表容器內部 ip，就像用 `localhost` 代表 `127.0.0.1`。

執行 `docker compose down` 停止並刪除之前的 nginx 容器，執行 `docker compose up -d` 新增並啟動容器。執行 `docker compose logs` 顯示紀錄檢查一下是否有錯誤，在 `src` 目錄新增 `index.php`，輸入程式碼如下

``` php
<?php
phpinfo();
```

然後在瀏覽器打開 http://localhost/ 應該會看到熟悉的 php 設定頁面

### MariaDB

`docker-compose.yaml` 修改成如下

``` yaml
# Services
services:

  # Nginx Service
  nginx:
    image: nginx:1.21-alpine
    ports:
      - 80:80
    volumes:
      - ./src:/var/www/php
      - ./.docker/nginx/conf.d:/etc/nginx/conf.d
    depends_on:
      - php

  # PHP Service
  php:
    build: ./.docker/php
    working_dir: /var/www/php
    volumes:
      - ./src:/var/www/php

  # MariaDB Service
  mariadb:
    image: mariadb:latest
    environment:
      - MARIADB_DATABASE=demo
      - MARIADB_USER=demo
      - MARIADB_PASSWORD=$MARIADB_PASSWORD
      - MARIADB_RANDOM_ROOT_PASSWORD=1
    volumes:
      - mariadbdata:/var/lib/mysql
    ports:
      - 127.0.0.1:3306:3306


# Volumes
volumes:

  mariadbdata:
```

需要 `pdo_mysql` extension 讓 PHP 查詢 MySQL，php-fpm 官方 image 沒有內建，但是有 `docker-php-ext-install` 指令來安裝 php 的 extension，所以 php 改用 `Dockerfile` 建立自訂的 docker image

新增目錄 `.docker/php`
``` bash
mkdir -p .docker/php
```

然後新增 Dockerfile 檔案到 `.docker/php` 目錄，填入以下設定

```
FROM php:8.1-fpm-alpine

RUN docker-php-ext-install pdo_mysql
```

原文使用 mysql/mysql-server，不過好像不能用了，老是顯示為 Error，所以改用 MariaDB。

`MARIADB_PASSWORD=$MARIADB_PASSWORD` 表示使用 `.env` 的 `MARIADB_PASSWORD` 值做為密碼，root 密碼則是自動亂數產生，表示不使用 root 登入。不須像原文那樣設定 my.cnf，使用 MariaDB 預設值即可。不再使用 healthy check ，實在太慢了。

新增檔案 `.env`，設定 MariaDB 密碼

```
MARIADB_PASSWORD='密碼'
```

## 前端 Vue.js 與後端 Laravel

### 後端 Laravel

#### 建立所需的 image 和權限

如果從前面的設定一路到此，需要刪除 `.docker/php` 目錄，以及檔案 `.docker/nginx/conf.d/php.conf` 和 `src/index.php`，目錄結構應該從以下那樣開始(執行 `tree -a .` 指令的結果)

```
├── .docker
│   └── nginx
│       └── conf.d
├── docker-compose.yaml
├── .env
└── src
```

修改 `docker-compose.yaml` 如下
``` yaml
# Services
services:

  # Nginx Service
  nginx:
    image: nginx:1.21-alpine
    ports:
      - 80:80
    volumes:
      - ./src/backend:/var/www/backend
      - ./.docker/nginx/conf.d:/etc/nginx/conf.d
    depends_on:
      - backend

  # Backend Service
  backend:
    build:
      context: ./src/backend
      args:
        HOST_UID: $HOST_UID
    working_dir: /var/www/backend
    volumes:
      - ./src/backend:/var/www/backend

  # MariaDB Service
  mariadb:
    image: mariadb:latest
    environment:
      - MARIADB_DATABASE=demo
      - MARIADB_USER=demo
      - MARIADB_PASSWORD=$MARIADB_PASSWORD
      - MARIADB_RANDOM_ROOT_PASSWORD=1
    volumes:
      - mariadbdata:/var/lib/mysql
    ports:
      - 127.0.0.1:3306:3306


# Volumes
volumes:

  mariadbdata:
```

修改的地方有：

- nginx 的 `volumes`：`- ./src:/var/www/php` 改為 `- ./src/backend:/var/www/backend`
- php 整個改成 backend

在 `.docker/nginx/conf.d` 目錄新增檔案 `backend.conf`，填入以下內容

```
server {
    listen      80;
    listen      [::]:80;
    root        /var/www/backend/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass  backend:9000;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include       fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

新增 `src/backend` 目錄

``` bash
mkdir -p src/backend
```

在 `src/backend` 目錄內新增 `Dockerfile`，並填入以下內容

``` yaml
FROM php:8.1-fpm-alpine

# Install extensions
RUN docker-php-ext-install pdo_mysql bcmath

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

# Create user based on provided user ID
ARG HOST_UID
RUN adduser --disabled-password --gecos "" --uid $HOST_UID demo

# Switch to that user
USER demo
```

使用 [multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/#use-multi-stage-builds) 方式直接複製檔案來安裝 Composer

`Dockerfile` 新增使用者 demo，並切換到 demo，這是為了讓之後新增的 Laravel 檔案權限是一般使用者，否則預設是 root。

執行以下的指令，顯示目前本機的使用者 uid

``` bash
id -u
```

然後在 `.env` 中設定環境變數

```
# 假設 id -u 的結果是 1000
HOST_UID=1000
```

最後執行 `docker compose build backend` 建立 image

#### 新增 Laravel 專案

新增 Laravel 專案 tmp，然後把 tmp 目錄內的檔案移出來，這樣 `src/backend/` 目錄內才會是 Laravel 檔案

``` bash
docker compose run --rm backend composer create-project --prefer-dist laravel/laravel tmp "10.*"

docker compose run --rm backend sh -c "mv -n tmp/.* ./ && mv tmp/* ./ && rm -Rf tmp"
```

修改 `src/backend/` 目錄的 `.env` 檔案，讓資料庫設定正確

```
DB_CONNECTION=mysql
# 要用 docker container 名稱 mariadb，不是用 127.0.0.1
DB_HOST=mariadb
DB_PORT=3306
# 因為 docker compose 的 MYSQL_DATABASE 設定是 demo
DB_DATABASE=demo
DB_USERNAME=demo
# .env 和 src/backend/.env 的 MARIADB_PASSWORD 設定值要一樣
DB_PASSWORD=密碼
```

`docker compose down -v` 刪除容器。`docker compose up -d` 新增容器，`docker compose ps -a` 確認容器正常運作 

執行指令 `docker compose exec backend php artisan migrate` 確認資料庫設定正確

#### OPcache

在 `src/backend` 目錄新增 `.docker` 目錄

``` bash
mkdir -p src/backend/.docker
```

在 `src/backend/.docker` 新增 `php.ini` 檔案，填入以下內容

```
[opcache]
opcache.enable=1
opcache.revalidate_freq=0
opcache.validate_timestamps=1
opcache.max_accelerated_files=10000
opcache.memory_consumption=192
opcache.max_wasted_percentage=10
opcache.interned_strings_buffer=16
opcache.fast_shutdown=1
```

修改 `src/backend` 目錄內的 `Dockerfile` 如下

```
FROM php:8.1-fpm-alpine

# Install extensions
RUN docker-php-ext-install pdo_mysql bcmath opcache

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

# Configure PHP
COPY .docker/php.ini $PHP_INI_DIR/conf.d/opcache.ini

# Use the default development or production configuration
ARG APP_ENV
RUN mv $PHP_INI_DIR/php.ini-$APP_ENV $PHP_INI_DIR/php.ini

# Create user based on provided user ID
ARG HOST_UID
RUN adduser --disabled-password --gecos "" --uid $HOST_UID demo

# Switch to that user
USER demo
```

`docker-compose.yaml` 需要設定 args APP_ENV

``` yaml
backend:
    build:
      context: ./src/backend
      args:
        HOST_UID: $HOST_UID
        APP_ENV: $APP_ENV
```

設定 `.env` 
```
# development 或是 production
APP_ENV='development'
```

最後執行 `docker compose build backend` 建立 image，`docker compose up -d` 新增並啟動容器

執行 `docker compose exec backend php -m` 確認有啟用 `Zend OPcache`

#### 安裝 Redis

修改 `src/backend` 目錄內的 `Dockerfile`，新增以下內容

```
# Install Redis extension
RUN apk add --no-cache pcre-dev $PHPIZE_DEPS \
    && pecl install redis 6.0.0 \
    && docker-php-ext-enable redis
```

執行 `docker compose build backend` 建立 image

修改 `docker-compose.yaml`，新增以下內容

``` yaml
# Redis Service
redis:
  image: redis:6-alpine
  command: ["redis-server", "--appendonly", "yes"]
  volumes:
    - redisdata:/data
```

`command` 用來覆蓋容器啟動後的預設指令。Redis 在此是用來執行佇列，最好保存排程紀錄，所以設定成 [Append Only File 模式](https://redis.io/docs/management/persistence/) (`--appendonly` 參數)

需要保存 redis 資料，所以新增一個 volume

``` yaml
volumes:
  redisdata:
```

因為 Redis 是給 backend 容器使用的，所以在 backend `depends_on` 新增以下內容，確保先啟動 Redis 容器

``` yaml
  depends_on:
    redis:
      condition: service_started
```

修改 `src/backend` 目錄內的 `.env`，redis host 要和資料庫一樣改用 docker 內部別名，所以改成以下這樣

```
QUEUE_CONNECTION=redis
REDIS_HOST=redis
```

執行指令 `docker compose up -d` 新增並啟動容器

#### 佇列 queue

接下來會有兩個執行 Laravel 的容器，一個執行一般的 web 請求，另一個執行佇列，因為兩個容器需要的 `Dockerfile` 很類似，所以用以下的方式

修改 `src/backend` 目錄內的 `Dockerfile`，開頭的 `FROM php:8.1-fpm-alpine` 加上 `as backend`，把這一行開始到另一個 FROM 之前的階段取名為 backend

```
FROM php:8.1-fpm-alpine as backend
```

以及以 backend 階段為基礎，取新的名稱 worker，在 `Dockerfile` 最底下新增以下的 FROM 和 CMD 內容

```
FROM backend as worker

# Start worker
CMD ["php", "/var/www/backend/artisan", "queue:work"]
```
`CMD` 指令設定 worker 容器啟動後自動執行 `php artisan queue:work`

修改 `docker-compose.yaml`，新增以下設定

``` yaml
# Worker Service
worker:
  build:
    context: ./src/backend
    target: worker
    args:
      HOST_UID: $HOST_UID
      APP_ENV: $APP_ENV
  working_dir: /var/www/backend
  volumes:
    - ./src/backend:/var/www/backend
  depends_on:
    - backend
```

上面的設定和 backend 容器很像，也用 `volume` 同步相同的 Laravel 檔案，唯一不同的是因為共用同一個 `Dockerfile`，所以要用 `target: worker` 指定使用的是 `worker` 階段。

因此 backend 容器也要加上 `target: backend`，指定使用 `backend` 階段

``` yaml
# Backend Service
backend:
  build:
    context: ./src/backend
    target: backend
    args:
      HOST_UID: $HOST_UID
      APP_ENV: $APP_ENV
```

建立兩個 image，所以執行兩個 build

``` bash
docker compose build backend
docker compose build worker
```

最後執行指令 `docker compose up -d` 新增並啟動容器

### 前端 Vue.js

#### 建立所需的 image 和權限

`docker-compose.yaml` 填入以下內容

``` yaml
# Frontend Service
frontend:
  build: ./src/frontend
  working_dir: /var/www/frontend
  volumes:
    - ./src/frontend:/var/www/frontend
  depends_on:
    - backend
```

以及在 nginx 的 depends_on 加上 `- frontend`

在 `src` 目錄新增 `frontend` 目錄

``` bash
mkdir -p src/frontend
```

然後新增 `Dockerfile` 檔案，填入以下設定

```
FROM node:20-alpine

USER node
```

切換到使用者 node，否則新增 Vue.js 專案的檔案權限預設是 root。在 [Docker for local web development, part 3: a three-tier architecture with frameworks](https://tech.osteel.me/posts/docker-for-local-web-development-part-3-a-three-tier-architecture-with-frameworks#the-frontend-application) 沒有如此設定，所以其實會產生問題

執行指令 `docker compose build frontend` 建立 image

#### 新增 Vue.js 專案

新增 Vue.js 專案 tmp，然後把 tmp 目錄內的檔案移出來，這樣 `src/frontend/` 目錄內才會是 Vue.js 檔案

``` bash
docker compose run --rm frontend yarn create vite tmp --template vue

docker compose run --rm frontend sh -c "mv -n tmp/.* ./ && mv tmp/* ./ && rm -Rf tmp"
```

安裝 Vue.js 依賴的套件

``` bash
docker compose run --rm frontend yarn
```

在 `.docker/nginx/conf.d` 目錄新增 `frontend.conf` 檔案，填入以下設定

```
server {
    listen      80;
    listen      [::]:80;

    location / {
        proxy_pass         http://frontend:8080;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection 'upgrade';
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   Host $host;
    }
}
```

上述設定連到 http://localhost ，會把請求轉到 frontend 容器處理，所以後端的 nginx 設定要改成 port 8000，否則會衝突，連到後端 Laravel

修改 `.docker/nginx/conf.d` 目錄內的 `backend.conf` 檔案，把 port 改成 8000
```
server {                                                                        
    listen      8000;                                                            
    listen      [::]:8000;                                                       
    root        /var/www/backend/public;    

```

還有 `docker-compose.yaml` 要修改 nginx ports，開放 8000

``` yaml
  nginx:
    image: nginx:1.21-alpine
    ports:
      - 80:80
      - 8000:8000
```

因為 nginx 設定成 port 80 直接轉到容器內部網址 http://frontend:8080 （使用 `proxy_pass`），由 Vite 提供 HTTP server，Vue.js 檔案不需要同步到 nginx，但是後端是交給 nginx 作為 HTTP server，轉交 php 請求給 php-fpm（使用 `fastcgi_pass`），php-fpm 會去找設定的 root 目錄中有沒有請求指定的 php 檔案，注意是檔案，不是網址，所以 `src/backend` 目錄的檔案需要同步給 nginx 。

在正式環境，直接打包 JavaScript 檔案，交給 nginx 做為一般的靜態檔案，所以不需要前端容器

在 `src/frontend` 新增 `vite.config.js` 檔案，填入以下設定，設置好 Vite 的 hot reload

``` javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
    plugins: [vue()],
    server: {
        host: true,
        hmr: {port: 80},
        port: 8080,
        watch: {
            usePolling: true
        }
    }
})
```

修改 `src/frontend` 目錄內的 `Dockerfile` 檔案，新增 `yarn dev` 指令，使容器啟動後自動執行測試環境

``` docker
FROM node:20-alpine

USER node

# Start application
CMD ["yarn", "dev"]
```

修改了 `Dockerfile`，所以要執行指令 `docker compose build frontend` 重建 image

最後執行指令 `docker compose up -d`，用瀏覽器打開 http://localhost 應該可以看到 Vue.js 的歡迎頁面

## 排程

文章中推薦使用 [Ofelia](https://hub.docker.com/r/mcuadros/ofelia) ，好處是如果要修改執行的週期、容器或指令，只需要修改 `config.ini` 設定，並重新啟動 Ofelia 容器即可。如果用 Linux cron，Google 了一下發現相當麻煩，這篇文章[如何在 Docker 中運行 Cron？](https://meishizaolunzi.com/zh-Hant/cron-in-docker/) 可以參考

在 `.docker` 目錄內新增 `scheduler` 目錄

``` bash
mkdir -p .docker/scheduler
```

然後在 `.docker/scheduler` 目錄內新增檔案 `config.ini`，填入以下設定

```
[job-exec "Laravel Scheduler"]
schedule = @every 1m
container = demo-backend-1
command = php /var/www/backend/artisan schedule:run
```

以上設定名稱為 `demo-backend-1` 的容器每分鐘執行一次 `php /var/www/backend/artisan schedule:run` 指令

修改 `docker-compose.yaml`，新增以下內容

``` yaml
# Scheduler Service
scheduler:
  image: mcuadros/ofelia:latest
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - ./.docker/scheduler/config.ini:/etc/ofelia/config.ini
  depends_on:
    - backend
```

執行 `docker compose up -d` 指令，新增並啟動容器，執行 `docker compose logs -f scheduler` 指令，確認 Ofelia 容器正常運作

PS. 可能需要設定 `backend container_name:demo-backend` ，以及修改 `.docker/scheduler/config.ini` 的 container 為 demo-backend。因為 docker container 名稱會用到資料夾名稱