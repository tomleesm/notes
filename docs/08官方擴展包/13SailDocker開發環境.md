# Sail Docker 開發環境

## 介紹

[Laravel Sail](https://github.com/laravel/sail) 是一個輕量級的命令列介面，用於與 Laravel 的默認 Docker 開發環境進行互動。Sail 為使用 PHP, MySQL, 和 Redis 建構 Laravel 應用提供了一個很好的起點，不需要事先有 Docker 經驗。

Sail 的核心是 `docker-compose.yml` 檔案和儲存在項目根目錄的 `sail` 指令碼。`sail` 指令碼為 CLI 提供了便捷的方法，可用於與 `docker-compose.yml` 檔案定義的 Docker 容器進行互動。

Laravel Sail 支援 macOS、Linux 和 Windows (通過 [WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)）。

## 安裝 & 設定

Laravel Sail 會隨著所有全新的 Laravel 應用程式一起自動安裝，因此你可以立即的開始使用它。要瞭解如何建立一個新的 Laravel 應用程式，請查閱適合您目前作業系統的 [安裝文件](https://learnku.com/docs/laravel/10.x/installation)。在安裝過程中，你將被要求選擇你的應用程式將與哪些 Sail 支援的服務進行互動。


### 安裝 Sail 到當前應用中

假如你有興趣在你現有的 Laravel 應用程式中使用 Sail，你可以透過 Composer 套件管理簡單的安裝 Sail。當然，這些步驟的前提是假設你現有的本地開發環境允許你安裝 Copmoser 依賴：

```shell
composer require laravel/sail --dev
```

在 Sail 完成安裝後，你可以運行 Artisan 命令 `sail:install`。這個命令將會發佈 Sail 的 `docker-compose.yml` 檔案到你應用程式的根目錄：

```shell
php artisan sail:install
```

最後，你可以啟動 Sail 的服務了。想要繼續學習如何使用 Sail，請接著閱讀本文擋的其餘部分：

```shell
./vendor/bin/sail up
```

#### 增加額外服務

如果你想在你現有的 Sail 安裝中新增一個額外的服務，你可以運行`sail:add` Artisan 命令。

```shell
php artisan sail:add
```

#### 使用開發容器

如果你想在 [Devcontainer](https://code.visualstudio.com/docs/remote/containers) 中進行開發，你可以在執行 `sail:install` 命令時新增 `--devcontainer` 參數。`--devcontainer` 將指示 `sail:install` 命令將默認的 `.devcontainer/devcontainer.json` 檔案發佈到你的應用程式根目錄：

```shell
php artisan sail:install --devcontainer
```

### 組態 Shell 別名

默認情況下，Sail 命令使用 `vendor/bin/sail` 指令碼呼叫，該指令碼已包含在所有新建的 Laravel 應用程式中：

```shell
./vendor/bin/sail up
```

但與其重複的輸入 `vendor/bin/sail` 來執行 Sail 命令，你可能會希望組態一個 Shell 別名方便你更容易的執行 Sail 命令：

```shell
alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'
```


為了確保這一點始終可用，你可以把它新增到你的主目錄下的 shell 組態檔案中，如 `~/.zshrc` 或 `~/.bashrc` ，然後重新啟動你的 shell。

一旦組態了 shell 別名，你可以通過簡單地輸入 `sail` 來執行 Sail 命令。本文接下來的示例都假定你已經組態了此別名：

```shell
sail up
```

## 啟動 & 停止 Sail

Laravel Sail 的 `docker-compose.yml` 檔案定義了各種 Docker 容器，它們可以協同工作以幫助你建構 Laravel 應用程式。每一個容器都定義在 `docker-compose.yml` 檔案的 `services` 的組態內。 `laravel.test` 容器是將服務於您的應用程式的主要應用程式容器。

在開始 Sail 之前，你應該確認沒有其他的網站伺服器或資料庫正運行在你的本地電腦上。要開始啟用 `docker-compose.yml` 檔案中定義的所有 Docker 容器，請執行 `up` 命令：

```shell
sail up
```

要在後台啟動所有的 Docker 容器，你可以以 "detached" 模式啟動 Sail。

```shell
sail up -d
```

啟動應用程式的容器後，你可以通過 Web 瀏覽器中訪問項目：[http://localhost](http://localhost/).

要停止所有的容器，你可以簡單的按 Control + C 來停止容器的執行。或者，如果容器是在背景執行的，你可以使用 `stop` 命令。

```shell
sail stop
```

## 執行命令

使用 Laravel Sail 時，應用程式在 Docker 容器中執行，並且與本地電腦隔離。不過 Sail 提供了一種針對應用程式運行各種命令的便捷方法，例如任意的 PHP 命令，Artisan 命令，Composer 命令和 Node / NPM 命令。



**當你閱讀 Laravel 文件時，你可能經常看到在未使用 Sail 的狀況下運行 Composer，Artisan 或是 Node / NPM 命令。** 以下示例假設你已經在本地電腦上安裝上述工具。如果你打算使用 Sail 建構你的本地開發環境 ，你需要改用 Sail 運行這些命令：

```shell
# 在本地運行 Artisan 命令...
php artisan queue:work

# 在 Laravel Sail 中運行 Artisan 命令...
sail artisan queue:work
```

### 執行 PHP 命令

PHP 命令可以使用 `php` 命令執行。當然，這些命令將使用為你的應用程式組態的 PHP 版本執行。要瞭解更多關於 PHP 版本可用的 Laravel Sail 資訊，請查閱 [PHP 版本文件](#sail-php-versions)：

```shell
sail php --version

sail php script.php
```

### 執行 Composer 命令

Composer 命令可以使用 `composer` 命令執行。Laravel Sail 的應用程式容器中已經安裝 Composer 2.x：

```nothing
sail composer require laravel/sanctum
```

#### 在已運行的應用中安裝 Composer 依賴

假如你與團隊一起開發應用程式，你也許不是最初建立 Laravel 應用程式的人。因此，當你克隆應用程式的倉庫到本地電腦後，倉庫默認不會安裝的任何 Composer 依賴項，也包括 Sail。

你可以進入到應用程式目錄下並執行以下命令來安裝應用所需的依賴，這個命令使用一個包含 PHP 與 Composer 的小型 Docker 容器進行應用程式依賴的安裝：

```shell
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php82-composer:latest \
    composer install --ignore-platform-reqs
```



當你使用 `laravelsail/phpXX-composer` 鏡像時，你應該選擇和你的應用程式所用環境相同的 PHP 版本（`74`、`80`、`81`或 `82`）。

### 執行 Artisan 命令

Artisan 命令可以使用 `artisan` 命令執行：

```shell
sail artisan queue:work
```

### 執行 Node / NPM 命令

Node 命令可以使用 `node` 命令執行，而 NPM 命令可以使用 `npm` 命令執行：

```shell
sail node --version

sail npm run dev
```

如果你願意，你可以使用 Yarn 代替 NPM：

```shell
sail yarn
```

## 與資料庫互動

### MySQL

你可能已經注意到，應用程式的 `docker-compose.yml` 檔案包含一個 MySQL 容器的組態。該容器使用了 [Docker volume](https://docs.docker.com/storage/volumes/)，以便即使在停止和重新啟動容器時依然可以持久儲存資料庫中儲存的資料。

此外，在MySQL容器第一次啟動時，它將為你建立兩個資料庫。第一個資料庫使用你的 `DB_DATABASE` 環境變數的值命名，用於你的本地開發。第二個是專門的測試資料庫，名為 `testing`，將確保你的測試不會干擾你的開發資料。

一旦你啟動了你的容器，你可以通過在你的應用程式的 `.env` 檔案中設定 `DB_HOST` 環境變數來連接到你的應用程式中的 MySQL 實例 `mysql`。

要從你的本地機器連接到你的應用程式的 MySQL 資料庫，你可以使用一個圖形化的資料庫管理應用程式，如 [TablePlus](https://tableplus.com/)。默認情況下，MySQL 資料庫可以通過 `localhost` 連接埠 3306 訪問，訪問憑證與 `DB_USERNAME` 和 `DB_PASSWORD` 環境變數的值一致。或者，你可以以 `root` 使用者的身份連接，它也利用 `DB_PASSWORD` 環境變數的值作為密碼。



### Redis

應用程式的 `docker-compose.yml` 檔案也包含 [Redis](https://redis.io/) 容器的組態，此容器使用 [Docker volume](https://docs.docker.com/storage/volumes/)，以便即使在停止和重新啟動容器後，Redis 資料中儲存的資料也可以持久保存。啟動容器後，可以通過將應用程式 `.env` 檔案中的環境變數 `REDIS_HOST` 設定為 `redis` 來連接到應用程式中的 Redis 實例。


要從本地電腦連接到應用程式的 Redis 資料庫，可以使用圖形資料庫管理應用程式，例如 [TablePlus](https://tableplus.com/)。默認情況下，可以從 `localhost` 的 6379 連接埠訪問 Redis 資料庫。

### Meilisearch

如果你在安裝 Sail 時選擇安裝 [MeiliSearch](https://www.meilisearch.com) 服務，你的應用程式的 `docker-compose.yml` 檔案將包含一個 [Laravel Scout](/docs/laravel/10.x/scout) 相容且強大的[搜尋引擎服務元件組態](https://github.com/meilisearch/meilisearch-laravel-scout)。啟動容器後，你可以通過將環境變數 `MEILISEARCH_HOST` 設定為 `http://meilisearch:7700` 來連接到應用程式中的 MeiliSearch 實例。

要從本地電腦訪問 MeiliSearch 的 Web 管理面板，你可以通過瀏覽器訪問 `http://localhost:7700`。

## 檔案儲存

如果你計畫在生產環境中運行應用程式時使用 Amazon S3 儲存檔案，你可能希望在安裝 Sail 時安裝 [MinIO](https://min.io) 服務。 MinIO 提供了一個與 S3 相容的 API，你可以使用 Laravel 的 `s3` 檔案儲存驅動程式在本地進行開發，而無需在生產 S3 環境中建立用於測試的儲存桶。如果在安裝 Sail 時選擇安裝 MinIO，部分 MinIO 相關的組態將新增到應用程式的 `docker-compose.yml` 檔案中。



默認情況下，應用程式的 `filesystems`  組態檔案已經包含 `s3` 磁碟的磁碟組態。除了使用此磁碟與 Amazon S3 互動之外，你還可以使用它與任何 S3 相容的檔案儲存服務（例如 MinIO）進行互動，只需修改控制其組態的關聯環境變數即可。例如，在使用 MinIO 時，你的檔案系統環境變數組態應定義如下：

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

為了讓 Laravel 的 Flysystem 整合在使用 MinIO 時產生正確的 URL，你應該定義 `AWS_URL` 環境變數，使其與你的應用程式的本地 URL 相匹配，並在 URL 路徑中包含桶的名稱。

```ini
AWS_URL=http://localhost:9000/local
```

你可以通過 MinIO 控制台建立桶，該控制台可在 `http://localhost:8900` 。MinIO 控制台的默認使用者名稱是 `sail`，默認密碼是 `password`。

> **警告**
> 使用 MinIO 時，不支援通過 `temporaryUrl` 方法生成臨時儲存的 URL。

## 運行測試

Laravel 提供了出色的開箱即用測試，你可以使用 Sail 的 `test` 命令運行應用程式的 [功能和單元測試](/docs/laravel/10.x/testing)。任何 PHPUnit 可接受的命令選項都可以透過 `test` 命令傳遞：

```shell
sail test

sail test --group orders
```

Sail `test` 命令相當於運行 Artisan `test` 命令：

```shell
sail artisan test
```



默認情況下, Sail會建立一個專門的 `測試` 資料庫, 這樣你的測試就不會干擾到你的資料庫的當前狀態. 在默認的Laravel安裝中, Sail也會組態你的 `phpunit.xml` 檔案, 在執行你的測試時使用這個資料庫:

```xml
<env name="DB_DATABASE" value="testing"/>
```

### Laravel Dusk

[Laravel Dusk](/docs/laravel/10.x/dusk) 提供了非常優雅、易於使用的瀏覽器自動化測試 API。有了 Sail，進行瀏覽器測試更加方便了，你甚至不用在你的本地電腦上安裝 Selenium 或者任何其他工具。要開啟這項功能，請在 `docker-compose.yml` 檔案中取消 Selenium 服務相關組態的註釋：

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

下一步，請確認 `docker-compose.yml` 檔案中的 `laravel.test` 服務組態 `depends_on` 是否包含了 `selenium` 選項：

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

最後，你可以透過啟動 Sail 並運行 `dusk` 命令來進行 Dusk 測試：

```shell
sail dusk
```

#### 在 Apple Silicon 上運行 Selenium

如果你的本地機器包含 Apple Silicon 晶片，你的 `selenium` 服務必須使用 `seleniarm/standalone-chromium` 鏡像：

```yaml
selenium:
    image: 'seleniarm/standalone-chromium'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

## 預覽電子郵件

Laravel Sail 默認的 `docker-compose.yml` 檔案中包含了一個服務項 [Mailpit](https://github.com/axllent/mailpit)。Mailpit 在本地開發過程中攔截應用程式傳送的郵件，並提供一個便捷的 Web 介面，這樣你就可以在瀏覽器中預覽你的郵件。當使用 Sail 時，Mailpit 的默認主機是 `mailpit` ，可通過連接埠 1025 使用。

```ini
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```



當 Sail 執行階段，你可以透過 [http://localhost:8025](http://localhost:8025/) 訪問 Mailpit 的 Web 介面。


## 容器 CLI

有時候，你可能想要在應用容器中開啟一個 Bash  session 。 可以通過執行 `shell` 命令，以訪問容器中的檔案和已安裝的服務，此外，你還可以執行其他任意 Shell 指令：

```shell
sail shell

sail root-shell
```

想打開一個新的 [Laravel Tinker](https://github.com/laravel/tinker)  session ，你可以執行 `tinker` 命令：

```shell
sail tinker
```

## PHP 版本

Sail目前支援通過 PHP 8.2、8.1、PHP 8.0 或 PHP 7.4 為你的應用程式提供服務。目前 Sail 使用的默認 PHP 版本是 PHP 8.2。想更改應用程式使用的 PHP 版本，請在 `docker-compose.yml` 檔案定義的容器 `laravel.test` 相應組態中調整 `build` 定義:

```yaml
# PHP 8.2
context: ./vendor/laravel/sail/runtimes/8.2

# PHP 8.1
context: ./vendor/laravel/sail/runtimes/8.1

# PHP 8.0
context: ./vendor/laravel/sail/runtimes/8.0

# PHP 7.4
context: ./vendor/laravel/sail/runtimes/7.4
```

此外，你如果想更新你的鏡像名稱來反映當前使用的 PHP 版本，你可以在 `docker-compose.yml` 檔案中調整 `image` 欄位：

```yaml
image: sail-8.1/app
```

在修改 `docker-compose.yml` 檔案過後，你需要重建容器鏡像並重啟 Sail：

```shell
sail build --no-cache

sail up
```

## Node 版本

Sail 默認安裝 Node 18。要更改鏡像建構時所安裝的 Node 版本，你可以在應用程式的 `docker-compose.yml` 檔案中更新 `laravel.test` 服務的 `build.args` 定義：

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '14'
```

在修改 `docker-compose.yml` 檔案過後，你需要重建容器鏡像並重啟 Sail：

```shell
sail build --no-cache

sail up
```



## 共享你的網站

有時候你可能需要公開分享你的網站給同事，或是測試應用與 Webhook 的整合。想共享你的網站時，可以使用 `share` 命令。當你執行此命令後，將會獲取一個隨機的網址，例如 `laravel-sail.site` 用來訪問你的應用程式：

```shell
sail share
```

當通過 `share` 命令共享你的站點時，你應該在 `TrustProxies` 中介軟體中組態應用程式的可信代理。否則，相關的URL 生成的助手函數，例如 `url` 和 `route` 將無法在生成 URL 生成過程中選擇正確 HTTP 主機地址：

    /**
     * 應用程式的受信任代理
     *
     * @var array|string|null
     */
    protected $proxies = '*';

如果你想為你的共享站點自訂子域名，可以在執行 `share` 命令時加上 `subdomain` 參數：

```shell
sail share --subdomain=my-sail-site
```

> **注意**
> `share` 命令是由 [Expose](https://github.com/beyondcode/expose) 提供，這是 [BeyondCode](https://beyondco.de/) 的一個開源網路隧道服務。

## 使用 Xdebug 進行偵錯

Laravel Sail 的 Docker 組態包含對 [Xdebug](https://xdebug.org/) 的支援，這是一個流行且強大的 PHP 偵錯程式。為了啟用 Xdebug，你需要在應用程式的 `.env` 檔案中新增一些變數以 [組態 Xdebug](https://xdebug.org/docs/step_debug#mode)。要啟用 Xdebug，你必須在啟動 Sail 之前設定適當的應用模式：

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

#### Linux 主機 IP 組態

在容器內部，`XDEBUG_CONFIG` 環境變數被定義為 `client_host=host.docker.internal` 以便為 Mac 和 Windows (WSL2) 正確組態 Xdebug。如果你的本地機器運行的是 Linux，確保你運行的是 Docker Engine 17.06.0+ 和 Compose 1.16.0+。否則，你將需要手動定義這個環境變數。

首先，你需要通過運行以下命令來確定要新增到環境變數中的正確主機 IP 地址。通常，`<container-name>` 應該是為你的應用程式提供服務的容器的名稱，並且通常以 `_laravel.test_1` 結尾：

```shell
docker inspect -f {{range.NetworkSettings.Networks}}{{.Gateway}}{{end}} <container-name>
```

在獲得正確的主機 IP 地址後，你需要在應用程式的 `.env` 檔案中定義 `SAIL_XDEBUG_CONFIG` 變數：

```ini
SAIL_XDEBUG_CONFIG="client_host=<host-ip-address>"
```

### 通過命令列使用 Xdebug 進行偵錯

在運行 Artisan 命令時，可以使用 `sail debug` 命令啟動偵錯 session ：

```shell
# 在沒有 Xdebug 的情況下運行 Artisan 命令...
sail artisan migrate

# 使用 Xdebug 運行 Artisan 命令...
sail debug migrate
```

### 通過瀏覽器使用 Xdebug 進行偵錯

要在通過 Web 瀏覽器與應用程式互動時偵錯你的應用程式，請按照 [Xdebug 提供的說明](https://xdebug.org/docs/step_debug#web-application) 從 Web 瀏覽器啟動 Xdebug  session 。

如果你使用的是 PhpStorm，請查看 JetBrains 關於 [零組態偵錯](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html) 的文件。

> **警告**
> Laravel Sail 依賴於 `artisan serve` 來為你的應用程式提供服務。從 Laravel 8.53.0 版本開始，`artisan serve` 命令只接受 `XDEBUG_CONFIG` 和 `XDEBUG_MODE` 變數。從 Laravel 8.53.0 版本開始，舊版本的 Laravel（8.52.0 及以下）不支援這些變數並且不接受偵錯連接。

## 定製化

因為 Sail 就是 Docker，所以你可以自由的定製任何內容，使用 `sail:publish` 命令可以將 Sail 預設的 Dockerfile 發佈到你的應用程式中，以便於進行定製：

```shell
sail artisan sail:publish
```

運行這個命令後，Laravel Sail 預設好的 Dockerfile 和其他組態檔案將被生成發佈到項目根目錄的 `docker` 目錄中。當你自行定製 Sail 組態之後，你可以在應用程式的 `docker-compose.yml` 檔案中更改應用程式容器的映像名稱。在完成上述操作後，你需要使用 `build` 命令重新建構容器。如果你使用 Sail 在單台機器上開發多個 Laravel 應用程式，那麼為應用程式的鏡像分配一個唯一的名稱將尤為重要：

```shell
sail build --no-cache
```

