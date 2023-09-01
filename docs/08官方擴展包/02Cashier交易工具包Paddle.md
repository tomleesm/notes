# Cashier 交易工具包 (Paddle)

## 介紹

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle) 為 [Paddle's](https://paddle.com) 訂閱計費服務提供了一個富有表現力、流暢的介面。它幾乎能夠處理所有你所恐懼的各種訂閱計費邏輯和程式碼。除了基本的訂閱管理，Cashier 還可以處理：優惠券、交換訂閱、訂閱「數量」、取消寬限期等。

在使用 Cashier 時，推薦你回顧一下 Paddle 的[使用者手冊](https://developer.paddle.com/guides) and [API 文件](https://developer.paddle.com/api-reference/intro)。


## 升級 Cashier

當升級到一個新版本的 Cashier 時，推薦仔細回顧下 [升級指南](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.) 這非常重要。

## 安裝

首先，使用 Composer 包管理器安裝 Paddle 的 Cashier 包：

```shell
composer require laravel/cashier-paddle
```

> 注意：為了確保 Cashier 正確處理所有 Paddle 事件，請記得 [組態 Cashier 的 webhook 處理](#handling-paddle-webhooks)。

### Paddle 沙盒

在本地和預發佈開發環境中，應該 [註冊一個 Paddle 沙盒帳號](https://developer.paddle.com/getting-started/sandbox)。這個帳號將為你提供一個沙盒環境來測試和開發你的應用，而不會產生真實的交易。你也許會使用 Paddle 的 [測試卡號](https://developer.paddle.com/getting-started/sandbox#test-cards) 來模擬各種交易場景。

在使用 Pable 沙盒環境時，你應在應用程式的 `.env` 環境檔案中將 `PADDLE_SANDBOX` 環境變數設定為 `true` ：

```ini
PADDLE_SANDBOX=true
```

在你已經完成你的應用開發之後，你也許會 [申請一個 Paddle 正式帳號](https://paddle.com/) 。 在你的應用程式投入生產環境之前，Paddle 需要批准你的應用程式的域。

### 資料遷移

Cashier 服務提供者註冊它自己的資料遷移目錄，所以你記得在安裝擴展包之後執行資料遷移。Cashier 資料遷移將生成新的 `customers` 表。另外，新的 `subscriptions` 表將被建立，來儲存所有你的使用者的訂閱。最後，新的 `receipts` 表也將被建立，來儲存所有你的收據資訊:

```shell
php artisan migrate
```

如果你需要重寫 Cashier 中的資料遷移，你可以使用 `vendor:publish` Artisan 命令來發佈它們：

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

如果你想阻止 Cashier 的資料遷移全部執行，你可以使用 Cashier 提供的 `ignoreMigrations`。通常，這個方法會在 `AppServiceProvider` 的 `register` 方法中被呼叫：

    use Laravel\Paddle\Cashier;

    /**
     * 註冊服務。
     */
    public function register(): void
    {
        Cashier::ignoreMigrations();
    }

## 組態

### Billable 模型

在使用 Cashier 之前，你必須將 `Billable` trait 新增到你的使用者模型定義中。 這裡的 trait 提供了多種方法來允許你執行常見的計費任務，例如建立訂閱、應用優惠券和更新付款方式資訊：

    use Laravel\Paddle\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

如果你有非使用者的計費實體，你還可以將特徵新增到這些類中：

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Paddle\Billable;

    class Team extends Model
    {
        use Billable;
    }

### API Keys

接下來，你應該在應用程式的 `.env` 檔案中組態你的 Paddle 。 你可以從 Paddle 控制面板檢索你的 Paddle API 金鑰：

```ini
PADDLE_VENDOR_ID=your-paddle-vendor-id
PADDLE_VENDOR_AUTH_CODE=your-paddle-vendor-auth-code
PADDLE_PUBLIC_KEY="your-paddle-public-key"
PADDLE_SANDBOX=true
```

當你使用 [Paddle 的沙箱環境](#paddle-sandbox) 時，`PADDLE_SANDBOX` 環境變數應該設定為 `true`。如果你將應用程式部署到生產環境並使用 Paddle 的即時供應商環境，則 `PADDLE_SANDBOX` 變數應該設定為 `false`。


### Paddle JS

Paddle 依賴其自己的 JavaScript 庫來啟動 Paddle 結賬小部件。你可以通過在應用程式佈局中的 `</head>` 標籤關閉之前放置 `@paddleJS` Blade 指令來載入 JavaScript 庫：

```blade
<head>
    ...

    @paddleJS
</head>
```

### 貨幣組態

默認 Cashier 貨幣是美元（USD）。你可以在 `.env` 檔案中定義 `CASHIER_CURRENCY` 環境變數來更改默認貨幣：

```ini
CASHIER_CURRENCY=EUR
```

除了組態 Cashier 的貨幣之外，你還可以指定在格式化貨幣值以顯示在發票上時要使用的區域。Cashier 內部利用 [PHP 的 NumberFormatter 類](https://www.php.net/manual/en/class.numberformatter.php)來設定貨幣區域：

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> 注意：為了使用 `en` 以外的語言環境，請確保你的伺服器上安裝並組態了 `ext-intl` PHP 擴展。

### 覆蓋默認模型

你可以通過定義自己的模型並繼承相應的 Cashier 模型來自由擴展 Cashier 模型：

    use Laravel\Paddle\Subscription as CashierSubscription;

    class Subscription extends CashierSubscription
    {
        // ...
    }

定義模型後，你可以通過 `Laravel\Paddle\Cashier` 類指示 Cashier 使用你的自訂模型。通常，你應該在應用的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中通知 Cashier 關於你的自訂模型：

    use App\Models\Cashier\Receipt;
    use App\Models\Cashier\Subscription;

    /**
     * 啟動應用服務。
     */
    public function boot(): void
    {
        Cashier::useReceiptModel(Receipt::class);
        Cashier::useSubscriptionModel(Subscription::class);
    }



## 核心概念

### 支付連結

Paddle 缺乏廣泛的 CRUD API 來執行訂閱狀態更改。因此，與 Paddle 的大多數互動都是通過其 [結帳小部件](https://developer.paddle.com/guides/how-tos/checkout/paddle-checkout) 完成的。在使用結賬小部件之前，我們必須使用 Cashier 生成一個 「支付連結」。 「支付連結」將通知結賬小部件我們希望執行的計費操作：

    use App\Models\User;
    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $request->user()->newSubscription('default', $premium = 34567)
            ->returnTo(route('home'))
            ->create();

        return view('billing', ['payLink' => $payLink]);
    });

Cashier 包括一個 `paddle-button` [Blade 元件](/docs/laravel/10.x/blade#components)。 我們可以將支付連結 URL 作為 「prop」傳遞給該元件。 點選此按鈕時，將顯示 Paddle 的結帳小部件：

```html
<x-paddle-button :url="$payLink" class="px-8 py-4">
    訂閱
</x-paddle-button>
```

默認情況下，這將顯示一個具有標準 Paddle 樣式的按鈕。 你可以通過向元件新增 `data-theme="none"` 屬性來刪除所有 Paddle 樣式：

```html
<x-paddle-button :url="$payLink" class="px-8 py-4" data-theme="none">
    訂閱
</x-paddle-button>
```

Paddle 結賬小部件是非同步的。 一旦使用者在小部件中建立或更新訂閱，Paddle 將傳送你的應用程式 webhook，以便你可以在我們自己的資料庫中正確更新訂閱狀態。 因此，正確 [設定 webhooks](#handling-paddle-webhook) 以同步 Paddle 的狀態變化非常重要。

有關支付連結的更多資訊，你可以查看 [有關支付連結生成的 Paddle API 文件](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink)。

> 注意：訂閱狀態更改後，接收相應 webhook 的延遲通常很小，但你應該在應用程式中考慮到這一點，因為你的使用者訂閱在完成結帳後可能不會立即生效。



#### 手動呈現支付連結

你也可以在不使用 Laravel 內建的 Blade 元件的情況下手動渲染支付連結。 首先，生成支付連結 URL，如先前所示：

    $payLink = $request->user()->newSubscription('default', $premium = 34567)
        ->returnTo(route('home'))
        ->create();

接下來，只需將支付連結 URL 附加到 HTML 中的 `a` 元素：

    <a href="#!" class="ml-4 paddle_button" data-override="{{ $payLink }}">
        Paddle 支付
    </a>

#### 需要額外確認的付款

有時需要額外的驗證才能確認和處理付款。發生這種情況時，Paddle 將顯示付款確認螢幕。Paddle 或 Cashier 顯示的付款確認螢幕可能會針對特定銀行或發卡機構的付款流程進行定製，並且可能包括額外的卡確認、臨時小額費用、單獨的裝置身份驗證或其他形式的驗證。

### 內聯結賬

如果你不想使用 Paddle 的 「疊加」樣式結帳小部件，Paddle 還提供了內嵌顯示小部件的選項。 雖然這種方法不允許你調整任何結帳的 HTML 欄位，但它允許你將小部件嵌入到你的應用中。

為了讓你輕鬆開始內聯結賬，Cashier 包含一個 `paddle-checkout` Blade 元件。 首先，你應該 [生成支付連結](#pay-links)並將支付連結傳遞給元件的 `override` 屬性：

```blade
<x-paddle-checkout :override="$payLink" class="w-full" />
```

要調整內聯結帳元件的高度，你可以將 `height` 屬性傳遞給 Blade 元件：

```blade
<x-paddle-checkout :override="$payLink" class="w-full" height="500" />
```



#### 沒有支付連結的內聯結賬

或者，你可以使用自訂選項而不是使用支付連結來自訂小部件：

```blade
@php
$options = [
    'product' => $productId,
    'title' => 'Product Title',
];
@endphp

<x-paddle-checkout :options="$options" class="w-full" />
```

請參閱 Paddle 的 [Inline Checkout 指南](https://developer.paddle.com/guides/how-tos/checkout/inline-checkout) 以及他們的 [參數參考](https://developer.paddle.com/reference/paddle-js/parameters) 以獲取有關內聯結帳可用選項的更多詳細資訊。

> 注意：如果你想在指定自訂選項時也使用 passthrough 選項，你應該提供一個鍵 / 值陣列作為其值。Cashier 將自動處理將陣列轉換為 JSON 字串。 此外，`customer_id` passthrough 選項保留供內部 Cashier 使用。

#### 手動呈現內聯結賬

你也可以在不使用 Laravel 的內建 Blade 元件的情況下手動渲染內聯結賬。 首先，生成支付連結 URL [如前面示例中所示](#pay-links)。

接下來，你可以使用 Paddle.js 來初始化結帳。 為了讓這個例子簡單，我們將使用 [Alpine.js](https://github.com/alpinejs/alpine) 來演示； 但是，你可以自由地將此示例轉換為你自己的前端技術堆疊：

```html
<div class="paddle-checkout" x-data="{}" x-init="
    Paddle.Checkout.open({
        override: {{ $payLink }},
        method: 'inline',
        frameTarget: 'paddle-checkout',
        frameInitialHeight: 366,
        frameStyle: 'width: 100%; background-color: transparent; border: none;'
    });
">
</div>
```

### 使用者識別

與 Stripe 相比，Paddle 使用者在所有 Paddle 中都是獨一無二的，而不是每個 Paddle 帳戶都是獨一無二的。因此，Paddle 的 API 目前不提供更新使用者詳細資訊（例如電子郵件地址）的方法。在生成支付連結時，Paddle 使用 `customer_email` 參數識別使用者。建立訂閱時，Paddle 將嘗試將使用者提供的電子郵件與現有 Paddle 使用者進行匹配。


鑑於這種行為，在使用 Cashier 和 Paddle 時需要記住一些重要的事情。首先，你應該知道，即使 Cashier 中的訂閱繫結到同一個應用程式使用者，**它們也可能繫結到 Paddle 內部系統中的不同使用者**。其次，每個訂閱都有自己的連接支付方式資訊，並且在 Paddle 的內部系統中也可能有不同的電子郵件地址（取決於建立訂閱時分配給使用者的電子郵件）。

因此，在顯示訂閱時，你應該始終告知使用者哪些電子郵件地址或付款方式資訊與訂閱相關聯。可以使用 `Laravel\Paddle\Subscription` 模型提供的以下方法檢索這些資訊：

    $subscription = $user->subscription('default');

    $subscription->paddleEmail();
    $subscription->paymentMethod();
    $subscription->cardBrand();
    $subscription->cardLastFour();
    $subscription->cardExpirationDate();

當前，沒有辦法通過 Paddle API 修改使用者的電子郵件地址。當使用者想在 Paddle 內更新他們的電子郵件地址時，他們唯一的方法是聯絡 Paddle 客戶支援。在與 Paddle 溝通時，他們需要提供訂閱的 `paddleEmail`，這樣 Paddle 就可以更新正確的使用者。

## 定價

Paddle 允許你自訂每種貨幣對應的價格，也就是說 Paddle 允許你為不同國家和地區組態不同的價格。Cashier Paddle 允許你使用 `productPrices` 方法檢索一個特定產品的所有價格。這個方法接受你希望檢索價格的產品的產品 ID：

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices([123, 456]);



貨幣將根據請求的 IP 地址來確定，當然你也可以傳入一個可選的國家和地區參數來檢索特定國家和地區的價格：

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices([123, 456], ['customer_country' => 'BE']);

檢索出價格後，你可以根據需要顯示它們：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->price()->gross() }}</li>
    @endforeach
</ul>
```

你也可以顯示淨價（不含稅）並將稅額顯示分離：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->price()->net() }} (+ {{ $price->price()->tax() }} tax)</li>
    @endforeach
</ul>
```

如果你檢索了訂閱的價格，你可以分別顯示其原始價格和連續訂閱價格：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - Initial: {{ $price->initialPrice()->gross() }} - Recurring: {{ $price->recurringPrice()->gross() }}</li>
    @endforeach
</ul>
```

更多相關資訊，請 [查看 Paddle 的價格 API 文件](https://developer.paddle.com/api-reference/checkout-api/prices/getprices)。

#### 客戶

如果使用者已經是客戶並且你希望顯示適用於該客戶的價格，你可以通過直接從客戶實例檢索價格來實現：

    use App\Models\User;

    $prices = User::find(1)->productPrices([123, 456]);

在內部，Cashier 將使用使用者的 [`paddleCountry` 方法](#customer-defaults) 來檢索以他們的貨幣表示的價格。例如，居住在美國的使用者將看到以美元為單位的價格，而位於比利時的使用者將看到以歐元為單位的價格。如果找不到匹配的貨幣，則將使用產品的默認貨幣。你可以在 Paddle 控制面板中自訂產品或訂閱計畫的所有價格。



#### 優惠券

你也可以展示選擇優惠券後的折扣價。 在呼叫 `productPrices` 方法時，優惠券可以作為逗號分隔的字串傳遞：

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices([123, 456], [
        'coupons' => 'SUMMERSALE,20PERCENTOFF'
    ]);

然後，使用 `price` 方法顯示計算出的價格：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->price()->gross() }}</li>
    @endforeach
</ul>
```

你可以使用 `listPrice` 方法顯示原價（沒有優惠券折扣）：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->listPrice()->gross() }}</li>
    @endforeach
</ul>
```

> 注意：使用價格 API 時，Paddle 僅允許將優惠券應用於一次性購買的產品，而不允許應用於訂閱計畫。

## 客戶

### 客戶預設值

Cashier 允許你在建立支付連結時為你的客戶定義一些預設值。 設定這些預設值允許你預先填寫客戶的電子郵件地址、國家 / 地區和郵政編碼，以便他們可以立即轉到結帳小部件的付款部分。 你可以通過覆蓋計費模型上的以下方法來設定這些預設值：

    /**
     * 獲取客戶的電子郵件地址以與 Paddle 關聯。
     */
    public function paddleEmail(): string|null
    {
        return $this->email;
    }

    /**
     * 獲取客戶的國家與 Paddle 關聯。
     *
     * 這需要一個 2 個字母的程式碼。 有關支援的國家 / 地區，請參閱以下連結。
     *
     * @link https://developer.paddle.com/reference/platform-parameters/supported-countries
     */
    public function paddleCountry(): string|null
    {
        // ...
    }

    /**
     * 獲取客戶的郵政編碼以與 Paddle 關聯。
     *
     * 有關需要此功能的國家 / 地區，請參閱以下連結。
     *
     * @link https://developer.paddle.com/reference/platform-parameters/supported-countries#countries-requiring-postcode
     */
    public function paddlePostcode(): string|null
    {
        // ...
    }



這些預設值將用於 Cashier 中生成 [支付連結](#pay-links) 的每個操作。

## 訂閱

### 建立訂閱

要建立訂閱，請首先檢索計費模型的實例，該實例通常是 `App\Models\User` 的實例。檢索模型實例後，你可以使用 `newSubscription` 方法來建立模型的訂閱支付連結：

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $request->user()->newSubscription('default', $premium = 12345)
            ->returnTo(route('home'))
            ->create();

        return view('billing', ['payLink' => $payLink]);
    });

傳遞給 `newSubscription` 方法的第一個參數應該是訂閱的名稱。 如果你的應用只提供一個訂閱，你可以將其稱為 `default` 或 `primary`。第二個參數是使用者訂閱的特定計畫。 該值應對應於 Paddle 中的計畫識別碼。`returnTo` 方法接受一個 URL，你的使用者在成功完成結帳後將被重新導向到該 URL。

`create` 方法將建立一個支付連結，你可以使用它來生成一個支付按鈕。可以使用 Cashier Paddle 附帶的 `paddle-button` [Blade 元件](/docs/laravel/10.x/blade#components) 生成支付按鈕：

```blade
<x-paddle-button :url="$payLink" class="px-8 py-4">
    訂閱
</x-paddle-button>
```



使用者完成結帳後，將從 Paddle 傳送一個 `subscription_created` webhook。 Cashier 將收到此 webhook 並為你的客戶設定訂閱。為了確保你的應用程式正確接收和處理所有 webhook，請確保你正確地 [設定 webhook 處理](#handling-paddle-webhooks)。

#### 額外細節

如果你想指定額外的客戶或訂閱詳細資訊，你可以通過將它們作為鍵 / 值對陣列傳遞給 `create` 方法來實現。要瞭解有關 Paddle 支援的其他欄位的更多資訊，請查看 Paddle 關於 [生成支付連結](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink) 的文件：

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->create([
            'vat_number' => $vatNumber,
        ]);

#### 優惠券

如果你想在建立訂閱時申請優惠券，你可以使用 `withCoupon` 方法：

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->withCoupon('code')
        ->create();

#### 中繼資料

你還可以使用 `withMetadata` 方法傳遞中繼資料陣列：

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->withMetadata(['key' => 'value'])
        ->create();

> 注意：提供中繼資料時，請避免使用 `subscription_name` 作為中繼資料鍵。 此金鑰保留供 Cashier 內部使用。

### 檢查訂閱狀態

一旦使用者訂閱了你的應用程式，你就可以使用各種便利的方法檢查他們的訂閱狀態。 首先，如果使用者有活動訂閱，`subscribed` 方法返回 `true`，即使訂閱當前處於試用期：

    if ($user->subscribed('default')) {
        // ...
    }



該 `subscribed` 方法也非常適合 [路由中介軟體](/docs/laravel/10.x/middleware)，允許你根據使用者的訂閱狀態來過濾對路由和 controller 的訪問：

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class EnsureUserIsSubscribed
    {
        /**
         * 處理請求。
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            if ($request->user() && ! $request->user()->subscribed('default')) {
                // 該使用者不是付費使用者。。。
                return redirect('billing');
            }

            return $next($request);
        }
    }

如果你想確定使用者是否仍在試用期內，你可以使用 `onTrial` 方法。這個方法用於確定是否應向使用者顯示他們仍在試用期的警告：

    if ($user->subscription('default')->onTrial()) {
        // ...
    }

該 `subscribedToPlan` 方法可用於根據給定的 Paddle 計畫 ID 來確定使用者是否訂閱了給定的計畫。 在這個例子中，我們將確定使用者的 `default` 訂閱是否訂閱了包月計畫：

    if ($user->subscribedToPlan($monthly = 12345, 'default')) {
        // ...
    }

通過將陣列傳遞給 `subscribedToPlan` 方法，你可以確定使用者的 `default` 訂閱是訂閱月度計畫或是年度計畫：

    if ($user->subscribedToPlan([$monthly = 12345, $yearly = 54321], 'default')) {
        // ...
    }

該 `recurring` 方法可用於確定使用者當前是否已訂閱並且不是處於試用期：

    if ($user->subscription('default')->recurring()) {
        // ...
    }



#### 已取消訂閱狀態

要確定使用者是否曾經是訂閱者但現在已取消訂閱，你可以使用 `cancelled` 方法：

    if ($user->subscription('default')->cancelled()) {
        // ...
    }

你還可以確定使用者是否已取消訂閱，但在訂閱完全到期之前會處於 「寬限期」。 例如，如果使用者在 3 月 5 日取消原定於 3 月 10 日到期的訂閱，則使用者將處於「寬限期」，直到 3 月 10 日。 請注意，在此期間 `subscribed` 方法仍然返回 `true`：

    if ($user->subscription('default')->onGracePeriod()) {
        // ...
    }

確定使用者是否已取消訂閱並且不處於「寬限期」內，你可以使用 `ended` 方法：

    if ($user->subscription('default')->ended()) {
        // ...
    }

#### 逾期狀態

如果訂閱的付款失敗，它將被標記為 `past_due`。當你的訂閱處於此狀態時，在客戶更新其付款資訊之前，它不會處於活動狀態。你可以使用訂閱實例上的 `pastDue` 方法來確定訂閱是否過期：

    if ($user->subscription('default')->pastDue()) {
        // ...
    }

當訂閱過期時，你應該指示使用者 [更新他們的付款資訊](#updating-payment-information)。 你可以在 [Paddle 訂閱設定](https://vendors.paddle.com/subscription-settings) 中組態逾期訂閱的處理方式。

如果你希望訂閱在 `past_due` 時仍被視為活動，你可以使用 Cashier 提供的 `keepPastDueSubscriptionsActive` 方法。通常，此方法應在你的 `AppServiceProvider` 的 `register` 方法中呼叫：

    use Laravel\Paddle\Cashier;

    /**
     * 註冊應用服務。
     */
    public function register(): void
    {
        Cashier::keepPastDueSubscriptionsActive();
    }

> 注意：當訂閱處於 `past_due` 狀態時，在付款資訊更新之前無法更改。 因此，當訂閱處於 `past_due` 狀態時，`swap` 和 `updateQuantity` 方法將拋出異常。



#### 訂閱範圍

大多數訂閱狀態也可用作查詢範圍，以便你可以輕鬆查詢資料庫中處於給定狀態的訂閱：

    // 獲取所有有效訂閱。。。
    $subscriptions = Subscription::query()->active()->get();

    // 獲取給定使用者的所有已取消訂閱。。。
    $subscriptions = $user->subscriptions()->cancelled()->get();

可用範圍的完整列表如下：

    Subscription::query()->active();
    Subscription::query()->onTrial();
    Subscription::query()->notOnTrial();
    Subscription::query()->pastDue();
    Subscription::query()->recurring();
    Subscription::query()->ended();
    Subscription::query()->paused();
    Subscription::query()->notPaused();
    Subscription::query()->onPausedGracePeriod();
    Subscription::query()->notOnPausedGracePeriod();
    Subscription::query()->cancelled();
    Subscription::query()->notCancelled();
    Subscription::query()->onGracePeriod();
    Subscription::query()->notOnGracePeriod();

### 訂閱單次收費

訂閱單次收費允許你在訂閱的基礎上向訂閱者收取一次性費用：

    $response = $user->subscription('default')->charge(12.99, 'Support Add-on');

與 [單一費用](#single-charges) 相比，此方法將立即向客戶儲存的訂閱付款方式收費。 收費金額應始終以訂閱的貨幣定義。

### 更新付款資訊

Paddle 始終為每個訂閱保存一種付款方式。 如果要更新訂閱的默認付款方式，則應首先使用訂閱模型上的 `updateUrl` 方法生成訂閱 「更新 URL」：

    use App\Models\User;

    $user = User::find(1);

    $updateUrl = $user->subscription('default')->updateUrl();

然後，你可以將生成的 URL 與 Cashier 提供的 `paddle-button` Blade 元件結合使用，以允許使用者啟動 Paddle 小部件並更新他們的付款資訊：

```html
<x-paddle-button :url="$updateUrl" class="px-8 py-4">
    更新付款資訊
</x-paddle-button>
```



當使用者更新完他們的資訊後，Paddle 將傳送一個 `subscription_updated` webhook，訂閱詳細資訊將在你的應用資料庫中更新。

### 改變計畫

使用者訂閱你的應用程式後，他們可能偶爾想要更改為新的訂閱計畫。 要為使用者更新訂閱計畫時，你應該將 Paddle 計畫的識別碼傳遞給訂閱的 `swap` 方法：

    use App\Models\User;

    $user = User::find(1);

    $user->subscription('default')->swap($premium = 34567);

如果你想變更計畫並立即為使用者開具發票，而不是等待他們的下一個計費週期，你可以使用 `swapAndInvoice` 方法：

    $user = User::find(1);

    $user->subscription('default')->swapAndInvoice($premium = 34567);

> 注意：試用活動期間不能變更計畫。有關此限制的更多資訊，請參閱 [Paddle 文件](https://developer.paddle.com/api-reference/subscription-api/users/updateuser#usage-notes)。

#### 按比例分配

默認情況下，Paddle 在計畫變更時按比例分配費用。 `noProrate` 方法可用於在不按比例分配費用的情況下更新訂閱：

    $user->subscription('default')->noProrate()->swap($premium = 34567);

### 訂閱數量

有時訂閱會受到 「數量」的影響。例如，項目管理應用可能對每個項目每月收費 10 美元。 要增加或減少訂閱數量，請使用 `incrementQuantity` 和 `decrementQuantity` 方法：

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity();

    // 訂閱增加 5 個。。。
    $user->subscription('default')->incrementQuantity(5);

    $user->subscription('default')->decrementQuantity();

    // 訂閱減少 5 個。。。
    $user->subscription('default')->decrementQuantity(5);



或者，你以使用 `updateQuantity` 方法設定特定數量：

    $user->subscription('default')->updateQuantity(10);

該 `noProrate` 方法可用於更新訂閱數量而不按比例分配費用：

    $user->subscription('default')->noProrate()->updateQuantity(10);

### 訂閱修改器

訂閱修改器允許你實施 [按量計費](https://developer.paddle.com/guides/how-tos/subscriptions/metered-billing#using-subscription-price-modifiers) 或使用附加元件擴展訂閱。

例如，你可能想為標準訂閱提供 「高級支援」附加元件。 你可以像這樣建立這個修改器：

    $modifier = $user->subscription('default')->newModifier(12.99)->create();

The example above will add a $12.99 add-on to the subscription. By default, this charge will recur on every interval you have configured for the subscription. If you would like, you can add a readable description to the modifier using the modifier's `description` method:
上例將向訂閱新增 $12.99 的附加元件。默認情況下，此費用將在你為訂閱組態的每個時間週期內重複收取。 如果你願意，可以使用修改器的 `description` 方法向修改器新增可讀的描述：

    $modifier = $user->subscription('default')->newModifier(12.99)
        ->description('Premium Support')
        ->create();

為了說明如何使用修改器實現計量計費，假設你的應用程式要對使用者傳送的每條 SMS 消息收費。首先，你應該在 Paddle 儀表板中建立一個 $0 的計畫。 使用者訂閱此計畫後，你可以向訂閱新增代表每個單獨費用的修改器：

    $modifier = $user->subscription('default')->newModifier(0.99)
        ->description('New text message')
        ->oneTime()
        ->create();

如你所見，我們在建立此調節器時呼叫了 `oneTime` 方法。此方法將確保修改器只收費一次，並且不會在每個計費週期重複。

#### 檢索修改器



你可以通過 `modifiers` 方法檢索訂閱的所有修改器列表：

    $modifiers = $user->subscription('default')->modifiers();

    foreach ($modifiers as $modifier) {
        $modifier->amount(); // $0.99
        $modifier->description; // 新的簡訊。
    }

#### 刪除修改器

修改器可以通過呼叫 `Laravel\Paddle\Modifier` 實例上的 `delete` 方法來刪除：

    $modifier->delete();

### 多個訂閱

Paddle 允許你的客戶同時擁有多個訂閱。例如，你可能經營一家健身房，提供游泳訂閱和舉重訂閱，每個訂閱可能有不同的定價。當然，客戶應該能夠訂閱其中一項或兩項計畫。

當你的應用程式建立訂閱時，你可以向 `newSubscription` 方法提供訂閱的名稱。該名稱可以是表示使用者正在發起的訂閱類型的任何字串：

    use Illuminate\Http\Request;

    Route::post('/swimming/subscribe', function (Request $request) {
        $request->user()
            ->newSubscription('swimming', $swimmingMonthly = 12345)
            ->create($request->paymentMethodId);

        // ...
    });

在本例中，我們為客戶發起了每月一次的游泳訂閱。然而，他們可能想在以後換成每年訂閱一次。當調整客戶的訂閱時，我們可以簡單地交換`游泳`訂閱的價格：

    $user->subscription('swimming')->swap($swimmingYearly = 34567);

當然，你也可以完全取消訂閱：

    $user->subscription('swimming')->cancel();

### 暫停訂閱

要暫停訂閱，請呼叫使用者訂閱的 `pause` 方法：

    $user->subscription('default')->pause();

當訂閱暫停時，Cashier 將自動在你的資料庫中設定 `paused_from` 列。此列用於確定 `paused` 方法何時應該開始返回 `true`。例如，如果客戶在 3 月 1 日暫停訂閱，但該訂閱直到 3 月 5 日才計畫重複發生，則 `paused` 方法將繼續返回 `false` ，直到 3 月 5 日。這樣做是因為使用者可以繼續使用應用程式，直到他們的計費週期結束。


你可以使用 `onPausedGracePeriod` 方法確定使用者是否已暫停訂閱但仍處於 「寬限期」：

    if ($user->subscription('default')->onPausedGracePeriod()) {
        // ...
    }

要恢復暫停的訂閱，你可以呼叫使用者訂閱的 `unpause` 方法：

    $user->subscription('default')->unpause();

> 注意：訂閱暫停時無法修改。 如果你想切換到不同的計畫或更新數量，你必須先恢復訂閱。

### 取消訂閱

要取消訂閱，請呼叫使用者訂閱的 `cancel` 方法：

    $user->subscription('default')->cancel();

當訂閱被取消時，Cashier 將自動在你的資料庫中設定 `ends_at` 列。 此列用於確定 `subscribed` 方法應該何時開始返回 `false`。例如，如果客戶在 3 月 1 日取消訂閱，但訂閱計畫在 3 月 5 日之前結束，則 `subscribed` 方法將在 3 月 5 日之前繼續返回 `true`。這樣做是因為通常允許使用者繼續使用應用程式，直到他們的計費週期結束。

你可以使用 `onGracePeriod` 方法確定使用者是否已取消訂閱但仍處於「寬限期」：

    if ($user->subscription('default')->onGracePeriod()) {
        // ...
    }

如果你想立即取消訂閱，你可以呼叫使用者訂閱的 `cancelNow` 方法：

    $user->subscription('default')->cancelNow();

> 注意：取消後無法恢復 Paddle 的訂閱。 如果你的客戶希望恢復訂閱，則他們必須重新訂閱。



## 訂閱試用

### 預先收集付費方式

> 注意：在預先試用和收集付款方式詳細資訊時，Paddle 會阻止任何訂閱更改，例如更換計畫或更新數量。 如果你想允許客戶在試用期間更換計畫，則必須取消並重新建立訂閱。

如果你想為你的客戶提供試用期，同時仍然預先收集付款方式資訊，你應該在建立訂閱付款連結時使用 `trialDays` 方法：

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $request->user()->newSubscription('default', $monthly = 12345)
                    ->returnTo(route('home'))
                    ->trialDays(10)
                    ->create();

        return view('billing', ['payLink' => $payLink]);
    });

此方法將在你的應用資料庫中的訂閱記錄上設定試用期結束日期，並指示 Paddle 在此日期之後才開始向客戶收費。

> 注意：如果客戶的訂閱未在試用結束日期之前取消，他們將在試用到期後立即收費，因此你務必將試用結束日期通知你的使用者。

你可以使用使用者實例的 `onTrial` 方法或訂閱實例的 `onTrial` 方法來確定使用者是否在試用期內。 下面的兩個例子是一樣的：

    if ($user->onTrial('default')) {
        // ...
    }

    if ($user->subscription('default')->onTrial()) {
        // ...
    }

要確定試用期是否已過期，你可以使用 `hasExpiredTrial` 方法：

    if ($user->hasExpiredTrial('default')) {
        // ...
    }

    if ($user->subscription('default')->hasExpiredTrial()) {
        // ...
    }



#### 在 Paddle / Cashier 中定義試用天數

你可以選擇在 Paddle 儀表板中定義你的計畫接收的試用天數，或者始終使用 Cashier 明確傳遞它們。如果你選擇在 Paddle 中定義計畫的試用天數，你應該知道新訂閱，包括過去訂閱過的客戶的新訂閱，將始終獲得試用期，除非你明確呼叫 `trialDays(0)` 方法。

### 未預先收集付款方式

如果你想提供試用期而不預先收集使用者的付款方式資訊，你可以將附加到你的使用者的客戶記錄上的 `trial_ends_at` 列設定為你想要的試用結束日期。這通常在使用者註冊期間完成：

    use App\Models\User;

    $user = User::create([
        // ...
    ]);

    $user->createAsCustomer([
        'trial_ends_at' => now()->addDays(10)
    ]);

Cashier 將這種類型的試用稱為「通用試用」，因為它不附屬於任何現有訂閱。如果當前日期未超過 `trial_ends_at` 的值，則 `User` 實例上的 `onTrial` 方法將返回 `true`：

    if ($user->onTrial()) {
        // 使用者在試用期內。。。
    }

一旦你準備好為使用者建立一個實際的訂閱，你可以像往常一樣使用 `newSubscription` 方法：

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $user->newSubscription('default', $monthly = 12345)
            ->returnTo(route('home'))
            ->create();

        return view('billing', ['payLink' => $payLink]);
    });

要檢索使用者的試用結束日期，你可以使用 `trialEndsAt` 方法。如果使用者正在試用，則此方法將返回一個 Carbon 日期實例，否則將返回 `null` 。如果你想獲取特定訂閱而不是默認訂閱的試用結束日期，你還可以傳遞一個可選的訂閱名稱參數：

    if ($user->onTrial()) {
        $trialEndsAt = $user->trialEndsAt('main');
    }



如果你希望明確知道使用者處於 「通用」試用期內並且尚未建立實際訂閱，則可以使用 `onGenericTrial` 方法：

    if ($user->onGenericTrial()) {
        // 使用者在通用試用期內。。。
    }

> 注意：建立 Paddle 訂閱後，無法延長或修改其試用期。

## 處理 Paddle Webhooks

Paddle 可以通過 webhook 通知你的應用各種事件。默認情況下，指向 Cashier 的 webhook  controller 的路由由 Cashier 服務提供商註冊。
該 controller 將處理所有傳入的 webhook 請求。

默認情況下，此 controller 將自動處理付費失敗過多的取消訂閱（[由你的 Paddle 訂閱設定定義](https://vendors.paddle.com/subscription-settings)）、訂閱更新和付款方式更改；但是，我們很快就會發現，你可以擴展這個 controller 來處理你喜歡的任何 Paddle webhook 事件。

為確保你的應用可以處理 Paddle webhooks，請務必 [在 Paddle 控制面板中組態 webhook URL](https://vendors.paddle.com/alerts-webhooks)。默認情況下，Cashier 的 webhook  controller 響應 `/paddle/webhook` URL 路徑。你應該在 Paddle 控制面板中啟用的所有 webhook 的完整列表是：

- 訂閱建立
- 訂閱更新
- 訂閱取消
- 付款成功
- 訂閱付款成功

> 注意：確保使用 Cashier 包含的 [webhook 簽名驗證](/docs/laravel/10.x/cashier-paddle#verifying-webhook-signatures) 中介軟體保護傳入請求。

#### Webhook 和 CSRF 保護

由於 Paddle webhooks 需要繞過 Laravel 的 [CSRF 保護](/docs/laravel/10.x/csrf)，請務必在你的 `App\Http\Middleware\VerifyCsrfToken` 中介軟體中將 URI 作為例外列出或列出外面的路由 `web` 中介軟體組的：

    protected $except = [
        'paddle/*',
    ];



#### Webhook 和本地開發

為了讓 Paddle 能夠在本地開發期間傳送你的應用程式 webhook，你需要通過站點共享服務公開你的應用程式，例如 [Ngrok](https://ngrok.com/) 或 [Expose](https://expose.dev/docs/introduction)。如果你使用 [Laravel Sail](/docs/laravel/10.x/sail) 在本地開發應用程式，你可以使用 Sail 的 [站點共享命令](/docs/laravel/10.x/sail#sharing-your-site)。

### 定義 webhook 事件處理程序

Cashier 會自動處理因收費失敗和其他常見的 paddle webhook 取消訂閱。 但是，如果你有其他想要處理的 webhook 事件，你可以通過監聽 Cashier 調度的以下事件來實現：

- `Laravel\Paddle\Events\WebhookReceived`
- `Laravel\Paddle\Events\WebhookHandled`

這兩個事件都包含 Paddle webhook 的完整負載。例如，如果你想處理 `invoice.payment_succeeded` webhook，你可以註冊一個 [listener](/docs/laravel/10.x/events#defining-listeners) 來處理事件：

    <?php

    namespace App\Listeners;

    use Laravel\Paddle\Events\WebhookReceived;

    class PaddleEventListener
    {
        /**
         * 處理收到的 Paddle webhook。
         */
        public function handle(WebhookReceived $event): void
        {
            if ($event->payload['alert_name'] === 'payment_succeeded') {
                // 處理傳入事件。。。
            }
        }
    }

一旦你的監聽器被定義，你可以在你的應用程式的 `EventServiceProvider` 中註冊它：

    <?php

    namespace App\Providers;

    use App\Listeners\PaddleEventListener;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
    use Laravel\Paddle\Events\WebhookReceived;

    class EventServiceProvider extends ServiceProvider
    {
        protected $listen = [
            WebhookReceived::class => [
                PaddleEventListener::class,
            ],
        ];
    }

Cashier 還會發出專用於接收到的 webhook 類型的事件。除了來自 Paddle 的完整有效負載之外，它們還包含用於處理 webhook 的相關模型，例如計費模型、訂閱或收據：

- `Laravel\Paddle\Events\PaymentSucceeded`
- `Laravel\Paddle\Events\SubscriptionPaymentSucceeded`
- `Laravel\Paddle\Events\SubscriptionCreated`
- `Laravel\Paddle\Events\SubscriptionUpdated`
- `Laravel\Paddle\Events\SubscriptionCancelled`

你還可以通過在應用程式的 `.env` 檔案中定義 `CASHIER_WEBHOOK` 環境變數來覆蓋默認的內建 webhook 路由。此值應該是你的 webhook 路由中的完整 URL，並且需要和你在 Paddle 控制面板中設定的 URL 相匹配：

```ini
CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url
```

### 驗證 Webhook 簽名

為了保護你的 webhook，你可以使用 [Paddle 的 webhook 簽名](https://developer.paddle.com/webhook-reference/verifying-webhooks)。 為方便起見，Cashier 自動包含一個中介軟體，用於驗證傳入的 Paddle webhook 請求是否有效。

要啟用 webhook 驗證，請確保在應用程式的 .env 檔案中定義了`PADDLE_PUBLIC_KEY` 環境變數。 可以從你的 Paddle 帳戶儀表板中檢索公鑰。

## 一次性收費

### 簡單收費

如果你想對客戶進行一次性收費，你可以在可計費模型實例上使用 `charge` 方法來生成收費的支付連結。`charge` 方法接受費用金額（浮點數）作為它的第一個參數和一個費用描述作為它的第二個參數：

    use Illuminate\Http\Request;

    Route::get('/store', function (Request $request) {
        return view('store', [
            'payLink' => $user->charge(12.99, 'Action Figure')
        ]);
    });

生成支付連結後，你可以使用 Cashier 提供的 `paddle-button` Blade 元件讓使用者啟動 Paddle 小部件並完成收費：

```blade
<x-paddle-button :url="$payLink" class="px-8 py-4">
    Buy
</x-paddle-button>
```

`charge` 方法接受一個陣列作為其第三個參數，允許你將任何你希望的選項傳遞給底層 Paddle 支付連結建立。請查閱 [Paddle 文件](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink) 瞭解更多關於建立費用時可用的選項：

    $payLink = $user->charge(12.99, 'Action Figure', [
        'custom_option' => $value,
    ]);



費用以 `cashier.currency` 組態選項中指定的貨幣進行。 默認設定是美元。 你可以通過在應用程式的 `.env` 檔案中定義 `CASHIER_CURRENCY` 環境變數來覆蓋默認貨幣：

```ini
CASHIER_CURRENCY=EUR
```

你還可以使用 Paddle 的動態定價匹配系統 [覆蓋每種貨幣的價格](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink#price-overrides)。為此，請通過價格陣列而不是固定金額：

    $payLink = $user->charge([
        'USD:19.99',
        'EUR:15.99',
    ], 'Action Figure');

### 收費產品

如果你想對 Paddle 中組態的特定產品進行一次性收費，你可以在計費模型實例上使用 `chargeProduct` 方法來生成付款連結：

    use Illuminate\Http\Request;

    Route::get('/store', function (Request $request) {
        return view('store', [
            'payLink' => $request->user()->chargeProduct($productId = 123)
        ]);
    });

然後，你可以提供 `paddle-button` 元件的支付連結，以允許使用者初始化 Paddle 小部件：

```blade
<x-paddle-button :url="$payLink" class="px-8 py-4">
    購買
</x-paddle-button>
```

`chargeProduct` 方法接受一個陣列作為其第二個參數，允許你將任何你希望的選項傳遞給底層 Paddle 支付連結建立。 請查閱 [Paddle 文件](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink) 關於建立費用時可用的選項：

    $payLink = $user->chargeProduct($productId, [
        'custom_option' => $value,
    ]);

### 退款訂單

如果你需要對槳訂單進行退款，你可以使用 `refund` 方法。 此方法接受 Paddle 訂單 ID 作為其第一個參數。 你可以使用 `receipts` 方法檢索給定計費模型的收據：

    use App\Models\User;

    $user = User::find(1);

    $receipt = $user->receipts()->first();

    $refundRequestId = $user->refund($receipt->order_id);



你可以選擇指定具體的退款金額以及退款原因：

    $receipt = $user->receipts()->first();

    $refundRequestId = $user->refund(
        $receipt->order_id, 5.00, 'Unused product time'
    );

> 技巧：聯絡 Paddle 支援時，你可以使用 `$refundRequestId` 作為退款參考。

## 收據
你可以通過 `receipts` 屬性輕鬆檢索可計費模型的收據陣列：

use App\Models\User;

    use App\Models\User;

    $user = User::find(1);

    $receipts = $user->receipts;

在為客戶列出收據時，你可以使用收據實例的方法來顯示相關的收據資訊。 例如，你可能希望在表格中列出每張收據，以便使用者輕鬆下載任何收據：

```html
<table>
    @foreach ($receipts as $receipt)
        <tr>
            <td>{{ $receipt->paid_at->toFormattedDateString() }}</td>
            <td>{{ $receipt->amount() }}</td>
            <td><a href="{{ $receipt->receipt_url }}" target="_blank">Download</a></td>
        </tr>
    @endforeach
</table>
```

### 過去和未來的付款

你可以使用 `lastPayment` 和 `nextPayment` 方法來檢索和顯示客戶過去或即將進行的定期訂閱付款：

    use App\Models\User;

    $user = User::find(1);

    $subscription = $user->subscription('default');

    $lastPayment = $subscription->lastPayment();
    $nextPayment = $subscription->nextPayment();

這兩種方法都會返回一個 `Laravel\Paddle\Payment` 的實例； 但是，當計費週期結束時（例如取消訂閱時），`nextPayment` 將返回 `null`：

```blade
Next payment: {{ $nextPayment->amount() }} due on {{ $nextPayment->date()->format('d/m/Y') }}
```

## 處理失敗的付款

訂閱支付失敗的原因有多種，例如卡過期或卡資金不足。 發生這種情況時，我們建議你讓 Paddle 為你處理付款失敗。具體來說，你可以在你的 Paddle 儀表板中 [設定 Paddle 的自動計費電子郵件](https://vendors.paddle.com/subscription-settings)

或者，你可以通過捕獲 [`subscription_payment_failed`](https://developer.paddle.com/webhook-reference/subscription-alerts/subscription-payment-failed) webhook 並啟用 “訂閱付款失敗” 來執行更精確的自訂 Paddle 儀表板的 Webhook 設定中的選項：

    <?php

    namespace App\Listeners;

    use Laravel\Paddle\Events\WebhookReceived;

    class PaddleEventListener
    {
        /**
         * 處理訂閱付款失敗。
         */
        public function handle(WebhookReceived $event): void
        {
            if ($event->payload['alert_name'] === 'subscription_payment_failed') {
                // 處理訂閱付款失敗。。。
            }
        }
    }

一旦定義了監聽器，就得在應用程式的 `EventServiceProvider` 中註冊它：

    <?php

    namespace App\Providers;

    use App\Listeners\PaddleEventListener;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
    use Laravel\Paddle\Events\WebhookReceived;

    class EventServiceProvider extends ServiceProvider
    {
        protected $listen = [
            WebhookReceived::class => [
                PaddleEventListener::class,
            ],
        ];
    }

## 測試

在測試時，你應該手動測試你的計費流程，以確保你的整合按預期工作。

對於自動化測試，包括在 CI 環境中執行的測試，你可以使用 [Laravel 的 HTTP 客戶端](/docs/laravel/10.x/http-client#testing) 來偽造對 Paddle 的 HTTP 呼叫。 儘管這不會測試來自 Paddle 的實際響應，但它確實提供了一種無需實際呼叫 Paddle API 即可測試你應用程式的方法。
