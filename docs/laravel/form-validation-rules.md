# 驗證規則清單

## accepted 同意

`yes`、`on`、`1` 或 `true`，或上述值的字串形式，適合「同意服務協議」

## active_url 有效網址

基於 PHP 函式 [dns_get_record()](https://www.php.net/manual/en/function.dns-get-record.php) 的，有 A 或 AAAA 記錄的值，也就是查的到 ip 的。驗證時會停一下，看來是真的用 dns_get_record() 查詢。

開頭必須有 `http(s)://` 的有效網址，例如 https://www.google.com ，或 http://127.0.0.1 也可以。網址會先用  PHP 函式 [parse_url()](https://www.php.net/manual/en/function.parse-url.php) 處理後再交給 `dns_get_record()`，所以和 `dns_get_record()` 官方文件上傳入的網址不一樣

## after:date 某天之後(不含那天)

 PHP 函式 [strtotime()](https://www.php.net/manual/en/function.strtotime)可以接受的日期格式之後，但是不含那天。或是指定欄位

``` php
<?php
[
    # 改成 next thursday 也可以
    'publish_date' => 'after:next Thursday',
    'birthday' => 'after:2022-07-20',
    # 指定欄位
    'start_date' => 'date',
    'end_date' => 'after:start_date'
]
```

## after_or_equal:date 某天之後(含那天)

和 after:date 相同，只是包含那天

## alpha 字母

只能是字母，包含英文字母、拉丁字母、希臘字母、泰文字母、阿拉伯文字母、日文、中文都可以，但是阿拉伯數字、逗號、全形問號等不行

## alpha_dash 字母、數字、橫線、底線

包含 alpha 規則，再加上阿拉伯數字、橫線 `-` 和底線 `_`

## alpha_num 字母、數字

包含 alpha 規則，再加上阿拉伯數字

## array

輸入必須是 PHP 陣列，例如以下這樣

``` html
<p>
    <label>author 0</label>
    <input type="text" name="author[]">
</p>
<p>
    <label>author 1</label>
    <input type="text" name="author[]">
</p>
<p>
    <label>author 2</label>
    <input type="text" name="author[]">
</p>
```

``` php
<?php
$rule = [
    'author' => 'array'
];
$validator = \Validator::make($request->all(), $rule)->validate();
```

## bail 驗證失敗後停止後續規則驗證

規則 `A|B|C`，預設是規則 A 沒有通過，會繼續檢查 B ，然後是 C，所以錯誤訊息列出最多 3 個；如果最前面加上 `bail`，則只要有規則沒有通過，後續的不會繼續檢查，所以錯誤訊息最多只有 1 個

## before:date 某天之前(不含那天)

和 after:date 相同，只是日期是在之前

## before_or_equal:date 某天之前(含那天)

和 after_or_equal:date 相同，只是日期是在之前

## between:min,max 字串、陣列、檔案範圍

`between:1,3` 是指字串長度或一維陣列元素數量從 1 到 3，或檔案大小 1 到 3 KB，所以

| 輸入值 | 驗證通過 |
| ------- | -------- |
| a | O |
| abc | O |
| abcd | X |
| 一 | O |
| 一二三 | O |
| 一二三四 | X |
| 數字 0 | O |
| 數字 1 | O |
| 數字 123 | O |
| 數字 1234 | X |
| 陣列 `[ 'a' ]` | O |
| 陣列 `[ 'a', 'b', 'c' ]` | O |
| 陣列 `[ 'a', 'b', 'c' => [ 'd', 'e', 'f', 'g' ] ]` | O |
| 陣列 `[ 'a', 'b', 'c', 'd' ]` | X |
| 檔案大小 1 KB (1024 bytes) | O |
| 檔案大小 3 KB (3072 bytes) | O |
| 檔案大小 3073 bytes | X |

## boolean 可以轉成 true 或 false 的

true 或 false，以及可以轉換成 true 或 false 的值

| 輸入值 | 驗證通過 |
| ------- | -------- |
| `true` | O |
| `false` | O |
| 數字 1 | O |
| 數字 0 | O |
| 字串 1 | O |
| 字串 0 | O |
| 數字 2 | X |
| 空格字串 `'   '` | O |
| 空字串 `''` | O |
| Tab 字串 `"\t\t\t"` | O |
| A | X |
| a | X |
| `null` | X |

## confirmed

規則 `[ 'test' => 'confirmed' ]` 是指要有另一個欄位 test_confirmation，並且兩個欄位的值要一樣。但是實測發現以下的輸入值會通過驗證，所以請過濾使用者的輸入值

``` php
<?php
$input = [
    // 或者 test => '' 也是一樣的結果
    'test' => ' ',
    // 或者 test_confirmation => "\t\t\t" 也是一樣的結果
    'test_confirmation' => '        '
]
$rule = [
    'test' => 'confirmed'
];
$validator = \Validator::make($input, $rule)->validate();
```

## date

PHP 函式 [strtotime()](https://www.php.net/manual/en/function.strtotime)可以接受的日期格式，但是僅限絕對日期。由以下的表格可知，最好使用 ISO 8601 (`YYYY-MM-DD`)，比較沒有問題

| 輸入值 | 驗證通過 |
| ------- | -------- |
| 2022-07-26 | O |
| 10 September 2000 | O |
| 07/26/2022 | O |
| 26/07/2022 | X |
| 07-26-2022 | X |
| now | X |
| last week | X |
| +1 day | X |

## date_equals:date 等於那天

等於指定的日期或是指定欄位，PHP 函式 [strtotime()](https://www.php.net/manual/en/function.strtotime)可以接受的日期格式。

``` php
<?php
[
    # 改成 next thursday 也可以
    'publish_date' => 'date_equals:next Thursday',
    # 改成 date_equals:20 July 2022 也可以
    'birthday' => 'date_equals:2022-07-20',
    # 指定欄位
    'start_date' => 'date',
    'end_date' => 'date_equals:start_date'
]
```

## date_format:format 指定日期格式

PHP 可以接受的[日期格式](https://www.php.net/manual/en/datetime.format.php)，實測結果比較特別的列在以下表格。「應該通過卻沒有」的最好都別用，「時分秒」出人意料的可以正常運作也都通過驗證

原始碼在  Laravel 6 的 
[vendor/laravel/framework/src/Illuminate/Validation/Concerns/ValidatesAttributes.php](https://github.com/illuminate/validation/blob/934f75b6afd5b731881e3226df429470c62a9f3a/Concerns/ValidatesAttributes.php#L402)

關鍵在於以下程式碼的最後兩行 [DateTime::createFromFormat()](https://www.php.net/manual/en/datetime.createfromformat.php)，使用範例參考[DateTimeImmutable::createFromFormat()](https://www.php.net/manual/en/datetimeimmutable.createfromformat.php)

``` php
<?php
/**
規則 date_format:Y-m-d，輸入值 2022-07-08

$attribute: 表單欄位名稱
$value: 表單欄位的值，例如 2022-07-08
$parameters: 冒號右邊的 Y-m-d 參數，可以用逗號分隔多個參數，所以是陣列
**/
public function validateDateFormat($attribute, $value, $parameters)
{
    $this->requireParameterCount(1, $parameters, 'date_format');
    
    if (! is_string($value) && ! is_numeric($value)) {
        return false;
    }

    $format = $parameters[0];

    $date = DateTime::createFromFormat('!'.$format, $value);

    return $date && $date->format($format) == $value;
}
```

| format | 輸入值(皆爲字串) | 說明 |
| ---- | ------------------------- | --- |
| d | 1 | 應該失敗卻通過 |
| j | 01 | 應該失敗卻通過 |
| N | 1 到 7 | 應該通過卻沒有 |
| Y-m-d-N | 2022-07-01-5 | 應該通過卻沒有 |
| jS | 1st, 2nd, 3rd 或 4th | S 要和 j 一起使用 |
| w | 0 到 6 | 應該通過卻沒有 |
| z | 365 | 應該通過卻沒有（但是 364 通過了） |
| W | 0 到 52 | 應該通過卻沒有 |
| t | 28 到 31 | 沒有通過 |
| L | 0 和 1 | 沒有通過 |
| Y | 0, 0001, 0787 到 3000 | 負值都不通過 |
| a | am 和 pm | 應該失敗卻通過 |
| A | AM 和 PM | 應該失敗卻通過 |
| B | 000 到 999 | 沒有通過 |
| g | 1 到 12 | 應該失敗卻通過 |
| G | 01, 1 到 23 | 應該失敗卻通過 |
| h | 01, 1 到 12 | 應該失敗卻通過 |
| H | 00, 01, 0, 1 到 23 | 應該失敗卻通過 |
| i | 00 到 59 | 應該失敗卻通過 |
| s | 00 到 59 | 應該失敗卻通過 |
| u | 0,00, 000000, 000001 到 999999 | 應該失敗卻通過 |
| v | 0, 100 到 999 | 應該失敗卻通過 |
| v | 1 到 99 | 驗證失敗，所以 0 直接跳到 100 |
| e | UTC, GMT, Asia/Taipei | 應該失敗卻通過 |
| 大寫 I | 0 和 1 | 沒有通過 |
| O | +0800 | 通過 |
| P | +08:00 | 通過 |
| p |     | 沒有實測(因爲 PHP 7.4)，但應該會通過 |
| T | EST, MDT | 通過(但是 +05 失敗) |
| Z | -43200 到 50400 | 失敗 |
| c | 2004-02-12T15:19:21+00:00 | 失敗 |
| r | Thu, 21 Dec 2000 16:01:07 +0200 | 失敗 |
| U | 1657160589 | 通過 |
| D Y-m-d H:i:s a | Thu 2022-07-07 04:31:33 am | 通過 |

## different:field 和指定欄位不同

test 指定和欄位 A 的值不同，所以以下會驗證通過

``` php
<?php
$input = [
    'test' => 'a',
    'A' => 'A'
];
$rule = [
    'test' => 'different:A'
];
$validator = \Validator::make($input, $rule)->validate();
```

## digits:value 指定位數

`digits:3` 是指欄位值是數字，而且是十進位 3 位數。注意，實測發現輸入空字串，不管規則是幾位數都會通過，但是輸入 `null` 則不會通過

## digits_between:min,max 指定位數範圍

`digits:1,3` 是指欄位值是數字，而且是十進位 1 到 3 位數。注意，實測發現輸入空字串，不管規則是幾位數都會通過，但是輸入 `null` 則不會通過

## dimensions:條件 圖片尺寸

可用條件包含 `min_width`, `max_width`, `min_height`, `max_height`, `width`, `height`, `ratio`

寬度/高度的比例 `ratio`，可以用分數例如 `4/5` 或小數例如 0.8

可以用 `Rule::dimensions` 建立規則

``` php
<?php
use Illuminate\Validation\Rule;

[
    'photo' => 'dimensions:min_width=100,min_height=200',
    'pic' => [
        Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2)
    ]
]
```

## distinct 陣列值沒有重複

``` html
<p>
    <label>author 0</label>
    <input type="text" name="author[]">
</p>
<p>
    <label>author 1</label>
    <input type="text" name="author[]">
</p>
<p>
    <label>author 2</label>
    <input type="text" name="author[]">
</p>
```

以下驗證 author 是否爲陣列，以及陣列值是否沒有重複

``` php
<?php
$rule = [
    'author' => 'array',
    'author.*' => 'distinct'
];
$validator = \Validator::make($request->all(), $rule)->validate();
```

## email

使用 [egulias/email-validator](https://github.com/egulias/EmailValidator) 驗證電子郵件位址的格式是否正確，有以下幾種驗證器，預設使用 rfc

-   `rfc`：`RFCValidation`
-   `strict`：`NoRFCWarningsValidation`
-   `dns`：`DNSCheckValidation` (會檢查 @ 之後的網址是否有效，所以 tom@example.com 會驗證失敗)
-   `spoof`：`SpoofCheckValidation` (文件提到：Will check for multi-utf-8 chars that can signal an erroneous email name，看不太懂)
-   `filter`：`FilterEmailValidation` (透過 PHP `filter_var()`，Laravel 5.8 之前使用)

``` php
<?php
[
    'email' => 'email:rfc,dns,spoof'
]
```

SpoofCheckValidation 需要安裝 php intl extension

``` bash
sudo apt install php-intl
```

## ends_with: A,B 結尾是 A 或 B

`ends_with:A,B` 結尾是 A 或 B，所以 `'123ab'` 會驗證失敗，`'123AB'` 會通過

## exists:table,column 輸入值存在資料庫

如果有一個資料表 states

|   name     | abbreviation | 
| ---------- | ------- |
| Alabama    | AL |
| Alaska     | AK |
| Arizona    | AZ |
| Arkansas   | AR |
| California | CA |

以下規則會去搜尋表單輸入值有沒有在資料表中，前三個的 SQL 是這樣的：

``` sql
select count(*) as aggregate from "states" where "abbreviation" = AL
```

但是最後一個條件，使用 `Rule::exists()` 那個，SQL 是這樣的：

``` sql
select count(*) as aggregate from "states" where "abbreviation" = AL and ("name" = ALABAMA)
```

``` php
<?php
use Illuminate\Validation\Rule;

[
    # 表單輸入 abbreviation => 'AL'
    # 沒有指定資料欄，則預設使用表單欄位名稱
    'abbreviation' => 'exists:states',
    # 表單輸入 test => 'AL'
    # exists:table,column
    'test' => 'exists:states,abbreviation',
    # 表單輸入 test => 'AL'
    # pqsql 是指定使用在 config/database.php 的 connections 設定
    # 請注意 pqsql 和 states 使用 . 連接
    'test' => 'exists:pqsql.states,abbreviation',
    # 表單輸入 abbreviation => 'AL'
    # 雖然用 exists() 的參數指定資料表是 states，但是它會把表單欄位名稱作爲 WHERE 條件
    # 同時合併 $query->where() 的條件
    'abbreviation' => [
        Rule::exists('states')->where(function ($query) {
            $query->where('name', 'ALABAMA');
        })
    ]
]
```

## file 上傳成功的檔案

