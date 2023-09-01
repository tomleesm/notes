# HTTP 測試

## 簡介

Laravel 提供了一個非常流暢的 API，用於嚮應用程序發出 HTTP 請求並檢查響應。例如，看看下面定義的特性測試：

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * 基本功能測試示例。
         */
        public function test_a_basic_request(): void
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

`get` 方法嚮應用程序發出 `Get` 請求，而 `assertStatus` 方法則斷言返回的響應應該具有給定的 HTTP 狀態程式碼。除了這個簡單的斷言之外，Laravel 還包含各種用於檢查響應頭、內容、JSON 結構等的斷言。

## 建立請求

要嚮應用程序發出請求，可以在測試中呼叫`get`、`post`、`put`、`patch`或`delete`方法。這些方法實際上不會嚮應用程序發出「真正的」HTTP 請求。相反，整個網路請求是在內部模擬的。

測試請求方法不返回`Illuminate\Http\Response`實例，而是返回`Illuminate\Testing\TestResponse`實例，該實例提供[各種有用的斷言](##available-assertions),允許你檢查應用程式的響應：

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * 基本功能測試示例。
         */
        public function test_a_basic_request(): void
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

通常，你的每個測試應該只向你的應用發出一個請求。如果在單個測試方法中執行多個請求，則可能會出現意外行為。

> **技巧**
> 為了方便起見，運行測試時會自動停用 CSRF 中介軟體。

### 自訂要求標頭

你可以使用此 `withHeaders` 方法自訂請求的標頭，然後再將其傳送到應用程式。這使你可以將任何想要的自訂標頭新增到請求中：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * 基本功能測試示例。
         */
        public function test_interacting_with_headers(): void
        {
            $response = $this->withHeaders([
                'X-Header' => 'Value',
            ])->post('/user', ['name' => 'Sally']);

            $response->assertStatus(201);
        }
    }

### Cookies

在傳送請求前你可以使用 `withCookie` 或 `withCookies` 方法設定 cookie。`withCookie` 接受 cookie 的名稱和值這兩個參數，而 `withCookies` 方法接受一個名稱 / 值對陣列：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_interacting_with_cookies(): void
        {
            $response = $this->withCookie('color', 'blue')->get('/');

            $response = $this->withCookies([
                'color' => 'blue',
                'name' => 'Taylor',
            ])->get('/');
        }
    }

###  session  (Session) / 認證 (Authentication)

Laravel 提供了幾個可在 HTTP 測試時使用 Session 的輔助函數。首先，你需要傳遞一個陣列給 `withSession` 方法來設定 session 資料。這樣在應用程式的測試請求傳送之前，就會先去給資料載入 session：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_interacting_with_the_session(): void
        {
            $response = $this->withSession(['banned' => false])->get('/');
        }
    }

Laravel 的 session 通常用於維護當前已驗證使用者的狀態。因此，`actingAs` 方法提供了一種將給定使用者作為當前使用者進行身份驗證的便捷方法。例如，我們可以使用一個[工廠模式](/docs/laravel/10.x/eloquent-factories)來生成和認證一個使用者：

    <?php

    namespace Tests\Feature;

    use App\Models\User;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_an_action_that_requires_authentication(): void
        {
            $user = User::factory()->create();

            $response = $this->actingAs($user)
                             ->withSession(['banned' => false])
                             ->get('/');
        }
    }

你也可以通過傳遞看守器名稱作為 `actingAs` 方法的第二參數以指定使用者通過哪種看守器來認證。提供給 `actingAs` 方法的防護也將成為測試期間的默認防護。

    $this->actingAs($user, 'web')

### 偵錯響應

在向你的應用程式發出測試請求之後，可以使用 `dump`、`dumpHeaders` 和 `dumpSession` 方法來檢查和偵錯響應內容：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * 基本功能測試示例。
         */
        public function test_basic_test(): void
        {
            $response = $this->get('/');

            $response->dumpHeaders();

            $response->dumpSession();

            $response->dump();
        }
    }

或者，你可以使用 `dd`、`ddHeaders` 和 `ddSession` 方法轉儲有關響應的資訊，然後停止執行：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * 基本功能測試示例。
         */
        public function test_basic_test(): void
        {
            $response = $this->get('/');

            $response->ddHeaders();

            $response->ddSession();

            $response->dd();
        }
    }

### 異常處理

有時你可能想要測試你的應用程式是否引發了特定異常。為了確保異常不會被 Laravel 的異常處理程序捕獲並作為 HTTP 響應返回，可以在發出請求之前呼叫 `withoutExceptionHandling` 方法：

    $response = $this->withoutExceptionHandling()->get('/');

此外，如果想確保你的應用程式沒有使用 PHP 語言或你的應用程式正在使用的庫已棄用的功能，你可以在發出請求之前呼叫 `withoutDeprecationHandling` 方法。停用棄用處理時，棄用警告將轉換為異常，從而導致你的測試失敗：

    $response = $this->withoutDeprecationHandling()->get('/');

## 測試 JSON APIs

Laravel 也提供了幾個輔助函數來測試 JSON APIs 和其響應。例如，`json`、`getJson`、`postJson`、`putJson`、`patchJson`、`deleteJson` 以及 `optionsJson` 可以被用於傳送各種 HTTP 動作。你也可以輕鬆地將資料和要求標頭傳遞到這些方法中。首先，讓我們實現一個測試示例，傳送 `POST` 請求到 `/api/user`，並斷言返回的期望資料：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * 基本功能測試示例。
         */
        public function test_making_an_api_request(): void
        {
            $response = $this->postJson('/api/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

此外，JSON 響應資料可以作為響應上的陣列變數進行訪問，從而使你可以方便地檢查 JSON 響應中返回的各個值：

    $this->assertTrue($response['created']);

> **技巧**
> `assertJson` 方法將響應轉換為陣列，並利用 `PHPUnit::assertArraySubset` 驗證給定陣列是否存在於應用程式返回的 JSON 響應中。因此，如果 JSON 響應中還有其他屬性，則只要存在給定的片段，此測試仍將通過。

#### 驗證 JSON 完全匹配

如前所述，`assertJson` 方法可用於斷言 JSON 響應中存在 JSON 片段。如果你想驗證給定陣列是否與應用程式返回的 JSON **完全匹配**，則應使用 `assertExactJson` 方法：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * 基本功能測試示例。
         */
        public function test_asserting_an_exact_json_match(): void
        {
            $response = $this->postJson('/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

#### 驗證 JSON 路徑

如果你想驗證 JSON 響應是否包含指定路徑上的某些給定資料，可以使用 `assertJsonPath` 方法：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * 基本功能測試示例。
         */
        public function test_asserting_a_json_paths_value(): void
        {
            $response = $this->postJson('/user', ['name' => 'Sally']);

            $response
                ->assertStatus(201)
                ->assertJsonPath('team.owner.name', 'Darian');
        }
    }

`assertJsonPath` 方法也接受一個閉包，可以用來動態地確定斷言是否應該通過。

    $response->assertJsonPath('team.owner.name', fn (string $name) => strlen($name) >= 3);

### JSON 流式測試

Laravel 還提供了一種漂亮的方式來流暢地測試應用程式的 JSON 響應。首先，將閉包傳遞給 `assertJson` 方法。這個閉包將使用 `Illuminate\Testing\Fluent\AssertableJson` 的實例呼叫，該實例可用於對應用程式返回的 JSON 進行斷言。 `where` 方法可用於對 JSON 的特定屬性進行斷言，而 `missing` 方法可用於斷言 JSON 中缺少特定屬性：

    use Illuminate\Testing\Fluent\AssertableJson;

    /**
     * 基本功能測試示例。
     */
    public function test_fluent_json(): void
    {
        $response = $this->getJson('/users/1');

        $response
            ->assertJson(fn (AssertableJson $json) =>
                $json->where('id', 1)
                     ->where('name', 'Victoria Faith')
                     ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                     ->whereNot('status', 'pending')
                     ->missing('password')
                     ->etc()
            );
    }

#### 瞭解 `etc` 方法

在上面的例子中, 你可能已經注意到我們在斷言鏈的末端呼叫了 `etc` 方法. 這個方法通知Laravel，在JSON對象上可能還有其他的屬性存在。如果沒有使用 `etc` 方法, 如果你沒有對JSON對象的其他屬性進行斷言, 測試將失敗.

這種行為背後的意圖是保護你不會在你的 JSON 響應中無意地暴露敏感資訊，因為它迫使你明確地對該屬性進行斷言或通過 `etc` 方法明確地允許額外的屬性。

然而，你應該知道，在你的斷言鏈中不包括 `etc` 方法並不能確保額外的屬性不會被新增到巢狀在 JSON 對象中的陣列。`etc` 方法只能確保在呼叫 `etc` 方法的巢狀層中不存在額外的屬性。

#### 斷言屬性存在/不存在

要斷言屬性存在或不存在，可以使用 `has` 和 `missing` 方法：

    $response->assertJson(fn (AssertableJson $json) =>
        $json->has('data')
             ->missing('message')
    );

此外，`hasAll` 和 `missingAll` 方法允許同時斷言多個屬性的存在或不存在：

    $response->assertJson(fn (AssertableJson $json) =>
        $json->hasAll(['status', 'data'])
             ->missingAll(['message', 'code'])
    );

你可以使用 `hasAny` 方法來確定是否存在給定屬性列表中的至少一個：

    $response->assertJson(fn (AssertableJson $json) =>
        $json->has('status')
             ->hasAny('data', 'message', 'code')
    );

#### 斷言反對 JSON 集合

通常，你的路由將返回一個 JSON 響應，其中包含多個項目，例如多個使用者：

    Route::get('/users', function () {
        return User::all();
    });

在這些情況下，我們可以使用 fluent JSON 對象的 `has` 方法對響應中包含的使用者進行斷言。例如，讓我們斷言 JSON 響應包含三個使用者。接下來，我們將使用 `first` 方法對集合中的第一個使用者進行一些斷言。 `first` 方法接受一個閉包，該閉包接收另一個可斷言的 JSON 字串，我們可以使用它來對 JSON 集合中的第一個對象進行斷言：

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->has(3)
                 ->first(fn (AssertableJson $json) =>
                    $json->where('id', 1)
                         ->where('name', 'Victoria Faith')
                         ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                         ->missing('password')
                         ->etc()
                 )
        );

#### JSON 集合範圍斷言

有時，你的應用程式的路由將返回分配有命名鍵的 JSON 集合：

    Route::get('/users', function () {
        return [
            'meta' => [...],
            'users' => User::all(),
        ];
    })

在測試這些路由時，你可以使用 `has` 方法來斷言集合中的項目數。此外，你可以使用 `has` 方法來確定斷言鏈的範圍：

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->has('meta')
                 ->has('users', 3)
                 ->has('users.0', fn (AssertableJson $json) =>
                    $json->where('id', 1)
                         ->where('name', 'Victoria Faith')
                         ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                         ->missing('password')
                         ->etc()
                 )
        );

但是，你可以進行一次呼叫，提供一個閉包作為其第三個參數，而不是對 `has` 方法進行兩次單獨呼叫來斷言 `users` 集合。這樣做時，將自動呼叫閉包並將其範圍限定為集合中的第一項：

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->has('meta')
                 ->has('users', 3, fn (AssertableJson $json) =>
                    $json->where('id', 1)
                         ->where('name', 'Victoria Faith')
                         ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                         ->missing('password')
                         ->etc()
                 )
        );

#### 斷言 JSON 類型

你可能只想斷言 JSON 響應中的屬性屬於某種類型。 `Illuminate\Testing\Fluent\AssertableJson` 類提供了 `whereType` 和 `whereAllType` 方法來做到這一點：

    $response->assertJson(fn (AssertableJson $json) =>
        $json->whereType('id', 'integer')
             ->whereAllType([
                'users.0.name' => 'string',
                'meta' => 'array'
            ])
    );

你可以使用 `|` 字元指定多種類型，或者將類型陣列作為第二個參數傳遞給 `whereType` 方法。如果響應值為任何列出的類型，則斷言將成功：

    $response->assertJson(fn (AssertableJson $json) =>
        $json->whereType('name', 'string|null')
             ->whereType('id', ['string', 'integer'])
    );

`whereType` 和 `whereAllType` 方法識別以下類型：`string`、`integer`、`double`、`boolean`、`array` 和 `null`。

## 測試檔案上傳

`Illuminate\Http\UploadedFile` 提供了一個 `fake` 方法用於生成虛擬的檔案或者圖像以供測試之用。它可以和 `Storage` facade 的 `fake` 方法相結合，大幅度簡化了檔案上傳測試。舉個例子，你可以結合這兩者的功能非常方便地進行頭像上傳表單測試：

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_avatars_can_be_uploaded(): void
        {
            Storage::fake('avatars');

            $file = UploadedFile::fake()->image('avatar.jpg');

            $response = $this->post('/avatar', [
                'avatar' => $file,
            ]);

            Storage::disk('avatars')->assertExists($file->hashName());
        }
    }

如果你想斷言一個給定的檔案不存在，則可以使用由 `Storage` facade 提供的 `AssertMissing` 方法：

    Storage::fake('avatars');

    // ...

    Storage::disk('avatars')->assertMissing('missing.jpg');

#### 虛擬檔案定製

當使用 `UploadedFile` 類提供的 `fake` 方法建立檔案時，你可以指定圖片的寬度、高度和大小（以千位元組為單位），以便更好地測試你的應用程式的驗證規則。

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

除建立圖像外，你也可以用 `create` 方法建立其他類型的檔案：

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

如果需要，可以向該方法傳遞一個 `$mimeType` 參數，以顯式定義檔案應返回的 MIME 類型：

    UploadedFile::fake()->create(
        'document.pdf', $sizeInKilobytes, 'application/pdf'
    );

## 測試檢視

Laravel 允許在不嚮應用程序發出模擬 HTTP 請求的情況下獨立呈現檢視。為此，可以在測試中使用 `view` 方法。`view` 方法接受檢視名稱和一個可選的資料陣列。這個方法返回一個 `Illuminate\Testing\TestView` 的實例，它提供了幾個方法來方便地斷言檢視的內容：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        public function test_a_welcome_view_can_be_rendered(): void
        {
            $view = $this->view('welcome', ['name' => 'Taylor']);

            $view->assertSee('Taylor');
        }
    }

`TestView` 對象提供了以下斷言方法：`assertSee`、`assertSeeInOrder`、`assertSeeText`、`assertSeeTextInOrder`、`assertDontSee` 和 `assertDontSeeText`。

如果需要，你可以通過將 `TestView` 實例轉換為一個字串獲得原始的檢視內容：

    $contents = (string) $this->view('welcome');

#### 共享錯誤

一些檢視可能依賴於 Laravel 提供的 [全域錯誤包](/docs/laravel/10.x/validation#quick-displaying-the-validation-errors) 中共享的錯誤。要在錯誤包中生成錯誤消息，可以使用 `withViewErrors` 方法：

    $view = $this->withViewErrors([
        'name' => ['Please provide a valid name.']
    ])->view('form');

    $view->assertSee('Please provide a valid name.');

### 渲染範本 & 元件

必要的話，你可以使用 `blade` 方法來計算和呈現原始的 [Blade](/docs/laravel/10.x/blade) 字串。與 `view` 方法一樣，`blade` 方法返回的是 `Illuminate\Testing\TestView` 的實例：

    $view = $this->blade(
        '<x-component :name="$name" />',
        ['name' => 'Taylor']
    );

    $view->assertSee('Taylor');

你可以使用 `component` 方法來評估和渲染 [Blade 元件](/docs/laravel/10.x/blade#components)。類似於 `view` 方法，`component` 方法返回一個 `Illuminate\Testing\TestView` 的實例：

    $view = $this->component(Profile::class, ['name' => 'Taylor']);

    $view->assertSee('Taylor');

## 可用斷言

### 響應斷言

Laravel 的 `Illuminate\Testing\TestResponse` 類提供了各種自訂斷言方法，你可以在測試應用程式時使用它們。可以在由 `json`、`get`、`post`、`put` 和 `delete` 方法返回的響應上訪問這些斷言：

<style>
    .collection-method-list > p {
        columns: 14.4em 2; -moz-columns: 14.4em 2; -webkit-columns: 14.4em 2;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertCookie](#assert-cookie)
[assertCookieExpired](#assert-cookie-expired)
[assertCookieNotExpired](#assert-cookie-not-expired)
[assertCookieMissing](#assert-cookie-missing)
[assertCreated](#assert-created)
[assertDontSee](#assert-dont-see)
[assertDontSeeText](#assert-dont-see-text)
[assertDownload](#assert-download)
[assertExactJson](#assert-exact-json)
[assertForbidden](#assert-forbidden)
[assertHeader](#assert-header)
[assertHeaderMissing](#assert-header-missing)
[assertJson](#assert-json)
[assertJsonCount](#assert-json-count)
[assertJsonFragment](#assert-json-fragment)
[assertJsonIsArray](#assert-json-is-array)
[assertJsonIsObject](#assert-json-is-object)
[assertJsonMissing](#assert-json-missing)
[assertJsonMissingExact](#assert-json-missing-exact)
[assertJsonMissingValidationErrors](#assert-json-missing-validation-errors)
[assertJsonPath](#assert-json-path)
[assertJsonMissingPath](#assert-json-missing-path)
[assertJsonStructure](#assert-json-structure)
[assertJsonValidationErrors](#assert-json-validation-errors)
[assertJsonValidationErrorFor](#assert-json-validation-error-for)
[assertLocation](#assert-location)
[assertContent](#assert-content)
[assertNoContent](#assert-no-content)
[assertStreamedContent](#assert-streamed-content)
[assertNotFound](#assert-not-found)
[assertOk](#assert-ok)
[assertPlainCookie](#assert-plain-cookie)
[assertRedirect](#assert-redirect)
[assertRedirectContains](#assert-redirect-contains)
[assertRedirectToRoute](#assert-redirect-to-route)
[assertRedirectToSignedRoute](#assert-redirect-to-signed-route)
[assertSee](#assert-see)
[assertSeeInOrder](#assert-see-in-order)
[assertSeeText](#assert-see-text)
[assertSeeTextInOrder](#assert-see-text-in-order)
[assertSessionHas](#assert-session-has)
[assertSessionHasInput](#assert-session-has-input)
[assertSessionHasAll](#assert-session-has-all)
[assertSessionHasErrors](#assert-session-has-errors)
[assertSessionHasErrorsIn](#assert-session-has-errors-in)
[assertSessionHasNoErrors](#assert-session-has-no-errors)
[assertSessionDoesntHaveErrors](#assert-session-doesnt-have-errors)
[assertSessionMissing](#assert-session-missing)
[assertStatus](#assert-status)
[assertSuccessful](#assert-successful)
[assertUnauthorized](#assert-unauthorized)
[assertUnprocessable](#assert-unprocessable)
[assertValid](#assert-valid)
[assertInvalid](#assert-invalid)
[assertViewHas](#assert-view-has)
[assertViewHasAll](#assert-view-has-all)
[assertViewIs](#assert-view-is)
[assertViewMissing](#assert-view-missing)

</div>

#### assertCookie

斷言響應中包含給定的 cookie：

    $response->assertCookie($cookieName, $value = null);

#### assertCookieExpired

斷言響應包含給定的過期的 cookie：

    $response->assertCookieExpired($cookieName);

#### assertCookieNotExpired

斷言響應包含給定的未過期的 cookie：

    $response->assertCookieNotExpired($cookieName);

#### assertCookieMissing

斷言響應不包含給定的 cookie:

    $response->assertCookieMissing($cookieName);

#### assertCreated

斷言做狀態程式碼為 201 的響應：

    $response->assertCreated();

#### assertDontSee

斷言給定的字串不包含在響應中。除非傳遞第二個參數 `false`，否則此斷言將給定字串進行轉義後匹配：

    $response->assertDontSee($value, $escaped = true);

#### assertDontSeeText

斷言給定的字串不包含在響應文字中。除非你傳遞第二個參數 `false`，否則該斷言將自動轉義給定的字串。該方法將在做出斷言之前將響應內容傳遞給 PHP 的 `strip_tags` 函數：

    $response->assertDontSeeText($value, $escaped = true);

#### assertDownload

斷言響應是「下載」。通常，這意味著返迴響應的呼叫路由返回了 `Response::download` 響應、`BinaryFileResponse` 或 `Storage::download` 響應：

    $response->assertDownload();

如果你願意，你可以斷言可下載的檔案被分配了一個給定的檔案名稱：

    $response->assertDownload('image.jpg');

#### assertExactJson

斷言響應包含與給定 JSON 資料的完全匹配：

    $response->assertExactJson(array $data);

#### assertForbidden

斷言響應中有禁止訪問 (403) 狀態碼：

    $response->assertForbidden();

#### assertHeader

斷言給定的 header 在響應中存在：

    $response->assertHeader($headerName, $value = null);

#### assertHeaderMissing

斷言給定的 header 在響應中不存在：

    $response->assertHeaderMissing($headerName);

#### assertJson

斷言響應包含給定的 JSON 資料：

    $response->assertJson(array $data, $strict = false);

`AssertJson` 方法將響應轉換為陣列，並利用 `PHPUnit::assertArraySubset` 驗證給定陣列是否存在於應用程式返回的 JSON 響應中。因此，如果 JSON 響應中還有其他屬性，則只要存在給定的片段，此測試仍將通過。

#### assertJsonCount

斷言響應 JSON 中有一個陣列，其中包含給定鍵的預期元素數量：

    $response->assertJsonCount($count, $key = null);

#### assertJsonFragment

斷言響應包含給定 JSON 片段：

    Route::get('/users', function () {
        return [
            'users' => [
                [
                    'name' => 'Taylor Otwell',
                ],
            ],
        ];
    });

    $response->assertJsonFragment(['name' => 'Taylor Otwell']);

#### assertJsonIsArray

斷言響應的 JSON 是一個陣列。

    $response->assertJsonIsArray();

#### assertJsonIsObject

斷言響應的 JSON 是一個對象。

    $response->assertJsonIsObject();

#### assertJsonMissing

斷言響應未包含給定的 JSON 片段：

    $response->assertJsonMissing(array $data);

#### assertJsonMissingExact

斷言響應不包含確切的 JSON 片段：

    $response->assertJsonMissingExact(array $data);

#### assertJsonMissingValidationErrors

斷言響應響應對於給定的鍵沒有 JSON 驗證錯誤：

    $response->assertJsonMissingValidationErrors($keys);

> **提示**
> 更通用的 [assertValid](#assert-valid) 方法可用於斷言響應沒有以 JSON 形式返回的驗證錯誤**並且**沒有錯誤被閃現到 session 儲存中。

#### assertJsonPath

斷言響應包含指定路徑上的給定資料：

    $response->assertJsonPath($path, $expectedValue);

例如，如果你的應用程式返回的 JSON 響應包含以下資料：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

你可以斷言 `user` 對象的 `name` 屬性匹配給定值，如下所示：

    $response->assertJsonPath('user.name', 'Steve Schoger');

#### assertJsonMissingPath

斷言響應具有給定的 JSON 結構：

    $response->assertJsonMissingPath($path);

例如，如果你的應用程式返回的 JSON 響應包含以下資料：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

你可以斷言它不包含 `user` 對象的 `email` 屬性。

    $response->assertJsonMissingPath('user.email');

#### assertJsonStructure

斷言響應具有給定的 JSON 結構：

    $response->assertJsonStructure(array $structure);

例如，如果你的應用程式返回的 JSON 響應包含以下資料：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

你可以斷言 JSON 結構符合你的期望，如下所示：

    $response->assertJsonStructure([
        'user' => [
            'name',
        ]
    ]);

有時，你的應用程式返回的 JSON 響應可能包含對象陣列：

```json
{
    "user": [
        {
            "name": "Steve Schoger",
            "age": 55,
            "location": "Earth"
        },
        {
            "name": "Mary Schoger",
            "age": 60,
            "location": "Earth"
        }
    ]
}
```

在這種情況下，你可以使用 `*` 字元來斷言陣列中所有對象的結構：

    $response->assertJsonStructure([
        'user' => [
            '*' => [
                 'name',
                 'age',
                 'location'
            ]
        ]
    ]);

#### assertJsonValidationErrors

斷言響應具有給定鍵的給定 JSON 驗證錯誤。在斷言驗證錯誤作為 JSON 結構返回而不是閃現到 session 的響應時，應使用此方法：

    $response->assertJsonValidationErrors(array $data, $responseKey = 'errors');

> **技巧**
> 更通用的 [assertInvalid](#assert-invalid) 方法可用於斷言響應具有以 JSON 形式返回的驗證錯誤**或**錯誤已快閃記憶體到 session 儲存。

#### assertJsonValidationErrorFor

斷言響應對給定鍵有任何 JSON 驗證錯誤：

    $response->assertJsonValidationErrorFor(string $key, $responseKey = 'errors');

#### assertLocation

斷言響應在 `Location` 頭部中具有給定的 URI 值：

    $response->assertLocation($uri);

#### assertContent

斷言給定的字串與響應內容匹配。

    $response->assertContent($value);

#### assertNoContent

斷言響應具有給定的 HTTP 狀態碼且沒有內容：

    $response->assertNoContent($status = 204);

#### assertStreamedContent

斷言給定的字串與流式響應的內容相匹配。

    $response->assertStreamedContent($value);

#### assertNotFound

斷言響應具有未找到（404）HTTP 狀態碼：

    $response->assertNotFound();

#### assertOk

斷言響應有 200 狀態碼：

    $response->assertOk();

#### assertPlainCookie

斷言響應包含給定的 cookie（未加密）:

    $response->assertPlainCookie($cookieName, $value = null);

#### assertRedirect

斷言響應會重新導向到給定的 URI：

    $response->assertRedirect($uri);

#### assertRedirectContains

斷言響應是否重新導向到包含給定字串的 URI：

    $response->assertRedirectContains($string);

#### assertRedirectToRoute

斷言響應是對給定的[命名路由](/docs/laravel/10.x/routing#named-routes)的重新導向。

    $response->assertRedirectToRoute($name = null, $parameters = []);

#### assertRedirectToSignedRoute

斷言響應是對給定[簽名路由](/docs/laravel/10.x/urls#signed-urls)的重新導向：

    $response->assertRedirectToSignedRoute($name = null, $parameters = []);

#### assertSee

斷言給定的字串包含在響應中。除非傳遞第二個參數 `false`，否則此斷言將給定字串進行轉義後匹配：

    $response->assertSee($value, $escaped = true);

#### assertSeeInOrder

斷言給定的字串按順序包含在響應中。除非傳遞第二個參數 `false`，否則此斷言將給定字串進行轉義後匹配：

    $response->assertSeeInOrder(array $values, $escaped = true);

#### assertSeeText

斷言給定字串包含在響應文字中。除非傳遞第二個參數 `false`，否則此斷言將給定字串進行轉義後匹配。在做出斷言之前，響應內容將被傳遞到 PHP 的 `strip_tags` 函數：

    $response->assertSeeText($value, $escaped = true);

#### assertSeeTextInOrder

斷言給定的字串按順序包含在響應的文字中。除非傳遞第二個參數 `false`，否則此斷言將給定字串進行轉義後匹配。在做出斷言之前，響應內容將被傳遞到 PHP 的 `strip_tags` 函數：

    $response->assertSeeTextInOrder(array $values, $escaped = true);

#### assertSessionHas

斷言 Session 包含給定的資料段：

    $response->assertSessionHas($key, $value = null);

如果需要，可以提供一個閉包作為 `assertSessionHas` 方法的第二個參數。如果閉包返回 `true`，則斷言將通過：

    $response->assertSessionHas($key, function (User $value) {
        return $value->name === 'Taylor Otwell';
    });

#### assertSessionHasInput

session 在 [快閃記憶體輸入陣列](/docs/laravel/9.x/responses#redirecting-with-flashed-session-data) 中斷言具有給定值：

    $response->assertSessionHasInput($key, $value = null);

如果需要，可以提供一個閉包作為 `assertSessionHasInput` 方法的第二個參數。如果閉包返回 `true`，則斷言將通過：

    $response->assertSessionHasInput($key, function (string $value) {
        return Crypt::decryptString($value) === 'secret';
    });

#### assertSessionHasAll

斷言 Session 中具有給定的鍵 / 值對列表：

    $response->assertSessionHasAll(array $data);

例如，如果你的應用程式 session 包含 `name` 和 `status` 鍵，則可以斷言它們存在並且具有指定的值，如下所示：

    $response->assertSessionHasAll([
        'name' => 'Taylor Otwell',
        'status' => 'active',
    ]);

#### assertSessionHasErrors

斷言 session 包含給定 `$keys` 的 Laravel 驗證錯誤。如果 `$keys` 是關聯陣列，則斷言 session 包含每個欄位（key）的特定錯誤消息（value）。測試將快閃記憶體驗證錯誤到 session 的路由時，應使用此方法，而不是將其作為 JSON 結構返回：

    $response->assertSessionHasErrors(
        array $keys, $format = null, $errorBag = 'default'
    );

例如，要斷言 `name` 和 `email` 欄位具有已快閃記憶體到 session 的驗證錯誤消息，可以呼叫 `assertSessionHasErrors` 方法，如下所示：

    $response->assertSessionHasErrors(['name', 'email']);

或者，你可以斷言給定欄位具有特定的驗證錯誤消息：

    $response->assertSessionHasErrors([
        'name' => 'The given name was invalid.'
    ]);

> **注意**
> 更加通用的 [assertInvalid](#assert-invalid) 方法可以用來斷言一個響應有驗證錯誤，以JSON形式返回，**或** 將錯誤被快閃記憶體到 session 儲存中。

#### assertSessionHasErrorsIn

斷言 session 在特定的[錯誤包](/docs/laravel/10.x/validation#named-error-bags)中包含給定 `$keys` 的錯誤。如果 `$keys` 是一個關聯陣列，則斷言該 session 在錯誤包內包含每個欄位（鍵）的特定錯誤消息（值）：

    $response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);

#### assertSessionHasNoErrors

斷言 session 沒有驗證錯誤：

    $response->assertSessionHasNoErrors();

#### assertSessionDoesntHaveErrors

斷言 session 對給定鍵沒有驗證錯誤：

    $response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');

> **注意**
> 更加通用的 [assertValid](#assert-valid) 方法可以用來斷言一個響應沒有以JSON形式返回的驗證錯誤，**同時** 不會將錯誤被快閃記憶體到 session 儲存中。

#### assertSessionMissing

斷言 session 中缺少指定的 $key：

    $response->assertSessionMissing($key);

#### assertStatus

斷言響應指定的 http 狀態碼：

    $response->assertStatus($code);

#### assertSuccessful

斷言響應一個成功的狀態碼 (>= 200 且 < 300) :

    $response->assertSuccessful();

#### assertUnauthorized

斷言一個未認證的狀態碼 (401)：

    $response->assertUnauthorized();

#### assertUnprocessable

斷言響應具有不可處理的實體 (422) HTTP 狀態程式碼：

    $response->assertUnprocessable();

#### assertValid

斷言響應對給定鍵沒有驗證錯誤。此方法可用於斷言驗證錯誤作為 JSON 結構返回或驗證錯誤已閃現到 session 的響應：

    // 斷言不存在驗證錯誤...
    $response->assertValid();

    // 斷言給定的鍵沒有驗證錯誤...
    $response->assertValid(['name', 'email']);

#### assertInvalid

斷言響應對給定鍵有驗證錯誤。此方法可用於斷言驗證錯誤作為 JSON 結構返回或驗證錯誤已快閃記憶體到 session 的響應：

    $response->assertInvalid(['name', 'email']);

你還可以斷言給定鍵具有特定的驗證錯誤消息。這樣做時，你可以提供整條消息或僅提供一部分消息：

    $response->assertInvalid([
        'name' => 'The name field is required.',
        'email' => 'valid email address',
    ]);

#### assertViewHas

斷言為響應檢視提供了一個鍵值對資料：

    $response->assertViewHas($key, $value = null);

將閉包作為第二個參數傳遞給 `assertViewHas` 方法將允許你檢查並針對特定的檢視資料做出斷言：

    $response->assertViewHas('user', function (User $user) {
        return $user->name === 'Taylor';
    });

此外，檢視資料可以作為陣列變數訪問響應，讓你可以方便地檢查它：

    $this->assertEquals('Taylor', $response['name']);

#### assertViewHasAll

斷言響應檢視具有給定的資料列表：

    $response->assertViewHasAll(array $data);

該方法可用於斷言該檢視僅包含與給定鍵匹配的資料：

    $response->assertViewHasAll([
        'name',
        'email',
    ]);

或者，你可以斷言該檢視資料存在並且具有特定值：

    $response->assertViewHasAll([
        'name' => 'Taylor Otwell',
        'email' => 'taylor@example.com,',
    ]);

#### assertViewIs

斷言當前路由返回的的檢視是給定的檢視：

    $response->assertViewIs($value);

#### assertViewMissing

斷言給定的資料鍵不可用於應用程式響應中返回的檢視：

    $response->assertViewMissing($key);

### 身份驗證斷言

Laravel 還提供了各種與身份驗證相關的斷言，你可以在應用程式的功能測試中使用它們。請注意，這些方法是在測試類本身上呼叫的，而不是由諸如 `get` 和 `post` 等方法返回的 `Illuminate\Testing\TestResponse` 實例。

#### assertAuthenticated

斷言使用者已通過身份驗證：

    $this->assertAuthenticated($guard = null);

#### assertGuest

斷言使用者未通過身份驗證：

    $this->assertGuest($guard = null);

#### assertAuthenticatedAs

斷言特定使用者已通過身份驗證：

    $this->assertAuthenticatedAs($user, $guard = null);

## 驗證斷言

Laravel 提供了兩個主要的驗證相關的斷言，你可以用它來確保在你的請求中提供的資料是有效或無效的。

#### assertValid

斷言響應對於給定的鍵沒有驗證錯誤。該方法可用於斷言響應中的驗證錯誤是以 JSON 結構返回的，或者驗證錯誤已經閃現到 session 中。

    // 斷言沒有驗證錯誤存在...
    $response->assertValid();

    //斷言給定的鍵沒有驗證錯誤...
    $response->assertValid(['name', 'email']);

#### assertInvalid

斷言響應對給定的鍵有驗證錯誤。這個方法可用於斷言響應中的驗證錯誤是以 JSON 結構返回的，或者驗證錯誤已經被閃現到 session 中。

    $response->assertInvalid(['name', 'email']);

你也可以斷言一個給定的鍵有一個特定的驗證錯誤資訊。當這樣做時，你可以提供整個消息或只提供消息的一小部分。

    $response->assertInvalid([
        'name' => 'The name field is required.',
        'email' => 'valid email address',
    ]);
