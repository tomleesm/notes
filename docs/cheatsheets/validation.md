``` php
Validator::make(
array('key' => 'Foo'),
array('key' => 'required|in:Foo')
);
Validator::extend('foo', function($attribute, $value, $params){});
Validator::extend('foo', 'FooValidator@validate');
Validator::resolver(function($translator, $data, $rules, $msgs)
{
return new FooValidator($translator, $data, $rules, $msgs);
});
```

## 驗證規則

### accepted

待驗證欄位必須是 「yes」 ，「on」 ，1 或 true。這對於驗證「服務條款」的接受或類似欄位時很有用。


### accepted_if:anotherfield,value,…

如果另一個正在驗證的欄位等於指定的值，則驗證中的欄位必須為 「yes」 ，「on」 ，1 或 true。 這對於驗證「服務條款」接受或類似欄位很有用。


### active_url

根據 dns_get_record PHP 函數，驗證中的欄位必須具有有效的 A 或 AAAA 記錄。 提供的 URL 的主機名使用 parse_url PHP 函數提取，然後傳遞給 dns_get_record。


### after:date

驗證中的欄位必須是給定日期之後的值。日期將被傳遞給 strtotime PHP 函數中，以便轉換為有效的 DateTime 實例：

```
'start_date' => 'required|date|after:tomorrow'
```

你也可以指定另一個要與日期比較的欄位，而不是傳遞要由 strtotime 處理的日期字串：

```
'finish_date' => 'required|date|after:start_date'
```

### after_or_equal:date

待驗證欄位的值對應的日期必須在給定日期之後或與給定的日期相同。可參閱 after 規則獲取更多資訊。


### alpha

待驗證欄位必須是包含在 \p{L} 和 \p{M} 中的 Unicode 字母字元。

為了將此驗證規則限制在 ASCII 範圍內的字元（a-z 和 A-Z），你可以為驗證規則提供 ascii 選項：

```
'username' => 'alpha:ascii',
```

### alpha_dash

被驗證的欄位必須完全是 Unicode 字母數字字元中的 \p{L}、\p{M}、\p{N}，以及 ASCII 破折號（-）和 ASCII 下劃線（_）。

為了將此驗證規則限制在 ASCII 範圍內的字元（a-z 和 A-Z），你可以為驗證規則提供 ascii 選項：

```
'username' => 'alpha_dash:ascii',
```

### alpha_num

被驗證的欄位必須完全是 Unicode 字母數字字元中的 \p{L}, \p{M} 和 \p{N}。

為了將此驗證規則限制在 ASCII 範圍內的字元（a-z 和 A-Z），你可以為驗證規則提供 ascii 選項：

```
'username' => 'alpha_num:ascii',
```

### array

待驗證欄位必須是有效的 PHP 陣列。

當向 array 規則提供附加值時，輸入陣列中的每個鍵都必須出現在提供給規則的值列表中。在以下示例中，輸入陣列中的 admin 鍵無效，因為它不包含在提供給 array 規則的值列表中：

``` php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => 'array:name,username',
]);
```

通常，你應該始終指定允許出現在陣列中的陣列鍵。

### ascii

正在驗證的欄位必須完全是 7 位的 ASCII 字元。

### bail

在首次驗證失敗後立即終止驗證。

雖然 bail 規則只會在遇到驗證失敗時停止驗證特定欄位，但 stopOnFirstFailure 方法會通知驗證器，一旦發生單個驗證失敗，它應該停止驗證所有屬性:

``` php
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

### before:date

待驗證欄位的值對應的日期必須在給定的日期之前。這個日期將被傳遞給 PHP 函數 strtotime 以便轉化為有效的 DateTime 實例。此外，與 after 規則一致，可以將另外一個待驗證的欄位作為 date 的值。

### before_or_equal:date

待驗證欄位的值必須是給定日期之前或等於給定日期的值。這個日期將被傳遞給 PHP 函數 strtotime 以便轉化為有效的 DateTime 實例。此外，與 after 規則一致， 可以將另外一個待驗證的欄位作為 date 的值。

### between:min,max

待驗證欄位值的大小必須介於給定的最小值和最大值（含）之間。字串、數字、陣列和檔案的計算方式都使用 size 方法。


### boolean

驗證的欄位必須可以轉換為 Boolean 類型。 可接受的輸入為 true, false, 1, 0, 「1」, 和 「0」。


### confirmed

驗證欄位必須與 {field}_confirmation 欄位匹配。例如，如果驗證欄位是 password，則輸入中必須存在相應的 password_confirmation 欄位。


### current_password

驗證欄位必須與已認證使用者的密碼匹配。 您可以使用規則的第一個參數指定 authentication guard:

```
'password' => 'current_password:api'
```

### date

驗證欄位必須是 strtotime PHP 函數可識別的有效日期。


### date_equals:date

驗證欄位必須等於給定日期。日期將傳遞到 PHP strtotime 函數中，以轉換為有效的 DateTime 實例。


### date_format:format,…

驗證欄位必須匹配給定的 format 。在驗證欄位時，您應該只使用 date 或 date_format 中的其中一個，而不是同時使用。該驗證規則支援 PHP 的 DateTime 類支援的所有格式。


### decimal:min,max

驗證欄位必須是數值類型，並且必須包含指定的小數位數：

```
// 必須正好有兩位小數（例如 9.99）...
'price' => 'decimal:2'
```

```
// 必須有 2 到 4 位小數...
'price' => 'decimal:2,4'
```

### declined

正在驗證的欄位必須是 「no」，「off」，0 或者 false。


### declined_if:anotherfield,value,…

如果另一個驗證欄位的值等於指定值，則驗證欄位的值必須為「no」、「off」、0 或 false。


### different:field

驗證的欄位值必須與欄位 field 的值不同。


### digits:value

驗證的整數必須具有確切長度 value 。


### digits_between:min,max

驗證的整數長度必須在給定的 min 和 max 之間。


### dimensions

驗證的檔案必須是符合規則參數指定尺寸限制的圖像：

```
'avatar' => 'dimensions:min_width=100,min_height=200'
```

可用的限制條件有: min_width , max_width , min_height , max_height , width , height , ratio .

ratio 約束應該表示為寬度除以高度。 這可以通過像 3/2 這樣的語句或像 1.5 這樣的浮點數來指定：

```
'avatar' => 'dimensions:ratio=3/2'
```

由於此規則需要多個參數，因此你可以 Rule::dimensions 方法來構造可讀性高的規則：

``` php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'avatar' => [
        'required',
        Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
    ],
]);
```

### distinct

驗證陣列時，正在驗證的欄位不能有任何重複值：

```
'foo.*.id' => 'distinct'
```

默認情況下，Distinct 使用鬆散的變數比較。要使用嚴格比較，您可以在驗證規則定義中新增 strict 參數：

```
'foo.*.id' => 'distinct:strict'
```

你可以在驗證規則的參數中新增 ignore_case ，以使規則忽略大小寫差異：

```
'foo.*.id' => 'distinct:ignore_case'
```

### doesnt_start_with:foo,bar,…

驗證的欄位不能以給定值之一開頭。


### doesnt_end_with:foo,bar,…

驗證的欄位不能以給定值之一結尾。


### email

驗證的欄位必須符合 e-mail 地址格式。當前版本，此種驗證規則由 egulias/email-validator 提供支援。默認情況下，使用 RFCValidation 驗證樣式，但你也可以應用其他驗證樣式：

```
'email' => 'email:rfc,dns'
```

上面的示例將應用 RFCValidation 和 DNSCheckValidation 驗證。以下是你可以應用的驗證樣式的完整列表：

- rfc: RFCValidation
- strict: NoRFCWarningsValidation
- dns: DNSCheckValidation
- spoof: SpoofCheckValidation
- filter: FilterEmailValidation
- filter_unicode: FilterEmailValidation::unicode()

filter 驗證器是 Laravel 內建的一個驗證器，它使用 PHP 的 filter_var 函數實現。在 Laravel 5.8 版本之前，它是 Laravel 默認的電子郵件驗證行為。

注意
dns 和 spoof 驗證器需要 PHP 的 intl 擴展。


### ends_with:foo,bar,…

被驗證的欄位必須以給定值之一結尾。


### enum

Enum 規則是一種基於類的規則，用於驗證被驗證欄位是否包含有效的列舉值。 Enum 規則的建構函式只接受列舉的名稱作為參數：

``` php
use App\Enums\ServerStatus;
use Illuminate\Validation\Rules\Enum;

$request->validate([
    'status' => [new Enum(ServerStatus::class)],
]);
```

### exclude

validate 和 validated 方法中會排除掉當前驗證的欄位。


### exclude_if:anotherfield,value

如果 anotherfield 等於 value ，validate 和 validated 方法中會排除掉當前驗證的欄位。

在一些複雜的場景，也可以使用 Rule::excludeIf 方法，這個方法需要返回一個布林值或者一個匿名函數。如果返回的是匿名函數，那麼這個函數應該返回 true 或 false 去決定被驗證的欄位是否應該被排除掉：

``` php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::excludeIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::excludeIf(fn () => $request->user()->is_admin),
]);
```

### exclude_unless:anotherfield,value

除非 anotherfield 等於 value ，否則 validate 和 validated 方法中會排除掉當前的欄位。如果 value 為 null （exclude_unless:name,null），那麼成立的條件就是被比較的欄位為 null 或者表單中沒有該欄位。


### exclude_with:anotherfield

如果表單資料中有 anotherfield ，validate 和 validated 方法中會排除掉當前的欄位。


### exclude_without:anotherfield

如果表單資料中沒有 anotherfield ，validate 和 validated 方法中會排除掉當前的欄位。


### exists:table,column

驗證的欄位值必須存在於指定的表中。


Exists 規則的基本用法

```
'state' => 'exists:states'
```

如果未指定 column 選項，則將使用欄位名稱。因此，在這種情況下，該規則將驗證 states 資料庫表是否包含一條記錄，該記錄的 state 列的值與請求的 state 屬性值匹配。


指定自訂列名
你可以將驗證規則使用的資料庫列名稱指定在資料庫表名稱之後：

```
'state' => 'exists:states,abbreviation'
```

有時候，你或許需要去明確指定一個具體的資料庫連接，用於 exists 查詢。你可以通過在表名前面新增一個連接名稱來實現這個效果。

```
'email' => 'exists:connection.staff,email'
```

你可以明確指定 Eloquent 模型，而不是直接指定表名：

```
'user_id' => 'exists:App\Models\User,id'
```

如果你想要自訂一個執行查詢的驗證規則，你可以使用 Rule 類去流暢地定義規則。在這個例子中，我們也將指定驗證規則為一個陣列，而不再是使用 | 分割他們：

``` php
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::exists('staff')->where(function (Builder $query) {
            return $query->where('account_id', 1);
        }),
    ],
]);
```

您可以通過將列名作為 exists 方法的第二個參數來明確指定 Rule::exists 方法生成的 exists 規則應該使用的資料庫列名：

```
'state' => Rule::exists('states', 'abbreviation'),
```

### file

要驗證的欄位必須是一個成功的已經上傳的檔案。


### filled

當欄位存在時，要驗證的欄位必須是一個非空的。


### gt:field

要驗證的欄位必須要大於給定的欄位。這兩個欄位必須是同一個類型。字串、數字、陣列和檔案都使用 size 進行相同的評估。


### gte:field

要驗證的欄位必須要大於或等於給定的欄位。這兩個欄位必須是同一個類型。字串、數字、陣列和檔案都使用 size 進行相同的評估。


### image

正在驗證的檔案必須是圖像（jpg、jpeg、png、bmp、gif、svg 或 webp）。


### in:foo,bar,…

驗證欄位必須包含在給定的值列表中。由於此規則通常要求你 implode 陣列，因此可以使用 Rule::in 方法來流暢地構造規則:

``` php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'zones' => [
        'required',
        Rule::in(['first-zone', 'second-zone']),
    ],
]);
```

當 in 規則與 array 規則組合使用時，輸入陣列中的每個值都必須出現在提供給 in 規則的值列表中。 在以下示例中，輸入陣列中的 LAS 機場程式碼無效，因為它不包含在提供給 in 規則的機場列表中：

``` php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$input = [
    'airports' => ['NYC', 'LAS'],
];

Validator::make($input, [
    'airports' => [
        'required',
        'array',
    ],
    'airports.*' => Rule::in(['NYC', 'LIT']),
]);
```

### in_array:anotherfield.*

驗證的欄位必須存在於_anotherfield_的值中。


### integer

驗證的欄位必須是一個整數。

警告
這個驗證規則並不會驗證輸入是否為”integer” 變數類型，它只會驗證輸入是否為 PHP 的 FILTER_VALIDATE_INT 規則接受的類型。如果你需要驗證輸入是否為數字，請結合 numeric 驗證規則使用。


### ip

驗證的欄位必須是一個 IP 地址。


### ipv4

驗證的欄位必須是一個 IPv4 地址。


### ipv6

驗證的欄位必須是一個 IPv6 地址。


### json

驗證的欄位必須是一個有效的 JSON 字串。


### lt:field

驗證的欄位必須小於給定的 field 欄位。兩個欄位必須是相同的類型。字串、數字、陣列和檔案的處理方式與 size 規則相同。


### lte:field

驗證的欄位必須小於或等於給定的 field 欄位。兩個欄位必須是相同的類型。字串、數字、陣列和檔案的處理方式與 size 規則相同。


### lowercase

驗證的欄位必須是小寫的。


### mac_address

驗證的欄位必須是一個 MAC 地址。


### max:value

驗證的欄位的值必須小於或等於最大值 value。字串、數字、陣列和檔案的處理方式與 size 規則相同。


### max_digits:value

驗證的整數必須具有最大長度 value。


### mimetypes:text/plain,…

驗證的檔案必須匹配給定的 MIME 類型之一：

'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'
為了確定上傳檔案的 MIME 類型，將讀取檔案內容並嘗試猜測 MIME 類型，這可能與客戶端提供的 MIME 類型不同。


### mimes:foo,bar,…

驗證的檔案必須具有與列出的擴展名之一對應的 MIME 類型。


MIME 規則的基本用法

```
'photo' => 'mimes:jpg,bmp,png'
```

儘管您只需要指定擴展名，但該規則實際上通過讀取檔案內容並猜測其 MIME 類型來驗證檔案的 MIME 類型。可以在以下位置找到 MIME 類型及其相應擴展名的完整列表：

svn.apache.org/repos/asf/httpd/htt...


### min:value

驗證的欄位的值必須大於或等於最小值 value。字串、數字、陣列和檔案的處理方式與 size 規則相同。


### min_digits:value

驗證的整數必須具有至少_value_位數。


### multiple_of:value

驗證的欄位必須是_value_的倍數。


### missing

驗證的欄位在輸入資料中必須不存在。


### missing_if:anotherfield,value,…

如果_anotherfield_欄位等於任何_value_，則驗證的欄位必須不存在。


### missing_unless:anotherfield,value

驗證的欄位必須不存在，除非_anotherfield_欄位等於任何_value_。


### missing_with:foo,bar,…

如果任何其他指定的欄位存在，則驗證的欄位必須不存在。


### missing_with_all:foo,bar,…

如果所有其他指定的欄位都存在，則驗證的欄位必須不存在。


### not_in:foo,bar,…

驗證的欄位不能包含在給定值列表中。可以使用 Rule::notIn 方法流暢地建構規則：

``` php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'toppings' => [
        'required',
        Rule::notIn(['sprinkles', 'cherries']),
    ],
]);
```

### not_regex:pattern

驗證的欄位必須不匹配給定的正規表示式。

在內部，此規則使用 PHP 的 preg_match 函數。指定的模式應遵守 preg_match 所需的相同格式要求，因此也應包括有效的分隔符。例如：`'email' => 'not_regex:/^.+$/i'`。

警告 使用 regex / not_regex 模式時，可能需要使用陣列指定驗證規則，而不是使用 | 分隔符，特別是如果正規表示式包含 | 字元。


### nullable

需要驗證的欄位可以為 null。


### numeric

需要驗證的欄位必須是數字類型。


### password

需要驗證的欄位必須與已認證使用者的密碼相匹配。

> **警告**
> 這個規則在 Laravel 9 中被重新命名為 current_password 並計畫刪除，請改用 Current Password 規則。


### present

需要驗證的欄位必須存在於輸入資料中。


### prohibited

需要驗證的欄位必須不存在或為空。如果符合以下條件之一，欄位將被認為是 “空”：

- 值為 null。
- 值為空字串。
- 值為空陣列或空的可計數對象。
- 值為上傳檔案，但檔案路徑為空。

### prohibited_if:anotherfield,value,…

如果 anotherfield 欄位等於任何 value，則需要驗證的欄位必須不存在或為空。如果符合以下條件之一，欄位將被認為是 “空”：

- 值為 null。
- 值為空字串。
- 值為空陣列或空的可計數對象。
- 值為上傳檔案，但檔案路徑為空。
- 如果需要複雜的條件禁止邏輯，則可以使用 Rule::prohibitedIf 方法。該方法接受一個布林值或一個閉包。當給定一個閉包時，閉包應返回 true 或 false，以指示是否應禁止驗證欄位：

``` php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::prohibitedIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::prohibitedIf(fn () => $request->user()->is_admin),
]);
```

### prohibited_unless:anotherfield,value,…

在 anotherfield 欄位等於任何 value 時，驗證的欄位必須為空或缺失。如果一個欄位符合以下任一標準，則它被認為是 “空” 的：

- 值為 null。
- 值為空字串。
- 值為一個空陣列或一個空的 Countable 對象。
- 值為上傳檔案且路徑為空。

### prohibits:anotherfield,…

如果驗證的欄位不為空或缺失，則 anotherfield 中的所有欄位都必須為空或缺失。如果一個欄位符合以下任一標準，則它被認為是 “空” 的：

- 值為 null。
- 值為空字串。
- 值為一個空陣列或一個空的 Countable 對象。
- 值為上傳檔案且路徑為空。

### regex:pattern

驗證的欄位必須匹配給定的正規表示式。

在內部，此規則使用 PHP 的 preg_match 函數。指定的模式應遵循 preg_match 所需的相同格式，並且也包括有效的分隔符。例如：`'email' => 'regex:/^.+@.+$/i'`。

> **警告**
> 當使用 regex / not_regex 模式時，可能需要使用陣列指定規則而不是使用 | 分隔符，特別是如果正規表示式包含 | 字元。


### required

驗證的欄位必須出現在輸入資料中且不為空。如果一個欄位符合以下任一標準，則它被認為是 “空” 的：

- 值為 null。
- 值為空字串。
- 值為一個空陣列或一個空的 Countable 對象。
- 值為上傳檔案且路徑為空。

### required_if:anotherfield,value,…

如果 anotherfield 欄位的值等於任何 value 值，則驗證的欄位必須存在且不為空。

如果您想要建構更複雜的 required_if 規則條件，可以使用 Rule::requiredIf 方法。該方法接受一個布林值或閉包。當傳遞一個閉包時，閉包應返回 true 或 false 來指示是否需要驗證的欄位：

``` php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf(fn () => $request->user()->is_admin),
]);
```

### required_unless:anotherfield,value,…

如果 anotherfield 欄位的值不等於任何 value 值，則驗證的欄位必須存在且不為空。這也意味著，除非 anotherfield 欄位等於任何 value 值，否則必須在請求資料中包含 anotherfield 欄位。如果 value 的值為 null （required_unless:name,null），則必須驗證該欄位，除非比較欄位是 null 或比較欄位不存在於請求資料中。


### required_with:foo,bar,…

僅當任何其他指定欄位存在且不為空時，才需要驗證欄位存在且不為空。


### required_with_all:foo,bar,…

僅當所有其他指定欄位存在且不為空時，才需要驗證欄位存在且不為空。


### required_without:foo,bar,…

驗證的欄位僅在任一其他指定欄位為空或不存在時，必須存在且不為空。


### required_without_all:foo,bar,…

驗證的欄位僅在所有其他指定欄位為空或不存在時，必須存在且不為空。


### required_array_keys:foo,bar,…

驗證的欄位必須是一個陣列，並且必須至少包含指定的鍵。


### same:field

給定的欄位必須與驗證的欄位匹配。


### size:value

驗證的欄位必須具有與給定的_value 相匹配的大小。對於字串資料，value 對應於字元數。對於數字資料，value 對應於給定的整數值（該屬性還必須具有 numeric 或 integer 規則）。對於陣列，size 對應於陣列的 count。對於檔案，size 對應於檔案大小（以千位元組為單位）。讓我們看一些例子：

``` php
// Validate that a string is exactly 12 characters long...
'title' => 'size:12';

// Validate that a provided integer equals 10...
'seats' => 'integer|size:10';

// Validate that an array has exactly 5 elements...
'tags' => 'array|size:5';

// Validate that an uploaded file is exactly 512 kilobytes...
'image' => 'file|size:512';
```

### starts_with:foo,bar,…

驗證的欄位必須以給定值之一開頭。


### string

驗證的欄位必須是一個字串。如果您希望允許欄位也可以為 null，則應將 nullable 規則分配給該欄位。


### timezone

驗證欄位必須是一個有效的時區識別碼，符合 timezone_identifiers_list PHP 函數的要求。


### unique:table,column

驗證欄位在給定的資料庫表中必須不存在。

指定自訂表 / 列名:

可以指定應使用哪個 Eloquent 模型來確定表名，而不是直接指定表名：

```
'email' => 'unique:App\Models\User,email_address'
```

column 選項可用於指定欄位對應的資料庫列。如果未指定 column 選項，則使用驗證欄位的名稱。

```
'email' => 'unique:users,email_address'
```

指定自訂資料庫連接

有時，您可能需要為 Validator 執行的資料庫查詢設定自訂連接。為此，可以在表名之前新增連接名稱：

```
'email' => 'unique:connection.users,email_address'
```

強制唯一規則忽略給定的 ID:

有時，您可能希望在唯一驗證期間忽略給定的 ID。例如，考慮一個 “更新個人資料” 螢幕，其中包括使用者的姓名、電子郵件地址和位置。您可能希望驗證電子郵件地址是否唯一。但是，如果使用者僅更改了名稱欄位而未更改電子郵件欄位，則不希望因為使用者已經擁有相關電子郵件地址而拋出驗證錯誤。

要指示驗證器忽略使用者的 ID，我們將使用 Rule 類來流暢地定義規則。在此示例中，我們還將指定驗證規則作為陣列，而不是使用 | 字元來分隔規則：

``` php
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

警告
您不應將任何使用者控制的請求輸入傳遞到 ignore 方法中。相反，您應僅傳遞系統生成的唯一 ID，例如 Eloquent 模型實例的自增 ID 或 UUID。否則，您的應用程式將容易受到 SQL 隱碼攻擊。

不需要將模型鍵的值傳遞給 ignore 方法，您也可以傳遞整個模型實例。Laravel 將自動從模型中提取鍵：

``` php
Rule::unique('users')->ignore($user)
```

如果您的表使用的是除 id 以外的主鍵列名，可以在呼叫 ignore 方法時指定列的名稱：

``` php
Rule::unique('users')->ignore($user->id, 'user_id')
```

默認情況下，unique 規則將檢查與正在驗證的屬性名稱匹配的列的唯一性。但是，您可以將不同的列名稱作為第二個參數傳遞給 unique 方法：

``` php
Rule::unique('users', 'email_address')->ignore($user->id)
```

新增額外的查詢條件：

您可以通過自訂查詢並使用 where 方法來指定其他查詢條件。例如，讓我們新增一個查詢條件，將查詢範圍限定為僅搜尋具有 account_id 列值為 1 的記錄：

``` php
'email' => Rule::unique('users')->where(fn (Builder $query) => $query->where('account_id', 1))
```

### uppercase

驗證欄位必須為大寫。


### url

驗證欄位必須為有效的 URL。


### ulid

驗證欄位必須為有效的通用唯一詞典排序識別碼（ULID）。


### uuid

驗證欄位必須為有效的 RFC 4122（版本 1、3、4 或 5）通用唯一識別碼（UUID）。

