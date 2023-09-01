# API 資源

## 簡介

在建構 API 時，你往往需要一個轉換層來聯結你的 Eloquent 模型和實際返回給使用者的 JSON 響應。比如，你可能希望顯示部分使用者屬性而不是全部，或者你可能希望在模型的 JSON 中包括某些關係。Eloquent 的資源類能夠讓你以更直觀簡便的方式將模型和模型集合轉化成 JSON。

當然，你可以始終使用 Eloquent 模型或集合的 `toJson` 方法將其轉換為 JSON ；但是，Eloquent 的資源提供了對模型及其關係的 JSON 序列化更加精細和更加健壯的控制。

## 生成資源

你可以使用 `make:resource` artisan 命令來生成一個資源類。默認情況下，資源將放在應用程式的 `app/Http/Resources` 目錄下。資源繼承自 `Illuminate\Http\Resources\Json\JsonResource` 類：

```shell
php artisan make:resource UserResource
```

#### 資源集合

除了生成轉換單個模型的資源之外，還可以生成負責轉換模型集合的資源。這允許你的 JSON 包含與給定資源的整個集合相關的其他資訊。

你應該在建立資源集合時使用 `--collection` 標誌來表明你要生成一個資源集合。或者，在資源名稱中包含 Collection 一詞將向 Laravel 表明它應該生成一個資源集合。資源集合繼承自 `Illuminate\Http\Resources\Json\ResourceCollection` 類：

```shell
php artisan make:resource User --collection

php artisan make:resource UserCollection
```

## 概念綜述

> **提示**  
> 提示：這是對資源和資源集合的高度概述。強烈建議您閱讀本文件的其他部分，以深入瞭解如何更好地自訂和使用資源。

在深入瞭解如何定製化編寫你的資源之前，讓我們首先從高層次上瞭解 Laravel 中如何使用資源。一個資源類表示一個單一模型需要被轉換成 JSON 格式。例如，下面是一個簡單的 `UserResource` 資源類：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * 將資源轉換為陣列。
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

每個資源類都定義了一個 `toArray` 方法，當資源從路由或 controller 方法作為響應被呼叫返回時，該方法返回應該轉換為 JSON 的屬性陣列。

注意，我們可以直接使用 `$this` 變數訪問模型屬性。這是因為資源類將自動代理屬性和方法訪問到底層模型以方便訪問。一旦定義了資源，你可以從路由或 controller 中呼叫並返回它。資源通過其建構函式接受底層模型實例：

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user/{id}', function (string $id) {
        return new UserResource(User::findOrFail($id));
    });



### 資源集合

如果你要返回一個資源集合或一個分頁響應，你應該在路由或 controller 中建立資源實例時使用你的資源類提供的 `collection` 方法：

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/users', function () {
        return UserResource::collection(User::all());
    });

當然了，使用如上方法你將不能新增任何附加的中繼資料和集合一起返回。如果你需要自訂資源集合響應，你需要建立一個專用的資源來表示集合：

```shell
php artisan make:resource UserCollection
```

此時，你就可以輕鬆地自訂響應中應該包含的任何中繼資料：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 將資源集合轉換為陣列。
         *
         * @return array<int|string, mixed>
         */
        public function toArray(Request $request): array
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

你可以在路由或者 controller 中返回已定義的資源集合：

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

#### 保護集合的鍵

當從路由返回一個資源集合時，Laravel 會重設集合的鍵，使它們按數字順序排列。但是，你可以在資源類中新增 `preserveKeys` 屬性，以指示是否應該保留集合的原始鍵：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * 指示是否應保留資源的集合原始鍵。
         *
         * @var bool
         */
        public $preserveKeys = true;
    }



如果 `preserveKeys` 屬性設定為 `true` ，那麼從路由或 controller 返回集合時，集合的鍵將會被保留：

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/users', function () {
        return UserResource::collection(User::all()->keyBy->id);
    });

#### 自訂基礎資源類

通常，資源集合的 `$this->collection` 屬性會被自動填充，結果是將集合的每個項對應到其單個資源類。單個資源類被假定為資源的類名，但沒有類名末尾的 `Collection` 部分。 此外，根據您的個人偏好，單個資源類可以帶著後綴 `Resource` ，也可以不帶。

例如，`UserCollection` 會嘗試將給定的使用者實例對應到 `User` 或 `UserResource` 資源。想要自訂該行為，你可以重寫資源集合中的 `$collects` 屬性指定自訂的資源：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 自訂資源類名。
         *
         * @var string
         */
        public $collects = Member::class;
    }

## 編寫資源

> **Note**  
> 技巧：如果您還沒有閱讀 [概念綜述](#concept-overview)，那麼在繼續閱讀本文件前，強烈建議您去閱讀一下，會更容易理解本節的內容。

從本質上說，資源的作用很簡單，它只需將一個給定的模型轉換為一個陣列。因此，每個資源都包含一個 `toArray` 方法，這個方法會將模型的屬性轉換為一個 API 友好的陣列，然後將該陣列通過路由或 controller 返回給使用者：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * 將資源轉換為陣列。
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }



一旦資源被定義，它可以直接從路由或 controller 返回：

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user/{id}', function (string $id) {
        return new UserResource(User::findOrFail($id));
    });

#### 關聯關係

如果你想在你的響應中包含關聯的資源，你可以將它們新增到你的資源的 `toArray` 方法返回的陣列中。在下面的例子中，我們將使用 `PostResource` 資源的 `collection` 方法來將使用者的部落格文章新增到資源響應中：

    use App\Http\Resources\PostResource;
    use Illuminate\Http\Request;

    /**
     * 將資源轉換為陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => PostResource::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

> 注意：如果你只希望在已經載入的關聯關係中包含它們，點這裡查看 [條件關聯](#conditional-relationships)。

#### 資源集合

當資源將單個模型轉換為陣列時，資源集合將模型集合轉換為陣列。當然，你並不是必須要為每個類都定義一個資源集合類，因為所有的資源都提供了一個 `collection` 方法來動態地生成一個「臨時」資源集合：

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/users', function () {
        return UserResource::collection(User::all());
    });

當然，如果你需要自訂資源集合返回的中繼資料，那就需要自己建立資源集合類：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 將資源集合轉換為陣列。
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }


與單一資源一樣，資源集合可以直接從路由或 controller 返回:

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

### 封包裹

默認情況下，當資源響應被轉換為 JSON 時，最外層的資源被包裹在 `data` 鍵中。因此一個典型的資源收集響應如下所示:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ]
}
```

如果你想使用自訂鍵而不是 `data`，你可以在資源類上定義一個 `$wrap` 屬性:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * 應該應用的「資料」包裝器。
         *
         * @var string|null
         */
        public static $wrap = 'user';
    }

如果你想停用最外層資源的包裹，你應該呼叫基類 `Illuminate\Http\Resources\Json\JsonResource` 的 `withoutWrapping` 方法。通常，你應該從你的 `AppServiceProvider` 或其他在程序每一個請求中都會被載入的 [服務提供者](/docs/laravel/10.x/providers) 中呼叫這個方法:

    <?php

    namespace App\Providers;

    use Illuminate\Http\Resources\Json\JsonResource;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 註冊任何應用程式服務。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 引導任何應用程式服務。
         */
        public function boot(): void
        {
            JsonResource::withoutWrapping();
        }
    }

> **注意**  
> `withoutWrapping` 方法只會停用最外層資源的包裹，不會刪除你手動新增到資源集合中的 `data` 鍵。



和單個資源一樣，你可以在路由或 controller 中直接返回資源集合：

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

### 封包裹

默認情況下，當資源響應被轉換為 JSON 時，最外層的資源被包裹在 `data` 鍵中。因此一個典型的資源收集響應如下所示：

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ]
}
```

如果你想使用自訂鍵而不是 `data`，你可以在資源類上定義一個 `$wrap` 屬性：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * 應該應用的「資料」包裝器。
         *
         * @var string|null
         */
        public static $wrap = 'user';
    }

如果你想停用最外層資源的包裹，你應該呼叫基類 `Illuminate\Http\Resources\Json\JsonResource` 的 `withoutWrapping` 方法。通常，你應該從你的 `AppServiceProvider` 或其他在程序每一個請求中都會被載入的 [服務提供者](/docs/laravel/10.x/providers) 中呼叫這個方法：

    <?php

    namespace App\Providers;

    use Illuminate\Http\Resources\Json\JsonResource;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 註冊任何應用程式服務。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 引導任何應用程式服務。
         */
        public function boot(): void
        {
            JsonResource::withoutWrapping();
        }
    }
> **注意**
>`withoutWrapping` 方法只會停用最外層資源的包裹，不會刪除你手動新增到資源集合中的 `data` 鍵。


#### 包裹巢狀資源

你可以完全自由地決定資源關聯如何被包裹。如果你希望無論怎樣巢狀，所有的資源集合都包裹在一個 `data` 鍵中，你應該為每個資源定義一個資源集合類，並將返回的集合包裹在 `data` 鍵中。

你可能會擔心這是否會導致最外層的資源包裹在兩層 `data` 鍵中。別擔心， Laravel 永遠不會讓你的資源被雙層包裹，所以你不必擔心資源集合被多重巢狀的問題：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class CommentsCollection extends ResourceCollection
    {
        /**
         * 將資源集合轉換成陣列。
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
        {
            return ['data' => $this->collection];
        }
    }

#### 封包裹和分頁

當通過資源響應返回分頁集合時，即使你呼叫了 `withoutWrapping` 方法，Laravel 也會將你的資源封包裹在 `data` 鍵中。這是因為分頁響應總會有 `meta` 和 `links` 鍵包含關於分頁狀態的資訊：

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

### 分頁

你可以將 Laravel 分頁實例傳遞給資源的 `collection` 方法或自訂資源集合：

    use App\Http\Resources\UserCollection;
    use App\Models\User;

    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });


分頁響應中總有 `meta` 和 `links` 鍵包含著分頁狀態資訊：

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

### 條件屬性

有些時候，你可能希望在給定條件滿足時新增屬性到資源響應裡。例如，你可能希望如果當前使用者是「管理員」時新增某個值到資源響應中。在這種情況下 Laravel 提供了一些輔助方法來幫助你解決問題。`when` 方法可以被用來有條件地向資源響應新增屬性：

    /**
     * 將資源轉換成陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when($request->user()->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

在上面這個例子中，只有當 `isAdmin` 方法返回 `true` 時，`secret` 鍵才會最終在資源響應中被返回。如果該方法返回 `false` 鍵將會在資源響應被傳送給客戶端之前被刪除。 `when` 方法可以使你避免使用條件語句拼接陣列，轉而用更優雅的方式來編寫你的資源。

`when` 方法也接受閉包作為其第二個參數，只有在給定條件為 `true` 時，才從閉包中計算返回的值：

    'secret' => $this->when($request->user()->isAdmin(), function () {
        return 'secret-value';
    }),


 `whenHas` 方法可以用來包含一個屬性，如果它確實存在於底層模型中：

    'name' => $this->whenHas('name'),

此外， `whenNotNull` 方法可用於在資源響應中包含一個屬性，如果該屬性不為空：

    'name' => $this->whenNotNull($this->name),

#### 有條件地合併資料

有些時候，你可能希望在給定條件滿足時新增多個屬性到資源響應裡。在這種情況下，你可以使用 `mergeWhen` 方法在給定的條件為 `true` 時將多個屬性新增到響應中：

    /**
     * 將資源轉換成陣列
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen($request->user()->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

同理，如果給定的條件為 `false` 時，則這些屬性將會在資源響應被傳送給客戶端之前被移除。

> **注意**  
>  `mergeWhen` 方法不應該被使用在混合字串和數字鍵的陣列中。此外，它也不應該被使用在不按順序排列的數字鍵的陣列中。

### 條件關聯

除了有條件地載入屬性之外，你還可以根據模型關聯是否已載入來有條件地在你的資源響應中包含關聯。這允許你在 controller 中決定載入哪些模型關聯，這樣你的資源可以在模型關聯被載入後才新增它們。最終，這樣做可以使你的資源輕鬆避免「N+1」查詢問題。



如果屬性確實存在於模型中，可以用`whenHas`來獲取：

    'name' => $this->whenHas('name'),

此外，`whenNotNull`可用於在資源響應中獲取一個不為空的屬性：

    'name' => $this->whenNotNull($this->name),

#### 有條件地合併資料

有些時候，你可能希望在給定條件滿足時新增多個屬性到資源響應裡。在這種情況下，你可以使用 `mergeWhen` 方法在給定的條件為 `true` 時將多個屬性新增到響應中：

    /**
     * 將資源轉換成陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen($request->user()->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

同理，如果給定的條件為 `false` 時，則這些屬性將會在資源響應被傳送給客戶端之前被移除。

> **注意**
>`mergeWhen` 方法不應該被使用在混合字串和數字鍵的陣列中。此外，它也不應該被使用在不按順序排列的數字鍵的陣列中。

### 條件關聯

除了有條件地載入屬性之外，你還可以根據模型關聯是否已載入來有條件地在你的資源響應中包含關聯。這允許你在 controller 中決定載入哪些模型關聯，這樣你的資源可以在模型關聯被載入後才新增它們。最終，這樣做可以使你的資源輕鬆避免「N+1」查詢問題。


可以使用`whenLoaded`方法來有條件的載入關聯。為了避免載入不必要的關聯，此方法接受關聯的名稱而不是關聯本身作為其參數：

    use App\Http\Resources\PostResource;

    /**
     * 將資源轉換成陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => PostResource::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

在上面這個例子中，如果關聯沒有被載入，則`posts`鍵將會在資源響應被傳送給客戶端之前被刪除。

#### 條件關係計數

除了有條件地包含關係之外，你還可以根據關係的計數是否已載入到模型上，有條件地包含資源響應中的關係「計數」:

    new UserResource($user->loadCount('posts'));

`whenCounted`方法可用於在資源響應中有條件地包含關係的計數。該方法避免在關係計數不存在時不必要地包含屬性:

    /**
     * 將資源轉換為一個陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts_count' => $this->whenCounted('posts'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

在這個例子中，如果`posts`關係的計數還沒有載入，`posts_count`鍵將在資源響應傳送到客戶端之前從資源響應中刪除。

#### 條件中間表資訊



除了在你的資源響應中有條件地包含關聯外，你還可以使用 `whenPivotLoaded` 方法有條件地從多對多關聯的中間表中新增資料。`whenPivotLoaded` 方法接受的第一個參數為中間表的名稱。第二個參數是一個閉包，它定義了在模型上如果中間表資訊可用時要返回的值：

    /**
     * 將資源轉換成陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_user', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

如果你的關聯使用的是 [自訂中間表](/docs/laravel/10.x/eloquent-relationships#defining-custom-intermediate-table-models)，你可以將中間表模型的實例作為 `whenPivotLoaded` 方法的第一個參數:

    'expires_at' => $this->whenPivotLoaded(new Membership, function () {
        return $this->pivot->expires_at;
    }),

如果你的中間表使用的是 `pivot` 以外的訪問器，你可以使用 `whenPivotLoadedAs` 方法：

    /**
     * 將資源轉換成陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
                return $this->subscription->expires_at;
            }),
        ];
    }

### 新增中繼資料

一些 JSON API 標準需要你在資源和資源集合響應中新增中繼資料。這通常包括資源或相關資源的 `links` ，或一些關於資源本身的中繼資料。如果你需要返回有關資源的其他中繼資料，只需要將它們包含在 `toArray` 方法中即可。例如在轉換資源集合時你可能需要新增 `link` 資訊：

    /**
     * 將資源轉換成陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }


當新增額外的中繼資料到你的資源中時，你不必擔心會覆蓋 Laravel 在返回分頁響應時自動新增的 `links` 或 `meta` 鍵。你新增的任何其他 `links` 會與分頁響應新增的 `links` 相合併。

#### 頂層中繼資料

有時候，你可能希望當資源被作為頂層資源返回時新增某些中繼資料到資源響應中。這通常包括整個響應的元資訊。你可以在資源類中新增 `with` 方法來定義中繼資料。此方法應返回一個中繼資料陣列，當資源被作為頂層資源渲染時，這個陣列將會被包含在資源響應中：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 將資源集合轉換成陣列。
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
        {
            return parent::toArray($request);
        }

        /**
         * 返回應該和資源一起返回的其他資料陣列。
         *
         * @return array<string, mixed>
         */
        public function with(Request $request): array
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }

#### 構造資源時新增中繼資料

你還可以在路由或者 controller 中構造資源實例時新增頂層資料。所有資源都可以使用 `additional` 方法來接受應該被新增到資源響應中的資料陣列：

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);



## 響應資源

就像你知道的那樣，資源可以直接在路由和 controller 中被返回：

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user/{id}', function (string $id) {
        return new UserResource(User::findOrFail($id));
    });

但有些時候，在傳送給客戶端前你可能需要自訂 HTTP 響應。你有兩種辦法。第一，你可以鏈式呼叫 `response` 方法。此方法將會返回 `Illuminate\Http\JsonResponse` 實例，允許你自訂響應頭資訊：

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

另外，你還可以在資源中定義一個 `withResponse` 方法。此方法將會在資源被作為頂層資源在響應時被呼叫：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\JsonResponse;
    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * 將資源轉換為陣列。
         *
         * @return array<string, mixed>
         */
        public function toArray(Request $request): array
        {
            return [
                'id' => $this->id,
            ];
        }

        /**
         * 自訂響應資訊。
         */
        public function withResponse(Request $request, JsonResponse $response): void
        {
            $response->header('X-Value', 'True');
        }
    }

