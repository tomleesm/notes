# Envoy 部署工具

## 簡介

[Laravel Envoy](https://github.com/laravel/envoy) 是一套在遠端伺服器上執行日常任務的工具。 使用 [Blade](/docs/laravel/10.x/blade) 風格語法，你可以輕鬆地組態部署任務、Artisan  命令的執行等。目前， Envoy 僅支援 Mac 和 Linux 作業系統。但是， Windows 上可以使用  [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10) 來實現支援。

## 安裝

首先，運行 Composer 將 Envoy 安裝到你的項目中：

```shell
composer require laravel/envoy --dev
```

安裝 Envoy 之後， Envoy 的可執行檔案將出現在項目的 `vendor/bin` 目錄下：

```shell
php vendor/bin/envoy
```

## 編寫任務

### 定義任務

任務是 Envoy 的基礎建構元素。任務定義了你想在遠端伺服器上當任務被呼叫時所執行的 Shell 命令。例如，你定義了一個任務：在佇列伺服器上執行 `php artisan queue:restart` 命令。

你所有的 Envoy 任務都應該在項目根目錄中的 `Envoy.blade.php` 檔案中定義。下面是一個幫助你入門的例子：

```blade
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

正如你所見，在檔案頂部定義了一個 `@servers` 陣列，使你可以通過任務聲明的 `on` 選項引用這些伺服器。`@servers` 聲明應始終放置在單行中。在你的 `@task` 聲明中，你應該放置任務被呼叫執行時你期望在伺服器上運行的 Shell 命令。

#### 本地任務

你可以通過將伺服器的 IP 地址指定為  `127.0.0.1`  來強制指令碼在本地運行：

```blade
@servers(['localhost' => '127.0.0.1'])
```

#### 匯入 Envoy 任務

使用 `@import` 指令，你可以從其他的 Envoy 檔案匯入它們的故事與任務並新增到你的檔案中。匯入檔案後，你可以像定義在自己的 Envoy 檔案中一樣執行其中包含的任務：

```blade
@import('vendor/package/Envoy.blade.php')
```

### 多伺服器

Envoy 允許你輕鬆地在多台伺服器上運行任務。 首先，在  `@servers` 聲明中新增其他伺服器。每台伺服器都應分配一個唯一的名稱。一旦定義了其他伺服器，你可以在任務的 `on` 陣列中列出每個伺服器：

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

#### 平行執行

默認情況下，任務將在每個伺服器上序列執行。 換句話說，任務將在第一台伺服器上完成運行後，再繼續在第二台伺服器上執行。如果你想在多個伺服器上平行運行任務，請在任務聲明中新增 `parallel` 選項：

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

如你所見，檔案頂部定義了一個 `@server` 陣列，允許你在任務聲明的 `on` 選項中引用這些伺服器。`@server` 聲明應始終放在一行中。在你的 `@task` 聲明中，你應該放置任務被呼叫執行時你期望在伺服器上運行的 Shell 命令。

#### 本地任務

你可以通過將伺服器的 IP 地址指定為 `127.0.0.1` 來強制指令碼在本地運行：

```blade
@servers(['localhost' => '127.0.0.1'])
```

#### 匯入 Envoy 任務

使用 `@import` 指令，你可以從其他的 Envoy 檔案匯入它們的故事與任務並新增到你的檔案中。檔案匯入後，你可以執行他們所定義的任務，就像這些任務是在你的 Envoy 檔案中被定義的一樣：

```blade
@import('vendor/package/Envoy.blade.php')
```

### 組態

有時，你可能需要在執行 Envoy 任務之前執行一些 PHP 程式碼。你可以使用 `@setup` 指令來定義一段 PHP 程式碼塊，在任務執行之前執行這段程式碼：

```php
@setup
    $now = new DateTime;
@endsetup
```

如果你需要在任務執行之前引入其他的 PHP 檔案，你可以在 `Envoy.blade.php` 檔案的頂部使用 `@include` 指令：

```blade
@include('vendor/autoload.php')

@task('restart-queues')
    # ...
@endtask
```

### 變數

如果需要的話，你可以在呼叫 Envoy 任務時通過在命令列中指定參數，將參數傳遞給 Envoy 任務：

```shell
php vendor/bin/envoy run deploy --branch=master
```

你可以使用 Blade 的「echo」語法訪問傳入任務中的命令列參數。你也可以在任務中使用 `if` 語句和循環。 例如，在執行 `git pull` 命令之前，我們先驗證 `$branch` 變數是否存在：

```blade
@servers(['web' => ['user@192.168.1.1']])

@task('deploy', ['on' => 'web'])
    cd /home/user/example.com

    @if ($branch)
        git pull origin {{ $branch }}
    @endif

    php artisan migrate --force
@endtask
```

### 故事

通過「故事」功能，你可以給一組任務起個名字，以便後續呼叫。例如，一個 `deploy` 故事可以運行 `update-code` 和 `install-dependencies` 等定義好的任務：

```blade
@servers(['web' => ['user@192.168.1.1']])

@story('deploy')
    update-code
    install-dependencies
@endstory

@task('update-code')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('install-dependencies')
    cd /home/user/example.com
    composer install
@endtask
```

一旦編寫好了故事，你可以像呼叫任務一樣呼叫它：

```shell
php vendor/bin/envoy run deploy
```



### 任務鉤子

當任務和故事指令碼執行階段，會執行很多鉤子。Envoy 支援的鉤子類型有`@before`, `@after`, `@error`, `@success`, and `@finished`。 這些鉤子中的所有程式碼都被解釋為 PHP 並在本地執行，而不是在你的任務與之互動的遠端伺服器上執行。

你可以根據需要定義任意數量的這些。這些鉤子將按照它們在您的 Envoy 指令碼中出現的順序執行。

#### `@before`

在每個任務執行之前，Envoy 指令碼中註冊的所有 `@before` 鉤子都會執行。 `@before` 鉤子負責接收將要執行的任務的名稱：

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

#### `@after`

每次任務執行後，Envoy 指令碼中註冊的所有 `@after` 鉤子都會執行。 `@after` 鉤子負責接收已執行任務的名稱：

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

#### `@error`

在每次任務失敗後（以大於 `0` 的狀態碼退出執行），Envoy 指令碼中註冊的所有 `@error` 鉤子都將執行。 `@error` 鉤子負責接收已執行任務的名稱：

```blade
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

#### `@success`

如果所有任務都已正確執行，則 Envoy 指令碼中註冊的所有 `@success` 鉤子都將執行：

```blade
@success
    // ...
@endsuccess
```

#### `@finished`

在所有任務都執行完畢後（不管退出狀態如何），所有的 `@finished` 鉤子都會被執行。 `@finished` 鉤子負責接收已完成任務的狀態碼，它可能是 `null` 或大於或等於 `0` 的 `integer`：

```blade
@finished
    if ($exitCode > 0) {
        // There were errors in one of the tasks...
    }
@endfinished
```

### 鉤子

當任務和故事執行階段，會執行許多鉤子。 Envoy 支援的鉤子類型有 `@before`、`@after`、`@error`、`@success` 和 `@finished`。這些鉤子中的所有程式碼都被解釋為 PHP 並在本地執行，而不是在與你的任務互動的遠端伺服器上執行。

你可以根據需要定義任意數量的鉤子。 它們將按照它們在你的 Envoy 指令碼中出現的順序執行。

#### `@before`

在每個任務執行之前，將執行在你的 Envoy 指令碼中註冊的所有 `@before` 鉤子。 `@before` 鉤子接收將要執行的任務的名稱：

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

#### `@after`

每次任務執行後，將執行在你的 Envoy 指令碼中註冊的所有 `@after` 鉤子。 `@after` 鉤子接收已執行任務的名稱：

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

#### `@error`

在每個任務失敗後（退出時的狀態大於 `0`），在你的 Envoy 指令碼中註冊的所有 `@error` 鉤子都將執行。 `@error` 鉤子接收已執行任務的名稱：

```blade
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

#### `@success`

如果所有任務都沒有出現錯誤，那麼在你的 Envoy 指令碼中註冊的所有 `@success` 鉤子都將執行：

```blade
@success
    // ...
@endsuccess
```

#### `@finished`

在執行完所有任務後（無論退出狀態如何），所有的 `@finished` 鉤子都將被執行。 `@finished` 鉤子接收已完成任務的狀態程式碼，它可以是 `null` 或大於或等於 `0` 的 `integer`：

```blade
@finished
    if ($exitCode > 0) {
        // There were errors in one of the tasks...
    }
@endfinished
```

## 運行任務

要運行在應用程式的 `Envoy.blade.php` 檔案中定義的任務或 story，請執行 Envoy 的 `run` 命令，傳遞你想要執行的任務或 story 的名稱。Envoy 將執行該任務，並在任務執行階段顯示來自遠端伺服器的輸出：

```shell
php vendor/bin/envoy run deploy
```

### 確認任務執行

如果你想在在伺服器上運行給定任務之前進行確認，請將 `confirm` 指令新增到您的任務聲明中。此選項特別適用於破壞性操作：

```blade
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

## 通知

### Slack

Envoy 支援在每次任務執行後向 [Slack](https://slack.com/) 傳送通知。`@slack` 指令接受一個 Slack 鉤子 URL 和一個頻道/使用者名稱稱。您可以通過在 Slack 控制面板中建立 「Incoming WebHooks」 整合來檢索您的 webhook URL。

你應該將整個 webhook URL 作為傳遞給 `@slack` 指令的第一個參數。`@slack` 指令給出的第二個參數應該是頻道名稱 (`#channel`) 或使用者名稱稱 (`@user`)：

```blade
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

默認情況下，Envoy 通知將向通知頻道傳送一條消息，描述已執行的任務。但是，你可以通過將第三個參數傳遞給 `@slack` 指令來覆蓋此消息，以自訂自己的消息：

```blade
@finished
    @slack('webhook-url', '#bots', 'Hello, Slack.')
@endfinished
```

### Discord

Envoy 還支援在每個任務執行後向 [Discord](https://discord.com/) 傳送通知。`@discord` 指令接受一個 Discord Webhook URL 和一條消息。您可以在伺服器設定中建立「Webhook」，並選擇 Webhook 應該發佈到哪個頻道來檢索 Webhook URL。您應該將整個 Webhook URL 傳遞到 `@discord` 指令中：
```blade
@finished
    @discord('discord-webhook-url')
@endfinished
```

### Telegram

Envoy 還支援在每個任務執行後向 [Telegram](https://telegram.org/) 傳送通知。`@telegram` 指令接受一個 Telegram Bot ID 和一個 Chat ID。你可以使用 [BotFather](https://t.me/botfather) 建立一個新的機器人來檢索Bot ID。你可以使用 [@username_to_id_bot](https://t.me/username_to_id_bot) 檢索有效的 Chat ID。你應該將整個Bot ID和Chat ID傳遞到 `@telegram` 指令中：

```blade
@finished
    @telegram('bot-id','chat-id')
@endfinished
```

### Microsoft Teams

Envoy 還支援在每個任務執行後向 [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams) 傳送通知。`@microsoftTeams` 指令接受Teams Webhook（必填）、消息、主題顏色（成功、資訊、警告、錯誤）和一組選項。你可以通過建立新的 [incoming webhook](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) 來檢索Teams Webhook。Teams API 有許多其他屬性可以自訂消息框，例如標題、摘要和部分。你可以在 [Microsoft Teams文件](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message) 中找到更多資訊。你應該將整個Webhook URL 傳遞到 `@microsoftTeams` 指令中：

```blade
@finished
    @microsoftTeams('webhook-url')
@endfinished
```

