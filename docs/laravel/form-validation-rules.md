# 驗證規則清單

## accepted 同意

`yes`、`on`、`1` 或 `true`，或上述值的字串形式，適合「同意服務協議」

## active_url 有效網址

基於 PHP 函式 [dns_get_record()](https://www.php.net/manual/en/function.dns-get-record.php) 的，有 A 或 AAAA 記錄的值，也就是查的到 ip 的。驗證時會停一下，看來是真的用 dns_get_record() 查詢。

開頭必須有 `http(s)://` 的有效網址，例如 https://www.google.com ，或 http://127.0.0.1 也可以

## after:date 某天之後(不含那天)

傳遞日期到 PHP 函式 [strtotime()](https://www.php.net/manual/en/function.strtotime)，所以日期格式必須是它可以接受的。或是指定欄位

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
$validator = \Validator::make($input, $rule)->validate();
```

## bail 驗證失敗後停止後續規則驗證

規則 `A|B|C`，預設是規則 A 沒有通過，會繼續檢查 B ，然後是 C，所以錯誤訊息列出最多 3 個；如果最前面加上 `bail`，則只要有規則沒有通過，後續的不會繼續檢查，所以錯誤訊息最多只有 1 個

## before:date 某天之前(不含那天)

和 after:date 相同，只是日期是在之前

## before_or_equal:date 某天之前(含那天)

和 after_or_equal:date 相同，只是日期是在之前

## between:min,max