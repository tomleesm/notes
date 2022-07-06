# 驗證規則清單

## accepted 同意

`yes`、`on`、`1` 或 `true`，或上述值的字串形式，適合「同意服務協議」

## active_url 有效網址

基於 PHP 函式 [dns_get_record()](https://www.php.net/manual/en/function.dns-get-record.php) 的，有 A 或 AAAA 記錄的值，也就是查的到 ip 的。驗證時會停一下，看來是真的用 dns_get_record() 查詢。

開頭必須有 `http(s)://` 的有效網址，例如 https://www.google.com ，或 http://127.0.0.1 也可以

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

輸入必須是 PHP 陣列，例如以下這樣，但是表單欄位無法產生這樣的輸入，所以實際上無法使用這個規則

``` php
<?php
$input = [
    'test' => [ 'A', 'B' ]
];
$rule = [
    'test' => 'array'
];
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

## date_format:format

