# Laravel Dusk

## 介紹

[Laravel Dusk](https://github.com/laravel/dusk) 提供了一套富有表現力、易於使用的瀏覽器自動化和測試 API。默認情況下，Dusk 不需要在本地電腦上安裝 JDK 或 Selenium。相反，Dusk 使用一個獨立的 [ChromeDriver](https://sites.google.com/chromium.org/driver) 安裝包。你可以自由地使用任何其他相容 Selenium 的驅動程式。

## 安裝

為了開始使用，你需要先安裝 [Google Chrome](https://www.google.com/chrome) 並將 `laravel/dusk` Composer 依賴新增到你的項目中：

```shell
composer require --dev laravel/dusk
```

> **警告**
> 如果你手動註冊 Dusk 的服務提供者，在生產環境中 **絕不要** 註冊，因為這可能導致任意使用者能夠認證你的應用程式。

安裝 Dusk 包後，執行 `dusk:install` Artisan 命令。`dusk:install` 命令將會建立一個 `tests/Browser` 目錄，一個示例 Dusk 測試，並為你的作業系統安裝 Chrome 驅動程式二進制檔案：

```shell
php artisan dusk:install
```

接下來，在應用程式的 `.env` 檔案中設定 `APP_URL` 環境變數。該值應該與你用於在瀏覽器中訪問應用程式的 URL 匹配。

> **注意**
> 如果你正在使用 [Laravel Sail](/docs/laravel/10.x/sail) 管理你的本地開發環境，請參閱 Sail 文件中有關[組態和運行 Dusk 測試](/docs/laravel/10.x/sail#laravel-dusk)的內容。

### 管理 ChromeDriver 安裝

如果你想安裝與 Laravel Dusk 通過 `dusk:install` 命令安裝的不同版本的 ChromeDriver，則可以使用 `dusk:chrome-driver` 命令：

```shell
# 為你的作業系統安裝最新版本的 ChromeDriver...
php artisan dusk:chrome-driver

# 為你的作業系統安裝指定版本的 ChromeDriver...
php artisan dusk:chrome-driver 86

# 為所有支援的作業系統安裝指定版本的 ChromeDriver...
php artisan dusk:chrome-driver --all

# 為你的作業系統安裝與 Chrome / Chromium 檢測到的版本匹配的 ChromeDriver...
php artisan dusk:chrome-driver --detect
```

> **警告**
> Dusk 需要 `chromedriver` 二進制檔案可執行。如果你無法運行 Dusk，你應該使用以下命令確保二進制檔案可執行：`chmod -R 0755 vendor/laravel/dusk/bin/`。

### 使用其他瀏覽器

默認情況下，Dusk 使用 Google Chrome 和獨立的 [ChromeDriver](https://sites.google.com/chromium.org/driver) 安裝來運行你的瀏覽器測試。但是，你可以啟動自己的 Selenium 伺服器，並運行你希望的任何瀏覽器來運行測試。

要開始，請打開你的 `tests/DuskTestCase.php` 檔案，該檔案是你的應用程式的基本 Dusk 測試用例。在這個檔案中，你可以刪除對 `startChromeDriver` 方法的呼叫。這將停止 Dusk 自動啟動 ChromeDriver：

    /**
     * 準備執行 Dusk 測試。
     *
     * @beforeClass
     */
    public static function prepare(): void
    {
        // static::startChromeDriver();
    }

接下來，你可以修改 `driver` 方法來連接到你選擇的 URL 和連接埠。此外，你可以修改應該傳遞給 WebDriver 的“期望能力”：

    use Facebook\WebDriver\Remote\RemoteWebDriver;

    /**
     * 建立 RemoteWebDriver 實例。
     */
    protected function driver(): RemoteWebDriver
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

## 入門

### 生成測試

要生成 Dusk 測試，請使用 `dusk:make` Artisan 命令。生成的測試將放在 `tests/Browser` 目錄中：

```shell
php artisan dusk:make LoginTest
```


### 在每次測試後重設資料庫

你編寫的大多數測試將與從應用程式資料庫檢索資料的頁面互動；然而，你的 Dusk 測試不應該使用 `RefreshDatabase` trait。`RefreshDatabase` trait 利用資料庫事務，這些事務將不適用或不可用於 HTTP 請求。相反，你有兩個選項：`DatabaseMigrations` trait 和 `DatabaseTruncation` trait。

#### 使用資料庫遷移

`DatabaseMigrations` trait 會在每次測試之前運行你的資料庫遷移。但是，為了每次測試而刪除和重新建立資料庫表通常比截斷表要慢：

    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;
    }

> **警告**
> 當執行 Dusk 測試時，不能使用 SQLite 記憶體資料庫。由於瀏覽器在其自己的處理程序中執行，因此它將無法訪問其他處理程序的記憶體資料庫。

#### 使用資料庫截斷

在使用 `DatabaseTruncation` trait 之前，你必須使用 Composer 包管理器安裝 `doctrine/dbal` 包：

```shell
composer require --dev doctrine/dbal
```

`DatabaseTruncation` trait 將在第一次測試時遷移你的資料庫，以確保你的資料庫表已經被正確建立。但是，在後續測試中，資料庫表將僅被截斷 - 相比重新運行所有的資料庫遷移，這樣做可以提高速度：

    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseTruncation;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseTruncation;
    }


默認情況下，此 trait 將截斷除 `migrations` 表以外的所有表。如果你想自訂應該截斷的表，則可以在測試類上定義 `$tablesToTruncate` 屬性：

    /**
     * 表示應該截斷哪些表。
     *
     * @var array
     */
    protected $tablesToTruncate = ['users'];


或者，你可以在測試類上定義 `$exceptTables` 屬性，以指定應該從截斷中排除的表：

    /**
     * 表示應該從截斷中排除哪些表。
     *
     * @var array
     */
    protected $exceptTables = ['users'];


為了指定需要清空表格的資料庫連接，你可以在測試類中定義一個 `$connectionsToTruncate` 屬性：

    /**
     * 表示哪些連接需要清空表格。
     *
     * @var array
     */
    protected $connectionsToTruncate = ['mysql'];


### 運行測試

要運行瀏覽器測試，執行 `dusk` Artisan 命令：

```shell
php artisan dusk
```

如果上一次運行 `dusk` 命令時出現了測試失敗，你可以通過 `dusk:fails` 命令先重新運行失敗的測試，以節省時間：

```shell
php artisan dusk:fails
```

`dusk` 命令接受任何 PHPUnit 測試運行器通常接受的參數，例如你可以只運行給定[組](https://phpunit.readthedocs.io/en/9.5/annotations.html#group)的測試：

```shell
php artisan dusk --group=foo
```

> **注意**
> 如果你正在使用 [Laravel Sail](/docs/laravel/10.x/sail) 來管理本地開發環境，請參考 Sail 文件中有關[組態和運行 Dusk 測試](/docs/laravel/10.x/sail#laravel-dusk)的部分。

#### 手動啟動 ChromeDriver

默認情況下，Dusk 會自動嘗試啟動 ChromeDriver。如果對於你的特定系統無法自動啟動，你可以在運行 `dusk` 命令之前手動啟動 ChromeDriver。如果你選擇手動啟動 ChromeDriver，則應該註釋掉 `tests/DuskTestCase.php` 檔案中的以下程式碼：

    /**
     * 為 Dusk 測試執行做準備。
     *
     * @beforeClass
     */
    public static function prepare(): void
    {
        // static::startChromeDriver();
    }

此外，如果你在連接埠 9515 以外的連接埠上啟動 ChromeDriver，你需要修改同一類中的 `driver` 方法以反映正確的連接埠：

    use Facebook\WebDriver\Remote\RemoteWebDriver;

    /**
     * 建立 RemoteWebDriver 實例。
     */
    protected function driver(): RemoteWebDriver
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

### 環境處理

如果要在運行測試時強制 Dusk 使用自己的環境檔案，請在項目根目錄中建立一個 `.env.dusk.{當前環境}` 檔案。例如，如果你將從你的 `local` 環境啟動 `dusk` 命令，你應該建立一個 `.env.dusk.local` 檔案。

在運行測試時，Dusk 將備份你的 `.env` 檔案，並將你的 Dusk 環境重新命名為 `.env`。測試完成後，會將你的 `.env` 檔案還原。

## 瀏覽器基礎知識

### 建立瀏覽器

為了開始學習，我們編寫一個測試，驗證我們能否登錄到我們的應用程式。生成測試後，我們可以修改它以導航到登錄頁面，輸入一些憑據並點選“登錄”按鈕。為了建立一個瀏覽器實例，你可以在 Dusk 測試中呼叫 `browse` 方法：

    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * 一個基本的瀏覽器測試示例。
         */
        public function test_basic_example(): void
        {
            $user = User::factory()->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function (Browser $browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'password')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

如上面的例子所示，`browse` 方法接受一個閉包。瀏覽器實例將由 Dusk 自動傳遞給此閉包，並且是與應用程式互動和進行斷言的主要對象。

#### 建立多個瀏覽器

有時你可能需要多個瀏覽器來正確地進行測試。例如，測試與 WebSockets 互動的聊天螢幕可能需要多個瀏覽器。要建立多個瀏覽器，只需將更多的瀏覽器參數新增到傳遞給 `browse` 方法的閉包簽名中即可：

    $this->browse(function (Browser $first, Browser $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');

        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');

        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });

### 導航

`visit` 方法可用於在應用程式中導航到給定的 URI：

    $browser->visit('/login');

你可以使用 `visitRoute` 方法來導航到 [命名路由](/docs/laravel/10.x/routing#named-routes)：

    $browser->visitRoute('login');

你可以使用 `back` 和 `forward` 方法來導航「後退」和「前進」：

    $browser->back();

    $browser->forward();

你可以使用 `refresh` 方法來刷新頁面：

    $browser->refresh();


### 調整瀏覽器窗口大小

你可以使用 `resize` 方法來調整瀏覽器窗口的大小：

    $browser->resize(1920, 1080);

你可以使用 `maximize` 方法來最大化瀏覽器窗口：

    $browser->maximize();

`fitContent` 方法將調整瀏覽器窗口的大小以匹配其內容的大小：

    $browser->fitContent();

當測試失敗時，Dusk 將在擷取螢幕截圖之前自動調整瀏覽器大小以適合內容。你可以在測試中呼叫 `disableFitOnFailure` 方法來停用此功能：

    $browser->disableFitOnFailure();

你可以使用`move`方法將瀏覽器窗口移動到螢幕上的其他位置：

    $browser->move($x = 100, $y = 100);

### 瀏覽器宏

如果你想定義一個可以在各種測試中重複使用的自訂瀏覽器方法，可以在`Browser`類中使用`macro`方法。通常，你應該從[服務提供者](/docs/laravel/10.x/providers)的`boot`方法中呼叫它：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Laravel\Dusk\Browser;

    class DuskServiceProvider extends ServiceProvider
    {
        /**
         * 註冊 《Dusk》 的瀏覽器宏。
         */
        public function boot(): void
        {
            Browser::macro('scrollToElement', function (string $element = null) {
                $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

                return $this;
            });
        }
    }

該 `macro` 函數接收方法名作為其第一個參數，並接收閉包作為其第二個參數。 將宏作為`Browser`實現上的方法呼叫宏時，將執行宏的閉包：

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/pay')
                ->scrollToElement('#credit-card-details')
                ->assertSee('Enter Credit Card Details');
    });

### 使用者認證

我們經常會測試需要身份驗證的頁面，你可以使用 Dusk 的`loginAs`方法來避免在每次測試期間與登錄頁面進行互動。該`loginAs`方法接收使用者 ID 或者使用者模型實例

    use App\Models\User;
    use Laravel\Dusk\Browser;

    $this->browse(function (Browser $browser) {
        $browser->loginAs(User::find(1))
              ->visit('/home');
    });

> **注意**
> 使用`loginAs`方法後，使用者 session 在檔案中的所有測試被維護。



### Cookies

你可以使用`cookie`方法來獲取或者設定加密過的 cookie 的值：

    $browser->cookie('name');

    $browser->cookie('name', 'Taylor');

使用`plainCookie`則可以獲取或者設定未加密過的 cookie 的值：

    $browser->plainCookie('name');

    $browser->plainCookie('name', 'Taylor');

你可以使用`deleteCookie`方法刪除指定的 cookie：

    $browser->deleteCookie('name');

### 運行 JavaScript

可以使用`script`方法在瀏覽器中執行任意 JavaScript 語句：

    $browser->script('document.documentElement.scrollTop = 0');

    $browser->script([
        'document.body.scrollTop = 0',
        'document.documentElement.scrollTop = 0',
    ]);

    $output = $browser->script('return window.location.pathname');

### 獲取截圖

你可以使用`screenshot`方法來截圖並將其指定檔案名稱儲存，所有截圖都將存放在`tests/Browser/screenshots`目錄下：

    $browser->screenshot('filename');

`responsiveScreenshots`方法可用於在不同斷點處擷取一系列截圖:

    $browser->responsiveScreenshots('filename');

### 控制台輸出結果保存到硬碟

你可以使用`storeConsoleLog`方法將控制台輸出指定檔案名稱並寫入磁碟，控制台輸出默認存放在`tests/Browser/console`目錄下：

    $browser->storeConsoleLog('filename');

### 頁面原始碼保存到硬碟

你可以使用`storeSource`方法將頁面當前原始碼指定檔案名稱並寫入磁碟，頁面原始碼默認會存放到`tests/Browser/source`目錄：

    $browser->storeSource('filename');

## 與元素互動

### Dusk 選擇器

編寫 Dusk 測試最困難的部分之一就是選擇良好的 CSS 選擇器與元素進行互動。 隨著時間的推移，前端的更改可能會導致如下所示的 CSS 選擇器無法通過測試：

    // HTML...

    <button>Login</button>

    // Test...

    $browser->click('.login-page .container div > button');

Dusk 選擇器可以讓你專注於編寫有效的測試，而不必記住 CSS 選擇器。要定義一個選擇器，你需要新增一個`dusk`屬性在 HTML 元素中。然後在選擇器前面加上`@`用來在 Dusk 測試中操作元素：

    // HTML...

    <button dusk="login-button">Login</button>

    // Test...

    $browser->click('@login-button');

### 文字、值 & 屬性

#### 獲取 & 設定值

Dusk 提供了多個方法用於和頁面元素的當前顯示文字、值和屬性進行互動，例如，要獲取匹配指定選擇器的元素的「值」，使用`value`方法：

    // 獲取值...
    $value = $browser->value('selector');

    // 設定值...
    $browser->value('selector', 'value');

你可以使用`inputValue`方法來獲取包含指定欄位名稱的輸入元素的「值」：

    $value = $browser->inputValue('field');

#### 獲取文字

該`text`方法可以用於獲取匹配指定選擇器元素文字：

    $text = $browser->text('selector');

#### 獲取屬性

最後，該`attribute`方法可以用於獲取匹配指定選擇器元素屬性：

    $attribute = $browser->attribute('selector', 'value');

### 使用表單

#### 輸入值

Dusk 提供了多種方法來與表單和輸入元素進行互動。首先，讓我們看一個在欄位中輸入值的示例：

    $browser->type('email', 'taylor@laravel.com');

注意，儘管該方法在需要時接收，但是我們不需要將 CSS 選擇器傳遞給`type`方法。如果沒有提供 CSS 選擇器，Dusk 會搜尋包含指定`name`屬性的`input`或`textarea`欄位。

要想將文字附加到一個欄位之後而且不清除其內容， 你可以使用`append`方法：

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');

你可以使用`clear`方法清除輸入值：

    $browser->clear('email');

你可以使用`typeSlowly`方法指示 Dusk 緩慢鍵入。 默認情況下，Dusk 在兩次按鍵之間將暫停 100 毫秒。 要自訂按鍵之間的時間量，你可以將適當的毫秒數作為方法的第二個參數傳遞：

    $browser->typeSlowly('mobile', '+1 (202) 555-5555');

    $browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);

你可以使用`appendSlowly`方法緩慢新增文字：

    $browser->type('tags', 'foo')
            ->appendSlowly('tags', ', bar, baz');

#### 下拉菜單

需要在下拉菜單中選擇值，你可以使用`select`方法。 類似於`type`方法， 該`select`方法並不是一定要傳入 CSS 選擇器。 當使用`select`方法時，你應該傳遞選項實際的值而不是它的顯示文字：

    $browser->select('size', 'Large');

你也可以通過省略第二個參數來隨機選擇一個選項：

    $browser->select('size');

通過將陣列作為`select`方法的第二個參數，可以指示該方法選擇多個選項：

    $browser->select('categories', ['Art', 'Music']);

#### 複選框

使用「check」 複選框時，你可以使用`check`方法。 像其他許多與 input 相關的方法，並不是必須傳入 CSS 選擇器。 如果精準的選擇器無法找到的時候，Dusk 會搜尋能夠與`name`屬性匹配的複選框：

    $browser->check('terms');

該`uncheck`方法可用於「取消選中」複選框輸入：

    $browser->uncheck('terms');

#### 單選按鈕

使用 「select」中單選按鈕選項時，你可以使用`radio`這個方法。 像很多其他的與輸入相關的方法一樣， 它也並不是必須傳入 CSS 選擇器。如果精準的選擇器無法被找到的時候， Dusk 會搜尋能夠與`name`屬性或者`value`屬性相匹配的`radio`單選按鈕：

    $browser->radio('size', 'large');

### 附件

該`attach`方法可以附加一個檔案到`file`input 元素中。 像很多其他的與輸入相關的方法一樣，他也並不是必須傳入 CSS 選擇器。如果精準的選擇器沒有被找到的時候，Dusk 會搜尋與`name`屬性匹配的檔案輸入框：

    $browser->attach('photo', __DIR__.'/photos/mountains.png');

> **注意**
> attach 方法需要使用 PHP`Zip`擴展，你的伺服器必須安裝了此擴展。

### 點選按鈕

可以使用`press`方法來點選頁面上的按鈕元素。該`press`方法的第一個參數可以是按鈕的顯示文字，也可以是 CSS/ Dusk 選擇器：

    $browser->press('Login');

提交表單時，許多應用程式在按下表單後會停用表單的提交按鈕，然後在表單提交的 HTTP 請求完成後重新啟用該按鈕。要按下按鈕並等待按鈕被重新啟用，可以使用`pressAndWaitFor`方法：

    // 按下按鈕並等待最多5秒，它將被啟用…
    $browser->pressAndWaitFor('Save');

    // 按下按鈕並等待最多1秒，它將被啟用…
    $browser->pressAndWaitFor('Save', 1);

### 點選連結

要點選連結，可以在瀏覽器實例下使用`clickLink`方法。該`clickLink`方法將點選指定文字的連結：

    $browser->clickLink($linkText);

你可以使用`seeLink`方法來確定具有給定顯示文字的連結在頁面上是否可見：

    if ($browser->seeLink($linkText)) {
        // ...
    }

> **注意**
> 這些方法與 jQuery 互動。 如果頁面上沒有 jQuery，Dusk 會自動將其注入到頁面中，以便在測試期間可用。

### 使用鍵盤

該`keys`方法讓你可以再指定元素中輸入比`type`方法更加複雜的輸入序列。例如，你可以在輸入值的同時按下按鍵。在這個例子中，輸入`taylor`時，`shift`鍵也同時被按下。當`taylor`輸入完之後， 將會輸入`swift`而不會按下任何按鍵：

    $browser->keys('selector', ['{shift}', 'taylor'], 'swift');

`keys`方法的另一個有價值的用例是向你的應用程式的主要 CSS 選擇器傳送「鍵盤快速鍵」組合：

    $browser->keys('.app', ['{command}', 'j']);

> **技巧**
> 所有修飾符鍵如`{command}`都包裹在`{}`字元中，並且與在 `Facebook\WebDriver\WebDriverKeys`類中定義的常數匹配，該類可以[在 GitHub 上找到](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php).

### 使用滑鼠

#### 點選元素

該`click`方法可用於「點選」與給定選擇器匹配的元素：

    $browser->click('.selector');

該`clickAtXPath`方法可用於「點選」與給定 XPath 表示式匹配的元素：

    $browser->clickAtXPath('//div[@class = "selector"]');

該`clickAtPoint`方法可用於「點選」相對於瀏覽器可視區域的給定坐標對上的最高元素：

    $browser->clickAtPoint($x = 0, $y = 0);

該`doubleClick`方法可用於模擬滑鼠的連按兩下：

    $browser->doubleClick();

該`rightClick`方法可用於模擬滑鼠的右擊：

    $browser->rightClick();

    $browser->rightClick('.selector');

該`clickAndHold`方法可用於模擬被點選並按住的滑鼠按鈕。 隨後呼叫 `releaseMouse` 方法將撤消此行為並釋放滑鼠按鈕：

    $browser->clickAndHold()
            ->pause(1000)
            ->releaseMouse();

#### 滑鼠懸停

該`mouseover`方法可用於與給定選擇器匹配的元素的滑鼠懸停動作：

    $browser->mouseover('.selector');

#### 拖放

該`drag`方法用於將與指定選擇器匹配的元素拖到其它元素：

    $browser->drag('.from-selector', '.to-selector');

或者，可以在單一方向上拖動元素：

    $browser->dragLeft('.selector', $pixels = 10);
    $browser->dragRight('.selector', $pixels = 10);
    $browser->dragUp('.selector', $pixels = 10);
    $browser->dragDown('.selector', $pixels = 10);

最後，你可以將元素拖動給定的偏移量：

    $browser->dragOffset('.selector', $x = 10, $y = 10);

### JavaScript 對話方塊

Dusk 提供了各種與 JavaScript 對話方塊進行互動的方法。例如，你可以使用`waitForDialog`方法來等待 JavaScript 對話方塊的出現。此方法接受一個可選參數，該參數指示等待對話方塊出現多少秒：

    $browser->waitForDialog($seconds = null);

該`assertDialogOpened`方法，斷言對話方塊已經顯示，並且其消息與給定值匹配：

    $browser->assertDialogOpened('Dialog message');

`typeInDialog`方法，在打開的 JavaScript 提示對話方塊中輸入給定值：

    $browser->typeInDialog('Hello World');

`acceptDialog`方法，通過點選確定按鈕關閉打開的 JavaScript 對話方塊：

    $browser->acceptDialog();

`dismissDialog`方法，通過點選取消按鈕關閉打開的 JavaScript 對話方塊（僅對確認對話方塊有效）：

    $browser->dismissDialog();

### 選擇器作用範圍

有時可能希望在給定的選擇器範圍內執行多個操作。比如，可能想要斷言表格中存在某些文字，然後點選表格中的一個按鈕。那麼你可以使用`with`方法實現此需求。在傳遞給`with`方法的閉包內執行的所有操作都將限於原始選擇器：

    $browser->with('.table', function (Browser $table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

你可能偶爾需要在當前範圍之外執行斷言。 你可以使用`elsewhere`和`elsewhereWhenAvailable`方法來完成此操作：

     $browser->with('.table', function ($table) {
        // 當前範圍是 `body .table`...

        $browser->elsewhere('.page-title', function ($title) {
            // 當前範圍是 `body .page-title`...
            $title->assertSee('Hello World');
        });

        $browser->elsewhereWhenAvailable('.page-title', function ($title) {
            // 當前範圍是 `body .page-title`...
            $title->assertSee('Hello World');
        });
     });

### 等待元素

在測試大面積使用 JavaScript 的應用時，在進行測試之前，通常有必要 「等待」 某些元素或資料可用。Dusk 可輕鬆實現。使用一系列方法，可以等到頁面元素可用，甚至給定的 JavaScript 表示式執行結果為`true`。

#### 等待

如果需要測試暫停指定的毫秒數， 使用`pause`方法：

    $browser->pause(1000);

如果你只需要在給定條件為`true`時暫停測試，請使用`pauseIf`方法:

    $browser->pauseIf(App::environment('production'), 1000);

同樣地，如果你需要暫停測試，除非給定的條件是`true`，你可以使用`pauseUnless`方法:

    $browser->pauseUnless(App::environment('testing'), 1000);

#### 等待選擇器

該`waitFor`方法可以用於暫停執行測試，直到頁面上與給定 CSS 選擇器匹配的元素被顯示。默認情況下，將在暫停超過 5 秒後拋出異常。如有必要，可以傳遞自訂超時時長作為其第二個參數：

    // 等待選擇器不超過 5 秒...
    $browser->waitFor('.selector');

    // 等待選擇器不超過 1 秒...
    $browser->waitFor('.selector', 1);

你也可以等待選擇器顯示給定文字：

    //  等待選擇器不超過 5 秒包含給定文字...
    $browser->waitForTextIn('.selector', 'Hello World');

    //  等待選擇器不超過 1 秒包含給定文字...
    $browser->waitForTextIn('.selector', 'Hello World', 1);

你也可以等待指定選擇器從頁面消失:

    // 等待不超過 5 秒 直到選擇器消失...
    $browser->waitUntilMissing('.selector');

    // 等待不超過 1 秒 直到選擇器消失...
    $browser->waitUntilMissing('.selector', 1);

或者，你可以等待與給定選擇器匹配的元素被啟用或停用：

    // 最多等待 5 秒鐘，直到選擇器啟用...
    $browser->waitUntilEnabled('.selector');

    // 最多等待 1 秒鐘，直到選擇器啟用...
    $browser->waitUntilEnabled('.selector', 1);

    // 最多等待 5 秒鐘，直到選擇器被停用...
    $browser->waitUntilDisabled('.selector');

    // 最多等待 1 秒鐘，直到選擇器被停用...
    $browser->waitUntilDisabled('.selector', 1);

#### 限定範疇範圍（可用時）

有時，你或許希望等待給定選擇器出現，然後與匹配選擇器的元素進行互動。例如，你可能希望等到模態窗口可用，然後在模態窗口中點選「確定」按鈕。在這種情況下，可以使用`whenAvailable`方法。給定回呼內的所有要執行的元素操作都將被限定在起始選擇器上:

    $browser->whenAvailable('.modal', function (Browser $modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

#### 等待文字

`waitForText`方法可以用於等待頁面上給定文字被顯示：

    // 等待指定文字不超過 5 秒...
    $browser->waitForText('Hello World');

    // 等待指定文字不超過 1 秒...
    $browser->waitForText('Hello World', 1);

你可以使用`waitUntilMissingText`方法來等待，直到顯示的文字已從頁面中刪除為止:

    // 等待 5 秒刪除文字...
    $browser->waitUntilMissingText('Hello World');

    // 等待 1 秒刪除文字...
    $browser->waitUntilMissingText('Hello World', 1);

#### 等待連結

`waitForLink`方法用於等待給定連結文字在頁面上顯示:

    // 等待連結 5 秒...
    $browser->waitForLink('Create');

    // 等待連結 1 秒...
    $browser->waitForLink('Create', 1);

#### 等待輸入

`waitForInput`方法可用於等待，直到給定的輸入欄位在頁面上可見:

    // 等待 5 秒的輸入…
    $browser->waitForInput($field);

    // 等待 1 秒的輸入…
    $browser->waitForInput($field, 1);

#### 等待頁面跳轉

當給出類似`$browser->assertPathIs('/home')`的路徑斷言時，如果`window.location.pathname`被非同步更新，斷言就會失敗。可以使用`waitForLocation`方法等待頁面跳轉到給定路徑：

    $browser->waitForLocation('/secret');

`waitForLocation`方法還可用於等待當前窗口位置成為完全限定的 URL：

    $browser->waitForLocation('https://example.com/path');

還可以使用[被命名的路由](/docs/laravel/10.x/routing#named-routes)等待跳轉：

    $browser->waitForRoute($routeName, $parameters);

#### 等待頁面重新載入

如果要在頁面重新載入後斷言，可以使用`waitForReload`方法：

    use Laravel\Dusk\Browser;

    $browser->waitForReload(function (Browser $browser) {
        $browser->press('Submit');
    })
    ->assertSee('Success!');

由於需要等待頁面重新載入通常發生在點選按鈕之後，為了方便起見，你可以使用`clickAndWaitForReload`方法：

    $browser->clickAndWaitForReload('.selector')
            ->assertSee('something');

#### 等待 JavaScript 表示式

有時候會希望暫停測試的執行，直到給定的 JavaScript 表示式執行結果為`true`。可以使用`waitUntil`方法輕鬆地達成此目的。 通過這個方法執行表示式，不需要包含`return`關鍵字或者結束分號：

    // 等待表示式為 true 5 秒時間...
    $browser->waitUntil('App.data.servers.length > 0');

    // 等待表示式為 true 1 秒時間...
    $browser->waitUntil('App.data.servers.length > 0', 1);

#### 等待 Vue 表示式

`waitUntilVue`和`waitUntilVueIsNot`方法可以一直等待，直到 [Vue 元件](https://vuejs.org) 的屬性包含給定的值：

    // 一直等待，直到元件屬性包含給定的值...
    $browser->waitUntilVue('user.name', 'Taylor', '@user');

    // 一直等待，直到元件屬性不包含給定的值...
    $browser->waitUntilVueIsNot('user.name', null, '@user');

#### 等待 JavaScript 事件

`waitForEvent`方法可用於暫停測試的執行，直到 JavaScript 事件發生:

    $browser->waitForEvent('load');

事件監聽器附加到當前範疇，默認情況下是`body`元素。當使用範圍選擇器時，事件監聽器將被附加到匹配的元素上:

    $browser->with('iframe', function (Browser $iframe) {
        // 等待 iframe 的載入事件…
        $iframe->waitForEvent('load');
    });

你也可以提供一個選擇器作為`waitForEvent`方法的第二個參數，將事件監聽器附加到特定的元素上:

    $browser->waitForEvent('load', '.selector');

你也可以等待`document`和`window`對象上的事件:

    // 等待文件被滾動…
    $browser->waitForEvent('scroll', 'document');

    // 等待 5 秒，直到窗口大小被調整…
    $browser->waitForEvent('resize', 'window', 5);

#### 等待回呼

Dusk 中的許多 「wait」 方法都依賴於底層方法 waitUsing。你可以直接用這個方法去等待一個回呼函數返回`waitUsing`。你可以直接用這個方法去等待一個回呼函數返回`true`。該`waitUsing`方法接收一個最大的等待秒數，閉包執行間隔時間，閉包，以及一個可選的失敗資訊：

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "有些東西沒有及時準備好。");

### 滾動元素到檢視中

有時你可能無法點選某個元素，因為該元素在瀏覽器的可見區域之外。該`scrollIntoView`方法可以將元素滾動到瀏覽器可視窗口內：

    $browser->scrollIntoView('.selector')
            ->click('.selector');

## 可用的斷言

Dusk 提供了各種你可以對應用使用的斷言。所有可用的斷言羅列如下：

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertTitle](#assert-title)
[assertTitleContains](#assert-title-contains)
[assertUrlIs](#assert-url-is)
[assertSchemeIs](#assert-scheme-is)
[assertSchemeIsNot](#assert-scheme-is-not)
[assertHostIs](#assert-host-is)
[assertHostIsNot](#assert-host-is-not)
[assertPortIs](#assert-port-is)
[assertPortIsNot](#assert-port-is-not)
[assertPathBeginsWith](#assert-path-begins-with)
[assertPathIs](#assert-path-is)
[assertPathIsNot](#assert-path-is-not)
[assertRouteIs](#assert-route-is)
[assertQueryStringHas](#assert-query-string-has)
[assertQueryStringMissing](#assert-query-string-missing)
[assertFragmentIs](#assert-fragment-is)
[assertFragmentBeginsWith](#assert-fragment-begins-with)
[assertFragmentIsNot](#assert-fragment-is-not)
[assertHasCookie](#assert-has-cookie)
[assertHasPlainCookie](#assert-has-plain-cookie)
[assertCookieMissing](#assert-cookie-missing)
[assertPlainCookieMissing](#assert-plain-cookie-missing)
[assertCookieValue](#assert-cookie-value)
[assertPlainCookieValue](#assert-plain-cookie-value)
[assertSee](#assert-see)
[assertDontSee](#assert-dont-see)
[assertSeeIn](#assert-see-in)
[assertDontSeeIn](#assert-dont-see-in)
[assertSeeAnythingIn](#assert-see-anything-in)
[assertSeeNothingIn](#assert-see-nothing-in)
[assertScript](#assert-script)
[assertSourceHas](#assert-source-has)
[assertSourceMissing](#assert-source-missing)
[assertSeeLink](#assert-see-link)
[assertDontSeeLink](#assert-dont-see-link)
[assertInputValue](#assert-input-value)
[assertInputValueIsNot](#assert-input-value-is-not)
[assertChecked](#assert-checked)
[assertNotChecked](#assert-not-checked)
[assertIndeterminate](#assert-indeterminate)
[assertRadioSelected](#assert-radio-selected)
[assertRadioNotSelected](#assert-radio-not-selected)
[assertSelected](#assert-selected)
[assertNotSelected](#assert-not-selected)
[assertSelectHasOptions](#assert-select-has-options)
[assertSelectMissingOptions](#assert-select-missing-options)
[assertSelectHasOption](#assert-select-has-option)
[assertSelectMissingOption](#assert-select-missing-option)
[assertValue](#assert-value)
[assertValueIsNot](#assert-value-is-not)
[assertAttribute](#assert-attribute)
[assertAttributeContains](#assert-attribute-contains)
[assertAriaAttribute](#assert-aria-attribute)
[assertDataAttribute](#assert-data-attribute)
[assertVisible](#assert-visible)
[assertPresent](#assert-present)
[assertNotPresent](#assert-not-present)
[assertMissing](#assert-missing)
[assertInputPresent](#assert-input-present)
[assertInputMissing](#assert-input-missing)
[assertDialogOpened](#assert-dialog-opened)
[assertEnabled](#assert-enabled)
[assertDisabled](#assert-disabled)
[assertButtonEnabled](#assert-button-enabled)
[assertButtonDisabled](#assert-button-disabled)
[assertFocused](#assert-focused)
[assertNotFocused](#assert-not-focused)
[assertAuthenticated](#assert-authenticated)
[assertGuest](#assert-guest)
[assertAuthenticatedAs](#assert-authenticated-as)
[assertVue](#assert-vue)
[assertVueIsNot](#assert-vue-is-not)
[assertVueContains](#assert-vue-contains)
[assertVueDoesNotContain](#assert-vue-does-not-contain)

</div>

#### assertTitle

斷言頁面標題為給定文字：

    $browser->assertTitle($title);

#### assertTitleContains

斷言頁面標題包含給定文字：

    $browser->assertTitleContains($title);

#### assertUrlIs

斷言當前的 URL（不包含 query string）是給定的字串：

    $browser->assertUrlIs($url);

#### assertSchemeIs

斷言當前的 URL scheme 是給定的 scheme：

    $browser->assertSchemeIs($scheme);

#### assertSchemeIsNot

斷言當前的 URL scheme 不是給定的 scheme：

    $browser->assertSchemeIsNot($scheme);

#### assertHostIs

斷言當前的 URL host 是給定的 host：

    $browser->assertHostIs($host);

#### assertHostIsNot

斷言當前的 URL host 不是給定的 host：

    $browser->assertHostIsNot($host);

#### assertPortIs

斷言當前的 URL 連接埠是給定的連接埠：

    $browser->assertPortIs($port);

#### assertPortIsNot

斷言當前的 URL 連接埠不是給定的連接埠：

    $browser->assertPortIsNot($port);

#### assertPathBeginsWith

斷言當前的 URL 路徑以給定的路徑開始：

    $browser->assertPathBeginsWith('/home');

#### assertPathIs

斷言當前的路徑是給定的路徑：

    $browser->assertPathIs('/home');

#### assertPathIsNot

斷言當前的路徑不是給定的路徑：

    $browser->assertPathIsNot('/home');

#### assertRouteIs

斷言給定的 URL 是給定的[命名路由](/docs/laravel/10.x/routing#named-routes)的 URL:

    $browser->assertRouteIs($name, $parameters);

#### assertQueryStringHas

斷言給定的查詢字串參數存在：

    $browser->assertQueryStringHas($name);

斷言給定的查詢字串參數存在並且具有給定的值：

    $browser->assertQueryStringHas($name, $value);

#### assertQueryStringMissing

斷言缺少給定的查詢字串參數：

    $browser->assertQueryStringMissing($name);

#### assertFragmentIs

斷言 URL 的當前雜湊片段與給定的片段匹配：

    $browser->assertFragmentIs('anchor');

#### assertFragmentBeginsWith

斷言 URL 的當前雜湊片段以給定片段開頭：

    $browser->assertFragmentBeginsWith('anchor');

#### assertFragmentIsNot

斷言 URL 的當前雜湊片段與給定的片段不匹配：

    $browser->assertFragmentIsNot('anchor');

#### assertHasCookie

斷言給定的加密 cookie 存在:

    $browser->assertHasCookie($name);

#### assertHasPlainCookie

斷言給定的未加密 cookie 存在：

    $browser->assertHasPlainCookie($name);

#### assertCookieMissing

斷言給定的加密 cookie 不存在：

    $browser->assertCookieMissing($name);

#### assertPlainCookieMissing

斷言給定的未加密 cookie 不存在：

    $browser->assertPlainCookieMissing($name);

#### assertCookieValue

斷言加密的 cookie 具有給定值：

    $browser->assertCookieValue($name, $value);

#### assertPlainCookieValue

斷言未加密的 cookie 具有給定值：

    $browser->assertPlainCookieValue($name, $value);

#### assertSee

斷言在頁面中有給定的文字：

    $browser->assertSee($text);

#### assertDontSee

斷言在頁面中沒有給定的文字：

    $browser->assertDontSee($text);

#### assertSeeIn

斷言在選擇器中有給定的文字：

    $browser->assertSeeIn($selector, $text);

#### assertDontSeeIn

斷言在選擇器中不存在給定的文字：

    $browser->assertDontSeeIn($selector, $text);

#### assertSeeAnythingIn

斷言在選擇器中存在任意的文字：

    $browser->assertSeeAnythingIn($selector);

斷言在選擇器中不存在文字：

    $browser->assertSeeNothingIn($selector);

#### assertScript

斷言給定的 JavaScript 表示式結果為給定的值：

    $browser->assertScript('window.isLoaded')
            ->assertScript('document.readyState', 'complete');

#### assertSourceHas

斷言在頁面中存在給定的原始碼：

    $browser->assertSourceHas($code);

#### assertSourceMissing

斷言頁面中沒有給定的原始碼：

    $browser->assertSourceMissing($code);

#### assertSeeLink

斷言在頁面中存在指定的連結：

    $browser->assertSeeLink($linkText);

#### assertDontSeeLink

斷言頁面中沒有指定的連結：

    $browser->assertDontSeeLink($linkText);

#### assertInputValue

斷言輸入框（input）有給定的值：

    $browser->assertInputValue($field, $value);

#### assertInputValueIsNot

斷言輸入框沒有給定的值：

    $browser->assertInputValueIsNot($field, $value);

#### assertChecked

斷言複選框（checkbox）被選中：

    $browser->assertChecked($field);

#### assertNotChecked

斷言複選框沒有被選中：

    $browser->assertNotChecked($field);

#### assertRadioSelected

斷言單選框（radio）被選中：

    $browser->assertRadioSelected($field, $value);

#### assertRadioNotSelected

斷言單選框（radio）沒有被選中：

    $browser->assertRadioNotSelected($field, $value);

<a name="assert-selected"></a>
#### assertSelected

斷言下拉框有給定的值:

    $browser->assertSelected($field, $value);


斷言下拉框沒有給定的值：

    $browser->assertNotSelected($field, $value);

<a name="assert-select-has-options"></a>
#### assertSelectHasOptions

斷言給定的陣列值是可選的：

    $browser->assertSelectHasOptions($field, $values);

<a name="assert-select-missing-options"></a>
#### assertSelectMissingOptions

斷言給定的陣列值是不可選的：

    $browser->assertSelectMissingOptions($field, $values);

<a name="assert-select-has-option"></a>
#### assertSelectHasOption

斷言給定的值在給定的地方是可供選擇的：

    $browser->assertSelectHasOption($field, $value);

<a name="assert-select-missing-option"></a>
#### assertSelectMissingOption

斷言給定的值不可選：

    $browser->assertSelectMissingOption($field, $value);

<a name="assert-value"></a>
#### assertValue

斷言選擇器範圍內的元素存在指定的值：

    $browser->assertValue($selector, $value);

<a name="assert-value-is-not"></a>
#### assertValueIsNot

斷言選擇器範圍內的元素不存在指定的值：

    $browser->assertValueIsNot($selector, $value);

<a name="assert-attribute"></a>
#### assertAttribute

斷言與給定選擇器匹配的元素在提供的屬性中具有給定的值：

    $browser->assertAttribute($selector, $attribute, $value);

<a name="assert-attribute-contains"></a>
#### assertAttributeContains

斷言匹配給定選擇器的元素在提供的屬性中包含給定值：

    $browser->assertAttributeContains($selector, $attribute, $value);

<a name="assert-aria-attribute"></a>
#### assertAriaAttribute

斷言與給定選擇器匹配的元素在給定的 aria 屬性中具有給定的值：

    $browser->assertAriaAttribute($selector, $attribute, $value);

例如，給定標記`<button aria-label="Add"></button>`，你可以像這樣聲明`aria-label`屬性：

    $browser->assertAriaAttribute('button', 'label', 'Add')

<a name="assert-data-attribute"></a>
#### assertDataAttribute

斷言與給定選擇器匹配的元素在提供的 data 屬性中具有給定的值：

    $browser->assertDataAttribute($selector, $attribute, $value);

例如，給定標記`<tr id="row-1" data-content="attendees"></tr>`，你可以像這樣斷言`data-label`屬性：

    $browser->assertDataAttribute('#row-1', 'content', 'attendees')

<a name="assert-visible"></a>
#### assertVisible

斷言匹配給定選擇器的元素可見:

    $browser->assertVisible($selector);

<a name="assert-present"></a>
#### assertPresent

斷言匹配給定選擇器的元素存在：

    $browser->assertPresent($selector);

<a name="assert-not-present"></a>
#### assertNotPresent

斷言源中不存在與給定選擇器匹配的元素：

    $browser->assertNotPresent($selector);

<a name="assert-missing"></a>
#### assertMissing

斷言匹配給定選擇器的元素不可見：

    $browser->assertMissing($selector);

<a name="assert-input-present"></a>
#### assertInputPresent

斷言具有給定名稱的輸入存在：

    $browser->assertInputPresent($name);

<a name="assert-input-missing"></a>
#### assertInputMissing

斷言源中不存在具有給定名稱的輸入：

    $browser->assertInputMissing($name);

<a name="assert-dialog-opened"></a>
#### assertDialogOpened

斷言已打開帶有給定消息的 JavaScript 對話方塊：

    $browser->assertDialogOpened($message);

<a name="assert-enabled"></a>
#### assertEnabled

斷言給定的欄位已啟用：

    $browser->assertEnabled($field);

<a name="assert-disabled"></a>
#### assertDisabled

斷言給定的欄位被停用：

    $browser->assertDisabled($field);

<a name="assert-button-enabled"></a>
#### assertButtonEnabled

斷言給定的按鈕已啟用：

    $browser->assertButtonEnabled($button);

<a name="assert-button-disabled"></a>
#### assertButtonDisabled

斷言給定的按鈕被停用：

    $browser->assertButtonDisabled($button);

<a name="assert-focused"></a>
#### assertFocused

斷言給定的欄位是焦點：

    $browser->assertFocused($field);

<a name="assert-not-focused"></a>
#### assertNotFocused

斷言給定欄位未聚焦：

    $browser->assertNotFocused($field);

<a name="assert-authenticated"></a>
#### assertAuthenticated

斷言使用者已通過身份驗證：

    $browser->assertAuthenticated();

<a name="assert-guest"></a>
#### assertGuest

斷言使用者未通過身份驗證：

    $browser->assertGuest();

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

斷言使用者已作為給定使用者進行身份驗證：

    $browser->assertAuthenticatedAs($user);

<a name="assert-vue"></a>
#### assertVue

Dusk 甚至允許你對 [Vue 元件](https://vuejs.org)資料的狀態進行斷言。例如，假設你的應用程式包含以下 Vue 元件：

    // HTML...

    <profile dusk="profile-component"></profile>

    // 元件定義...

    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',

        data: function () {
            return {
                user: {
                    name: 'Taylor'
                }
            };
        }
    });

你可以像這樣斷言 Vue 元件的狀態：

    /**
     * 一個基本的 Vue 測試示例
     *
     * @return void
     */
    public function testVue()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertVue('user.name', 'Taylor', '@profile-component');
        });
    }

<a name="assert-vue-is-not"></a>
#### assertVueIsNot

斷言 Vue 元件資料的屬性不匹配給定的值：

    $browser->assertVueIsNot($property, $value, $componentSelector = null);

<a name="assert-vue-contains"></a>
#### assertVueContains

斷言 Vue 元件資料的屬性是一個陣列，并包含給定的值：

    $browser->assertVueContains($property, $value, $componentSelector = null);

<a name="assert-vue-does-not-contain"></a>
#### assertVueDoesNotContain

斷言 Vue 元件資料的屬性是一個陣列，且不包含給定的值：

    $browser->assertVueDoesNotContain($property, $value, $componentSelector = null);

<a name="pages"></a>
## Pages

有時，測試需要按順序執行幾個複雜的操作。這會使測試程式碼更難閱讀和理解。 Dusk Pages 允許你定義語義化的操作，然後可以通過單一方法在給定頁面上執行這些操作。Pages 還可以為應用或單個頁面定義通用選擇器的快捷方式。

<a name="generating-pages"></a>
### 生成 Pages

`dusk:page`Artisan 命令可以生成頁面對象。所有的頁面對象都位於`tests/Browser/Pages`目錄：

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### 組態 Pages

默認情況下，頁面具有三種方法：`url`、`assert`和`elements`。我們現在將討論 `url`和`assert`方法。`elements`方法將[在下面更詳細地討論](#shorthand-selectors)。

<a name="the-url-method"></a>
#### `url` 方法

`url`方法應該返回表示頁面 URL 的路徑。 Dusk 將會在瀏覽器中使用這個 URL 來導航到具體頁面：

    /**
     * 獲取頁面的 URL。
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }

<a name="the-assert-method"></a>
#### `assert` 方法

`assert`方法可以作出任何斷言來驗證瀏覽器是否在指定頁面上。實際上沒有必要在這個方法中放置任何東西；但是，你可以按自己的需求來做出這些斷言。導航到頁面時，這些斷言將自動運行：

    /**
     * 斷言瀏覽器當前處於指定頁面。
     */
    public function assert(Browser $browser): void
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### 導航至頁面

一旦頁面定義好之後，你可以使用`visit`方法導航至頁面：

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

有時你可能已經在給定的頁面上，需要將頁面的選擇器和方法「載入」到當前的測試上下文中。 這在通過按鈕重新導向到指定頁面而沒有明確導航到該頁面時很常見。 在這種情況下，你可以使用`on`方法載入頁面：

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### 選擇器簡寫

該`elements`方法允許你為頁面中的任何 CSS 選擇器定義簡單易記的簡寫。例如，讓我們為應用登錄頁中的 email 輸入框定義一個簡寫：

    /**
     * 獲取頁面元素的簡寫。
     *
     * @return array<string, string>
     */
    public function elements(): array
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

一旦定義了簡寫，你就可以用這個簡寫來代替之前在頁面中使用的完整 CSS 選擇器：

    $browser->type('@email', 'taylor@laravel.com');

<a name="global-shorthand-selectors"></a>
#### 全域的選擇器簡寫

安裝 Dusk 之後，`Page`基類存放在你的`tests/Browser/Pages`目錄。該類中包含一個`siteElements`方法，這個方法可以用來定義全域的選擇器簡寫，這樣在你應用中每個頁面都可以使用這些全域選擇器簡寫了：

    /**
     * 獲取站點全域的選擇器簡寫。
     *
     * @return array<string, string>
     */
    public static function siteElements(): array
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>
### 頁面方法

除了頁面中已經定義的默認方法之外，你還可以定義在整個測試過程中會使用到的其他方法。例如，假設我們正在開發一個音樂管理應用，在應用中每個頁面都可能需要一個公共的方法來建立播放列表，而不是在每一個測試類中都重寫一遍建立播放列表的邏輯，這時候你可以在你的頁面類中定義一個`createPlaylist`方法：

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // 其他頁面方法...

        /**
         * 建立一個新的播放列表。
         */
        public function createPlaylist(Browser $browser, string $name): void
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

方法被定義之後，你可以在任何使用到該頁的測試中使用它了。瀏覽器實例會自動作為第一個參數傳遞給自訂頁面方法：

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="components"></a>
## 元件

元件類似於 Dusk 的 「頁面對象」，不過它更多的是貫穿整個應用程式中頻繁重用的 UI 和功能片斷，比如說導覽列或資訊通知彈窗。因此，元件並不會繫結於某個明確的 URL。

<a name="generating-components"></a>
### 生成元件

使用`dusk:component`Artisan 命令即可生成元件。新生成的元件位於`tests/Browser/Components`目錄下：

    php artisan dusk:component DatePicker

如上所示，這是生成一個「日期選擇器」（date picker）元件的示例，這個元件可能會貫穿使用在你應用程式的許多頁面中。在整個測試套件的大量測試頁面中，手動編寫日期選擇的瀏覽器自動化邏輯會非常麻煩。 更方便的替代辦法是，定義一個表示日期選擇器的 Dusk 元件，然後把自動化邏輯封裝在該元件內：

    <?php

    namespace Tests\Browser\Components;

    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Component as BaseComponent;

    class DatePicker extends BaseComponent
    {
        /**
         * 獲取元件的 root selector。
         */
        public function selector(): string
        {
            return '.date-picker';
        }

        /**
         * 斷言瀏覽器包含元件。
         */
        public function assert(Browser $browser): void
        {
            $browser->assertVisible($this->selector());
        }

        /**
         * 讀取元件的元素簡寫。
         *
         * @return array<string, string>
         */
        public function elements(): array
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@year-list' => 'div > div.datepicker-years',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }

        /**
         * 選擇給定日期。
         */
        public function selectDate(Browser $browser, int $year, int $month, int $day): void
        {
            $browser->click('@date-field')
                    ->within('@year-list', function (Browser $browser) use ($year) {
                        $browser->click($year);
                    })
                    ->within('@month-list', function (Browser $browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function (Browser $browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }

<a name="using-components"></a>
### 使用元件

當元件被定義了之後，我們就可以輕鬆的在任意測試頁面通過日期選擇器選擇一個日期。並且，如果選擇日期的邏輯發生了變化，我們只需要更新元件即可：

    <?php

    namespace Tests\Browser;

    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Browser;
    use Tests\Browser\Components\DatePicker;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        /**
         * 一個基礎的元件測試用例.
         */
        public function test_basic_example(): void
        {
            $this->browse(function (Browser $browser) {
                $browser->visit('/')
                        ->within(new DatePicker, function (Browser $browser) {
                            $browser->selectDate(2019, 1, 30);
                        })
                        ->assertSee('January');
            });
        }
    }

<a name="continuous-integration"></a>
## 持續整合

> **注意**
> 大多數 Dusk 持續整合組態都希望你的 Laravel 應用程式使用連接埠 8000 上的內建 PHP 開發伺服器提供服務。因此，你應該確保持續整合環境有一個值為 `http://127.0.0.1:8000` 的 `APP_URL` 環境變數。

<a name="running-tests-on-heroku-ci"></a>
### Heroku CI

要在 [Heroku CI](https://www.heroku.com/continuous-integration) 中運行 Dusk 測試，請將以下 Google Chrome buildpack 和 指令碼新增到 Heroku 的 `app.json` 檔案中：

    {
      "environments": {
        "test": {
          "buildpacks": [
            { "url": "heroku/php" },
            { "url": "https://github.com/heroku/heroku-buildpack-google-chrome" }
          ],
          "scripts": {
            "test-setup": "cp .env.testing .env",
            "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve --no-reload > /dev/null 2>&1 &' && php artisan dusk"
          }
        }
      }
    }

<a name="running-tests-on-travis-ci"></a>
### Travis CI

要在 [Travis CI](https://travis-ci.org) 運行 Dusk 測試，可以使用下面這個 `.travis.yml` 組態。由於 Travis CI 不是一個圖形化的環境，我們還需要一些額外的步驟以便啟動 Chrome 瀏覽器。此外，我們將會使用 `php artisan serve` 來啟動 PHP 自帶的 Web 伺服器：

```yaml
language: php

php:
  - 7.3

addons:
  chrome: stable

install:
  - cp .env.testing .env
  - travis_retry composer install --no-interaction --prefer-dist
  - php artisan key:generate
  - php artisan dusk:chrome-driver

before_script:
  - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
  - php artisan serve --no-reload &

script:
  - php artisan dusk
```

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

如果你正在使用 [Github Actions](https://github.com/features/actions) 來運行你的 Dusk 測試，你應該使用以下這份組態檔案為範本。像 TravisCI 一樣，我們使用 `php artisan serve` 命令來啟動 PHP 的內建 Web 服務：

```yaml
name: CI
on: [push]
jobs:

  dusk-php:
    runs-on: ubuntu-latest
    env:
      APP_URL: "http://127.0.0.1:8000"
      DB_USERNAME: root
      DB_PASSWORD: root
      MAIL_MAILER: log
    steps:
      - uses: actions/checkout@v3
      - name: Prepare The Environment
        run: cp .env.example .env
      - name: Create Database
        run: |
          sudo systemctl start mysql
          mysql --user="root" --password="root" -e "CREATE DATABASE \`my-database\` character set UTF8mb4 collate utf8mb4_bin;"
      - name: Install Composer Dependencies
        run: composer install --no-progress --prefer-dist --optimize-autoloader
      - name: Generate Application Key
        run: php artisan key:generate
      - name: Upgrade Chrome Driver
        run: php artisan dusk:chrome-driver --detect
      - name: Start Chrome Driver
        run: ./vendor/laravel/dusk/bin/chromedriver-linux &
      - name: Run Laravel Server
        run: php artisan serve --no-reload &
      - name: Run Dusk Tests
        run: php artisan dusk
      - name: Upload Screenshots
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: screenshots
          path: tests/Browser/screenshots
      - name: Upload Console Logs
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: console
          path: tests/Browser/console
```
