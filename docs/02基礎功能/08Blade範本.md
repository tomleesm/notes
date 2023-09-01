# Blade 範本

## 簡介

Blade 是 Laravel 提供的一個簡單而又強大的範本引擎。 和其他流行的 PHP 範本引擎不同，Blade 並不限制你在檢視中使用原生 PHP 程式碼。實際上，所有 Blade 檢視檔案都將被編譯成原生的 PHP 程式碼並快取起來，除非它被修改，否則不會重新編譯，這就意味著 Blade 基本上不會給你的應用增加任何負擔。Blade 範本檔案使用 `.blade.php` 作為副檔名，被存放在 `resources/views` 目錄。



Blade 檢視可以使用全域 `view` 函數從 Route 或 controller 返回。當然，正如有關 [views](/docs/laravel/10.x/views) 的文件中所描述的，可以使用 `view` 函數的第二個參數將資料傳遞到 Blade 檢視：

    Route::get('/', function () {
        return view('greeting', ['name' => 'Finn']);
    });

### 用 Livewire 為 Blade 賦能

想讓你的 Blade 範本更上一層樓，輕鬆建構動態介面嗎？看看[Laravel Livewire](https://laravel-livewire.com)。Livewire 允許你編寫 Blade 元件，這些元件具有動態功能，通常只能通過 React 或 Vue 等前端框架來實現，這提供了一個很好的方法來建構現代，沒有複雜前端對應，基於客戶端渲染，無須很多的建構步驟的  JavaScript 框架。


## 顯示資料

你可以把變數置於花括號中以在檢視中顯示資料。例如，給定下方的路由：

    Route::get('/', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

你可以像如下這樣顯示 `name` 變數的內容：

```blade
Hello, {{ $name }}.
```

> **技巧**：Blade 的 `{{ }}` 語句將被 PHP 的 `htmlspecialchars` 函數自動轉義以防範 XSS 攻擊。

你不僅限於顯示傳遞給檢視的變數的內容。你也可以回顯任何 PHP 函數的結果。實際上，你可以將所需的任何 PHP 程式碼放入 Blade echo 語句中：

```blade
The current UNIX timestamp is {{ time() }}.
```

### HTML 實體編碼



默認情況下，Blade（和 Laravel `e` 助手）將對 HTML 實體進行雙重編碼。如果你想停用雙重編碼，請從 `AppServiceProvider` 的 `boot` 方法呼叫 `Blade::withoutDoubleEncoding` 方法：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Blade::withoutDoubleEncoding();
        }
    }

#### 展示非轉義資料

默認情況下， Blade `{{ }}` 語句將被 PHP 的 `htmlspecialchars` 函數自動轉義以防範 XSS 攻擊。如果不想你的資料被轉義，那麼你可使用如下的語法：

```blade
Hello, {!! $name !!}.
```

> **注意：**在應用中顯示使用者提供的資料時請格外小心，請儘可能的使用轉義和雙引號語法來防範 XSS 攻擊。

### Blade & JavaScript 框架

由於許多 JavaScript 框架也使用「花括號」來標識將顯示在瀏覽器中的表示式，因此，你可以使用 `@` 符號來表示 Blade 渲染引擎應當保持不變。例如：

```blade
<h1>Laravel</h1>

Hello, @{{ name }}.
```

在這個例子中， `@` 符號將被 Blade 移除；當然，Blade 將不會修改 `{{ name }}` 表示式，取而代之的是 JavaScript 範本來對其進行渲染。

`@` 符號也用於轉義 Blade 指令：

```blade
{{-- Blade template --}}
@@if()

<!-- HTML output -->
@if()
```

#### 渲染 JSON

有時，你可能會將陣列傳遞給檢視，以將其呈現為 JSON，以便初始化 JavaScript 變數。 例如：

```blade
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```



或者，你可以使用 `Illuminate\Support\Js::from` 方法指令，而不是手動呼叫 `json_encode`。 `from` 方法接受與 PHP 的 `json_encode` 函數相同的參數；但是，它將確保正確轉義生成的 JSON 以包含在 HTML 引號中。 `from` 方法將返回一個字串 `JSON.parse` JavaScript 語句，它將給定對象或陣列轉換為有效的 JavaScript 對象：

```blade
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

Laravel 框架的最新版本包括一個 `Js` 門面，它提供了在 Blade 範本中方便地訪問此功能：

```blade
<script>
    var app = {{ Js::from($array) }};
</script>
```

> **注意：**你應該只使用 `Js::from` 渲染已經存在的變數為 JSON。 Blade 範本基於正規表示式，如果嘗試將複雜表示式傳遞給 `Js::from` 可能會導致無法預測的錯誤。

#### `@verbatim` 指令

如果你在範本中顯示很大一部分 JavaScript 變數，你可以將 HTML 嵌入到 `@verbatim` 指令中，這樣，你就不需要在每一個 Blade 回顯語句前新增 `@` 符號：

```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

## Blade 指令

除了範本繼承和顯示資料以外， Blade 還為常見的 PHP 控制結構提供了便捷的快捷方式，例如條件語句和循環。這些快捷方式為 PHP 控制結構提供了一個非常清晰、簡潔的書寫方式，同時，還與 PHP 中的控制結構保持了相似的語法特性。



### If 語句

你可以使用 `@if` ， `@elseif` ， `@else` 和 `@endif` 指令構造 `if` 語句。這些指令功能與它們所對應的 PHP 語句完全一致：

```blade
@if (count($records) === 1)
    有一條記錄
@elseif (count($records) > 1)
    有多條記錄
@else
    沒有記錄
@endif
```

為了方便， Blade 還提供了一個 `@unless` 指令：

```blade
@unless (Auth::check())
    你還沒有登錄
@endunless
```

> 譯註：相當於 `@if (! Auth::check()) @endif`

除了上面所說條件指令外， `@isset` 和 `@empty` 指令亦可作為它們所對應的 PHP 函數的快捷方式：

```blade
@isset($records)
    // $records 已經被定義且不為 null ……
@endisset

@empty($records)
    // $records 為「空」……
@endempty
```

#### 授權指令

`@auth` 和 `@guest` 指令可用於快速判斷當前使用者是否已經獲得 [授權](/docs/laravel/10.x/authentication) 或是遊客：

```blade
@auth
    // 使用者已經通過認證……
@endauth

@guest
    // 使用者沒有通過認證……
@endguest
```

如有需要，你亦可在使用 `@auth` 和 `@guest` 指令時指定 [認證守衛](https://learnku.com/docs/laravel/10.x/authentication "認證守衛")：

```blade
@auth('admin')
    // 使用者已經通過認證...
@endauth

@guest('admin')
    // 使用者沒有通過認證...
@endguest
```

#### 環境指令

你可以使用 `@production` 指令來判斷應用是否處於生產環境：

```blade
@production
    // 生產環境特定內容...
@endproduction
```

或者，你可以使用 `@env` 指令來判斷應用是否運行於指定的環境：

```blade
@env('staging')
    //  應用運行於「staging」環境...
@endenv

@env(['staging', 'production'])
    // 應用運行於 「staging」或 [生產] 環境...
@endenv
```



#### 區塊指令

你可以使用 `@hasSection` 指令來判斷區塊是否有內容：

```blade
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

你可以使用 `sectionMissing` 指令來判斷區塊是否沒有內容：

```blade
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

### Switch 語句

你可使用 `@switch` ， `@case` ， `@break` ， `@default` 和 `@endswitch` 語句來構造 Switch 語句：

```blade
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

### 循環

除了條件語句， Blade 還提供了與 PHP 循環結構功能相同的指令。同樣，這些語句的功能和它們所對應的 PHP 語法一致：

```blade
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

> **技巧：**在遍歷 `foreach` 循環時，你可以使用 [循環變數](#the-loop-variable) 去獲取有關循環的有價值的資訊，例如，你處於循環的第一個迭代亦或是處於最後一個迭代。

使用循環時，還可以使用 `@continue` 和 `@break` 循環或跳過當前迭代：

```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

你還可以在指令聲明中包含繼續或中斷條件：

```blade
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```



### Loop 變數

在遍歷 `foreach` 循環時，循環內部可以使用 `$loop` 變數。該變數提供了訪問一些諸如當前的循環索引和此次迭代是首次或是末次這樣的資訊的方式：

```blade
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

如果你處於巢狀循環中，你可以使用循環的 `$loop` 變數的 `parent` 屬性訪問父級循環：

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

該 `$loop` 變數還包含各種各樣有用的屬性：

| 屬性  | 描述 |
| ------------- | ------------- |
| `$loop->index`  |  當前迭代的索引（從 0 開始）。 |
| `$loop->iteration`  |  當前循環的迭代次數（從 1 開始）。 |
| `$loop->remaining`  |  循環剩餘的迭代次數。 |
| `$loop->count`  |  被迭代的陣列的元素個數。 |
| `$loop->first`  |  當前迭代是否是循環的首次迭代。 |
| `$loop->last`  |  當前迭代是否是循環的末次迭代。 |
| `$loop->even`  |  當前循環的迭代次數是否是偶數。 |
| `$loop->odd`  |  當前循環的迭代次數是否是奇數。 |
| `$loop->depth`  |  當前循環的巢狀深度。 |
| `$loop->parent`  |  巢狀循環中的父級循環。 |

### 有條件地編譯 class 樣式

該 `@class` 指令有條件地編譯 CSS class 樣式。該指令接收一個陣列，其中陣列的鍵包含你希望新增的一個或多個樣式的類名，而值是一個布林值表示式。如果陣列元素有一個數值的鍵，它將始終包含在呈現的 class 列表中：

```blade
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```

同樣，`@style` 指令可用於有條件地將內聯 CSS 樣式新增到一個 HTML 元素中。

```blade
@php
    $isActive = true;
@endphp

<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>

<span style="background-color: red; font-weight: bold;"></span>
```

### 附加屬性

為方便起見，你可以使用該 `@checked` 指令輕鬆判斷給定的 HTML 複選框輸入是否被「選中（checked）」。如果提供的條件判斷為 `true` ，則此指令將回顯 `checked`：

```blade
<input type="checkbox"
        name="active"
        value="active"
        @checked(old('active', $user->active)) />
```

同樣，該 `@selected` 指令可用於判斷給定的選項是否被「選中（selected）」：

```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

此外，該 `@disabled` 指令可用於判斷給定元素是否為「停用（disabled）」:

```blade
<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>
```

此外，`@readonly` 指令可以用來指示某個元素是否應該是「唯讀 （readonly）」的。

```blade
<input type="email"
        name="email"
        value="email@laravel.com"
        @readonly($user->isNotAdmin()) />
```

此外，`@required` 指令可以用來指示一個給定的元素是否應該是「必需的（required）」。

```blade
<input type="text"
        name="title"
        value="title"
        @required($user->isAdmin()) />
```

### 包含子檢視

> **技巧：**雖然你可以自由使用該 `@include` 指令，但是 Blade [元件](#components) 提供了類似的功能，並提供了優於該 `@include` 指令的功能，如資料和屬性繫結。

Blade 的 `@include` 指令允許你從一個檢視中包含另外一個 Blade 檢視。父檢視中的所有變數在子檢視中都可以使用：

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

儘管子檢視可以繼承父檢視中所有可以使用的資料，但是你也可以傳遞一個額外的陣列，這個陣列在子檢視中也可以使用:

```blade
@include('view.name', ['status' => 'complete'])
```

如果你想要使用 `@include` 包含一個不存在的檢視，Laravel 將會拋出一個錯誤。如果你想要包含一個可能存在也可能不存在的檢視，那麼你應該使用 `@includeIf` 指令:

```blade
@includeIf('view.name', ['status' => 'complete'])
```

如果想要使用 `@include`  包含一個給定值為 `true` 或 `false`的布林值表示式的檢視，那麼你可以使用 `@includeWhen` 和 `@includeUnless` 指令:

```blade
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

如果想要包含一個檢視陣列中第一個存在的檢視，你可以使用 `includeFirst` 指令:

```blade
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

> **注意：**在檢視中，你應該避免使用 `__DIR__` 和 `__FILE__` 這些常數，因為他們將引用已快取的和已編譯的檢視。

#### 為集合渲染檢視

你可以使用 Blade 的 `@each` 指令將循環合併在一行內：

```blade
@each('view.name', $jobs, 'job')
```

該 `@each` 指令的第一個參數是陣列或集合中的元素的要渲染的檢視片段。第二個參數是你想要迭代的陣列或集合，當第三個參數是一個表示當前迭代的檢視的變數名。因此，如果你遍歷一個名為 `jobs` 的陣列，通常會在檢視片段中使用 `job` 變數來訪問每一個 job （jobs 陣列的元素）。在你的檢視片段中，可以使用 `key` 變數來訪問當前迭代的鍵。

你亦可傳遞第四個參數給 `@each` 指令。當給定的陣列為空時，將會渲染該參數所對應的檢視。

```blade
@each('view.name', $jobs, 'job', 'view.empty')
```

> **注意：**通過 `@each` 指令渲染的檢視不會繼承父檢視的變數。如果子檢視需要使用這些變數，你可以使用 `@foreach` 和 `@include` 來代替它。

### `@once` 指令

該 `@once` 指令允許你定義範本的一部分內容，這部分內容在每一個渲染週期中只會被計算一次。該指令在使用 [堆疊](#stacks) 推送一段特定的 JavaScript 程式碼到頁面的頭部環境下是很有用的。例如，如果你想要在循環中渲染一個特定的 [元件](#components) ，你可能希望僅在元件渲染的首次推送 JavaScript 程式碼到頭部：

```blade
@once
    @push('scripts')
        <script>
            // 你自訂的 JavaScript 程式碼...
        </script>
    @endpush
@endonce
```

由於該 `@once` 指令經常與 `@push` 或 `@prepend` 指令一起使用，為了使用方便，我們提供了 `@pushOnce` 和 `@prependOnce` 指令：

```blade
@pushOnce('scripts')
    <script>
        // 你自訂的 JavaScript 程式碼...
    </script>
@endPushOnce
```

### 原始 PHP 語法

在許多情況下，嵌入 PHP 程式碼到你的檢視中是很有用的。你可以在範本中使用 Blade 的 `@php` 指令執行原生的 PHP 程式碼塊：

```blade
@php
    $counter = 1;
@endphp
```

如果只需要寫一條 PHP 語句，可以在 `@php` 指令中包含該語句。

```blade
@php($counter = 1)
```

### 註釋

Blade 也允許你在檢視中定義註釋。但是，和 HTML 註釋不同， Blade 註釋不會被包含在應用返回的 HTML 中：

```blade
{{-- 這個註釋將不會出現在渲染的HTML中。 --}}
```

## 元件

元件和插槽的作用與區塊和佈局的作用一致；不過，有些人可能覺著元件和插槽更易於理解。有兩種書寫元件的方法：基於類的元件和匿名元件。

你可以使用 `make:component` Artisan 命令來建立一個基於類的元件。我們將會建立一個簡單的  `Alert` 元件用於說明如何使用元件。該 `make:component` 命令將會把元件置於 `App\View\Components` 目錄中：

```shell
php artisan make:component Alert
```

該 `make:component` 命令將會為元件建立一個檢視範本。建立的檢視被置於 `resources/views/components` 目錄中。在為自己的應用程式編寫元件時，會在 `app/View/Components` 目錄和 `resources/views/components` 目錄中自動發現元件，因此通常不需要進一步的元件註冊。

你還可以在子目錄中建立元件：

```shell
php artisan make:component Forms/Input
```

上面的命令將在目錄中建立一個 `Input` 元件， `App\View\Components\Forms` 檢視將放置在 `resources/views/components/forms` 目錄中。

如果你想建立一個匿名元件（一個只有 Blade 範本並且沒有類的元件），你可以在呼叫命令  `make:component` 使用該 `--view` 標誌：

```shell
php artisan make:component forms.input --view
```

上面的命令將在 `resources/views/components/forms/input.blade.php`建立一個 Blade 檔案，該檔案中可以通過 `<x-forms.input />`作為元件呈現。

#### 手動註冊包元件

當為你自己的應用編寫元件的時候，Laravel 將會自動發現位於 `app/View/Components` 目錄和 `resources/views/components` 目錄中的元件。

當然，如果你使用 Blade 元件編譯一個包，你可能需要手動註冊元件類及其 HTML 標籤別名。你應該在包的服務提供者的 `boot` 方法中註冊你的元件：

    use Illuminate\Support\Facades\Blade;

    /**
     * 註冊你的包的服務
     */
    public function boot(): void
    {
        Blade::component('package-alert', Alert::class);
    }

當元件註冊完成後，便可使用標籤別名來對其進行渲染。

```blade
<x-package-alert/>
```

或者，你可以使用該 `componentNamespace` 方法按照約定自動載入元件類。例如，一個 `Nightshade` 包可能有 `Calendar` 和 `ColorPicker` 元件駐留在 `Package\Views\Components` 命名空間中：

    use Illuminate\Support\Facades\Blade;

    /**
     * 註冊你的包的服務
     */
    public function boot(): void
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }


這將允許他們的供應商命名空間使用包元件，使用以下 `package-name::` 語法：

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade 將自動檢測連結到該元件的類，通過對元件名稱進行帕斯卡大小寫。使用「點」表示法也支援子目錄。

### 顯示元件

要顯示一個元件，你可以在 Blade 範本中使用 Blade 元件標籤。 Blade 元件以  `x-` 字串開始，其後緊接元件類 kebab case 形式的名稱（即單詞與單詞之間使用短橫線 `-` 進行連接）：

```blade
<x-alert/>

<x-user-profile/>
```

如果元件位於 `App\View\Components` 目錄的子目錄中，你可以使用 `.` 字元來指定目錄層級。例如，假設我們有一個元件位於 `App\View\Components\Inputs\Button.php`，那麼我們可以像這樣渲染它：

```blade
<x-inputs.button/>
```

如果你想有條件地渲染你的元件，你可以在你的元件類上定義一個 `shouldRender` 方法。如果 `shouldRender` 方法返回 `false`，該元件將不會被渲染。

    use Illuminate\Support\Str;

    /**
     * 該元件是否應該被渲染
     */
    public function shouldRender(): bool
    {
        return Str::length($this->message) > 0;
    }

### 傳遞資料到元件中

你可以使用 HTML 屬性傳遞資料到 Blade 元件中。普通的值可以通過簡單的 HTML 屬性來傳遞給元件。PHP 表示式和變數應該通過以 `:` 字元作為前綴的變數來進行傳遞：

```blade
<x-alert type="error" :message="$message"/>
```

你應該在類的構造器中定義元件的必要資料。在元件的檢視中，元件的所有 public 類型的屬性都是可用的。不必通過元件類的 `render` 方法傳遞：

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;
    use Illuminate\View\View;

    class Alert extends Component
    {
        /**
         * 建立元件實例。
         */
        public function __construct(
            public string $type,
            public string $message,
        ) {}

        /**
         * 獲取代表該元件的檢視/內容
         */
        public function render(): View
        {
            return view('components.alert');
        }
    }



渲染元件時，你可以回顯變數名來顯示元件的 public 變數的內容：

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

#### 命名方式（Casing）

元件的構造器的參數應該使用 `駝峰式` 類型，在 HTML 屬性中引用參數名時應該使用 `短橫線隔開式 kebab-case ：單詞與單詞之間使用短橫線 - 進行連接）` 。例如，給定如下的元件構造器：

    /**
     * 建立一個元件實例
     */
    public function __construct(
        public string $alertType,
    ) {}

`$alertType`  參數可以像這樣使用：

```blade
<x-alert alert-type="danger" />
```

#### 短屬性語法/省略屬性語法

當向元件傳遞屬性時，你也可以使用「短屬性語法/省略屬性語法」（省略屬性書寫）。這通常很方便，因為屬性名稱經常與它們對應的變數名稱相匹配。

```blade
{{-- 短屬性語法/省略屬性語法... --}}
<x-profile :$userId :$name />

{{-- 等價於... --}}
<x-profile :user-id="$userId" :name="$name" />
```

#### 轉義屬性渲染

因為一些 JavaScript 框架，例如 Alpine.js 還可以使用冒號前綴屬性，你可以使用雙冒號 (`::`) 前綴通知 Blade 屬性不是 PHP 表示式。例如，給定以下元件：

```blade
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

Blade 將渲染出以下 HTML 內容：

```blade
<button :class="{ danger: isDeleting }">
    Submit
</button>
```

#### #### 元件方法

除了元件範本可用的公共變數外，還可以呼叫元件上的任何公共方法。例如，假設一個元件有一個 `isSelected` 方法：

    /**
     * 確定給定選項是否為當前選定的選項。
     */
    public function isSelected(string $option): bool
    {
        return $option === $this->selected;
    }



你可以通過呼叫與方法名稱匹配的變數，從元件範本執行此方法：

```blade
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

#### 訪問元件類中的屬性和插槽

Blade 元件還允許你訪問類的 render 方法中的元件名稱、屬性和插槽。但是，為了訪問這些資料，應該從元件的 `render` 方法返回閉包。閉包將接收一個  `$data` 陣列作為它的唯一參數。此陣列將包含幾個元素，這些元素提供有關元件的資訊：

    use Closure;

    /**
     * 獲取表示元件的檢視 / 內容
     */
    public function render(): Closure
    {
        return function (array $data) {
            // $data['componentName'];
            // $data['attributes'];
            // $data['slot'];

            return '<div>Components content</div>';
        };
    }

`componentName` 等於 `x-` 前綴後面的 HTML 標記中使用的名稱。所以 `<x-alert />` 的 `componentName` 將是 `alert` 。 `attributes` 元素將包含 HTML 標記上的所有屬性。 `slot` 元素是一個 `Illuminate\Support\HtmlString`實例，包含元件的插槽內容。

閉包應該返回一個字串。如果返回的字串與現有檢視相對應，則將呈現該檢視；否則，返回的字串將作為內聯 Blade 檢視進行計算。

#### 附加依賴項

如果你的元件需要引入來自 Laravel 的 [服務容器](/docs/laravel/10.x/container)的依賴項，你可以在元件的任何資料屬性之前列出這些依賴項，這些依賴項將由容器自動注入：

```php
use App\Services\AlertCreator;

/**
 * 建立元件實例
 */
public function __construct(
    public AlertCreator $creator,
    public string $type,
    public string $message,
) {}
```



#### 隱藏屬性/方法

如果要防止某些公共方法或屬性作為變數公開給元件範本，可以將它們新增到元件的 `$except` 陣列屬性中：

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;

    class Alert extends Component
    {
        /**
         * 不應向元件範本公開的屬性/方法。
         *
         * @var array
         */
        protected $except = ['type'];

        /**
         * Create the component instance.
         */
        public function __construct(
            public string $type,
        ) {}
    }

### 元件屬性

我們已經研究了如何將資料屬性傳遞給元件；但是，有時你可能需要指定額外的 HTML 屬性，例如  `class`，這些屬性不是元件運行所需的資料的一部分。通常，你希望將這些附加屬性向下傳遞到元件範本的根元素。例如，假設我們要呈現一個 `alert` 元件，如下所示：

```blade
<x-alert type="error" :message="$message" class="mt-4"/>
```

所有不屬於元件的構造器的屬性都將被自動新增到元件的「屬性包」中。該屬性包將通過 `$attributes` 變數自動傳遞給元件。你可以通過回顯這個變數來渲染所有的屬性：

```blade
<div {{ $attributes }}>
    <!-- 元件內容 -->
</div>
```

> **注意：**此時不支援在元件中使用諸如 `@env` 這樣的指令。例如， `<x-alert :live="@env('production')"/>` 不會被編譯。

#### 默認 / 合併屬性

某些時候，你可能需要指定屬性的預設值，或將其他值合併到元件的某些屬性中。為此，你可以使用屬性包的 `merge`方法。 此方法對於定義一組應始終應用於元件的默認 CSS 類特別有用：

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

假設我們如下方所示使用該元件：

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

最終呈現的元件 HTML 將如下所示：

```blade
<div class="alert alert-error mb-4">
    <!-- Contents of the $message variable -->
</div>
```

#### 有條件地合併類

有時你可能希望在給定條件為 `true` 時合併類。 你可以通過該 `class` 方法完成此操作，該方法接受一個類陣列，其中陣列鍵包含你希望新增的一個或多個類，而值是一個布林值表示式。如果陣列元素有一個數字鍵，它將始終包含在呈現的類列表中：

```blade
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

如果需要將其他屬性合併到元件中，可以將 `merge` 方法連結到 `class` 方法中：

```blade
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

> **技巧：**如果你需要有條件地編譯不應接收合併屬性的其他 HTML 元素上的類，你可以使用 [`@class` 指令](#conditional-classes)。

#### 非 class 屬性的合併

當合並非 `class` 屬性的屬性時，提供給 `merge` 方法的值將被視為該屬性的「default」值。但是，與 `class` 屬性不同，這些屬性不會與注入的屬性值合併。相反，它們將被覆蓋。例如， `button` 元件的實現可能如下所示：

```blade
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```



若要使用自訂 `type` 呈現按鈕元件，可以在使用該元件時指定它。如果未指定 `type`，則將使用 `button` 作為 type 值：

```blade
<x-button type="submit">
    Submit
</x-button>
```

本例中 `button` 元件渲染的 HTML 為：

```blade
<button type="submit">
    Submit
</button>
```

如果希望 `class` 以外的屬性將其預設值和注入值連接在一起，可以使用 `prepends` 方法。在本例中， `data-controller` 屬性始終以 `profile-controller` 開頭，並且任何其他注入 `data-controller` 的值都將放在該預設值之後：

```blade
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

#### 保留屬性 / 過濾屬性

可以使用 `filter` 方法篩選屬性。如果希望在屬性包中保留屬性，此方法接受應返回 `true` 的閉包：

```blade
{{ $attributes->filter(fn (string $value, string $key) => $key == 'foo') }}
```

為了方便起見，你可以使用 `whereStartsWith` 方法檢索其鍵以給定字串開頭的所有屬性：

```blade
{{ $attributes->whereStartsWith('wire:model') }}
```

相反，該 `whereDoesntStartWith` 方法可用於排除鍵以給定字串開頭的所有屬性：

```blade
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

使用 `first` 方法，可以呈現給定屬性包中的第一個屬性：

```blade
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

如果要檢查元件上是否存在屬性，可以使用 `has` 方法。此方法接受屬性名稱作為其唯一參數，並返回一個布林值，指示該屬性是否存在：

```blade
@if ($attributes->has('class'))
    <div>Class attribute is present</div>
@endif
```



你可以使用 `get` 方法檢索特定屬性的值：

```blade
{{ $attributes->get('class') }}
```

### 保留關鍵字

默認情況下，為了渲染元件，會保留一些關鍵字供 Blade 內部使用。以下關鍵字不能定義為元件中的公共屬性或方法名稱：

<div class="content-list" markdown="1">

- `data`
- `render`
- `resolveView`
- `shouldRender`
- `view`
- `withAttributes`
- `withName`

</div>

### 插槽

你通常需要通過「插槽」將其他內容傳遞給元件。通過回顯 `$slot` 變數來呈現元件插槽。為了探索這個概念，我們假設 `alert` 元件具有以下內容：

```blade
<!-- /resources/views/components/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

我們可以通過向元件中注入內容將內容傳遞到 `slot` ：

```blade
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

有時候一個元件可能需要在它內部的不同位置放置多個不同的插槽。我們來修改一下 alert 元件，使其允許注入 「title」:

```blade
<!-- /resources/views/components/alert.blade.php -->

<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

你可以使用 `x-slot` 標籤來定義命名插槽的內容。任何沒有在 `x-slot` 標籤中的內容都將傳遞給  `$slot` 變數中的元件：

```xml
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

#### 範疇插槽

如果你使用諸如 Vue 這樣的 JavaScript 框架，那麼你應該很熟悉「範疇插槽」，它允許你從插槽中的元件訪問資料或者方法。 Laravel 中也有類似的用法，只需在你的元件中定義 public 方法或屬性，並且使用 `$component` 變數來訪問插槽中的元件。在此示例中，我們將假設元件在其元件類上定義了 `x-alert` 一個公共方法： `formatAlert`

```blade
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```



#### 插槽屬性

像 Blade 元件一樣，你可以為插槽分配額外的 [屬性](#component-attributes) ，例如 CSS 類名：

```xml
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Heading
    </x-slot>

    Content

    <x-slot:footer class="text-sm">
        Footer
    </x-slot>
</x-card>
```

要與插槽屬性互動，你可以訪問 `attributes` 插槽變數的屬性。有關如何與屬性互動的更多資訊，請參閱有關 [元件屬性](#component-attributes) 的文件：

```blade
@props([
    'heading',
    'footer',
])

<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>

    {{ $slot }}

    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

### 內聯元件檢視

對於小型元件而言，管理元件類和元件檢視範本可能會很麻煩。因此，你可以從 `render` 方法中返回元件的內容：

    /**
     * 獲取元件的檢視 / 內容。
     */
    public function render(): string
    {
        return <<<'blade'
            <div class="alert alert-danger">
                {{ $slot }}
            </div>
        blade;
    }

#### 生成內聯檢視元件

要建立一個渲染內聯檢視的元件，你可以在運行 `make:component` 命令時使用  `inline` ：

```shell
php artisan make:component Alert --inline
```

### 動態元件

有時你可能需要渲染一個元件，但直到執行階段才知道應該渲染哪個元件。在這種情況下, 你可以使用 Laravel 內建的 `dynamic-component` 元件, 根據執行階段的值或變數來渲染元件:

```blade
<x-dynamic-component :component="$componentName" class="mt-4" />
```

### 手動註冊元件

> **注意：**以下關於手動註冊元件的文件主要適用於那些正在編寫包含檢視元件的 Laravel 包的使用者。如果你不是在寫包，這一部分的元件文件可能與你無關。



當為自己的應用程式編寫元件時，元件會在`app/View/Components`目錄和`resources/views/components`目錄下被自動發現。

但是，如果你正在建立一個利用 Blade 元件的包，或者將元件放在非傳統的目錄中，你將需要手動註冊你的元件類和它的 HTML 標籤別名，以便 Laravel 知道在哪裡可以找到這個元件。你通常應該在你的包的服務提供者的`boot`方法中註冊你的元件：

    use Illuminate\Support\Facades\Blade;
    use VendorPackage\View\Components\AlertComponent;

    /**
     * 註冊你的包的服務。
     */
    public function boot(): void
    {
        Blade::component('package-alert', AlertComponent::class);
    }

一旦你的元件被註冊，它就可以使用它的標籤別名進行渲染。

```blade
<x-package-alert/>
```

#### 自動載入包元件

另外，你可以使用`componentNamespace`方法來自動載入元件類。例如，一個`Nightshade`包可能有`Calendar`和`ColorPicker`元件，它們位於`PackageViews\Components`命名空間中。

    use Illuminate\Support\Facades\Blade;

    /**
     * 註冊你的包的服務。
     */
    public function boot(): void
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

這將允許使用`package-name::`語法的供應商名稱空間來使用包的元件。

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade 將通過元件名稱的駝峰式大小寫 (pascal-casing) 自動檢測與該元件連結的類。也支援使用 "點 "符號的子目錄。

### 匿名元件

與行內元件相同，匿名元件提供了一個通過單個檔案管理元件的機制。然而，匿名元件使用的是一個沒有關聯類的單一檢視檔案。要定義一個匿名元件，你只需將 Blade 範本置於 `resources/views/components` 目錄下。例如，假設你在 `resources/views/components/alert.blade.php`中定義了一個元件：

```blade
<x-alert/>
```


如果元件在 `components` 目錄的子目錄中，你可以使用 `.` 字元來指定其路徑。例如，假設元件被定義在 `resources/views/components/inputs/button.blade.php` 中，你可以像這樣渲染它：

```blade
<x-inputs.button/>
```

#### 匿名索引元件

有時，當一個元件由許多 Blade 範本組成時，你可能希望將給定元件的範本分組到一個目錄中。例如，想像一個具有以下目錄結構的「可摺疊」元件：

```none
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

此目錄結構允許你像這樣呈現元件及其項目：

```blade
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

然而，為了通過 `x-accordion` 渲染元件， 我們被迫將「索引」元件範本放置在 `resources/views/components` 目錄中，而不是與其他相關的範本巢狀在 `accordion` 目錄中。

幸運的是，Blade 允許你 `index.blade.php` 在元件的範本目錄中放置檔案。當 `index.blade.php` 元件存在範本時，它將被呈現為元件的「根」節點。因此，我們可以繼續使用上面示例中給出的相同 Blade 語法；但是，我們將像這樣調整目錄結構：

```none
/resources/views/components/accordion/index.blade.php
/resources/views/components/accordion/item.blade.php
```

#### 資料 / 屬性

由於匿名元件沒有任何關聯類，你可能想要區分哪些資料應該被作為變數傳遞給元件，而哪些屬性應該被存放於 [屬性包](#component-attributes)中。



你可以在元件的 Blade 範本的頂層使用 `@props` 指令來指定哪些屬性應該作為資料變數。元件中的其他屬性都將通過屬性包的形式提供。如果你想要為某個資料變數指定一個預設值，你可以將屬性名作為陣列鍵，預設值作為陣列值來實現：

```blade
<!-- /resources/views/components/alert.blade.php -->

@props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

給定上面的元件定義，我們可以像這樣渲染元件：

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

#### 訪問父元件資料

有時你可能希望從子元件中的父元件訪問資料。在這些情況下，你可以使用該 `@aware` 指令。例如，假設我們正在建構一個由父 `<x-menu>` 和 子組成的複雜菜單元件 `<x-menu.item>`：

```blade
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

該 `<x-menu>` 元件可能具有如下實現：

```blade
<!-- /resources/views/components/menu/index.blade.php -->

@props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

因為 `color` 只被傳遞到父級 (`<x-menu>`)中，所以 `<x-menu.item>` 在內部是不可用的。但是，如果我們使用該 `@aware` 指令，我們也可以使其在內部可用 `<x-menu.item>` ：

```blade
<!-- /resources/views/components/menu/item.blade.php -->

@aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

> **注意：**該 `@aware` 指令無法訪問未通過 HTML 屬性顯式傳遞給父元件的父資料。`@aware` 指令 不能訪問未顯式傳遞給父元件的預設值 `@props` 。


### 匿名元件路徑

如前所述，匿名元件通常是通過在你的`resources/views/components`目錄下放置一個 Blade 範本來定義的。然而，你可能偶爾想在 Laravel 註冊其他匿名元件的路徑，除了默認路徑。

`anonymousComponentPath`方法接受匿名元件位置的「路徑」作為它的第一個參數，並接受一個可選的「命名空間」作為它的第二個參數，元件應該被放在這個命名空間下。通常，這個方法應該從你的應用程式的一個[服務提供者](/docs/laravel/10.x/providers) 的`boot`方法中呼叫。

    /**
     * 引導任何應用服務。
     */
    public function boot(): void
    {
        Blade::anonymousComponentPath(__DIR__.'/../components');
    }

當元件路徑被註冊而沒有指定前綴時，就像上面的例子一樣，它們在你的 Blade 元件中可能也沒有相應的前綴。例如，如果一個`panel.blade.php`元件存在於上面註冊的路徑中，它可能會被呈現為這樣。

```blade
<x-panel />
```

前綴「命名空間」可以作為第二個參數提供給`anonymousComponentPath`方法。

    Blade::anonymousComponentPath(__DIR__.'/../components', 'dashboard');

當提供一個前綴時，在該「命名空間」內的元件可以在渲染時將該元件的命名空間前綴到該元件的名稱。

```blade
<x-dashboard::panel />
```

## 建構佈局

### 使用元件佈局

大多數 web 應用程式在不同的頁面上有相同的總體佈局。如果我們必須在建立的每個檢視中重複整個佈局 HTML，那麼維護我們的應用程式將變得非常麻煩和困難。謝天謝地，將此佈局定義為單個 [Blade 元件](#components) 並在整個應用程式中非常方便地使用它。


#### 定義佈局元件

例如，假設我們正在建構一個「todo list」應用程式。我們可以定義如下所示的 `layout` 元件：

```blade
<!-- resources/views/components/layout.blade.php -->

<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

#### 應用佈局元件

一旦定義了 `layout` 元件，我們就可以建立一個使用該元件的 Blade 檢視。在本例中，我們將定義一個顯示任務列表的簡單檢視：

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    @foreach ($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

請記住，注入到元件中的內容將提供給 `layout` 元件中的默認 `$slot` 變數。正如你可能已經注意到的，如果提供了 `$title` 插槽，那麼我們的 `layout` 也會尊從該插槽；否則，將顯示默認的標題。我們可以使用元件文件中討論的標準槽語法從任務列表檢視中插入自訂標題。 我們可以使用[元件文件](#components)中討論的標準插槽語法從任務列表檢視中注入自訂標題：

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    <x-slot:title>
        Custom Title
    </x-slot>

    @foreach ($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

現在我們已經定義了佈局和任務列表檢視，我們只需要從路由中返回 `task` 檢視即可：

    use App\Models\Task;

    Route::get('/tasks', function () {
        return view('tasks', ['tasks' => Task::all()]);
    });

### 使用範本繼承進行佈局

#### 定義一個佈局

佈局也可以通過 「範本繼承」 建立。在引入 [元件](#components) 之前，這是建構應用程式的主要方法。



讓我們看一個簡單的例子做開頭。首先，我們將檢查頁面佈局。由於大多數 web 應用程式在不同的頁面上保持相同的總體佈局，因此將此佈局定義為單一檢視非常方便：

```blade
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            這是一個主要的側邊欄
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

如你所見，此檔案包含經典的 HTML 標記。但是，請注意 `@section` 和 `@yield` 指令。顧名思義， `@section` 指令定義內容的一部分，而 `@yield` 指令用於顯示給定部分的內容。

現在我們已經為應用程式定義了一個佈局，讓我們定義一個繼承該佈局的子頁面。

#### 繼承佈局

定義子檢視時，請使用 `@extends` Blade 指令指定子檢視應「繼承」的佈局。擴展 Blade 佈局的檢視可以使用 `@section` 指令將內容注入佈局的節點中。請記住，如上面的示例所示，這些部分的內容將使用 `@yield` 顯示在佈局中：

```blade
<!-- resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

在本例中，`sidebar` 部分使用 `@parent`  指令將內容追加（而不是覆蓋）到局部的側欄位置。在呈現檢視時， `@parent` 指令將被佈局的內容替換。

> **技巧：**與前面的示例相反，本 `sidebar` 節以 `@endsection` 結束，而不是以 `@show` 結束。 `@endsection` 指令將只定義一個節，`@show` 將定義並 **立即 yield** 該節。


該 `@yield` 指令還接受預設值作為其第二個參數。如果要生成的節點未定義，則將呈現此內容：

```blade
@yield('content', 'Default content')
```

## 表單

### CSRF 欄位

無論何時在應用程式中定義 HTML 表單，都應該在表單中包含一個隱藏的 CSRF 令牌欄位，以便 [CSRF 保護中介軟體](/docs/laravel/10.x/csrf) 可以驗證請求。你可以使用 `@csrf` Blade 指令生成令牌欄位：

```blade
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

### Method 欄位

由於 HTML 表單不能發出 `PUT`、`PATCH`或 `DELETE` 請求，因此需要新增一個隱藏的 `_method` 欄位來欺騙這些 HTTP 動詞。 `@method` Blade 指令可以為你建立此欄位：

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

### 表單校驗錯誤

該 `@error` 指令可用於快速檢查給定屬性是否存在 [驗證錯誤消息](/docs/laravel/10.x/validation#quick-displaying-the-validation-errors) 。在 `@error` 指令中，可以回顯 `$message` 變數以顯示錯誤消息：

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title"
    type="text"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

由於該 `@error` 指令編譯為「if」語句，因此你可以在 `@else` 屬性沒有錯誤時使用該指令來呈現內容：

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input id="email"
    type="email"
    class="@error('email') is-invalid @else is-valid @enderror">
```



你可以將 [特定錯誤包的名稱](/docs/laravel/10.x/validation#named-error-bags) 作為第二個參數傳遞給 `@error` 指令，以便在包含多個表單的頁面上檢索驗證錯誤消息：

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input id="email"
    type="email"
    class="@error('email', 'login') is-invalid @enderror">

@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

## 堆疊

Blade 允許你推送到可以在其他檢視或佈局中的其他地方渲染的命名堆疊。這對於指定子檢視所需的任何 JavaScript 庫特別有用：

```blade
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

如果你想在給定的布林值表示式評估為 `true` 時 `@push` 內容，你可以使用 `@pushIf` 指令。

```blade
@pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
@endPushIf
```

你可以根據需要多次推入堆疊。要呈現完整的堆疊內容，請將堆疊的名稱傳遞給 `@stack` 指令：

```blade
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

如果要將內容前置到堆疊的開頭，應使用 `@prepend` 指令：

```blade
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```

## 服務注入

該 `@inject` 指令可用於從 Laravel [服務容器](/docs/laravel/10.x/container)中檢索服務。傳遞給 `@inject` 的第一個參數是要將服務放入的變數的名稱，而第二個參數是要解析的服務的類或介面名稱：

```blade
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```



## 渲染內聯 Blade 範本

有時你可能需要將原始 Blade 範本字串轉換為有效的 HTML。你可以使用 `Blade` 門面提供的 `render` 方法來完成此操作。該 `render` 方法接受 Blade 範本字串和提供給範本的可選資料陣列：

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

Laravel 通過將內聯 Blade 範本寫入 `storage/framework/views` 目錄來呈現它們。如果你希望 Laravel 在渲染 Blade 範本後刪除這些臨時檔案，你可以為 `deleteCachedView` 方法提供參數：

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);
```

## 渲染 Blade 片段

當使用 [Turbo](https://turbo.hotwired.dev/) 和 [htmx](https://htmx.org/) 等前端框架時，你可能偶爾需要在你的HTTP響應中只返回Blade範本的一個部分。Blade「片段（fragment）」允許你這樣做。要開始，將你的Blade範本的一部分放在`@fragment`和`@endfragment`指令中。

```blade
@fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

然後，在渲染使用該範本的檢視時，你可以呼叫 `fragment` 方法來指定只有指定的片段應該被包含在傳出的 HTTP 響應中。

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

`fragmentIf` 方法允許你根據一個給定的條件有條件地返回一個檢視的片段。否則，整個檢視將被返回。

```php
return view('dashboard', ['users' => $users])
    ->fragmentIf($request->hasHeader('HX-Request'), 'user-list');
```



`fragments` 和 `fragmentsIf` 方法允許你在響應中返回多個檢視片段。這些片段將被串聯起來。

```php
view('dashboard', ['users' => $users])
    ->fragments(['user-list', 'comment-list']);

view('dashboard', ['users' => $users])
    ->fragmentsIf(
        $request->hasHeader('HX-Request'),
        ['user-list', 'comment-list']
    );
```

## 擴展 Blade

Blade 允許你使用 `directive` 方法定義自己的自訂指令。當 Blade 編譯器遇到自訂指令時，它將使用該指令包含的表示式呼叫提供的回呼。

下面的示例建立了一個 `@datetime($var)` 指令，該指令格式化給定的 `$var`，它應該是 `DateTime` 的一個實例：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 註冊應用的服務
         */
        public function register(): void
        {
            // ...
        }

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Blade::directive('datetime', function (string $expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }
    }

正如你所見，我們將 `format` 方法應用到傳遞給指令中的任何表示式上。因此，在本例中，此指令生成的最終 PHP 將是：

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> **注意：**更新 Blade 指令的邏輯後，需要刪除所有快取的 Blade 檢視。可以使用 `view:clear` Artisan 命令。

### 自訂回顯處理程序

如果你試圖使用 Blade 來「回顯」一個對象， 該對象的 `__toString` 方法將被呼叫。該[`__toString`](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring) 方法是 PHP 內建的「魔術方法」之一。但是，有時你可能無法控制 `__toString` 給定類的方法，例如當你與之互動的類屬於第三方庫時。

在這些情況下，Blade 允許您為該特定類型的對象註冊自訂回顯處理程序。為此，您應該呼叫 Blade 的 `stringable` 方法。該 `stringable` 方法接受一個閉包。這個閉包類型應該提示它負責呈現的對象的類型。通常，應該在應用程式的 `AppServiceProvider` 類的 `boot` 方法中呼叫該 `stringable` 方法：

    use Illuminate\Support\Facades\Blade;
    use Money\Money;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::stringable(function (Money $money) {
            return $money->formatTo('en_GB');
        });
    }

定義自訂回顯處理程序後，您可以簡單地回顯 Blade 範本中的對象：

```blade
Cost: {{ $money }}
```

### 自訂 if 聲明

在定義簡單的自訂條件語句時，編寫自訂指令通常比較複雜。因此，Blade 提供了一個 Blade::if 方法，允許你使用閉包快速定義自訂條件指令。例如，讓我們定義一個自訂條件來檢查為應用程式組態的默認 「儲存」。我們可以在 AppServiceProvider 的 boot 方法中執行此操作：

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::if('disk', function (string $value) {
            return config('filesystems.default') === $value;
        });
    }

一旦定義了自訂條件，就可以在範本中使用它:

```blade
@disk('local')
    <!-- The application is using the local disk... -->
@elsedisk('s3')
    <!-- The application is using the s3 disk... -->
@else
    <!-- The application is using some other disk... -->
@enddisk

@unlessdisk('local')
    <!-- The application is not using the local disk... -->
@enddisk
```

