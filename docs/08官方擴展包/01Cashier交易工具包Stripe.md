# Cashier 交易工具包 (Stripe)

## 簡介

[Laravel Cashier Stripe](https://github.com/laravel/cashier-stripe) 為 [Stripe](https://stripe.com) 的訂閱計費服務提供了一個富有表現力、流暢的介面。它處理了幾乎所有你害怕編寫的訂閱計費樣板程式碼。除了基本的訂閱管理，Cashier 還可以處理優惠券、交換訂閱、訂閱 「數量」、取消寬限期，甚至生成發票 PDF。

## 升級 Cashier

升級到新版本的 Cashier 時，請務必仔細閱讀[升級指南](https://github.com/laravel/cashier-stripe/blob/master/UPGRADE.)。

> **注意**
> 為了防止破壞性變更，Cashier 使用固定的 Stripe API 版本。 Cashier 14 使用 Stripe API 版本 `2022-11-15` 。Stripe API 版本將在次要版本上更新，以利用新的 Stripe 功能和改進。

## 安裝

首先，使用 Composer 為 Stripe 安裝 Cashier 擴展包：

```shell
composer require laravel/cashier
```
> **注意**
> 為確保 Cashier 正確處理所有 Stripe 事件，請記得[設定 Cashier 的 webhook](#handling-stripe-webhooks)。

### 資料庫遷移

Cashier 的服務提供器註冊了自己的資料庫遷移目錄，因此請記住在安裝此包後遷移資料庫。Cashier 遷移將向 `users` 表中新增多個列，並建立一個新的 `subscriptions` 表來保存客戶的所有訂閱：

```shell
php artisan migrate
```

如果需要覆蓋 Cashier 附帶的遷移，可以使用 `vendor:publish` Artisan 命令發佈它們：

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

如果你想阻止 Cashier 的遷移完全運行，可以使用 Cashier 提供的`ignoreMigrations` 方法。通常應在 `AppServiceProvider` 類的 `register` 方法中呼叫此方法：

    use Laravel\Cashier\Cashier;

    /**
     * 註冊任何應用程式服務。
     *
     */
    public function register(): void
    {
        Cashier::ignoreMigrations();
    }

> **注意**
>  Stripe 建議用於儲存 Stripe 識別碼的任何列都應區分大小寫。因此，在使用 MySQL 時，應該確保將 `stripe_id` 列排序規則設定為 `utf8_bin` 。更多關於這方面的資訊可以在 [Stripe 文件](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible)中找到。

## 組態

### 訂單模型

在使用 Cashier 之前，需要將 `Billable` trait 新增到可訂單模型定義中。通常會放在 `App\Models\User` 模型中。這個特性提供了多個方法以便執行常用支付任務，如建立訂閱、應用優惠券和更新支付方法資訊：

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Cashier 默認假設你的 Billable 模型是 Laravel 自帶的 `App\Models\User` 類。如果需要修改可以在 `useCustomerModel` 方法定義一個不同的模型。通常此方法在 `AppServiceProvider` 類的`boot`方法中被呼叫：

    use App\Models\Cashier\User;
    use Laravel\Cashier\Cashier;

    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        Cashier::useCustomerModel(User::class);
    }

> **注意**
> 如果你使用的不是 Laravel 自帶的 `App\Models\User` 模型，需要發佈並修改默認的 [Cashier 遷移](#installation)檔案以匹配你使用模型對應的表名。

### API 秘鑰

接下來需要在 `.env` 檔案中組態 Stripe 秘鑰，可以在 Stripe 後台控制面板中獲取Stripe API 秘鑰：

```ini
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
```
> **注意**
> 你應該確保在應用程式的`.env`檔案中定義了`STRIPE_WEBHOOK_SECRET`環境變數，因為該變數用於確保傳入的 Webhook 確實來自 Stripe。

### 貨幣組態

Cashier 默認貨幣是美元 (USD)，可以在 `.env` 中設定 `CASHIER_CURRENCY` 環境變數來修改默認的貨幣組態：

```ini
CASHIER_CURRENCY=eur
```

除了組態 Cashier 的貨幣之外，還可以在格式化用於顯示在發票上的金額時指定本地化組態。在底層，Cashier 使用了 [PHP 的 `NumberFormatter` 類](https://www.php.net/manual/en/class.numberformatter.php)來設定本地貨幣：

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> **注意**
> 為了使用本地化組態而不是 `en`，需要確保安裝了 PHP `ext-intl` PHP 擴展並在伺服器上啟用組態。

### 稅務組態

感謝[Stripe  稅務](https://stripe.com/tax)，可以自動計算 Stripe 生成的所有發票的稅費。 可以通過應用程式的 `App\Providers\AppServiceProvider`類的 `boot` 方法中呼叫 `calculateTaxes` 來啟用自動稅務計算：

    use Laravel\Cashier\Cashier;

    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        Cashier::calculateTaxes();
    }

啟動稅務計算後，任何新訂閱和生成的一次性發票都會進行自動稅務計算。

為了使這個功能正常使用，客戶的賬單明細中例如客戶姓名、住址、發票 ID 需要同步到 Stripe。你可以使用 Cashier 提供的[客戶資料同步](#syncing-customer-data-with-stripe)和 [Tax ID](#tax-ids) 方法來完成此操作。

> **注意**
> 對於[單次收費](#single-charges)或[單次支付結賬](#single-charge-checkouts)，不支援計算稅費。

### 日誌

Cashier 允許你指定日誌通道來記錄所有與 Stripe 相關的異常。可以通過在  `.env` 中組態 `CASHIER_LOGGER` 來指定：

```ini
CASHIER_LOGGER=stack
```

對 Stripe 的 API 呼叫生成的異常將通過應用程式的默認日誌通道記錄。

### 使用自訂模型

你可以通過定義自己的模型並擴展相應的 `Cashier` 模型來自由擴展 Cashier 內部的模型，增加一些方法：

    use Laravel\Cashier\Subscription as CashierSubscription;

    class Subscription extends CashierSubscription
    {
        // ...
    }

定義模型後，可以通過 `Laravel\Cashier\Cashier` 類組態 Cashier 使用自訂的模型。通常還需要在 `App\Providers\AppServiceProvider` 類的 `boot` 中註冊一下：

    use App\Models\Cashier\Subscription;
    use App\Models\Cashier\SubscriptionItem;

    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        Cashier::useSubscriptionModel(Subscription::class);
        Cashier::useSubscriptionItemModel(SubscriptionItem::class);
    }

## 消費者

### 獲取消費者

你可以使用 `Cashier::findBillable` 方法通過 Stripe ID 查詢消費者資訊。該方法返回的是一個 billable 模型實例：

    use Laravel\Cashier\Cashier;

    $user = Cashier::findBillable($stripeId);

### 建立客戶

有時，你可能希望在不開始訂閱的情況下建立一個 Stripe 客戶。 你可以使用 `createAsStripeCustomer` 方法完成此操作：

    $stripeCustomer = $user->createAsStripeCustomer();

在 Stripe 中建立客戶後，你可以在以後開始訂閱。 你可以提供一個可選的 `$options` 陣列來傳遞任何額外的 [Stripe API 支援的客戶建立參數](https://stripe.com/docs/api/customers/create)：

    $stripeCustomer = $user->createAsStripeCustomer($options);

如果你想為計費模型返回 Stripe 客戶對象，你可以使用 `asStripeCustomer` 方法：

    $stripeCustomer = $user->asStripeCustomer();

如果你想為給定的計費模型檢索 Stripe 客戶對象，但不確定該計費模型是否已經是 Stripe 中的客戶，則可以使用 createOrGetStripeCustomer 方法。 如果尚不存在，此方法將在 Stripe 中建立一個新客戶：

    $stripeCustomer = $user->createOrGetStripeCustomer();

### 更新客戶

有時，你可能希望直接向 Stripe 客戶更新其他資訊。 你可以使用 `updateStripeCustomer` 方法完成此操作。 此方法接受一組 [Stripe API 支援的客戶更新選項](https://stripe.com/docs/api/customers/update)：

    $stripeCustomer = $user->updateStripeCustomer($options);

### 餘額

Stripe 允許你貸記或借記客戶的「餘額」。 稍後，此餘額將在新發票上貸記或借記。 要檢查客戶的總餘額，你可以使用計費模型上提供的「餘額」方法。 `balance` 方法將返回以客戶貨幣表示的餘額的格式化字串表示形式：

    $balance = $user->balance();

要記入客戶的餘額，可以為該 `creditBalance` 方法提供一個值。如果你願意，還可以提供描述：

    $user->creditBalance(500, 'Premium customer top-up.');

為該方法提供一個值 `debitBalance` 將從客戶的餘額中扣除：

    $user->debitBalance(300, 'Bad usage penalty.');

`applyBalance` 方法會建立一條客戶餘額流水記錄。可以通過呼叫 `balanceTransactions` 方法獲取餘額交易記錄，這有助於提供借記或貸記記錄給客戶查看：

    // 檢索所有交易...
    $transactions = $user->balanceTransactions();

    foreach ($transactions as $transaction) {
        // 交易量...
        $amount = $transaction->amount(); // $2.31

        // 在可用的情況下檢索相關發票...
        $invoice = $transaction->invoice();
    }

### 稅號

Cashier 提供了一種管理客戶稅號的簡便方法。`taxIds` 例如，`taxIds` 方法可用於檢索作為集合分配給客戶的所有[稅號](https://stripe.com/docs/api/customer_tax_ids/object)：

    $taxIds = $user->taxIds();

你還可以通過識別碼檢索客戶的特定稅號：

    $taxId = $user->findTaxId('txi_belgium');

你可以通過向 `createTaxId` 方法提供有效的 [type](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type) 和值來建立新的稅號：

    $taxId = $user->createTaxId('eu_vat', 'BE0123456789');

`createTaxId` 方法將立即將增值稅 ID 新增到客戶的帳戶中。 [增值稅 ID 的驗證也由 Stripe 完成](https://stripe.com/docs/invoicing/customer/tax-ids#validation)； 然而，這是一個非同步的過程。 你可以通過訂閱 `customer.tax_id.updated` webhook 事件並檢查 [增值稅 ID `verification` 參數](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification)。 有關處理 webhook 的更多資訊，請參閱[有關定義 webhook 處理程序的文件](#handling-stripe-webhooks)。

你可以使用 `deleteTaxId` 方法刪除稅號：

    $user->deleteTaxId('txi_belgium');

### 使用 Stripe 同步客戶資料

通常，當你的應用程式的使用者更新他們的姓名、電子郵件地址或其他也由 Stripe 儲存的資訊時，你應該通知 Stripe 更新。 這樣一來，Stripe 的資訊副本將與你的應用程式同步。

要自動執行此操作，你可以在計費模型上定義一個事件偵聽器，以響應模型的`updated` 事件。然後，在你的事件監聽器中，你可以在模型上呼叫 `syncStripeCustomerDetails` 方法：

    use App\Models\User;
    use function Illuminate\Events\queueable;

    /**
     * 模型的「引導」方法。
     */
    protected static function booted(): void
    {
        static::updated(queueable(function (User $customer) {
            if ($customer->hasStripeId()) {
                $customer->syncStripeCustomerDetails();
            }
        }));
    }

現在，每次更新你的客戶模型時，其資訊都會與 Stripe 同步。 為方便起見，Cashier 會在初始建立客戶時自動將你客戶的資訊與 Stripe 同步。

你可以通過覆蓋 Cashier 提供的各種方法來自訂用於將客戶資訊同步到 Stripe 的列。 例如，當 Cashier 將客戶資訊同步到 Stripe 時，你可以重寫 `stripeName` 方法來自訂應該被視為客戶「姓名」的屬性：

    /**
     * 獲取應同步到 Stripe 的客戶名稱。
     */
    public function stripeName(): string|null
    {
        return $this->company_name;
    }

同樣，你可以複寫 `stripeEmail`、`stripePhone` 和 `stripeAddress` 方法。 當[更新 Stripe 客戶對象](https://stripe.com/docs/api/customers/update)時，這些方法會將資訊同步到其相應的客戶參數。 如果你希望完全控制客戶資訊同步過程，你可以複寫 `syncStripeCustomerDetails` 方法。

### 訂單入口

Stripe 提供了一個簡單的方式來[設定訂單入口](https://stripe.com/docs/billing/subscriptions/customer-portal)以便使用者可以管理訂閱、支付方法、以及查看歷史賬單。你可以在 controller 或路由中使用 `redirectToBillingPortal` 方法將使用者重新導向到賬單入口：

    use Illuminate\Http\Request;

    Route::get('/billing-portal', function (Request $request) {
        return $request->user()->redirectToBillingPortal();
    });

默認情況下，當使用者完成對訂閱的管理後，會將能夠通過 Stripe 計費門戶中的連結返回到應用的 home 路由，你可以通過傳遞 URL 作為 `redirectToBillingPortal` 方法的參數來自訂使用者返回的 URL：

    use Illuminate\Http\Request;

    Route::get('/billing-portal', function (Request $request) {
        return $request->user()->redirectToBillingPortal(route('billing'));
    });

如果你只想要生成訂單入口的 URL，可以使用 `billingPortalUrl` 方法：

    $url = $request->user()->billingPortalUrl(route('billing'));

## 支付方式

### 儲存支付方式

為了使用 Stripe 建立訂閱或者進行「一次性」支付，你需要儲存支付方法並從 Stripe 中獲取對應的識別碼。這種方式可用於實現你是否計畫使用這個支付方法進行訂閱還是單次收費，下面我們分別來介紹這兩種方法。

#### 訂閱付款方式

當儲存客戶的信用卡資訊以備將來訂閱使用時，必須使用 Stripe「Setup Intents」API 來安全地收集客戶的支付方式詳細資訊。 「Setup Intent」向 Stripe 指示向客戶的付款方式收費的目的。 Cashier 的 `Billable` 特性包括 `createSetupIntent` 方法，可輕鬆建立新的設定目的。 你應該從將呈現收集客戶付款方式詳細資訊的表單的路由或 controller 呼叫此方法：

    return view('update-payment-method', [
        'intent' => $user->createSetupIntent()
    ]);

建立設定目的並將其傳遞給檢視後，你應該將其秘密附加到將收集付款方式的元素。 例如，考慮這個「更新付款方式」表單：

```html
<input id="card-holder-name" type="text">

<!-- Stripe 元素預留位置 -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    更新付款方式
</button>
```

接下來，可以使用 Stripe.js 庫將 [Stripe 元素](https://stripe.com/docs/stripe-js) 附加到表單並安全地收集客戶的付款詳細資訊：

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接下來，可以驗證卡並使用 [Stripe 的 `confirmCardSetup` 方法](https://stripe.com/docs/js/setup_intents/confirm_card_setup)從 Stripe 檢索安全的「支付方式識別碼」：

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');
const clientSecret = cardButton.dataset.secret;

cardButton.addEventListener('click', async (e) => {
    const { setupIntent, error } = await stripe.confirmCardSetup(
        clientSecret, {
            payment_method: {
                card: cardElement,
                billing_details: { name: cardHolderName.value }
            }
        }
    );

    if (error) {
        // 向使用者顯示「error.message」...
    } else {
        // 卡已驗證成功...
    }
});
```

#### 訂閱付款方式

儲存客戶的銀行卡資訊以備將來訂閱時使用，必須使用 Stripe「Setup Intents」API 來安全地收集客戶的支付方式詳細資訊。 「設定意圖」 向Stripe 指示向客戶的付款方式收費的目的。 Cashier 的 `Billable` 特性包括 `createSetupIntent` 方法，可輕鬆建立新的設定意圖。你應該從路由或 controller 呼叫此方法，該路由或 controller 將呈現收集客戶付款方法詳細資訊的表單:

    return view('update-payment-method', [
        'intent' => $user->createSetupIntent()
    ]);

建立設定意圖並將其傳遞給檢視後，你應該將其秘密附加到將收集付款方式的元素。 例如，考慮這個「更新付款方式」表單:

```html
<input id="card-holder-name" type="text">

<!-- Stripe 元素預留位置 -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    更新付款方式
</button>
```

接下來，可以使用 Stripe.js 庫將 [Stripe 元素](https://stripe.com/docs/stripe-js)附加到表單並安全地收集客戶的付款詳細資訊:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接下來，可以從 Stripe 搜尋安全的「支付方式識別碼」驗證銀行卡並使用 [Stripe 的 `confirmCardSetup` 方法](https://stripe.com/docs/js/setup_intents/confirm_card_setup):

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');
const clientSecret = cardButton.dataset.secret;

cardButton.addEventListener('click', async (e) => {
    const { setupIntent, error } = await stripe.confirmCardSetup(
        clientSecret, {
            payment_method: {
                card: cardElement,
                billing_details: { name: cardHolderName.value }
            }
        }
    );

    if (error) {
        // 向使用者顯示「error.message」...
    } else {
        // 該銀行卡已驗證成功...
    }
});
```

銀行卡通過 Stripe 驗證後，你可以將生成的 `setupIntent.payment_method` 識別碼傳遞給你的 Laravel 應用程式，在那裡它可以附加到客戶。 付款方式可以[新增為新付款方式](#adding-payment-methods)或[用於更新默認付款方式](#updating-the-default-payment-method)。 你還可以立即使用付款方式識別碼來[建立新訂閱](#creating-subscriptions)。

> **技巧**  
> 如果你想瞭解有關設定目的和收集客戶付款詳細資訊的更多資訊，請[查看 Stripe 提供的概述](https://stripe.com/docs/payments/save-and-reuse#php)。

#### 單筆費用的支付方式

當然，在針對客戶的支付方式進行單筆收費時，我們只需要使用一次支付方式識別碼。 由於 Stripe 的限制，你不能使用客戶儲存的默認付款方式進行單筆收費。 你必須允許客戶使用 Stripe.js 庫輸入他們的付款方式詳細資訊。 例如，考慮以下形式：

```html
<input id="card-holder-name" type="text">

<!-- Stripe 元素預留位置 -->
<div id="card-element"></div>

<button id="card-button">
    處理付款
</button>
```

定義這樣的表單後，可以使用 Stripe.js 庫將[Stripe 元素](https://stripe.com/docs/stripe-js)附加到表單並安全地收集客戶的付款詳細資訊：

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接下來，可以驗證卡並使用 [Stripe 的 `createPaymentMethod` 方法](https://stripe.com/docs/stripe-js/reference#stripe-create-payment) 從 Stripe 檢索安全的「支付方式識別碼」-方法）：

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');

cardButton.addEventListener('click', async (e) => {
    const { paymentMethod, error } = await stripe.createPaymentMethod(
        'card', cardElement, {
            billing_details: { name: cardHolderName.value }
        }
    );

    if (error) {
        // 向使用者顯示「error.message」...
    } else {
        // 卡已驗證成功...
    }
});
```

銀行卡通過 Stripe 驗證後，你可以將生成的 `setupIntent.payment_method` 識別碼傳遞給你的 Laravel 應用程式，在那裡它可以附加到客戶。付款方式可以[新增為新付款方式](#adding-payment-methods)或[用於更新默認付款方式](#updating-the-default-payment-method)。 你還可以立即使用付款方式識別碼來[建立新訂閱](#creating-subscriptions)。

> **筆記**
> 如果你想瞭解有關設定目的和收集客戶付款詳細資訊的更多資訊，請[查看 Stripe 提供的概述](https://stripe.com/docs/payments/save-and-reuse#php).

#### 單筆費用的支付方式

當然，在針對客戶的支付方式進行單筆收費時，我們只需要使用一次支付方式識別碼。 由於 Stripe 的限制，你不能使用客戶儲存的默認付款方式進行單筆收費。 你必須允許客戶使用 Stripe.js 庫輸入他們的付款方式詳細資訊。 例如，考慮以下形式：

```html
<input id="card-holder-name" type="text">

<!-- Stripe 元素預留位置 -->
<div id="card-element"></div>

<button id="card-button">
    付款流程
</button>
```

在定義了這樣一個表單之後，可以使用 Stripe.js 庫將 [Stripe Element](https://stripe.com/docs/stripe-js) 附加到表單並安全地收集客戶的付款詳細資訊：

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接下來，可以驗證銀行卡並使用 [Stripe 的 `createPaymentMethod` 方法](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method):

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');

cardButton.addEventListener('click', async (e) => {
    const { paymentMethod, error } = await stripe.createPaymentMethod(
        'card', cardElement, {
            billing_details: { name: cardHolderName.value }
        }
    );

    if (error) {
        // 向使用者顯示「error.message」...
    } else {
        // 該銀行卡已驗證成功...
    }
});
```

如果卡驗證成功，你可以將 `paymentMethod.id` 傳遞給你的 Laravel 應用程式並處理[單次收費](#simple-charge)。

### 檢索付款方式

計費模型實例上的 `paymentMethods` 方法返回 `Laravel\Cashier\PaymentMethod` 實例的集合：

    $paymentMethods = $user->paymentMethods();

默認情況下，此方法將返回 `card` 類型的支付方式。要檢索不同類型的付款方式，你可以將 `type` 作為參數傳遞給該方法：

    $paymentMethods = $user->paymentMethods('sepa_debit');

要檢索客戶的默認付款方式，可以使用 `defaultPaymentMethod` 方法：

    $paymentMethod = $user->defaultPaymentMethod();

你可以使用 `findPaymentMethod` 方法檢索附加到計費模型的特定付款方式：

    $paymentMethod = $user->findPaymentMethod($paymentMethodId);

### 確定使用者是否有付款方式

要確定計費模型是否有附加到其帳戶的默認付款方式，請呼叫 `hasDefaultPaymentMethod` 方法：

    if ($user->hasDefaultPaymentMethod()) {
        // ...
    }

你可以使用 `hasPaymentMethod` 方法來確定計費模型是否至少有一種支付方式附加到他們的帳戶：

    if ($user->hasPaymentMethod()) {
        // ...
    }

此方法將確定計費模型是否具有 `card` 類型的支付方式。 要確定該模型是否存在另一種類型的支付方式，你可以將 `type` 作為參數傳遞給該方法：

    if ($user->hasPaymentMethod('sepa_debit')) {
        // ...
    }

### 更新默認付款方式

`updateDefaultPaymentMethod` 方法可用於更新客戶的默認支付方式資訊。 此方法接受 Stripe 支付方式識別碼，並將新支付方式指定為默認支付方式：

    $user->updateDefaultPaymentMethod($paymentMethod);

如果銀行卡驗證成功，你可以將`paymentMethod.id` 傳遞給你的 Laravel 應用程式並處理 [單次收費](#simple-charge).

### 搜尋付款方式

計費模型實例上的 `paymentMethods` 方法返回一組 `Laravel\Cashier\PaymentMethod` 實例：

    $paymentMethods = $user->paymentMethods();

默認情況下，此方法將返回 `card` 類型的支付方式。 要搜尋不同類型的付款方式，你可以將 `type` 作為參數傳遞給該方法：

    $paymentMethods = $user->paymentMethods('sepa_debit');

要搜尋客戶的默認付款方式，可以使用 `defaultPaymentMethod` 方法：

    $paymentMethod = $user->defaultPaymentMethod();

你可以使用 `findPaymentMethod` 方法搜尋附加到計費模型的特定付款方式：

    $paymentMethod = $user->findPaymentMethod($paymentMethodId);

### 確定使用者是否有付款方式

要確定計費模型是否有附加到其帳戶的默認付款方式，請呼叫 `hasDefaultPaymentMethod` 方法：

    if ($user->hasDefaultPaymentMethod()) {
        // ...
    }

你可以 `hasPaymentMethod` 方法來確定計費模型是否至少有一種支付方式附加到他們的帳戶：

    if ($user->hasPaymentMethod()) {
        // ...
    }

此方法將確定計費模型是否具有 `card` 類型的支付方式。 要確定該模型是否存在另一種類型的支付方式，你可以將 `type` 作為參數傳遞給該方法：

    if ($user->hasPaymentMethod('sepa_debit')) {
        // ...
    }

### 更新默認付款方式

`updateDefaultPaymentMethod` 方法可用於更新客戶的默認支付方式資訊。 此方法接受 Stripe 付款方式識別碼，並將新付款方式指定為默認付款方式：

    $user->updateDefaultPaymentMethod($paymentMethod);

要將你的默認支付方式資訊與客戶在 Stripe 中的默認支付方式資訊同步，你可以使用 `updateDefaultPaymentMethodFromStripe` 方法：

    $user->updateDefaultPaymentMethodFromStripe();

> **注意**  
> 客戶的默認付款方式只能用於開具發票和建立新訂閱。 由於 Stripe 施加的限制，它可能無法用於單次收費。

### 新增付款方式

要新增新的支付方式，你可以在計費模型上呼叫 `addPaymentMethod` 方法，並傳遞支付方式識別碼：

    $user->addPaymentMethod($paymentMethod);

> **技巧**  
> 要瞭解如何檢索付款方式識別碼，請查看[付款方式儲存文件](#storing-payment-methods)。

### 刪除付款方式

要刪除付款方式，你可以在要刪除的 `Laravel\Cashier\PaymentMethod` 實例上呼叫 `delete` 方法：

    $paymentMethod->delete();

`deletePaymentMethod` 方法將從計費模型中刪除特定的支付方式：

    $user->deletePaymentMethod('pm_visa');

`deletePaymentMethods` 方法將刪除計費模型的所有付款方式資訊：

    $user->deletePaymentMethods();

默認情況下，此方法將刪除 `card` 類型的支付方式。 要刪除不同類型的付款方式，你可以將 `type` 作為參數傳遞給該方法：

    $user->deletePaymentMethods('sepa_debit');

> **注意**  
> 如果使用者有一個有效的訂閱，你的應用程式不應該允許他們刪除他們的默認支付方式。

## 訂閱

訂閱提供了一種為你的客戶設定定期付款的方法。 Cashier 管理的 Stripe 訂閱支援多種訂閱價格、訂閱數量、試用等。

### 建立訂閱

要建立訂閱，首先檢索你的計費模型的實例，通常是 `App\Models\User` 的實例。 檢索到模型實例後，你可以使用 `newSubscription` 方法建立模型的訂閱：

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription(
            'default', 'price_monthly'
        )->create($request->paymentMethodId);

        // ...
    });

傳遞給 `newSubscription` 方法的第一個參數應該是訂閱的內部名稱。 如果你的應用程式僅提供單一訂閱，你可以稱其為 `default` 或 `primary`。 此訂閱名稱僅供內部應用程式使用，無意向使用者顯示。 此外，它不應包含空格，並且在建立訂閱後絕不能更改。 第二個參數是使用者訂閱的具體價格。 該值應對應於 Stripe 中的價格識別碼。

`create` 方法接受 [Stripe 支付方式標識](#storing-payment-methods)或 Stripe `PaymentMethod` 對象，將開始訂閱並使用計費模型的 Stripe 客戶 ID 和其他相關資訊更新你的資料庫賬單資訊。

> **注意**  
> 將支付方式識別碼直接傳遞給 `create` 訂閱方法也會自動將其新增到使用者儲存的支付方式中。

#### 通過發票電子郵件收取定期付款

你可以指示 Stripe 在每次定期付款到期時通過電子郵件向客戶傳送發票，而不是自動收取客戶的經常性付款。 然後，客戶可以在收到發票後手動支付。 通過發票收取經常性付款時，客戶無需預先提供付款方式：

    $user->newSubscription('default', 'price_monthly')->createAndSendInvoice();

客戶在取消訂閱之前必須支付發票的時間由 `days_until_due` 選項決定。 默認情況下，這是 30 天； 但是，如果你願意，可以為此選項提供特定值：

    $user->newSubscription('default', 'price_monthly')->createAndSendInvoice([], [
        'days_until_due' => 30
    ]);

#### 數量

如果你想在建立訂閱時為價格設定特定的[數量](https://stripe.com/docs/billing/subscriptions/quantities)，你應該在建立之前呼叫訂閱建構器上的 `quantity` 方法 訂閱：

    $user->newSubscription('default', 'price_monthly')
         ->quantity(5)
         ->create($paymentMethod);

#### 其它詳細資訊

如果你想指定支援的其他[客戶](https://stripe.com/docs/api/customers/create)或[訂閱](https://stripe.com/docs/api/subscriptions/create)選項 通過 Stripe，你可以通過將它們作為第二個和第三個參數傳遞給 `create` 方法來實現：

    $user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
        'email' => $email,
    ], [
        'metadata' => ['note' => 'Some extra information.'],
    ]);

#### 優惠券

如果你想在建立訂閱時使用優惠券，你可以使用 `withCoupon` 方法：

    $user->newSubscription('default', 'price_monthly')
         ->withCoupon('code')
         ->create($paymentMethod);

或者，如果你想應用 [Stripe 促銷程式碼](https://stripe.com/docs/billing/subscriptions/discounts/codes)，你可以使用 `withPromotionCode` 方法：

    $user->newSubscription('default', 'price_monthly')
         ->withPromotionCode('promo_code_id')
         ->create($paymentMethod);

給定的促銷程式碼 ID 應該是分配給促銷程式碼的 Stripe API ID，而不是面向客戶的促銷程式碼。 如果你需要根據給定的面向客戶的促銷程式碼尋找促銷程式碼 ID，你可以使用 `findPromotionCode` 方法：

    // 通過面向客戶的程式碼尋找促銷程式碼 ID...
    $promotionCode = $user->findPromotionCode('SUMMERSALE');

    // 通過面向客戶的程式碼尋找有效的促銷程式碼 ID...
    $promotionCode = $user->findActivePromotionCode('SUMMERSALE');



在上面的示例中，返回的 `$promotionCode` 對像是 `Laravel\Cashier\PromotionCode` 的一個實例。 這個類裝飾了一個底層的 `Stripe\PromotionCode` 對象。 你可以通過呼叫 `coupon` 方法來檢索與促銷程式碼相關的優惠券：

    $coupon = $user->findPromotionCode('SUMMERSALE')->coupon();

優惠券實例允許你確定折扣金額以及優惠券是代表固定折扣還是基於百分比的折扣：

    if ($coupon->isPercentage()) {
        return $coupon->percentOff().'%'; // 21.5%
    } else {
        return $coupon->amountOff(); // $5.99
    }

你還可以檢索當前應用於客戶或訂閱的折扣：

    $discount = $billable->discount();

    $discount = $subscription->discount();

返回的 `Laravel\Cashier\Discount` 實例裝飾底層的 `Stripe\Discount` 對象實例。 你可以通過呼叫 `coupon` 方法獲取與此折扣相關的優惠券：

    $coupon = $subscription->discount()->coupon();

如果你想將新的優惠券或促銷程式碼應用於客戶或訂閱，你可以通過 `applyCoupon` 或 `applyPromotionCode` 方法進行：

    $billable->applyCoupon('coupon_id');
    $billable->applyPromotionCode('promotion_code_id');

    $subscription->applyCoupon('coupon_id');
    $subscription->applyPromotionCode('promotion_code_id');

請記住，你應該使用分配給促銷程式碼的 Stripe API ID，而不是面向客戶的促銷程式碼。 在給定時間只能將一個優惠券或促銷程式碼應用於客戶或訂閱。

有關此主題的更多資訊，請參閱有關[優惠券](https://stripe.com/docs/billing/subscriptions/coupons)和[促銷程式碼](https://stripe.com/docs/billing)的 Stripe 文件 /訂閱/優惠券/程式碼）。

#### 新增訂閱

如果你想向已有默認付款方式的客戶新增訂閱，你可以在訂閱建構器上呼叫 `add` 方法：

    use App\Models\User;

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')->add();

#### 從 Stripe 儀表板建立訂閱

你還可以從 Stripe 儀表板本身建立訂閱。 這樣做時，Cashier 將同步新新增的訂閱並為其分配一個名稱 `default`。 要自訂分配給儀表板建立的訂閱的訂閱名稱，[擴展 `WebhookController`](#defining-webhook-event-handlers)並覆蓋 `newSubscriptionName` 方法。

此外，你只能通過 Stripe 儀表板建立一種類型的訂閱。 如果你的應用程式提供多個使用不同名稱的訂閱，則只能通過 Stripe 儀表板新增一種類型的訂閱。

最後，你應該始終確保你的應用程式提供的每種訂閱類型只新增一個活動訂閱。 如果客戶有兩個 `default` 訂閱，Cashier 只會使用最近新增的訂閱，即使兩者都會與你的應用程式資料庫同步。

### 檢查訂閱狀態

客戶訂閱你的應用程式後，你可以使用各種方便的方法輕鬆檢查他們的訂閱狀態。 首先，如果客戶有有效訂閱， `subscribed` 方法會返回 `true` ，即使該訂閱當前處於試用期內。 `subscribed` 方法接受訂閱的名稱作為它的第一個參數：

    if ($user->subscribed('default')) {
        // ...
    }

`subscribed` 方法也非常適合[路由中介軟體](/docs/laravel/10.x/middleware)，允許你根據使用者的訂閱狀態過濾對路由和 controller 的訪問：

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class EnsureUserIsSubscribed
    {
        /**
         * 處理傳入請求。
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            if ($request->user() && ! $request->user()->subscribed('default')) {
                // 該使用者不是付費客戶...
                return redirect('billing');
            }

            return $next($request);
        }
    }

如果你想確定使用者是否仍在試用期內，你可以使用 `onTrial` 方法。 此方法可用於確定是否應向使用者顯示他們仍在試用期的警告：

    if ($user->subscription('default')->onTrial()) {
        // ...
    }

`subscribedToProduct` 方法可用於根據給定的 Stripe 產品識別碼確定使用者是否訂閱了給定的產品。 在 Stripe 中，產品是價格的集合。 在此示例中，我們將確定使用者的 `default` 訂閱是否主動訂閱了應用程式的「高級」產品。 給定的 Stripe 產品識別碼應與你在 Stripe 儀表板中的產品識別碼之一相對應：

    if ($user->subscribedToProduct('prod_premium', 'default')) {
        // ...
    }

通過將陣列傳遞給 `subscribedToProduct` 方法，你可以確定使用者的 `default` 訂閱是否主動訂閱了應用程式的「基本」或「高級」產品：

    if ($user->subscribedToProduct(['prod_basic', 'prod_premium'], 'default')) {
        // ...
    }

`subscribedToPrice` 方法可用於確定客戶的訂閱是否對應於給定的價格 ID：

    if ($user->subscribedToPrice('price_basic_monthly', 'default')) {
        // ...
    }

`recurring` 方法可用於確定使用者當前是否已訂閱並且不再處於試用期內：

    if ($user->subscription('default')->recurring()) {
        // ...
    }

> **注意**  
> 如果使用者有兩個同名訂閱，則 `subscription` 方法將始終返回最近的訂閱。 例如，一個使用者可能有兩個名為 `default`的訂閱記錄； 但是，其中一個訂閱可能是舊的、過期的訂閱，而另一個是當前的、有效的訂閱。 最近的訂閱將始終返回，而較舊的訂閱將保留在資料庫中以供歷史審查。

#### 取消訂閱狀態

要確定使用者是否曾經是活躍訂閱者但已取消訂閱，你可以使用 `canceled` 方法：

    if ($user->subscription('default')->canceled()) {
        // ...
    }

你還可以確定使用者是否已取消他們的訂閱，但在訂閱完全到期之前是否仍處於「寬限期」。 例如，如果使用者在 3 月 5 日取消了原定於 3 月 10 日到期的訂閱，則使用者在 3 月 10 日之前處於「寬限期」。 請注意，`subscribed` 方法在此期間仍會返回 `true`：

    if ($user->subscription('default')->onGracePeriod()) {
        // ...
    }

要確定使用者是否已取消訂閱並且不再處於「寬限期」內，你可以使用 `ended` 方法：

    if ($user->subscription('default')->ended()) {
        // ...
    }

#### 不完整和逾期狀態

如果訂閱在建立後需要二次支付操作，訂閱將被標記為「不完整」。 訂閱狀態儲存在 Cashier 的 `subscriptions` 資料庫表的 `stripe_status` 列中。

同樣，如果在交換價格時需要二次支付操作，訂閱將被標記為 `past_due` 。 當你的訂閱處於這些狀態中的任何一種時，在客戶確認付款之前，它不會啟動。 可以使用計費模型或訂閱實例上的  `hasIncompletePayment` 方法來確定訂閱是否有未完成的付款：

    if ($user->hasIncompletePayment('default')) {
        // ...
    }

    if ($user->subscription('default')->hasIncompletePayment()) {
        // ...
    }

當訂閱有未完成的付款時，你應該將使用者引導至 Cashier 的付款確認頁面，並傳遞 `latestPayment` 識別碼。 你可以使用訂閱實例上可用的 `latestPayment` 方法來檢索此識別碼：

```html
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    請確認你的付款。
</a>
```

如果你希望訂閱在處於 `past_due` 或 `incomplete` 狀態時仍被視為有效，你可以使用 Cashier 提供的 `keepPastDueSubscriptionsActive` 和 `keepIncompleteSubscriptionsActive` 方法。 通常，應在 `App\Providers\AppServiceProvider` 的 `register` 方法中呼叫這些方法：

    use Laravel\Cashier\Cashier;

    /**
     * 註冊任何應用程式服務。
     */
    public function register(): void
    {
        Cashier::keepPastDueSubscriptionsActive();
        Cashier::keepIncompleteSubscriptionsActive();
    }

> **注意**  
> 當訂閱處於 `incomplete` 狀態時，在確認付款之前無法更改。 因此，當訂閱處於 `incomplete` 狀態時， `swap` 和 `updateQuantity` 方法將拋出異常。

#### 訂閱範圍

大多數訂閱狀態也可用作查詢範圍，以便你可以輕鬆查詢資料庫以尋找處於給定狀態的訂閱：

    // 獲取所有活動訂閱...
    $subscriptions = Subscription::query()->active()->get();

    // 獲取使用者所有已取消的訂閱...
    $subscriptions = $user->subscriptions()->canceled()->get();

可用範圍的完整列表如下：

    Subscription::query()->active();
    Subscription::query()->canceled();
    Subscription::query()->ended();
    Subscription::query()->incomplete();
    Subscription::query()->notCanceled();
    Subscription::query()->notOnGracePeriod();
    Subscription::query()->notOnTrial();
    Subscription::query()->onGracePeriod();
    Subscription::query()->onTrial();
    Subscription::query()->pastDue();
    Subscription::query()->recurring();

### 更改價格

客戶訂閱你的應用程式後，他們可能偶爾想要更改為新的訂閱價格。 要將客戶換成新價格，請將 Stripe 價格的識別碼傳遞給 `swap` 方法。 交換價格時，假設使用者想要重新啟動他們的訂閱（如果之前已取消訂閱）。 給定的價格識別碼應對應於 Stripe 儀表板中可用的 Stripe 價格識別碼：

    use App\Models\User;

    $user = App\Models\User::find(1);

    $user->subscription('default')->swap('price_yearly');

如果客戶處於試用期，則將保持試用期。 此外，如果訂閱存在「數量」，則也將保留該數量。

如果你想交換價格並取消客戶當前的任何試用期，你可以呼叫 `skipTrial` 方法：

    $user->subscription('default')
            ->skipTrial()
            ->swap('price_yearly');

如果你想交換價格並立即向客戶開具發票而不是等待他們的下一個結算週期，你可以使用 `swapAndInvoice` 方法：

    $user = User::find(1);

    $user->subscription('default')->swapAndInvoice('price_yearly');

#### 按比例分配

默認情況下，Stripe 在價格之間交換時按比例分配費用。 `noProrate` 方法可用於在不按比例分配費用的情況下更新訂閱價格：

    $user->subscription('default')->noProrate()->swap('price_yearly');

有關訂閱按比例分配的更多資訊，請參閱 [Stripe 文件](https://stripe.com/docs/billing/subscriptions/prorations)。

> **注意**  
> 在 `swapAndInvoice` 方法之前執行 `noProrate` 方法將不會影響按比例分配。 將始終開具發票。

### 認購數量

有時訂閱會受到「數量」的影響。 例如，項目管理應用程式可能對每個項目收取每月 10 美元的費用。 你可以使用 `incrementQuantity` 和 `decrementQuantity` 方法輕鬆增加或減少你的訂閱數量：

    use App\Models\User;

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity();

    // 向訂閱的當前數量新增五個...
    $user->subscription('default')->incrementQuantity(5);

    $user->subscription('default')->decrementQuantity();

    // 從訂閱的當前數量中減去五...
    $user->subscription('default')->decrementQuantity(5);

或者，你可以使用 `updateQuantity` 方法設定特定數量：

    $user->subscription('default')->updateQuantity(10);

`noProrate` 方法可用於在不按比例分配費用的情況下更新訂閱的數量：

    $user->subscription('default')->noProrate()->updateQuantity(10);

有關訂閱數量的更多資訊，請參閱 [Stripe 文件](https://stripe.com/docs/subscriptions/quantities)。

#### 多個產品的訂閱數量

如果你的訂閱是[包含多個產品的訂閱](#subscriptions-with-multiple-products)，你應該將你希望增加或減少的數量的價格 ID 作為第二個參數傳遞給增量/減量方法：

    $user->subscription('default')->incrementQuantity(1, 'price_chat');

### 訂閱多個產品

[訂閱多個產品](https://stripe.com/docs/billing/subscriptions/multiple-products)允許你將多個計費產品分配給一個訂閱。 例如，假設你正在建構一個客戶服務「幫助台」應用程式，其基本訂閱價格為每月 10 美元，但提供即時聊天附加產品，每月額外收費 15 美元。 包含多個產品的訂閱資訊儲存在 Cashier 的 `subscription_items` 資料庫表中。

你可以通過將價格陣列作為第二個參數傳遞給 `newSubscription` 方法來為給定訂閱指定多個產品：

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default', [
            'price_monthly',
            'price_chat',
        ])->create($request->paymentMethodId);

        // ...
    });

在上面的例子中，客戶將有兩個附加到他們的 `default` 訂閱的價格。 兩種價格都將按各自的計費間隔收取。 如有必要，你可以使用 `quantity` 方法為每個價格指定具體數量：

    $user = User::find(1);

    $user->newSubscription('default', ['price_monthly', 'price_chat'])
        ->quantity(5, 'price_chat')
        ->create($paymentMethod);

如果你想為現有訂閱新增另一個價格，你可以呼叫訂閱的 `addPrice` 方法：

    $user = User::find(1);

    $user->subscription('default')->addPrice('price_chat');

上面的示例將新增新價格，客戶將在下一個計費週期為此付費。 如果你想立即向客戶開具賬單，你可以使用 `addPriceAndInvoice` 方法：

    $user->subscription('default')->addPriceAndInvoice('price_chat');

如果你想新增具有特定數量的價格，你可以將數量作為 `addPrice` 或 `addPriceAndInvoice` 方法的第二個參數傳遞：

    $user = User::find(1);

    $user->subscription('default')->addPrice('price_chat', 5);

你可以使用 `removePrice` 方法從訂閱中刪除價格：

    $user->subscription('default')->removePrice('price_chat');

> **注意**  
> 你不得刪除訂閱的最後價格。 相反，你應該簡單地取消訂閱。

#### 交換價格

你還可以更改附加到具有多個產品的訂閱的價格。 例如，假設客戶訂閱了帶有 `price_chat` 附加產品的 `price_basic` 訂閱，而你想要將客戶從 `price_basic` 升級到 `price_pro` 價格：

    use App\Models\User;

    $user = User::find(1);

    $user->subscription('default')->swap(['price_pro', 'price_chat']);

執行上述示例時，刪除帶有 `price_basic` 的基礎訂閱項，保留帶有 `price_chat` 的訂閱項。 此外，還會為 `price_pro` 建立一個新的訂閱項目。

你還可以通過將鍵/值對陣列傳遞給 `swap` 方法來指定訂閱項選項。 例如，你可能需要指定訂閱價格數量：

    $user = User::find(1);

    $user->subscription('default')->swap([
        'price_pro' => ['quantity' => 5],
        'price_chat'
    ]);

如果你想交換訂閱的單一價格，你可以在訂閱項目本身上使用 `swap` 方法。 如果你想保留訂閱的其他價格的所有現有中繼資料，此方法特別有用：

    $user = User::find(1);

    $user->subscription('default')
            ->findItemOrFail('price_basic')
            ->swap('price_pro');

#### 按比例分配

默認情況下，Stripe 會在為多個產品的訂閱新增或刪除價格時按比例收費。 如果你想在不按比例分配的情況下進行價格調整，你應該將 `noProrate` 方法連結到你的價格操作中：

    $user->subscription('default')->noProrate()->removePrice('price_chat');

#### 數量

如果你想更新單個訂閱價格的數量，你可以使用[現有數量方法](#subscription-quantity) 將價格名稱作為附加參數傳遞給該方法：

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity(5, 'price_chat');

    $user->subscription('default')->decrementQuantity(3, 'price_chat');

    $user->subscription('default')->updateQuantity(10, 'price_chat');

> **注意**  
> 當訂閱有多個價格時，`Subscription` 模型上的 `stripe_price` 和 `quantity` 屬性將為 `null`。 要訪問單個價格屬性，你應該使用 `Subscription` 模型上可用的 `items` 關係。

#### 訂閱項目

當訂閱有多個價格時，它會在資料庫的 `subscription_items` 表中儲存多個訂閱 `items`。 你可以通過訂閱上的 `items` 關係訪問這些：

    use App\Models\User;

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->items->first();

    // 檢索特定商品的 Stripe 價格和數量...
    $stripePrice = $subscriptionItem->stripe_price;
    $quantity = $subscriptionItem->quantity;

你還可以使用 `findItemOrFail` 方法檢索特定價格：

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->findItemOrFail('price_chat');

### 多個訂閱

Stripe 允許你的客戶同時擁有多個訂閱。 例如，你可能經營一家提供游泳訂閱和舉重訂閱的健身房，並且每個訂閱可能有不同的定價。 當然，客戶應該能夠訂閱其中一個或兩個計畫。

當你的應用程式建立訂閱時，你可以將訂閱的名稱提供給 `newSubscription` 方法。 該名稱可以是表示使用者正在啟動的訂閱類型的任何字串：

    use Illuminate\Http\Request;

    Route::post('/swimming/subscribe', function (Request $request) {
        $request->user()->newSubscription('swimming')
            ->price('price_swimming_monthly')
            ->create($request->paymentMethodId);

        // ...
    });

在此示例中，我們為客戶發起了每月一次的游泳訂閱。 但是，他們以後可能想換成按年訂閱。 在調整客戶的訂閱時，我們可以簡單地交換 `swimming` 訂閱的價格：

    $user->subscription('swimming')->swap('price_swimming_yearly');

當然，你也可以完全取消訂閱：

    $user->subscription('swimming')->cancel();

### 計量計費

[計量計費](https://stripe.com/docs/billing/subscriptions/metered-billing) 允許你根據客戶在計費週期內的產品使用情況向客戶收費。 例如，你可以根據客戶每月傳送的簡訊或電子郵件的數量向客戶收費。

要開始使用計量計費，你首先需要在 Stripe 控制面板中建立一個具有計量價格的新產品。 然後，使用 `meteredPrice` 將計量價格 ID 新增到客戶訂閱：

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default')
            ->meteredPrice('price_metered')
            ->create($request->paymentMethodId);

        // ...
    });

你還可以通過 [Stripe Checkout](#checkout) 開始計量訂閱：

    $checkout = Auth::user()
            ->newSubscription('default', [])
            ->meteredPrice('price_metered')
            ->checkout();

    return view('your-checkout-view', [
        'checkout' => $checkout,
    ]);

#### 報告使用情況

當你的客戶使用你的應用程式時，你將向 Stripe 報告他們的使用情況，以便他們可以精準地計費。 要增加計量訂閱的使用，你可以使用 `reportUsage` 方法：

    $user = User::find(1);

    $user->subscription('default')->reportUsage();

默認情況下，「使用數量」1 會新增到計費週期。 或者，你可以傳遞特定數量的「使用量」以新增到客戶在計費期間的使用量中：

    $user = User::find(1);

    $user->subscription('default')->reportUsage(15);

如果你的應用程式在單個訂閱中提供多個價格，你將需要使用 `reportUsageFor` 方法來指定你要報告使用情況的計量價格：

    $user = User::find(1);

    $user->subscription('default')->reportUsageFor('price_metered', 15);

有時，你可能需要更新之前報告的使用情況。 為此，你可以將時間戳或 `DateTimeInterface` 實例作為第二個參數傳遞給 `reportUsage`。 這樣做時，Stripe 將更新在給定時間報告的使用情況。 你可以繼續更新以前的使用記錄，因為給定的日期和時間仍在當前計費週期內：

    $user = User::find(1);

    $user->subscription('default')->reportUsage(5, $timestamp);

#### 檢索使用記錄

要檢索客戶過去的使用情況，你可以使用訂閱實例的 `usageRecords` 方法：

    $user = User::find(1);

    $usageRecords = $user->subscription('default')->usageRecords();

如果你的應用程式為單個訂閱提供多個價格，你可以使用 `usageRecordsFor` 方法指定你希望檢索使用記錄的計量價格：

    $user = User::find(1);

    $usageRecords = $user->subscription('default')->usageRecordsFor('price_metered');

`usageRecords` 和 `usageRecordsFor` 方法返回一個包含使用記錄關聯陣列的 Collection 實例。 你可以遍歷此陣列以顯示客戶的總使用量：

    @foreach ($usageRecords as $usageRecord)
        - Period Starting: {{ $usageRecord['period']['start'] }}
        - Period Ending: {{ $usageRecord['period']['end'] }}
        - Total Usage: {{ $usageRecord['total_usage'] }}
    @endforeach

有關返回的所有使用資料的完整參考以及如何使用 Stripe 基於游標的分頁，請參閱[官方 Stripe API 文件](https://stripe.com/docs/api/usage_records/subscription_item_summary_list)。

### 訂閱稅

> **注意**  
> 你可以[使用 Stripe Tax 自動計算稅費](#tax-configuration)，而不是手動計算稅率

要指定使用者為訂閱支付的稅率，你應該在計費模型上實施 `taxRates` 方法並返回一個包含 Stripe 稅率 ID 的陣列。你可以在[你的 Stripe 控制面板](https://dashboard.stripe.com/test/tax-rates) 中定義這些稅率：

    /**
     * 適用於客戶訂閱的稅率。
     *
     * @return array<int, string>
     */
    public function taxRates(): array
    {
        return ['txr_id'];
    }

`taxRates` 方法使你能夠在逐個客戶的基礎上應用稅率，這對於跨越多個國家和稅率的使用者群可能會有所幫助。

如果你提供多種產品的訂閱，你可以通過在計費模型上實施 `priceTaxRates` 方法為每個價格定義不同的稅率：

    /**
     * 適用於客戶訂閱的稅率。
     *
     * @return array<string, array<int, string>>
     */
    public function priceTaxRates(): array
    {
        return [
            'price_monthly' => ['txr_id'],
        ];
    }

> **注意**  
> `taxRates` 方法僅適用於訂閱費用。 如果你使用 Cashier 進行「一次性」收費，你將需要手動指定當時的稅率。

#### 同步稅率

當更改由 `taxRates` 方法返回的硬編碼稅率 ID 時，使用者任何現有訂閱的稅收設定將保持不變。 如果你希望使用新的 `taxRates` 值更新現有訂閱的稅值，你應該在使用者的訂閱實例上呼叫  `syncTaxRates` 方法：

    $user->subscription('default')->syncTaxRates();

這還將同步具有多個產品的訂閱的任何項目稅率。 如果你的應用程式提供多種產品的訂閱，你應該確保你的計費模型實施 `priceTaxRates` 方法[如上所述](#subscription-taxes)。

#### 免稅

Cashier 還提供 `isNotTaxExempt`、`isTaxExempt` 和 `reverseChargeApplies` 方法來確定客戶是否免稅。 這些方法將呼叫 Stripe API 來確定客戶的免稅狀態：

    use App\Models\User;

    $user = User::find(1);

    $user->isTaxExempt();
    $user->isNotTaxExempt();
    $user->reverseChargeApplies();

> **注意**  
> 這些方法也適用於任何 `Laravel\Cashier\Invoice` 對象。 但是，當在 `Invoice` 對象上呼叫時，這些方法將確定發票建立時的豁免狀態。

### 訂閱錨定日期

默認情況下，計費週期錨是建立訂閱的日期，或者如果使用試用期，則為試用結束的日期。 如果你想修改計費錨點日期，你可以使用 `anchorBillingCycleOn` 方法：

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $anchor = Carbon::parse('first day of next month');

        $request->user()->newSubscription('default', 'price_monthly')
                    ->anchorBillingCycleOn($anchor->startOfDay())
                    ->create($request->paymentMethodId);

        // ...
    });

有關管理訂閱計費週期的更多資訊，請參閱 [Stripe 計費週期文件](https://stripe.com/docs/billing/subscriptions/billing-cycle)

### 取消訂閱

要取消訂閱，請在使用者的訂閱上呼叫 `cancel` 方法：

    $user->subscription('default')->cancel();

當訂閱被取消時，Cashier 將自動在你的 `subscriptions` 資料庫表中設定 `ends_at` 列。 此列用於瞭解 `subscribed` 方法何時應開始返回 `false`。

例如，如果客戶在 3 月 1 日取消訂閱，但訂閱計畫要到 3 月 5 日才結束，則 `subscribed` 方法將繼續返回 `true` 直到 3 月 5 日。 這樣做是因為通常允許使用者繼續使用應用程式，直到他們的計費週期結束。

你可以使用 `onGracePeriod` 方法確定使用者是否已取消訂閱但仍處於「寬限期」：

    if ($user->subscription('default')->onGracePeriod()) {
        // ...
    }

如果你希望立即取消訂閱，請在使用者的訂閱上呼叫 `cancelNow` 方法：

    $user->subscription('default')->cancelNow();

如果你希望立即取消訂閱並為任何剩餘的未開具發票的計量使用或新的/待定的按比例分配發票項目開具發票，請在使用者的訂閱上呼叫 `cancelNowAndInvoice` 方法：

    $user->subscription('default')->cancelNowAndInvoice();

你也可以選擇在特定時間取消訂閱：

    $user->subscription('default')->cancelAt(
        now()->addDays(10)
    );

### 恢復訂閱

如果客戶取消了他們的訂閱，而你希望恢復訂閱，你可以在訂閱上呼叫 `resume` 方法。 客戶必須仍在「寬限期」內才能恢復訂閱：

    $user->subscription('default')->resume();

如果客戶取消訂閱，然後在訂閱完全到期之前恢復訂閱，則不會立即向客戶收費。 相反，他們的訂閱將被重新啟動，並且他們將按照原始計費週期進行計費。

## 訂閱試用

### 預先支付方式

如果你想為客戶提供試用期，同時仍然預先收集付款方式資訊，則應在建立訂閱時使用 `trialDays` 方法：

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default', 'price_monthly')
                    ->trialDays(10)
                    ->create($request->paymentMethodId);

        // ...
    });

此方法將在資料庫中的訂閱記錄上設定試用期結束日期，並指示 Stripe 在該日期之前不要開始向客戶收費。 使用 `trialDays` 方法時，Cashier 將覆蓋為 Stripe 中的價格組態的任何默認試用期。

> **注意**  
> 如果客戶的訂閱在試用結束日期之前沒有取消，他們將在試用期滿後立即收費，因此你應該確保通知你的使用者他們的試用結束日期。

`trialUntil` 方法允許你提供一個 `DateTime` 實例，指定試用期何時結束：

    use Carbon\Carbon;

    $user->newSubscription('default', 'price_monthly')
                ->trialUntil(Carbon::now()->addDays(10))
                ->create($paymentMethod);

你可以使用使用者實例的 `onTrial` 方法或訂閱實例的 `onTrial` 方法來確定使用者是否在試用期內。 下面的兩個例子是等價的：

    if ($user->onTrial('default')) {
        // ...
    }

    if ($user->subscription('default')->onTrial()) {
        // ...
    }

你可以使用 `endTrial` 方法立即結束訂閱試用：

    $user->subscription('default')->endTrial();

要確定現有試用版是否已過期，你可以使用 `hasExpiredTrial` 方法：

    if ($user->hasExpiredTrial('default')) {
        // ...
    }

    if ($user->subscription('default')->hasExpiredTrial()) {
        // ...
    }

#### 在 Stripe / Cashier 中定義試用日

你可以選擇在 Stripe 儀表板中定義你的價格接收的試用天數，或者始終使用 Cashier 明確傳遞它們。 如果你選擇在 Stripe 中定義價格的試用日，你應該知道新訂閱，包括過去有訂閱的客戶的新訂閱，將始終收到試用期，除非你明確呼叫 `skipTrial()` 方法 。

### 沒有預先付款方式

如果你想在不預先收集使用者付款方式資訊的情況下提供試用期，你可以將使用者記錄中的 `trial_ends_at` 列設定為你想要的試用結束日期。 這通常在使用者註冊期間完成：

    use App\Models\User;

    $user = User::create([
        // ...
        'trial_ends_at' => now()->addDays(10),
    ]);

> **注意**  
> 請務必在計費模型的類定義中為 `trial_ends_at` 屬性新增 [date cast](/docs/laravel/10.x/eloquent-mutators#date-casting)。

Cashier 將這種類型的試用稱為「通用試用」，因為它不附加到任何現有訂閱。 如果當前日期沒有超過 `trial_ends_at` 的值，計費模型實例上的 `onTrial` 方法將返回 `true`：

    if ($user->onTrial()) {
        // 使用者在試用期內…
    }

一旦你準備好為使用者建立一個實際的訂閱，你可以像往常一樣使用 `newSubscription` 方法：

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')->create($paymentMethod);

要檢索使用者的試用結束日期，你可以使用 `trialEndsAt` 方法。 如果使用者正在試用，此方法將返回一個 Carbon 日期實例，否則將返回 `null`。 如果你想獲取特定訂閱而非默認訂閱的試用結束日期，你還可以傳遞一個可選的訂閱名稱參數：

    if ($user->onTrial()) {
        $trialEndsAt = $user->trialEndsAt('main');
    }

如果你希望明確知道使用者處於他們的「通用」試用期並且尚未建立實際訂閱，你也可以使用 `onGenericTrial` 方法：

    if ($user->onGenericTrial()) {
        // 使用者正處於「通用」試用期內…
    }

### 延長試驗

`extendTrial` 方法允許你在建立訂閱後延長訂閱的試用期。 如果試用期已經過期並且已經向客戶收取訂閱費用，你仍然可以為他們提供延長試用期。 在試用期內花費的時間將從客戶的下一張發票中扣除：

    use App\Models\User;

    $subscription = User::find(1)->subscription('default');

    // 從現在起 7 天結束試用...
    $subscription->extendTrial(
        now()->addDays(7)
    );

    // 試用期再延長 5 天...
    $subscription->extendTrial(
        $subscription->trial_ends_at->addDays(5)
    );

## 處理 Stripe Webhook

> **技巧**  
> 你可以使用 [Stripe CLI](https://stripe.com/docs/stripe-cli) 在本地開發期間幫助測試 webhook。

Stripe 可以通過 webhook 通知你的應用程式各種事件。 默認情況下，指向 Cashier 的 webhook  controller 的路由由 Cashier 服務提供商自動註冊。 該 controller 將處理所有傳入的 webhook 請求。

默認情況下，Cashier webhook  controller 將自動處理取消訂閱失敗次數過多（由你的 Stripe 設定定義）、客戶更新、客戶刪除、訂閱更新和付款方式更改； 然而，我們很快就會發現，你可以擴展這個 controller 來處理你喜歡的任何 Stripe webhook 事件。

為確保你的應用程式可以處理 Stripe webhook，請務必在 Stripe 控制面板中組態 webhook URL。 默認情況下，Cashier 的 webhook  controller 響應 `/stripe/webhook` URL 路徑。 你應該在 Stripe 控制面板中啟用的所有 webhooks 的完整列表是：

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `invoice.payment_succeeded`
- `invoice.payment_action_required`

為方便起見，Cashier 包含一個 `cashier:webhook` Artisan 命令。 此命令將在 Stripe 中建立一個 webhook，用於偵聽 Cashier 所需的所有事件：

```shell
php artisan cashier:webhook
```

默認情況下，建立的 webhook 將指向由 `APP_URL` 環境變數和 Cashier 包含的 `cashier.webhook` 路由定義的 URL。 如果你想使用不同的 URL，你可以在呼叫命令時提供 `--url` 選項：

```shell
php artisan cashier:webhook --url "https://example.com/stripe/webhook"
```

建立的 webhook 將使用與你的 Cashier 版本相容的 Stripe API 版本。 如果你想使用不同的 Stripe 版本，你可以提供 `--api-version` 選項：

```shell
php artisan cashier:webhook --api-version="2019-12-03"
```

建立後，webhook 將立即啟動。 如果你想建立 webhook 但在你準備好之前將其停用，你可以在呼叫命令時提供 `--disabled` 選項：

```shell
php artisan cashier:webhook --disabled
```

> **注意**  
> 確保使用 Cashier 包含的 [webhook 簽名驗證](#verifying-webhook-signatures)中介軟體保護傳入的 Stripe webhook 請求。

#### Webhook 和 CSRF 保護

由於 Stripe webhooks 需要繞過 Laravel 的 [CSRF 保護](/docs/laravel/10.x/csrf)，請務必在應用程式的 `App\Http\Middleware\VerifyCsrfToken` 中介軟體中將 URI 列為異常或列出路由 在 `web` 中介軟體組之外：

    protected $except = [
        'stripe/*',
    ];

### 定義 Webhook 事件處理程序

Cashier 自動處理失敗收費和其他常見 Stripe webhook 事件的訂閱取消。 但是，如果你有其他想要處理的 webhook 事件，你可以通過收聽以下由 Cashier 調度的事件來實現：

- `Laravel\Cashier\Events\WebhookReceived`
- `Laravel\Cashier\Events\WebhookHandled`

這兩個事件都包含 Stripe webhook 的完整負載。 例如，如果你希望處理 `invoice.payment_succeeded` webhook，你可以註冊一個 [listener](/docs/laravel/10.x/events#defining-listeners) 來處理該事件：

    <?php

    namespace App\Listeners;

    use Laravel\Cashier\Events\WebhookReceived;

    class StripeEventListener
    {
        /**
         * 處理收到的 Stripe webhooks。
         */
        public function handle(WebhookReceived $event): void
        {
            if ($event->payload['type'] === 'invoice.payment_succeeded') {
                // 處理傳入事件...
            }
        }
    }

定義監聽器後，你可以在應用程式的 `EventServiceProvider` 中註冊它：

    <?php

    namespace App\Providers;

    use App\Listeners\StripeEventListener;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
    use Laravel\Cashier\Events\WebhookReceived;

    class EventServiceProvider extends ServiceProvider
    {
        protected $listen = [
            WebhookReceived::class => [
                StripeEventListener::class,
            ],
        ];
    }

### 驗證 Webhook 簽名

為了保護你的 webhook，你可以使用 [Stripe 的 webhook 簽名](https://stripe.com/docs/webhooks/signatures)。為方便起見，Cashier 自動包含一個中介軟體，用於驗證傳入的 Stripe webhook 請求是否有效。

要啟用 webhook 驗證，請確保在應用程式的 `.env` 檔案中設定了 `STRIPE_WEBHOOK_SECRET` 環境變數。 Webhook `secret` 可以從你的 Stripe 帳戶儀表板中檢索。

## 單次收費

### 簡單收費

如果你想對客戶進行一次性收費，你可以在計費模型實例上使用 charge 方法。 你需要[提供支付方式識別碼](#payment-methods-for-single-charges)作為 `charge` 方法的第二個參數：

    use Illuminate\Http\Request;

    Route::post('/purchase', function (Request $request) {
        $stripeCharge = $request->user()->charge(
            100, $request->paymentMethodId
        );

        // ...
    });

`charge` 方法接受一個陣列作為它的第三個參數，允許你將你希望的任何選項傳遞給底層的 Stripe 費用建立。 有關建立費用時可用選項的更多資訊，請參見 [Stripe 文件](https://stripe.com/docs/api/charges/create)：

    $user->charge(100, $paymentMethod, [
        'custom_option' => $value,
    ]);

你也可以在沒有基礎客戶或使用者的情況下使用 `charge` 方法。 為此，請在應用程式的計費模型的新實例上呼叫 `charge` 方法：

    use App\Models\User;

    $stripeCharge = (new User)->charge(100, $paymentMethod);

如果收費失敗， `charge` 方法將拋出異常。 如果收費成功，`Laravel\Cashier\Payment` 的實例將從該方法返回：

    try {
        $payment = $user->charge(100, $paymentMethod);
    } catch (Exception $e) {
        // ...
    }

> **注意**  
> `charge` 方法接受以你的應用程式使用的貨幣的最低分母表示的付款金額。 例如，如果客戶以美元付款，則應以美分指定金額。

### 用發票收費

有時你可能需要一次性收費並向客戶提供 PDF 收據。 `invoicePrice` 方法可以讓你做到這一點。例如，讓我們為客戶開具5件新襯衫的發票：

    $user->invoicePrice('price_tshirt', 5);

發票將立即根據使用者的默認付款方式收取。`invoicePrice` 方法也接受一個陣列作為它的第三個參數。 此陣列包含發票項目的計費選項。該方法接受的第四個參數也是一個陣列，其中應包含發票本身的計費選項：

    $user->invoicePrice('price_tshirt', 5, [
        'discounts' => [
            ['coupon' => 'SUMMER21SALE']
        ],
    ], [
        'default_tax_rates' => ['txr_id'],
    ]);

與 `invoicePrice` 類似，你可以使用 `tabPrice` 方法為多個項目（每張發票最多250個項目）建立一次性收費，將它們新增到客戶的「標籤」，然後向客戶開具發票。例如，我們可以為客戶開具5件襯衫和2個杯子的發票：

    $user->tabPrice('price_tshirt', 5);
    $user->tabPrice('price_mug', 2);
    $user->invoice();

或者，你可以使用 `invoiceFor` 方法對客戶的默認付款方式進行「一次性」收費：

    $user->invoiceFor('One Time Fee', 500);

雖然 `invoiceFor` 方法可供你使用，但建議你使用具有預定義價格的 `invoicePrice` 和 `tabPrice` 方法。通過這樣做，你可以在 Stripe 儀表板中獲得更好的分析和資料，以瞭解你在每個產品的基礎上的銷售情況。

> **注意**  
> `invoice`、`invoicePrice` 和 `invoiceFor` 方法將建立一個 Stripe 發票，失敗的發票會繼續嘗試扣費。如果你不希望失敗的發票繼續嘗試扣費，則需要在第一次扣費失敗後使用 Stripe API 關閉它們。

### 建立付款意向

你可以通過在計費模型實例上呼叫 `pay` 方法來建立新的 Stripe 支付意圖。 呼叫此方法將建立一個包裝在 `Laravel\Cashier\Payment` 實例中的支付意圖：

    use Illuminate\Http\Request;

    Route::post('/pay', function (Request $request) {
        $payment = $request->user()->pay(
            $request->get('amount')
        );

        return $payment->client_secret;
    });

建立支付意圖後，你可以將客戶端密碼返回到應用程式的前端，以便使用者可以在其瀏覽器中完成支付。 要閱讀有關使用 Stripe 支付意圖建構整個支付流程的更多資訊，請參閱 [Stripe 文件](https://stripe.com/docs/payments/accept-a-payment?platform=web)。

使用 `pay` 方法時，你的 Stripe 控制面板中啟用的默認付款方式將可供客戶使用。 或者，如果你只想允許使用某些特定的支付方式，你可以使用 `payWith` 方法：

    use Illuminate\Http\Request;

    Route::post('/pay', function (Request $request) {
        $payment = $request->user()->payWith(
            $request->get('amount'), ['card', 'bancontact']
        );

        return $payment->client_secret;
    });

> **注意**  
> `pay` 和 `payWith` 方法接受以你的應用程式使用的貨幣的最低分母表示的付款金額。 例如，如果客戶以美元付款，則應以美分指定金額。

### 退還費用

如果你需要退還 Stripe 費用，你可以使用 refund 方法。 此方法接受 Stripe [payment intent ID](#payment-methods-for-single-charges) 作為其第一個參數：

    $payment = $user->charge(100, $paymentMethodId);

    $user->refund($payment->id);

## 發票

### 檢索發票

你可以使用 `invoices` 方法輕鬆檢索可計費模型的發票陣列。 `invoices` 方法返回 `Laravel\Cashier\Invoice` 實例的集合：

    $invoices = $user->invoices();

如果你想在結果中包含待處理的發票，你可以使用 `invoicesIncludingPending` 方法：

    $invoices = $user->invoicesIncludingPending();

你可以使用 `findInvoice` 方法通過其 ID 檢索特定發票：

    $invoice = $user->findInvoice($invoiceId);

#### 顯示發票資訊

在為客戶列出發票時，你可以使用發票的方法顯示相關的發票資訊。 例如，你可能希望在表格中列出每張發票，以便使用者輕鬆下載其中任何一張：

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>

### 即將收到的發票

要檢索客戶即將收到的發票，你可以使用 `upcomingInvoice` 方法：

    $invoice = $user->upcomingInvoice();

類似地，如果客戶有多個訂閱，你還可以檢索特定訂閱的即將到來的發票：

    $invoice = $user->subscription('default')->upcomingInvoice();

### 預覽訂閱發票

使用 `previewInvoice` 方法，你可以在更改價格之前預覽發票。 這將允許你確定在進行給定價格更改時客戶發票的外觀：

    $invoice = $user->subscription('default')->previewInvoice('price_yearly');

你可以將一組價格傳遞給 `previewInvoice` 方法，以便預覽具有多個新價格的發票：

    $invoice = $user->subscription('default')->previewInvoice(['price_yearly', 'price_metered']);

### 生成發票 PDF

在生成發票 PDF 之前，你應該用 Composer 安裝 Dompdf 庫，它是 Cashier 的默認發票渲染器：

```php
composer require dompdf/dompdf
```

在路由或 controller 中，你可以使用 `downloadInvoice` 方法生成給定發票的 PDF 下載。此方法將自動生成下載發票所需的正確 HTTP 響應：

    use Illuminate\Http\Request;

    Route::get('/user/invoice/{invoice}', function (Request $request, string $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId);
    });

默認情況下，發票上的所有資料都來自儲存在 Stripe 中的客戶和發票資料。檔案名稱是基於你的 `app.name` 組態值。但是，你可以通過提供一個陣列作為 `downloadInvoice` 方法的第二個參數來自訂其中的一些資料。 此陣列允許你自訂資訊，例如你的公司和產品詳細資訊：

    return $request->user()->downloadInvoice($invoiceId, [
        'vendor' => 'Your Company',
        'product' => 'Your Product',
        'street' => 'Main Str. 1',
        'location' => '2000 Antwerp, Belgium',
        'phone' => '+32 499 00 00 00',
        'email' => 'info@example.com',
        'url' => 'https://example.com',
        'vendorVat' => 'BE123456789',
    ]);

`downloadInvoice` 方法還允許通過其第三個參數自訂檔案名稱。此檔案名稱將自動以 `.pdf` 為後綴：

    return $request->user()->downloadInvoice($invoiceId, [], 'my-invoice');

#### 自訂發票渲染器

Cashier 還可以使用自訂發票渲染器。 默認情況下，Cashier 使用 `DompdfInvoiceRenderer` 實現，它利用 [dompdf](https://github.com/dompdf/dompdf) PHP 庫來生成 Cashier 的發票。但是，你可以通過實現 `Laravel\Cashier\Contracts\InvoiceRenderer` 介面來使用任何你想要的渲染器。 例如，你可能希望使用對第三方 PDF 呈現服務的 API 呼叫來呈現發票 PDF：

    use Illuminate\Support\Facades\Http;
    use Laravel\Cashier\Contracts\InvoiceRenderer;
    use Laravel\Cashier\Invoice;

    class ApiInvoiceRenderer implements InvoiceRenderer
    {
        /**
         * 呈現給定的發票並返回原始 PDF 位元組。
         */
        public function render(Invoice $invoice, array $data = [], array $options = []): string
        {
            $html = $invoice->view($data)->render();

            return Http::get('https://example.com/html-to-pdf', ['html' => $html])->get()->body();
        }
    }

一旦你實現了發票渲染器合約，你應該在你的應用程式的 `config/cashier.php` 組態檔案中更新 `cashier.invoices.renderer` 組態值。 此組態值應設定為自訂渲染器實現的類名。

## 結賬

Cashier Stripe 還提供對 [Stripe Checkout](https://stripe.com/payments/checkout) 的支援。 Stripe Checkout 通過提供預建構的託管支付頁面，消除了實施自訂頁面以接受付款的痛苦。

以下文件包含有關如何開始使用 Stripe Checkout with Cashier 的資訊。 要瞭解有關 Stripe Checkout 的更多資訊，你還應該考慮查看 [Stripe 自己的 Checkout 文件](https://stripe.com/docs/payments/checkout) 。

### 產品結賬

你可以在計費模型上使用 `checkout` 方法對已在 Stripe 儀表板中建立的現有產品執行結帳。 `checkout` 方法將啟動一個新的 Stripe Checkout  session 。 默認情況下，你需要傳遞 Stripe Price ID：

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout('price_tshirt');
    });

如果需要，你還可以指定產品數量：

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 15]);
    });

當客戶訪問此路由時，他們將被重新導向到 Stripe 的結帳頁面。 默認情況下，當使用者成功完成或取消購買時，他們將被重新導向到你的 `home` 路由位置，但你可以使用 `success_url` 和 `cancel_url` 參數指定自訂回呼 URL：

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 1], [
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
    });

在定義 `success_url` 結帳選項時，你可以指示 Stripe 在呼叫 URL 時將結帳 session  ID 新增為查詢字串參數。為此，請將文字字串 `{CHECKOUT_SESSION_ID}` 新增到你的 `success_url` 查詢字串。Stripe 將用實際的結帳 session  ID 替換此預留位置：

    use Illuminate\Http\Request;
    use Stripe\Checkout\Session;
    use Stripe\Customer;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 1], [
            'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
            'cancel_url' => route('checkout-cancel'),
        ]);
    });

    Route::get('/checkout-success', function (Request $request) {
        $checkoutSession = $request->user()->stripe()->checkout->sessions->retrieve($request->get('session_id'));

        return view('checkout.success', ['checkoutSession' => $checkoutSession]);
    })->name('checkout-success');

#### 優惠碼

默認情況下，Stripe Checkout 不允許[使用者可兌換促銷程式碼](https://stripe.com/docs/billing/subscriptions/discounts/codes) 。幸運的是，有一種簡單的方法可以為你的結帳頁面啟用這些功能。為此，你可以呼叫 `allowPromotionCodes` 方法：

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()
            ->allowPromotionCodes()
            ->checkout('price_tshirt');
    });

### 單次收費結賬

你還可以對尚未在 Stripe 儀表板中建立的臨時產品進行簡單收費。為此，你可以在計費模型上使用 `checkoutCharge` 方法，並向其傳遞可計費金額、產品名稱和可選數量。當客戶訪問此路由時，他們將被重新導向到 Stripe 的結帳頁面：

    use Illuminate\Http\Request;

    Route::get('/charge-checkout', function (Request $request) {
        return $request->user()->checkoutCharge(1200, 'T-Shirt', 5);
    });

> **注意**  
> 當使用 `checkoutCharge` 方法時，Stripe 將始終在你的 Stripe 儀表板中建立新產品和價格。因此，我們建議你在 Stripe 儀表板中預先建立產品，並改用 `checkout` 方法。

### 訂閱結帳

> **注意**  
> 使用 Stripe Checkout 進行訂閱需要你在 Stripe 儀表板中啟用 `customer.subscription.created` webhook。 此 webhook 將在你的資料庫中建立訂閱記錄並儲存所有相關的訂閱項。

你也可以使用 Stripe Checkout 來啟動訂閱。 在使用 Cashier 的訂閱建構器方法定義你的訂閱後，你可以呼叫 `checkout` 方法。 當客戶訪問此路由時，他們將被重新導向到 Stripe 的結帳頁面：

    use Illuminate\Http\Request;

    Route::get('/subscription-checkout', function (Request $request) {
        return $request->user()
            ->newSubscription('default', 'price_monthly')
            ->checkout();
    });

與產品結帳一樣，你可以自訂成功和取消 URL：

    use Illuminate\Http\Request;

    Route::get('/subscription-checkout', function (Request $request) {
        return $request->user()
            ->newSubscription('default', 'price_monthly')
            ->checkout([
                'success_url' => route('your-success-route'),
                'cancel_url' => route('your-cancel-route'),
            ]);
    });

當然，你也可以為訂閱結帳啟用優惠碼：

    use Illuminate\Http\Request;

    Route::get('/subscription-checkout', function (Request $request) {
        return $request->user()
            ->newSubscription('default', 'price_monthly')
            ->allowPromotionCodes()
            ->checkout();
    });

> **注意**  
> 不幸的是，在開始訂閱時，Stripe Checkout 不支援所有訂閱計費選項。在訂閱生成器上使用 `anchorBillingCycleOn` 方法、設定按比例分配行為或設定支付行為在 Stripe Checkout  session 期間不會有任何影響。請查閱 [Stripe Checkout Session API 文件](https://stripe.com/docs/api/checkout/sessions/create)以查看可用的參數。

#### Stripe Checkout 和試用期

當然，你可以在建構將使用 Stripe Checkout 完成的訂閱時定義一個試用期：

    $checkout = Auth::user()->newSubscription('default', 'price_monthly')
        ->trialDays(3)
        ->checkout();

但是，試用期必須至少為 48 小時，這是 Stripe Checkout 支援的最短試用時間。

#### 訂閱和 Webhooks

請記住，Stripe 和 Cashier 通過 webhook 更新訂閱狀態，因此當客戶在輸入付款資訊後返回應用程式時，訂閱可能尚未啟動。要處理這種情況，你可能希望顯示一條消息，通知使用者他們的付款或訂閱處於待處理狀態。

### 收集稅號 ID

Checkout 還支援收集客戶的稅號。要在結帳 session 上啟用此功能，請在建立 session 時呼叫 `collectTaxIds` 方法：

    $checkout = $user->collectTaxIds()->checkout('price_tshirt');

呼叫此方法時，客戶會顯示一個新的複選框，允許他們選擇是否作為公司進行採購。如果選擇是，他們可以提供他們的稅號。

> **注意**  
> 如果你已經在應用程式的服務提供者中組態了 [自動徵稅](#tax-configuration) ，那麼該功能將自動啟用，無需呼叫 `collectTaxIds` 方法。

### 訪客結賬

使用 `Checkout::guest` 方法，你可以為你的應用程式中沒有註冊過「帳戶」的訪客啟動結賬 session ：

    use Illuminate\Http\Request;
    use Laravel\Cashier\Checkout;

    Route::get('/product-checkout', function (Request $request) {
        return Checkout::guest()->create('price_tshirt', [
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
    });

與為現有使用者建立結賬 session 類似，你可以利用 `Laravel\Cashier\CheckoutBuilder` 實例上的其他方法來定製訪客結賬 session ：

    use Illuminate\Http\Request;
    use Laravel\Cashier\Checkout;

    Route::get('/product-checkout', function (Request $request) {
        return Checkout::guest()
            ->withPromotionCode('promo-code')
            ->create('price_tshirt', [
                'success_url' => route('your-success-route'),
                'cancel_url' => route('your-cancel-route'),
            ]);
    });

在訪客結賬完成後，Stripe 會傳送一個 `checkout.session.completed` 的 webhook 事件，所以請確保[組態你的 Stripe webhook](https://dashboard.stripe.com/webhooks) 以實際傳送這個事件到你的應用程式。一旦 webhook 在 Stripe 儀表板中被啟用，你就可以[用 Cashier 處理 webhook ](#handling-stripe-webhooks) 。webhook 負載中包含的對象將是一個[`checkout` 對象](https://stripe.com/docs/api/checkout/sessions/object) ，你可以檢查它以完成你的客戶的訂單。

## 處理失敗的付款

有時，訂閱或單筆費用的付款可能會失敗。當這種情況發生時，Cashier 會拋出一個 `Laravel\Cashier\Exceptions\IncompletePayment` 異常，通知你發生了這種情況。捕獲此異常後，你有兩個選擇如何繼續。

首先，你可以將你的客戶重新導向到 Cashier 附帶的專用付款確認頁面。該頁面已經有一個通過 Cashier 的服務提供商註冊的關聯命名路由。因此，你可能會捕獲 `IncompletePayment` 異常並將使用者重新導向到付款確認頁面：

    use Laravel\Cashier\Exceptions\IncompletePayment;

    try {
        $subscription = $user->newSubscription('default', 'price_monthly')
                                ->create($paymentMethod);
    } catch (IncompletePayment $exception) {
        return redirect()->route(
            'cashier.payment',
            [$exception->payment->id, 'redirect' => route('home')]
        );
    }

在付款確認頁面上，將提示客戶再次輸入他們的信用卡資訊並執行 Stripe 要求的任何其他操作，例如 「3D Secure」確認。 確認付款後，使用者將被重新導向到上面指定的 `redirect` 參數提供的 URL 。重新導向後， `message` （字串）和 `success` （整數）查詢字串變數將被新增到 URL 。支付頁面目前支援以下支付方式類型：

- Credit Cards
- Alipay
- Bancontact
- BECS Direct Debit
- EPS
- Giropay
- iDEAL
- SEPA Direct Debit

或者，你可以讓 Stripe 為你處理付款確認。 在這種情況下，你可以在 Stripe 控制面板中[設定 Stripe 的自動計費電子郵件](https://dashboard.stripe.com/account/billing/automatic) ，而不是重新導向到付款確認頁面。 但是，如果捕獲到 `IncompletePayment` 異常，你仍應通知使用者他們將收到一封包含進一步付款確認說明的電子郵件。

以下方法可能會拋出支付異常：使用 `Billable` 特性的模型上的 `charge` 、 `invoiceFor` 和 `invoice`。 與訂閱互動時，`SubscriptionBuilder` 上的`create` 方法以及`Subscription` 和`SubscriptionItem` 模型上的`incrementAndInvoice` 和`swapAndInvoice` 方法可能會拋出未完成支付異常。

可以使用計費模型或訂閱實例上的 `hasIncompletePayment` 方法來確定現有訂閱是否有未完成的付款：

    if ($user->hasIncompletePayment('default')) {
        // ...
    }

    if ($user->subscription('default')->hasIncompletePayment()) {
        // ...
    }

你可以通過檢查異常實例上的 `payment` 屬性來獲取未完成付款的具體狀態：

    use Laravel\Cashier\Exceptions\IncompletePayment;

    try {
        $user->charge(1000, 'pm_card_threeDSecure2Required');
    } catch (IncompletePayment $exception) {
        // 獲取支付意目標狀態...
        $exception->payment->status;

        // 檢查具體條件...
        if ($exception->payment->requiresPaymentMethod()) {
            // ...
        } elseif ($exception->payment->requiresConfirmation()) {
            // ...
        }
    }

## 強大的客戶認證

如果你的企業或你的客戶之一位於歐洲，你將需要遵守歐盟的強客戶認證 (SCA) 法規。 歐盟於 2019 年 9 月實施了這些規定，以防止支付欺詐。 幸運的是，Stripe 和 Cashier 已準備好建構符合 SCA 的應用程式。

> **注意**
> 在開始之前，請查看 [Stripe 關於 PSD2 和 SCA 的指南](https://stripe.com/guides/strong-customer-authentication)以及他們的[關於新 SCA API 的文件](https://stripe.com/docs/strong-customer-authentication).

### 需要額外確認的付款

SCA 法規通常需要額外驗證以確認和處理付款。 發生這種情況時，Cashier 將拋出一個 `Laravel\Cashier\Exceptions\IncompletePayment` 異常，通知你需要額外的驗證。 有關如何處理這些異常的更多資訊，請參閱有關的 [handling failed payments](#handling-failed-payments) 文件。

Stripe 或 Cashier 顯示的支付確認螢幕可能會根據特定銀行或發卡機構的支付流程進行定製，並且可能包括額外的銀行卡確認、臨時小額收費、單獨的裝置身份驗證或其他形式的驗證。

#### 未完成和逾期狀態

當付款需要額外確認時，訂閱將保持在 `incomplete` 或 `past_due` 狀態，如其  `stripe_status` 資料庫列所示。 付款確認完成後，Cashier 將自動啟動客戶的訂閱，並且 Stripe 通過 webhook 通知你的應用程式已完成。

有關 `incomplete` 和 `past_due` 狀態的更多資訊，請參閱[我們關於這些狀態的附加文件](#incomplete-and-past-due-status)。

### 非 session 付款通知

由於 SCA 法規要求客戶偶爾驗證他們的付款細節，即使他們的訂閱處於活動狀態，Cashier 也可以在需要非 session 付款確認時向客戶傳送通知。 例如，這可能在訂閱續訂時發生。 可以通過將 `CASHIER_PAYMENT_NOTIFICATION` 環境變數設定為通知類來啟用收銀員的付款通知。 默認情況下，此通知處於停用狀態。 當然，Cashier 包含一個你可以用於此目的的通知類，但如果需要，你可以自由提供自己的通知類：

```ini
CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment
```

為確保傳送 session 外付款確認通知，請驗證你的應用程式的 [Stripe webhooks 已組態](#handling-stripe-webhooks)並且在你的 Stripe 儀表板中啟用了 `invoice.payment_action_required` 。 此外，你的 `Billable` 模型還應該使用 Laravel 的 `Illuminate\Notifications\Notifiable` 特性。

> **注意**
> 即使客戶手動進行需要額外確認的付款，也會傳送通知。 不幸的是，Stripe 無法知道付款是手動完成的還是「離線」完成的。 但是，如果客戶在確認付款後訪問付款頁面，他們只會看到「付款成功」消息。 不允許客戶不小心確認兩次相同的付款而招致意外的二次扣款。

## Stripe SDK

Cashier 的許多對象都是 Stripe SDK 對象的包裝器。 如果你想直接與 Stripe 對象互動，你可以使用 `asStripe` 方法方便地搜尋它們：:

    $stripeSubscription = $subscription->asStripeSubscription();

    $stripeSubscription->application_fee_percent = 5;

    $stripeSubscription->save();

你還可以使用 `updateStripeSubscription` 方法直接更新 Stripe 訂閱：

    $subscription->updateStripeSubscription(['application_fee_percent' => 5]);

如果你想直接使用 `Stripe\StripeClient` 客戶端，你可以呼叫 `Cashier` 類的 `stripe` 方法。 例如，你可以使用此方法訪問「StripeClient」實例並從你的 Stripe 帳戶中搜尋價格列表：

    use Laravel\Cashier\Cashier;

    $prices = Cashier::stripe()->prices->all();

## 測試

在測試使用 Cashier 的應用程式時，你可以模擬對 Stripe API 的實際 HTTP 請求； 但是，這需要你重新實現部分地 Cashier 自己的行為。 因此，我們建議讓你的測試命中實際的 Stripe API。 雖然速度較慢，但它可以讓你更加確信你的應用程式正在按預期工作，並且任何緩慢的測試都可以放在他們自己的 PHPUnit 測試組中。

測試時，請記住 Cashier 本身已經有一個很棒的測試套件，因此你應該只專注於測試自己的應用程式的訂閱和支付流程，而不是每個底層的 Cashier 行為。

首先，將 Stripe 金鑰的 **testing** 版本新增到你的 `phpunit.xml` 檔案中：

    <env name="STRIPE_SECRET" value="sk_test_<your-key>"/>

現在，每當你在測試時與 Cashier 互動時，它都會向你的 Stripe 測試環境傳送實際的 API 請求。為方便起見，你應該使用可能在測試期間使用的訂閱 / 計畫預先填寫你的 Stripe 測試帳戶。

> **技巧**  
> 為了測試各種計費場景，例如信用卡拒付和失敗，你可以使用 Stripe 提供的大量的[測試卡號和令牌](https://stripe.com/docs/testing) 。

