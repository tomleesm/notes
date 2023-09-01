# 契約 Contract

## 簡介

Laravel 的「契約（Contract）」是一組介面，它們定義由框架提供的核心服務。例如，`illuste\Contracts\Queue\Queue` Contract 定義了佇列所需的方法，而 `illuste\Contracts\Mail\Mailer` Contract 定義了傳送郵件所需的方法。

每個契約都有由框架提供的相應實現。例如，Laravel 提供了一個支援各種驅動的佇列實現，還有一個由 [SwiftMailer](https://symfony.com/doc/6.0/mailer.html) 提供支援的郵件程序實現等等。

所有的 Laravel Contract 都存在於它們各自的 [GitHub 倉庫](https://github.com/illuminate/contracts)。這為所有可用的契約提供了一個快速的參考點，以及一個可以被包開發人員使用的獨立的包。

### Contract 對比 Facade

Laravel 的 [Facade](/docs/laravel/10.x/facades) 和輔助函數提供了一種利用 Laravel 服務的簡單方法，無需類型提示並可以從服務容器中解析 Contract。在大多數情況下，每個 Facade 都有一個等效的 Contract。

和 Facade（不需要在建構函式中引入）不同，Contract 允許你為類定義顯式依賴關係。一些開發者更喜歡以這種方式顯式定義其依賴項，所以更喜歡使用 Contract，而其他開發者則享受 Facade 帶來的便利。**通常，大多數應用都可以在開發過程中使用 Facade。**


## 何時使用 Contract

使用 Contract 或 Facades 取決於個人喜好和開發團隊的喜好。Contract 和 Facade 均可用於建立功能強大且經過良好測試的 Laravel 應用。Contract 和 Facade 並不是一道單選題，你可以在同一個應用內同時使用 Contract 和 Facade。只要聚焦在類的職責應該單一上，你會發現 Contract 和 Facade 的實際差異其實很小。

通常情況下，大部分使用 Facade 的應用都不會在開發中遇到問題。但如果你在建立一個可以由多個 PHP 框架使用的擴展包，你可能會希望使用 `illuminate/contracts` 擴展包來定義該包和 Laravel 整合，而不需要引入完整的 Laravel 實現（不需要在 `composer.json` 中具體顯式引入 Laravel 框架來實現）。

## 如何使用 Contract

那麼，如何實現契約呢？它其實很簡單。

Laravel 中的許多類都是通過 [服務容器](https://learnku.com/docs/Laravel/10.x/container) 解析的，包括 controller 、事件偵聽器、中介軟體、佇列任務，甚至路由閉包。因此，要實現契約，你只需要在被解析的類的建構函式中「類型提示」介面。

例如，看看下面的這個事件監聽器：

    <?php

    namespace App\Listeners;

    use App\Events\OrderWasPlaced;
    use App\Models\User;
    use Illuminate\Contracts\Redis\Factory;

    class CacheOrderInformation
    {
        /**
         * 建立一個新的事件監聽器實例
         */
        public function __construct(
            protected Factory $redis,
        ) {}

        /**
         * 處理該事件。
         */
        public function handle(OrderWasPlaced $event): void
        {
            // ...
        }
    }



當解析事件監聽器時，服務容器將讀取建構函式上的類型提示，並注入適當的值。 要瞭解更多有關在服務容器中註冊內容的資訊，請查看 [其文件](/docs/laravel/10.x/container)。

## Contract 參考

下表提供了所有 Laravel Contract 及對應的 Facade 的快速參考：

不列印表格
