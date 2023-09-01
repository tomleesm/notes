# Scout 全文搜尋

## 介紹

[Laravel Scout](https://github.com/laravel/scout) 為 [Eloquent models](/docs/laravel/10.x/eloquent) 的全文搜尋提供了一個簡單的基於驅動程式的解決方案，通過使用模型觀察者，Scout 將自動同步 Eloquent 記錄的搜尋索引。

目前，Scout 附帶 [Algolia](https://www.algolia.com/), [Meilisearch](https://www.meilisearch.com), 和 MySQL / PostgreSQL (`database`) 驅動程式。此外，Scout 包括一個「collection」驅動程式，該驅動程式專為本地開發使用而設計，不需要任何外部依賴項或第三方服務。此外，編寫自訂驅動程式很簡單，你可以使用自己的搜尋實現自由擴展 Scout。

## 安裝

首先，通過 Composer 軟體包管理器安裝 Scout：

```shell
composer require laravel/scout
```

Scout 安裝完成後，使用 Artisan 命令 `vendor:publish` 生成 Scout 組態檔案。此命令將會在你的 `config` 目錄下 生成一個 `scout.php` 組態檔案:

```shell
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

最後，在你要做搜尋的模型中新增 `Laravel\Scout\Searchable` trait 。這個 trait 會註冊一個模型觀察者來保持模型和搜尋驅動的同步:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;
    }

### 驅動的先決條件

#### Algolia

使用 Algolia 驅動時，需要在 `config/scout.php` 組態檔案組態你的 `Algolia` `id` 和 `secret` 憑證。組態好憑證之後，還需要使用 Composer 安裝 Algolia PHP SDK：

```shell
composer require algolia/algoliasearch-client-php
```

#### Meilisearch

[Meilisearch](https://www.meilisearch.com) 是一個速度極快的開源搜尋引擎。如果你不確定如何在本地機器上安裝 MeiliSearch，你可以使用 Laravel 官方支援的 Docker 開發環境 [Laravel Sail](/docs/laravel/10.x/sail#meilisearch)。

使用 MeiliSearch 驅動程式時，你需要通過 Composer 包管理器安裝 MeiliSearch PHP SDK：

```shell
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

然後，在應用程式的 `.env` 檔案中設定 `SCOUT_DRIVER` 環境變數以及你的 MeiliSearch `host` 和 `key` 憑據：

```ini
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

更多關於 MeiliSearch 的資訊，請參考 [MeiliSearch 技術文件](https://docs.meilisearch.com/learn/getting_started/quick_start.html)。

此外，你應該通過查看 [MeiliSearch 關於二進制相容性的文件](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch)確保安裝與你的 MeiliSearch 二進製版本相容的 `meilisearch/meilisearch-php` 版本。

>  Meilisearch service itself.
注意：在使用 MeiliSearch 的應用程式上升級 Scout 時，你應該始終留意查看關於 MeiliSearch 升級發佈的[其他重大（破壞性）更改](https://github.com/meilisearch/MeiliSearch/releases)，以保證升級順利。

### 佇列

雖然不強制要求使用 Scout，但在使用該庫之前，強烈建議組態一個[佇列驅動](/docs/laravel/10.x/queues)。運行佇列 worker 將允許 Scout 將所有同步模型資訊到搜尋索引的操作都放入佇列中，從而為你的應用程式的Web介面提供更快的響應時間。

一旦你組態了佇列驅動程式，請將 `config/scout.php` 組態檔案中的 `queue` 選項的值設定為 `true`：

    'queue' => true,

即使將 `queue` 選項設定為 `false`，也要記住有些 Scout 驅動程式（如 Algolia 和 Meilisearch）始終非同步記錄索引。也就是說，即使索引操作已在 Laravel 應用程式中完成，但搜尋引擎本身可能不會立即反映新記錄和更新記錄。

要指定 Scout 使用的連接和佇列，請將 `queue` 組態選項定義為陣列：

    'queue' => [
        'connection' => 'redis',
        'queue' => 'scout'
    ],

## 組態

### 組態模型索引

每個 Eloquent 模型都與一個給定的搜尋「索引」同步，該索引包含該模型的所有可搜尋記錄。換句話說，可以將每個索引視為 MySQL 表。默認情況下，每個模型將持久化到與模型的典型「表」名稱匹配的索引中。通常，這是模型名稱的複數形式；但是，你可以通過在模型上重寫 `searchableAs` 方法來自訂模型的索引：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * 獲取與模型關聯的索引的名稱.
         */
        public function searchableAs(): string
        {
            return 'posts_index';
        }
    }


### 組態可搜尋資料

默認情況下，給定模型的 `toArray` 形式的整個內容將被持久化到其搜尋索引中。如果要自訂同步到搜尋索引的資料，可以重寫模型上的 `toSearchableArray` 方法：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * 獲取模型的可索引資料。
         *
         * @return array<string, mixed>
         */
        public function toSearchableArray(): array
        {
            $array = $this->toArray();

            // 自訂資料陣列...

            return $array;
        }
    }

一些搜尋引擎（如 Meilisearch）只會在正確的資料類型上執行過濾操作（`>` 、 `<` 等）。因此，在使用這些搜尋引擎並自訂可搜尋資料時，你應該確保數值類型被轉換為正確的類型：

    public function toSearchableArray()
    {
        return [
            'id' => (int) $this->id,
            'name' => $this->name,
            'price' => (float) $this->price,
        ];
    }

#### 組態可過濾資料和索引設定 (Meilisearch)

與 Scout 的其他驅動程式不同，Meilisearch 要求你預定義索引搜尋設定，例如可過濾屬性、可排序屬性和[其他支援的設定欄位](https://docs.meilisearch.com/reference/api/settings.html)。

可過濾屬性是你在呼叫 Scout 的 `where` 方法時想要過濾的任何屬性，而可排序屬性是你在呼叫 Scout 的 `orderBy` 方法時想要排序的任何屬性。要定義索引設定，請調整應用程式的 `scout` 組態檔案中 `meilisearch` 組態條目的 `index-settings` 部分：

```php
use App\Models\User;
use App\Models\Flight;

'meilisearch' => [
    'host' => env('MEILISEARCH_HOST', 'http://localhost:7700'),
    'key' => env('MEILISEARCH_KEY', null),
    'index-settings' => [
        User::class => [
            'filterableAttributes'=> ['id', 'name', 'email'],
            'sortableAttributes' => ['created_at'],
            // 其他設定欄位...
        ],
        Flight::class => [
            'filterableAttributes'=> ['id', 'destination'],
            'sortableAttributes' => ['updated_at'],
        ],
    ],
],
```



如果給定索引下的模型可以進行軟刪除，並且已包含在`index-settings`陣列中，Scout 將自動支援在該索引上過濾軟刪除的模型。如果你沒有其他可過濾或可排序的屬性來定義軟刪除的模型索引，則可以簡單地向該模型的`index-settings`陣列新增一個空條目：

```php
'index-settings' => [
    Flight::class => []
],
```

在組態應用程式的索引設定之後，你必須呼叫 `scout:sync-index-settings` Artisan 命令。此命令將向 Meilisearch 通知你當前組態的索引設定。為了方便起見，你可能希望將此命令作為部署過程的一部分：

```shell
php artisan scout:sync-index-settings
```

### 組態模型ID

默認情況下，Scout 將使用模型的主鍵作為儲存在搜尋索引中的模型唯一ID/鍵。如果你需要自訂此行為，可以重寫模型的 `getScoutKey` 和 `getScoutKeyName` 方法：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * 獲取這個模型用於索引的值.
         */
        public function getScoutKey(): mixed
        {
            return $this->email;
        }

        /**
         * 獲取這個模型用於索引的鍵.
         */
        public function getScoutKeyName(): mixed
        {
            return 'email';
        }
    }

### 設定模型的搜尋引擎

當進行搜尋時，Scout 通常會使用應用程式的 `scout` 組態檔案中指定的默認搜尋引擎。但是，可以通過在模型上覆蓋 `searchableUsing` 方法來更改特定模型的搜尋引擎：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Engines\Engine;
    use Laravel\Scout\EngineManager;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * 獲取這個模型用於索引的搜尋引擎.
         */
        public function searchableUsing(): Engine
        {
            return app(EngineManager::class)->engine('meilisearch');
        }
    }

### 組態模型ID

默認情況下，Scout 將使用模型的主鍵作為儲存在搜尋索引中的模型的唯一ID /鍵。如果你需要自訂此行為，你可以覆蓋模型上的`getScoutKey`和`getScoutKeyName`方法:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * 獲取用於索引模型的值.
         */
        public function getScoutKey(): mixed
        {
            return $this->email;
        }

        /**
         * 獲取用於索引模型的鍵名.
         */
        public function getScoutKeyName(): mixed
        {
            return 'email';
        }
    }

### 按型號組態搜尋引擎

搜尋時，Scout 通常使用你應用程式的 Scout 組態檔案中指定的默認搜尋引擎。然而，可以通過覆蓋模型上的`searchableUsing`方法來更改特定模型的搜尋引擎:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Engines\Engine;
    use Laravel\Scout\EngineManager;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * 獲取用於索引模型的引擎.
         */
        public function searchableUsing(): Engine
        {
            return app(EngineManager::class)->engine('meilisearch');
        }
    }



### 識別使用者

如果你在使用 [Algolia](https://algolia.com/) 時想要自動識別使用者，Scout 可以幫助你。將已認證的使用者與搜尋操作相關聯，可以在 Algolia 的儀表板中查看搜尋分析時非常有用。你可以通過在應用程式的 `.env` 檔案中將 `SCOUT_IDENTIFY` 環境變數定義為 `true` 來啟用使用者識別：

```ini
SCOUT_IDENTIFY=true
```

啟用此功能還會將請求的 IP 地址和已驗證的使用者的主要識別碼傳遞給 Algolia，以便將此資料與使用者發出的任何搜尋請求相關聯。

## 資料庫/集合引擎

### 資料庫引擎

> 注意：目前，資料庫引擎支援 MySQL 和 PostgreSQL。

如果你的應用程式與小到中等大小的資料庫互動或工作負載較輕，你可能會發現使用 Scout 的 「database」 引擎更為方便。資料庫引擎將使用 「where like」子句和全文索引來過濾你現有資料庫的結果，以確定適用於你查詢的搜尋結果。

要使用資料庫引擎，你可以簡單地將 `SCOUT_DRIVER` 環境變數的值設定為 `database`，或直接在你的應用程式的 `scout` 組態檔案中指定 `database` 驅動程式：

```ini
SCOUT_DRIVER=database
```

一旦你已將資料庫引擎指定為首選驅動程式，你必須[組態你的可搜尋資料](#configuring-searchable-data)。然後，你可以開始[執行搜尋查詢](#searching)來查詢你的模型。使用資料庫引擎時，不需要進行搜尋引擎索引，例如用於填充 Algolia 或 Meilisearch 索引所需的索引。

#### 自訂資料庫搜尋策略

默認情況下，資料庫引擎將對你所[組態為可搜尋的](#configuring-searchable-data)每個模型屬性執行 「where like」 查詢。然而，在某些情況下，這可能會導致性能不佳。因此，資料庫引擎的搜尋策略可以組態，以便某些指定的列利用全文搜尋查詢，或者僅使用 「where like」 約束來搜尋字串的前綴(`example%`)，而不是在整個字串中搜尋(`%example%`)。

為了定義這種行為，你可以將 PHP 屬性賦值給你的模型的 toSearchableArray 方法。任何未被分配其他搜尋策略行為的列將繼續使用默認的「where like」策略：

```php
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;

/**
 * 獲取模型的可索引資料陣列。
 *
 * @return array<string, mixed>
 */
#[SearchUsingPrefix(['id', 'email'])]
#[SearchUsingFullText(['bio'])]
public function toSearchableArray(): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'bio' => $this->bio,
    ];
}
```

> 注意：在指定列應使用全文查詢約束之前，請確保已為該列分配[全文索引](/docs/laravel/10.x/migrations#available-index-types)。


### 集合引擎

在本地開發過程中，你可以自由地使用 Algolia 或 Meilisearch 搜尋引擎，但你可能會發現使用「集合」引擎更加方便。集合引擎將使用「where」子句和集合過濾器來從你現有的資料庫結果中確定適用於你查詢的搜尋結果。當使用此引擎時，無需對可搜尋模型進行「索引」，因為它們只需從本地資料庫中檢索即可。

要使用收集引擎，你可以簡單地將 `SCOUT_DRIVER` 環境變數的值設定為 `collection`，或者直接在你的應用的 `scout` 組態檔案中指定 `collection` 驅動程式：

```ini
SCOUT_DRIVER=collection
```

一旦你將收集驅動程式指定為首選驅動程式，你就可以開始針對你的模型[執行搜尋查詢](#searching)。使用收集引擎時，不需要進行搜尋引擎索引，如種子 Algolia 或 Meilisearch 索引所需的索引。

#### 與資料庫引擎的區別

乍一看，「資料庫」和「收集」引擎非常相似。它們都直接與你的資料庫互動以檢索搜尋結果。然而，收集引擎不使用全文索引或 `LIKE` 子句來尋找匹配的記錄。相反，它會拉取所有可能的記錄，並使用 Laravel 的 `Str::is` 助手來確定搜尋字串是否存在於模型屬性值中。

收集引擎是最便攜的搜尋引擎，因為它適用於 Laravel 支援的所有關係型資料庫（包括 SQLite 和 SQL Server）；然而，它的效率比 Scout 的資料庫引擎低。

## 索引

### 批次匯入

如果你要將 Scout 安裝到現有項目中，你可能已經有需要匯入到你的索引中的資料庫記錄。Scout 提供了一個 `scout:import` Artisan 命令，你可以使用它將所有現有記錄匯入到你的搜尋索引中：

```shell
php artisan scout:import "App\Models\Post"
```

`flush` 命令可用於從你的搜尋索引中刪除模型的所有記錄：

```shell
php artisan scout:flush "App\Models\Post"
```

#### 修改匯入查詢

如果你想修改用於獲取所有批次匯入模型的查詢，你可以在你的模型上定義一個`makeAllSearchableUsing`方法。這是一個很好的地方，可以在匯入模型之前新增任何必要的關係載入:

    use Illuminate\Database\Eloquent\Builder;

    /**
     * 修改用於檢索模型的查詢，使所有模型都可搜尋.
     */
    protected function makeAllSearchableUsing(Builder $query): Builder
    {
        return $query->with('author');
    }

### 新增記錄

一旦你將`Laravel\Scout\Searchable` Trait新增到模型中，你所需要做的就是`保存`或`建立`一個模型實例，它將自動新增到你的搜尋索引中。如果你將Scout組態為[使用佇列](#queueing)，則此操作將由你的佇列工作者在後台執行:

    use App\Models\Order;

    $order = new Order;

    // ...

    $order->save();

#### 通過查詢新增記錄

如果你想通過Eloquent查詢將模型集合新增到你的搜尋索引中，你可以將`searchable`方法連結到Eloquent查詢中。`searchable`方法會將查詢的[結果分塊](/docs/laravel/10.x/eloquent#chunking-results)並將記錄新增到你的搜尋索引中。同樣，如果你已經組態了Scout來使用佇列，那麼所有的塊都將由你的佇列工作程序在後台匯入:

    use App\Models\Order;

    Order::where('price', '>', 100)->searchable();

你也可以在Eloquent關係實例上呼叫`searchable`方法:

    $user->orders()->searchable();

如果你已經有一組Eloquent模型對像在記憶體中，可以在該集合實例上呼叫`searchable`方法，將模型實例新增到對應的索引中：

    $orders->searchable();

> **注意**
searchable 方法可以被視為「upsert」操作。換句話說，如果模型記錄已經存在於索引中，它將被更新。如果它在搜尋索引中不存在，則將其新增到索引中。

### 更新記錄

要更新可搜尋的模型，只需更新模型實例的屬性並將模型保存到你的資料庫中。Scout 將自動將更改持久化到你的搜尋索引中：

    use App\Models\Order;

    $order = Order::find(1);

    // 更新訂單…

    $order->save();

你還可以在Eloquent查詢實例上呼叫 `searchable` 方法，以更新模型的集合。如果模型不存在於搜尋索引中，則將建立它們：

    Order::where('price', '>', 100)->searchable();

如果想要更新關係中所有模型的搜尋索引記錄，可以在關係實例上呼叫`searchable`方法：

    $user->orders()->searchable();

或者，如果你已經在記憶體中有一組 Eloquent 模型，可以在該集合實例上呼叫`searchable` 方法，以更新對應索引中的模型實例：

    $orders->searchable();

### 刪除記錄

要從索引中刪除記錄，只需從資料庫中刪除模型即可。即使你正在使用[軟刪除](/docs/laravel/10.x/eloquent#soft-deleting)模型，也可以這樣做：

    use App\Models\Order;

    $order = Order::find(1);

    $order->delete();



如果你不想在刪除記錄之前檢索模型，你可以在 Eloquent 查詢實例上使用 `unsearchable` 方法：

    Order::where('price', '>', 100)->unsearchable();

如果你想刪除與關係中所有模型相關的搜尋索引記錄，你可以在關係實例上呼叫 `unsearchable` 方法：

    $user->orders()->unsearchable();

或者，如果你已經有一組 Eloquent 模型在記憶體中，你可以在集合實例上呼叫 `unsearchable` 方法，將模型實例從相應的索引中移除：

    $orders->unsearchable();

### 暫停索引

有時你可能需要在不將模型資料同步到搜尋索引的情況下對模型執行一批 Eloquent 操作。你可以使用 `withoutSyncingToSearch` 方法來實現這一點。該方法接受一個閉包，將立即執行。在閉包內發生的任何模型操作都不會同步到模型的索引：

    use App\Models\Order;

    Order::withoutSyncingToSearch(function () {
        // 執行模型動作…
    });

### 有條件地搜尋模型實例

有時你可能需要在某些條件下使模型可搜尋。例如，假設你有一個 `App\Models\Post` 模型，它可能處於兩種狀態之一：「draft」（草稿）和 「published」（已發佈）。你可能只想讓 「published」（已發佈）的帖子可以被搜尋。為了實現這一點，你可以在你的模型中定義一個 `shouldBeSearchable` 方法：

    /**
     * 確定模型是否應該可搜尋。
     */
    public function shouldBeSearchable(): bool
    {
        return $this->isPublished();
    }

`shouldBeSearchable` 方法僅在通過 `save` 和 `create` 方法、查詢或關係操作模型時應用。直接使用 `searchable` 方法使模型或集合可搜尋將覆蓋 `shouldBeSearchable` 方法的結果。

> **警告**  
> 當使用 Scout 的「database」（資料庫）引擎時，`shouldBeSearchable` 方法不適用，因為所有可搜尋的資料都儲存在資料庫中。要在使用資料庫引擎時實現類似的行為，你應該使用 [where 子句](#where-clauses)代替。



## 搜尋

你可以使用 `search` 方法開始搜尋一個模型。搜尋方法接受一個將用於搜尋模型的字串。然後，你應該在搜尋查詢上連結 `get` 方法以檢索與給定搜尋查詢匹配的 Eloquent 模型：

    use App\Models\Order;

    $orders = Order::search('Star Trek')->get();

由於 Scout 搜尋返回一組 Eloquent 模型，你甚至可以直接從路由或 controller 返回結果，它們將自動轉換為 JSON：

    use App\Models\Order;
    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return Order::search($request->search)->get();
    });

如果你想在將搜尋結果轉換為 Eloquent 模型之前獲取原始搜尋結果，你可以使用 `raw` 方法：

    $orders = Order::search('Star Trek')->raw();

#### 自訂索引

搜尋查詢通常會在模型的 [`searchableAs`](#configuring-model-indexes) 方法指定的索引上執行。然而，你可以使用 `within` 方法指定一個應該被搜尋的自訂索引：

    $orders = Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

### Where 子句

Scout 允許你在搜尋查詢中新增簡單的「where」子句。目前，這些子句只支援基本的數值相等檢查，主要用於通過所有者 ID 限定搜尋查詢：

    use App\Models\Order;

    $orders = Order::search('Star Trek')->where('user_id', 1)->get();

你可以使用 `whereIn` 方法將結果約束在給定的一組值中：

    $orders = Order::search('Star Trek')->whereIn(
        'status', ['paid', 'open']
    )->get();



由於搜尋索引不是關聯式資料庫，所以目前不支援更高級的「where」子句。

> **警告 **
> 如果你的應用程式使用了 Meilisearch，你必須在使用 Scout 的「where」子句之前組態你的應用程式的[可過濾屬性](#configuring-filterable-data-for-meilisearch)。

### 分頁

除了檢索模型集合之外，你還可以使用 `paginate` 方法對搜尋結果進行分頁。此方法將返回一個 `Illuminate\Pagination\LengthAwarePaginator` 實例，就像你對[傳統的 Eloquent 查詢進行分頁](/docs/laravel/10.x/pagination)一樣：

    use App\Models\Order;

    $orders = Order::search('Star Trek')->paginate();

你可以通過將數量作為第一個參數傳遞給 `paginate` 方法來指定每頁檢索多少個模型：

    $orders = Order::search('Star Trek')->paginate(15);

一旦你檢索到了結果，你可以使用 [Blade](/docs/laravel/10.x/blade) 顯示結果並渲染頁面連結，就像你對傳統的 Eloquent 查詢進行分頁一樣：

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

當然，如果你想將分頁結果作為 JSON 檢索，可以直接從路由或 controller 返回分頁器實例：

    use App\Models\Order;
    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        return Order::search($request->input('query'))->paginate(15);
    });

> **警告**  
> 由於搜尋引擎不瞭解你的 Eloquent 模型的全域範疇定義，因此在使用 Scout 分頁的應用程式中，你不應該使用全域範疇。或者，你應該在通過 Scout 搜尋時重新建立全域範疇的約束。


### 軟刪除

如果你的索引模型使用了軟刪除並且你需要搜尋已軟刪除的模型，請將 `config/scout.php` 組態檔案中的 `soft_delete` 選項設定為 `true`。

    'soft_delete' => true,

當這個組態選項為 `true` 時，Scout 不會從搜尋索引中刪除已軟刪除的模型。相反，它會在索引記錄上設定一個隱藏的 `__soft_deleted` 屬性。然後，你可以使用 `withTrashed` 或 `onlyTrashed` 方法在搜尋時檢索已軟刪除的記錄：

    use App\Models\Order;

    // 在檢索結果時包含已刪除的記錄。。。
    $orders = Order::search('Star Trek')->withTrashed()->get();

    // 僅在檢索結果時包含已刪除的記錄。。。
    $orders = Order::search('Star Trek')->onlyTrashed()->get();

> 技巧：當使用 `forceDelete` 永久刪除軟刪除模型時，Scout 將自動從搜尋索引中移除它。

### 自訂引擎搜尋

如果你需要對一個引擎的搜尋行為進行高級定製，你可以將一個閉包作為 `search` 方法的第二個參數傳遞進去。例如，你可以使用這個回呼在搜尋查詢傳遞給 Algolia 之前將地理位置資料新增到你的搜尋選項中：

    use Algolia\AlgoliaSearch\SearchIndex;
    use App\Models\Order;

    Order::search(
        'Star Trek',
        function (SearchIndex $algolia, string $query, array $options) {
            $options['body']['query']['bool']['filter']['geo_distance'] = [
                'distance' => '1000km',
                'location' => ['lat' => 36, 'lon' => 111],
            ];

            return $algolia->search($query, $options);
        }
    )->get();

#### 自訂 Eloquent 結果查詢

在 Scout 從你的應用程式搜尋引擎中檢索到匹配的 Eloquent 模型列表後，Eloquent 會使用模型的主鍵檢索所有匹配的模型。你可以通過呼叫 `query` 方法來自訂此查詢。`query` 方法接受一個閉包，它將接收 Eloquent 查詢建構器實例作為參數：

```php
use App\Models\Order;
use Illuminate\Database\Eloquent\Builder;

$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```


由於此回呼是在從應用程式的搜尋引擎中已經檢索到相關模型之後呼叫的，因此 `query` 方法不應用於「過濾」結果。相反，你應該使用 [Scout where 子句](#where-clauses)。

## 自訂引擎

#### 編寫引擎

如果 Scout 內建的搜尋引擎不符合你的需求，你可以編寫自己的自訂引擎並將其註冊到 Scout。你的引擎應該繼承 `Laravel\Scout\Engines\Engine` 抽象類。這個抽象類包含了你的自訂引擎必須實現的八個方法：

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map(Builder $builder, $results, $model);
    abstract public function getTotalCount($results);
    abstract public function flush($model);

你可能會發現，查看 `Laravel\Scout\Engines\AlgoliaEngine` 類上這些方法的實現會很有幫助。這個類將為你提供一個良好的起點，以學習如何在自己的引擎中實現每個方法。


#### 註冊引擎

一旦你編寫好自己的引擎，就可以使用 Scout 的引擎管理器的 `extend` 方法將其註冊到 Scout 中。Scout 的引擎管理器可以從Laravel服務容器中解析。你應該從 `App\Providers\AppServiceProvider` 類或應用程式使用的任何其他服務提供程序的 `boot` 方法中呼叫 `extend` 方法：

    use App\ScoutExtensions\MySqlSearchEngine;
    use Laravel\Scout\EngineManager;

    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }



引擎註冊後，你可以在 `config/scout.php` , 組態檔案中指定它為默認的 Scout `driver`

    'driver' => 'mysql',

## 生成宏命令

如果你想要自訂生成器方法，你可以使用 `Laravel\Scout\Builder` 類下的 "macro" 方法。 通常，定義「macros」時，需要實現 [service provider’s](/docs/laravel/10.x/providers) `boot` 方法:

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;
    use Laravel\Scout\Builder;

    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        Builder::macro('count', function () {
            return $this->engine()->getTotalCount(
                $this->engine()->search($this)
            );
        });
    }

`macro` 函數接受一個名字作為第一個參數，第二個參數為一個閉包涵數。當呼叫 `Laravel\Scout\Builder` 宏命令時，呼叫這個函數.

    use App\Models\Order;

    Order::search('Star Trek')->count();

