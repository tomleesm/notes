# Homestead 虛擬機器

## 簡介

Laravel 致力於讓整個 PHP 開發體驗變得愉悅，包括您的本地開發環境。 [Laravel Homestead](https://github.com/laravel/homestead) 是一個官方的預封裝的 Vagrant box 套件，它為你提供了一個絕佳的開發環境，而無需你在本地機器上安裝 PHP、Web 伺服器及任何其他伺服器軟體。

[Vagrant](https://www.vagrantup.com) 提供了一種簡單、優雅的方式來管理和組態虛擬機器。 Vagrant Box 完全是一次性的。如果出現問題，你可以在幾分鐘內銷毀並重新建立 Box !

Homestead 可以在任何 Windows、 macOS 或 Linux 系統上運行，它預裝好了 Nginx、 PHP、 MySQL、 PostgreSQL、 Redis、 Memcached、 Node 以及開發令人驚嘆的 Laravel 應用程式所需的所有其他軟體。

> **注意**  
> 如果你使用的是 Windows ，你可能需要啟用硬體虛擬化（ VT-x ）。該功能通常需要通過你的 BIOS 啟用。如果你在 UEFI 系統上使用 Hyper-V ，則可能還需要停用 Hyper-V 才能訪問 VT-x 。

### 內建軟體

- Ubuntu 20.04
- Git
- PHP 8.2
- PHP 8.1
- PHP 8.0
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL 8.0
- lmm
- Sqlite3
- PostgreSQL 15
- Composer
- Docker
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- avahi
- ngrok
- Xdebug
- XHProf / Tideways / XHGui
- wp-cli

### 可選軟體

- Apache
- Blackfire
- Cassandra
- Chronograf
- CouchDB
- Crystal & Lucky Framework
- Elasticsearch
- EventStoreDB
- Gearman
- Go
- Grafana
- InfluxDB
- MariaDB
- Meilisearch
- MinIO
- MongoDB
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- R
- RabbitMQ
- RVM (Ruby Version Manager)
- Solr
- TimescaleDB
- Trader <small>(PHP extension)</small>
- Webdriver & Laravel Dusk Utilities

## 安裝 & 設定

### 第一步

在啟動 Homestead 環境之前，你必須安裝 [Vagrant](https://developer.hashicorp.com/vagrant/downloads) 及以下受支援的虛擬機器之一：

- [VirtualBox 6.1.x](https://www.virtualbox.org/wiki/Downloads)
- [Parallels](https://www.parallels.com/products/desktop/)

以上這些軟體包都為所有流行的作業系統提供了易於使用的可視化安裝程序。

如果要使用 Parallels 提供虛擬機器服務，你需要安裝 [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels)。這個外掛是免費的。

#### 安裝 Homestead

你可以通過將 Homestead 儲存庫克隆到你的主機上來安裝 Homestead。 考慮將儲存庫克隆到 `home` 目錄中的 `Homestead` 資料夾中，因為 Homestead 虛擬機器將作為所有 Laravel 應用程式的主機。 在本文件中，我們將此目錄稱為你的「Homestead 目錄」：

```shell
git clone https://github.com/laravel/homestead.git ~/Homestead
```

克隆 Laravel Homestead 儲存庫後，你應該檢出 `release` 分支。 這個分支總是包含 Homestead 的最新穩定版本：

```shell
cd ~/Homestead

git checkout release
```

接下來，從 Homestead 目錄執行 `bash init.sh` 命令以建立 `Homestead.yaml` 組態檔案。 `Homestead.yaml` 檔案是你為 Homestead 安裝組態所有設定的地方。 這個檔案將被放置在 Homestead 目錄中：

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

### 組態 Homestead

#### 設定提供服務的虛擬機器程序

`Homestead.yaml` 檔案中的 `provider` 鍵指示應該使用哪個 Vagrant 提供虛擬機器服務：`virtualbox` 或 `parallels`：

    provider: virtualbox

> **注意**  
> 如果你使用的是 Apple Silicon，你應該將 `box: laravel/homestead-arm` 新增到你的 `Homestead.yaml` 檔案中。 Apple Silicon 下需要使用 Parallels 提供虛擬機器服務。

#### 組態共享資料夾

`Homestead.yaml` 檔案的 `folders` 屬性列出了你希望與 Homestead 環境共享的所有資料夾。 當這些資料夾中的檔案發生更改時，它們將在你的本地機器和 Homestead 虛擬環境之間保持同步。 你可以根據需要組態任意數量的共享資料夾：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

> **注意**  
> Windows 使用者不應使用 `~/` 路徑語法，而應使用其項目的完整路徑，例如 `C:\Users\user\Code\project1`。

你應該始終將單個應用程式對應到它們自己的資料夾對應，而不是對應包含所有應用程式的單個大目錄。 對應資料夾時，虛擬機器需要跟蹤資料夾中每個檔案的所有磁碟 IO。 如果資料夾中有大量檔案，性能可能會降低：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
    - map: ~/code/project2
      to: /home/vagrant/project2
```

> **注意**  
> 在使用 Homestead 時，你永遠不應該掛載 `.`（當前目錄）。 這會導致 Vagrant 不會將當前資料夾對應到 `/vagrant`，並且會在組態時破壞可選功能並導致意外結果。

要啟用 [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html)，你可以在資料夾對應中新增一個 `type` 選項：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "nfs"
```

> **注意**  
> 在 Windows 上使用 NFS 時，應考慮安裝 [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd) 外掛。 該外掛將維護 Homestead 虛擬機器中檔案和目錄的正確使用者 / 組權限。

你還可以通過在 `options` 鍵下列出它們來傳遞 Vagrant 的 [同步資料夾](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) 支援的任何選項：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "rsync"
      options:
          rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
          rsync__exclude: ["node_modules"]
```

### 組態 Nginx 站點

不熟悉 Nginx？ 沒問題。 你的 `Homestead.yaml` 檔案的 `sites` 屬性允許你輕鬆地將「域」對應到 Homestead 環境中的資料夾。 `Homestead.yaml` 檔案中包含一個示例站點組態。 同樣，你可以根據需要向 Homestead 環境新增任意數量的站點。 Homestead 可以為你正在開發的每個 Laravel 應用程式提供方便的虛擬化環境：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
```

如果你在組態 Homestead 虛擬機器後更改了 `sites` 屬性，你應該在終端中執行 `vagrant reload --provision` 命令來更新虛擬機器上的 Nginx 組態。

> **注意**  
> Homestead 指令碼被建構為儘可能具有冪等性。 但是，如果你在組態時遇到問題，你應該通過執行 `vagrant destroy && vagrant up` 命令來銷毀和重建機器。

#### 主機名解析

Homestead 使用 `mDNS` 發佈主機名以進行自動主機解析。 如果你在 `Homestead.yaml` 檔案中設定 `hostname: homestead`，主機將在 `homestead.local` 中可用。 macOS、iOS 和 Linux 桌面發行版默認包含 `mDNS` 支援。 如果你使用的是 Windows，則必須安裝 [Bonjour Print Services for Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US)。

使用自動主機名最適合 Homestead 的 [每個項目安裝](#per-project-installation)。 如果你在單個 Homestead 實例上託管多個站點，你可以將你網站的「域名」新增到你機器上的 `hosts` 檔案中。 `hosts` 檔案會將你對 Homestead 站點的請求重新導向到你的 Homestead 虛擬機器中。 在 macOS 和 Linux 上，此檔案位於 `/etc/hosts`。 在 Windows 上，它位於 `C:\Windows\System32\drivers\etc\hosts` 。 你新增到此檔案的行將如下所示：

    192.168.56.56  homestead.test

確保列出的 IP 地址是你在 `Homestead.yaml` 檔案中設定的地址。將域名新增到 `hosts` 檔案並啟動 Vagrant 盒子後，你將能夠通過 Web 瀏覽器訪問該站點：

```shell
http://homestead.test
```

### 組態服務

Homestead 默認會啟動好幾個服務； 但你可以在組態的時候自訂啟用或停用哪些服務。 例如，你可以通過修改 `Homestead.yaml` 檔案中的 `services` 選項來啟用 PostgreSQL 並停用 MySQL：

```yaml
services:
    - enabled:
        - "postgresql"
    - disabled:
        - "mysql"
```

指定的服務將根據它們在 `enabled` 和 `disabled` 指令中的順序啟動或停止。

### 啟動 The Vagrant Box

你根據自己的需求修改 `Homestead.yaml` 後，你可以通過在 Homestead 目錄運行 `vagrant up` 命令來啟動 Vagrant 虛擬機器。 Vagrant 將啟動虛擬機器並自動組態你的共享資料夾和 Nginx 站點。

要銷毀虛擬機器實例，你可以使用 `vagrant destroy` 命令。

### 為項目單獨安裝

你可以為你管理的每個項目組態一個 Homestead 實例，而不是全域安裝 Homestead 並在所有項目中共享相同的 Homestead 虛擬機器。 如果你希望隨項目一起提供 `Vagrantfile`，允許其他人在克隆項目的儲存庫後立即 `vagrant up`，則為每個項目安裝 Homestead 可能會有所幫助。

你可以使用 Composer 包管理器將 Homestead 安裝到你的項目中：

```shell
composer require laravel/homestead --dev
```

安裝 Homestead 後，呼叫 Homestead 的 `make` 命令為你的項目生成 `Vagrantfile` 和 `Homestead.yaml` 檔案。 這些檔案將放置在項目的根目錄中。 `make` 命令將自動組態 `Homestead.yaml` 檔案中的站點和資料夾指令：

```shell
# macOS / Linux...
php vendor/bin/homestead make

# Windows...
vendor\\bin\\homestead make
```

接下來，在終端中運行 `vagrant up` 命令並在瀏覽器中通過 `http://homestead.test` 訪問你的項目。 請記住，如果你不使用自動 [主機名解析](#hostname-resolution)，你仍然需要為 `homestead.test` 或你選擇的域在 `/etc/hosts` 檔案中新增一個主機名對應。

### 安裝可選功能

使用 `Homestead.yaml` 檔案中的 `features` 選項可以安裝可選軟體。 大多數功能可以使用布林值啟用或停用，部分功能允許使用多個組態選項：

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
    - cassandra: true
    - chronograf: true
    - couchdb: true
    - crystal: true
    - elasticsearch:
        version: 7.9.0
    - eventstore: true
        version: 21.2.0
    - gearman: true
    - golang: true
    - grafana: true
    - influxdb: true
    - mariadb: true
    - meilisearch: true
    - minio: true
    - mongodb: true
    - mysql: true
    - neo4j: true
    - ohmyzsh: true
    - openresty: true
    - pm2: true
    - python: true
    - r-base: true
    - rabbitmq: true
    - rvm: true
    - solr: true
    - timescaledb: true
    - trader: true
    - webdriver: true
```

#### Elasticsearch

你可以指定支援的 Elasticsearch 版本，該版本必須是確切的版本號 (major.minor.patch)。 默認安裝將建立一個名為「homestead」的叢集。 你永遠不應該給 Elasticsearch 超過作業系統一半的記憶體，所以確保你的 Homestead 虛擬機器至少有 Elasticsearch 分配的兩倍。

> **注意**  
> 查看 [Elasticsearch 文件](https://www.elastic.co/guide/en/elasticsearch/reference/current) 瞭解如何自訂你的組態.



#### MariaDB

啟用 MariaDB 將會移除 MySQL 並安裝 MariaDB。MariaDB 通常是 MySQL 的替代品，完全相容 MySQL，所以在應用資料庫組態中你仍然可以使用 `mysql` 驅動。

#### MongoDB

默認安裝的 MongoDB 將會設定資料庫使用者名稱為 `homestead` 及對應的密碼為 `secret`。

#### Neo4j

Neo4j 是一個圖形資料庫，默認安裝的 Neo4j 會設定資料庫使用者名稱為 `homestead` 及對應的密碼 `secret`。要通過瀏覽器訪問 Neo4j ，請通過 Web 瀏覽器訪問 `http://homestead.test:7474`。默認情況下，服務預設了連接埠 `7687`（Bolt）、`7474`（HTTP）和 `7473`（HTTPS）為來自 Neo4j 客戶端的請求提供服務。

### 系統命令別名

您可以通過修改 Homestead 目錄中的 `aliases` 檔案將 Bash 命令別名新增到 Homestead 虛擬機器：

```shell
alias c='clear'
alias ..='cd ..'
```

當你更新完 `aliases` 檔案後，你需要通過 `vagrant reload --provision` 命令重啟 Homestead 機器，以確保新的別名在機器上生效。

## 更新 Homestead

更新 Homestead 之前確保你已經在 Homestead 目錄下通過如下命令移除了當前的虛擬機器：

```shell
vagrant destroy
```

接下來，需要更新 Homestead 原始碼，如果你已經克隆倉庫到本地，可以在項目根目錄下運行如下命令進行更新：

```shell
git fetch

git pull origin release
```

這些命令會從 Github 儲存庫中拉取最新的 Homestead 倉庫程式碼到本地，包括最新的標籤版本。你可以在 Homestead 的 [GitHub 發佈頁面](https://github.com/laravel/homestead/releases) 上找到最新的穩定版本。



如果你是通過 Composer 在指定 Laravel 項目中安裝的 Homestead，需要確保 `composer.json` 中包含了 `"laravel/homestead": "^12"`，然後更新這個依賴：

```shell
composer update
```

之後，你需要通過 `vagrant box update` 命令更新 Vagrant：

```shell
vagrant box update
```

接下來，你可以從 Homestead 目錄下運行 `bash init.sh` 命令來更新 Homestead 額外的組態檔案，你會被詢問是否覆蓋已存在的 `Homestead.yaml`、`after.sh` 以及 `aliases` 檔案：

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

最後，你需要重新生成新的 Homestead 虛擬機器來使用最新安裝的 Vagrant：

```shell
vagrant up
```

## 日常使用方法

### 通過 SSH 連接

你可以在 Homestead 目錄下通過運行 `vagrant ssh` 以 SSH 方式連接到虛擬機器。如果你設定了全部訪問 Homestead，也可以在任意路徑下通過 homestead ssh 登錄到虛擬機器。

### 新增其他站點

Homestead 虛擬機器在執行階段，可能需要新增多個 Laravel 應用到 Nginx 站點。如果是在單個 Homestead 環境中運行多個 Laravel 應用，新增站點很簡單，只需將站點新增到 `Homestead.yaml` 檔案：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    - map: another.test
      to: /home/vagrant/project2/public
```

> **注意**  
> 在新增站點之前，你應該確保已經為項目的目錄組態了[組態共享資料夾](#configuring-shared-folders)。

如果 Vagrant 沒有自動管理你的「hosts」檔案，你可能還需要將新站點新增到該檔案中。在 macOS 和 Linux 上，此檔案位於 `/etc/hosts`。在 Windows 上，它位於 `C:\Windows\System32\drivers\etc\hosts`：

    192.168.56.56  homestead.test
    192.168.56.56  another.test



新增站點後，你需要從 Homestead 目錄執行 `vagrant reload --provision` 命令以保證 Vagrant 載入新的站點。

#### 站點類型

Homestead 支援多種「類型」的站點，讓你可以輕鬆運行不是基於 Laravel 的項目。 例如，我們可以使用 `statamic` 站點類型輕鬆地將 Statamic 應用程式新增到 Homestead：

```yaml
sites:
    - map: statamic.test
      to: /home/vagrant/my-symfony-project/web
      type: "statamic"
```
可用的站點類型有： `apache`、`apigility`、`expressive`、`laravel`（默認）、`proxy`、`silverstripe`、`statamic`、`symfony2`、`symfony4` 和 `zf`。

#### 站點參數

你可以通過 `params` 站點指令向你的站點新增額外的 Nginx `fastcgi_param` 值：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      params:
          - key: FOO
            value: BAR
```

### 環境變數

你可以 `Homestead.yaml` 檔案來定義全域環境變數：

```yaml
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

更新 `Homestead.yaml` 檔案後，請務必通過執行 `vagrant reload --provision` 命令重新組態機器。 這將更新所有已安裝 PHP 版本的 PHP-FPM 組態，並為 `vagrant` 使用者更新環境。

### 連接埠

默認情況下，以下連接埠會轉發到你的 Homestead 環境：

- **HTTP:** 8000 &rarr;  轉發到 80
- **HTTPS:** 44300 &rarr;  轉發到 443

#### 轉發額外的連接埠

如你所願，你可以通過在你的 `Homestead.yaml` 檔案中定義一個 `ports` 組態項來將額外的連接埠轉發到 Vagrant 虛擬機器。 更新 `Homestead.yaml` 檔案後，請務必通過執行 `vagrant reload --provision` 命令重新載入虛擬機器組態：

```yaml
ports:
    - send: 50000
      to: 5000
    - send: 7777
      to: 777
      protocol: udp
```

以下是你可能希望從主機對應到 Vagrant box 的其他 Homestead 服務的連接埠清單：

- **SSH:** 2222 &rarr; 轉發到 22
- **ngrok UI:** 4040 &rarr; 轉發到 4040
- **MySQL:** 33060 &rarr; 轉發到 3306
- **PostgreSQL:** 54320 &rarr; 轉發到 5432
- **MongoDB:** 27017 &rarr; 轉發到 27017
- **Mailhog:** 8025 &rarr; 轉發到 8025
- **Minio:** 9600 &rarr; 轉發到 9600

### 多 PHP 版本

Homestead 引入了對在同一虛擬機器上運行多個版本的 PHP 的支援。 你可以在 `Homestead.yaml` 檔案中指定用於特定站點的 PHP 版本。 可用的 PHP 版本有：「5.6」、「7.0」、「7.1」、「7.2」、「7.3」、「7.4」、「8.0」、「8.1」和「8.2」（默認）：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "7.1"
```

[在你的 Homestead 虛擬機器中](#connecting-via-ssh)，你可以通過 CLI 使用任何支援的 PHP 版本：

```shell
php5.6 artisan list
php7.0 artisan list
php7.1 artisan list
php7.2 artisan list
php7.3 artisan list
php7.4 artisan list
php8.0 artisan list
php8.1 artisan list
php8.2 artisan list
```

你可以通過在 Homestead 虛擬機器中發出以下命令來更改 CLI 使用的默認 PHP 版本：

```shell
php56
php70
php71
php72
php73
php74
php80
php81
php82
```

### 連接到資料庫

Homestead 開箱即用地為 MySQL 和 PostgreSQL 組態了一個 `homestead` 資料庫。如果你想用宿主機的資料庫客戶端連接到 MySQL 或 PostgreSQL 資料庫，你可以通過連接 `127.0.0.1` （本地網路）的 `33060` 連接埠（MySQL） 或 `54320` 連接埠（PostgreSQL）。 兩個資料庫的使用者名稱和密碼都是 `homestead`/`secret`。

> **注意**  
> 只有在從宿主機連接到資料庫時，你才需要使用這些非標準連接埠。 由於 Laravel 在虛擬機器中運行，因此你將在 Laravel 應用程式的資料庫組態檔案中使用默認的 3306 和 5432 連接埠。

### 資料庫備份

當你的 Homestead 虛擬機器被銷毀時，Homestead 可以自動備份你的資料庫。 要使用此功能，你必須使用 Vagrant 2.1.0 或更高版本。 或者，如果你使用的是舊版本的 Vagrant，則必須安裝 `vagrant-triggers` 外掛。要啟用自動資料庫備份，請將以下行新增到你的 `Homestead.yaml` 檔案中：

    backup: true

組態完成後，當執行 `vagrant destroy` 命令時，Homestead 會將你的資料庫匯出到 `.backup/mysql_backup` 和 `.backup/postgres_backup` 目錄。 如果你選擇了[為項目單獨安裝](#per-project-installation) Homestead，你可以在項目安裝 Homestead 的資料夾中找到這些目錄，或者在你的項目根目錄中找到它們。

### 組態 Cron 工作排程

Laravel 提供了一種便捷方式來滿足[任務調度](/docs/laravel/10.x/scheduling)，通過 Artisan 命令 `schedule:run` 實現了定時運行（每分鐘執行一次）。 `schedule:run` 命令將檢查在 `App\Console\Kernel` 類中定義的作業計畫，以確定要運行哪些工作排程。

如果你想為 Homestead 站點運行 `schedule:run` 命令，可以在定義站點時將 `schedule` 選項設定為 `true`：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

站點的 cron 作業將在 Homestead 虛擬機器的 `/etc/cron.d` 目錄中被定義。

### 組態 MailHog

[MailHog](https://github.com/mailhog/MailHog) 會在你本地開發的過程中攔截應用程式傳送的電子郵件，而不是將郵件實際傳送給收件人。如果要使用 MailHog，你需要參考以下郵件組態並更新應用程式的 `.env` 檔案：

```ini
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```



組態 MailHog 後，你可以通過 http://localhost:8025 訪問 MailHog 儀表盤。

### 組態 Minio

[Minio](https://github.com/minio/minio) 是一個具有 Amazon S3 相容 API 的開源對象儲存伺服器。 要安裝 Minio，請使用 [features](#installing-optional-features) 部分中的以下組態選項更新你的 `Homestead.yaml` 檔案

    minio: true

默認情況下，Minio 在連接埠 9600 上可用。你可以通過訪問 `http://localhost:9600` 訪問 Minio 控制面板。 默認訪問金鑰是 `homestead`，而默認金鑰是 `secretkey`。 訪問 Minio 時，應始終使用區域 `us-east-1`

為了使用 Minio，你需要在應用程式的 `config/filesystems.php` 組態檔案中調整 S3 磁碟組態。 你需要將 `use_path_style_endpoint` 選項新增到磁碟組態中，並將 `url` 鍵更改為 `endpoint`:

    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'endpoint' => env('AWS_URL'),
        'use_path_style_endpoint' => true,
    ]

最後，確保你的 `.env` 檔案包含以下選項：

```ini
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
AWS_URL=http://localhost:9600
```

要組態 Minio 支援的「S3」儲存桶，請在你的 `Homestead.yaml` 檔案中新增 `buckets` 指令。 定義儲存桶後，你應該在終端中執行 `vagrant reload --provision` 命令多載虛擬機器：

```yaml
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

支援的 `policy` 值包括：`none`、`download`、`upload` 和 `public`。

### Laravel Dusk 測試工具

為了在 Homestead 中運行 [Laravel Dusk](/docs/laravel/10.x/duskk) 測試，你應該在 Homestead 組態中啟用 [`webdriver` 功能](#installing-optional-features):

```yaml
features:
    - webdriver: true
```

啟用 `webdriver` 功能後，你應該在終端中執行 `vagrant reload --provision` 命令多載虛擬機器。

### 共享你的環境

有時，你可能希望與同事或客戶分享你目前正在做的事情。 Vagrant 通過 `vagrant share` 命令內建了對此的支援； 但是，如果你在 `Homestead.yaml` 檔案中組態了多個站點，這個功能將不可用。

為瞭解決這個問題，Homestead 包含了自己的 `share` 命令。 首先，通過 `vagrant ssh` [SSH 到你的 Homestead 虛擬機器](#connecting-via-ssh) 並執行 `share homestead.test` 命令。 此命令將從你的 `Homestead.yaml` 組態檔案中共享 `homestead.test` 站點。 你可以將任何其他組態的站點取代為 `homestead.test`：

```shell
share homestead.test
```

運行該命令後，你將看到一個 Ngrok 螢幕出現，其中包含活動日誌和共享站點的可公開訪問的 URL。 如果你想指定自訂區域、子域或其他 Ngrok 執行階段選項，你可以將它們新增到你的 `share` 命令中：

```shell
share homestead.test -region=eu -subdomain=laravel
```

> **注意**  
> 請記住，Vagrant 本質上是不安全的，並且你在運行 `share` 命令時會將虛擬機器暴露在網際網路上。

## 偵錯和分析

### 使用 Xdebug 偵錯 Web 請求



Homestead 支援使用 [Xdebug](https://xdebug.org) 進行步驟偵錯。例如，你可以在瀏覽器中訪問一個頁面，PHP 將連接到你的 IDE 以允許檢查和修改正在運行的程式碼。

默認情況下，Xdebug 將自動運行並準備好接受連接。 如果需要在 CLI 上啟用 Xdebug，請在 Homestead 虛擬機器中執行 `sudo phpenmod xdebug` 命令。接下來，按照 IDE 的說明啟用偵錯。最後，組態你的瀏覽器以使用擴展或 [bookmarklet](https://www.jetbrains.com/phpstorm/marklets/) 觸發 Xdebug。

> **注意**  
> Xdebug 導致 PHP 運行速度明顯變慢。要停用 Xdebug，請在 Homestead 虛擬機器中運行 `sudo phpdismod xdebug` 並重新啟動 FPM 服務。

#### 自動啟動 Xdebug

在偵錯向 Web 伺服器發出請求的功能測試時，自動啟動偵錯比修改測試以通過自訂標頭或 cookie 來觸發偵錯更容易。 要強制 Xdebug 自動啟動，請修改 Homestead 虛擬機器中的 `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` 檔案並新增以下組態:

```ini
; 如果 Homestead.yaml 包含 IP 地址的不同子網，則這個 IP 地址可能會不一樣
xdebug.client_host = 192.168.10.1
xdebug.mode = debug
xdebug.start_with_request = yes
```

### 偵錯 CLI 應用程式

要偵錯 PHP CLI 應用程式，請在 Homestead 虛擬機器中使用 `xphp` shell 別名：

    xphp /path/to/script

### 使用 Blackfire 分析應用程式

[Blackfire](https://blackfire.io/docs/introduction) 是一種用於分析 Web 請求和 CLI 應用程式的服務。它提供了一個互動式使用者介面，可在呼叫圖和時間線中顯示組態檔案資料。Blackfire 專為在開發、登台和生產中使用而建構，對終端使用者沒有任何開銷。此外，Blackfire 還提供對程式碼和 `php.ini` 組態設定的性能、質量和安全檢查。



[Blackfire Player](https://blackfire.io/docs/player/index) 是一個開放原始碼的 Web 爬行、Web 測試和 Web 抓取應用程式，可以與 Blackfire 聯合使用以編寫分析場景的指令碼。

要啟用 Blackfire，請使用 Homestead 組態檔案中的「features」組態項：

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Blackfire 伺服器憑據和客戶端憑據需要使用 [Blackfire 帳戶](https://blackfire.io/signup)。 Blackfire 提供了多種選項來分析應用程式，包括 CLI 工具和瀏覽器擴充套件。 請查看 [Blackfire 文件](https://blackfire.io/docs/php/integrations/laravel/index)以獲取更多詳細資訊。

## 網路介面

`Homestead.yaml` 檔案的 `networks` 屬性為你的 Homestead 虛擬機器組態網路介面。 你可以根據需要組態任意數量的介面：

```yaml
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

要啟用 [bridged](https://www.vagrantup.com/docs/networking/public_network.html) 介面，請為將網路組態調整為 `bridge` 並將網路類型更改為 `public_network`：

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

要啟用 [DHCP](https://www.vagrantup.com/docs/networking/public_network.html) 功能，你只需從組態中刪除 `ip` 選項：

```yaml
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

## 擴展 Homestead

你可以使用 Homestead 目錄根目錄中的 `after.sh` 指令碼擴展 Homestead。 在此檔案中，你可以新增正確組態和自訂虛擬機器所需的任何 shell 命令。



當你自訂 Homestead 時，Ubuntu 可能會詢問你是要保留軟體包的原始組態還是使用新的組態檔案覆蓋它。 為了避免這種情況，你應該在安裝軟體包時使用以下命令，以避免覆蓋 Homestead 之前編寫的任何組態：

```shell
sudo apt-get -y \
    -o Dpkg::Options::="--force-confdef" \
    -o Dpkg::Options::="--force-confold" \
    install package-name
```

### 使用者自訂

與你的團隊一起使用 Homestead 時，你可能需要調整 Homestead 以更好地適應你的個人開發風格。 為此，你可以在 Homestead 目錄（包含 `Homestead.yaml` 檔案的同一目錄）的根目錄中建立一個 `user-customizations.sh` 檔案。 在此檔案中，你可以進行任何你想要的自訂； 但是， `user-customizations.sh` 不應受版本管理工具控制。

## 針對虛擬機器軟體的特殊設定

### VirtualBox

#### `natdnshostresolver`

默認情況下，Homestead 將 `natdnshostresolver` 設定組態為 `on`。 這允許 Homestead 使用你的主機作業系統的 DNS 設定。 如果你想覆蓋此行為，請將以下組態選項新增到你的 `Homestead.yaml` 檔案中：

```yaml
provider: virtualbox
natdnshostresolver: 'off'
```

#### Windows 上的符號連結

如果符號連結在你的 Windows 機器上不能正常工作，你可能需要將以下程式碼塊新增到你的 `Vagrantfile`：

```ruby
config.vm.provider "virtualbox" do |v|
    v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
end
```

