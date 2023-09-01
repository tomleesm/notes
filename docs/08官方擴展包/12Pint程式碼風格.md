# Pint 程式碼風格

## 介紹

[Laravel Pint](https://github.com/laravel/pint) 是一款面向極簡主義者的 PHP 程式碼風格固定工具。Pint 是建立在 PHP-CS-Fixer 基礎上，使保持程式碼風格的整潔和一致變得簡單。

Pint 會隨著所有新的 Laravel 應用程式自動安裝，所以你可以立即開始使用它。默認情況下，Pint 不需要任何組態，將通過遵循 Laravel 的觀點性編碼風格來修復你的程式碼風格問題。

## 安裝

Pint 已包含在 Laravel 框架的最近版本中，所以無需安裝。然而，對於舊的應用程式，你可以通過 Composer 安裝 Laravel Pint：

```shell
composer require laravel/pint --dev
```

## 運行 Pint

可以通過呼叫你項目中的 `vendor/bin` 目錄下的 `pint` 二進制檔案來指示 Pint 修復程式碼風格問題：

```shell
./vendor/bin/pint
```

你也可以在特定的檔案或目錄上運行 Pint：

```shell
./vendor/bin/pint app/Models

./vendor/bin/pint app/Models/User.php
```

Pint 將顯示它所更新的所有檔案的詳細列表。 你可以在呼叫 Pint 時提供 `-v` 選項來查看更多關於 Pint 修改的細節。：

```shell
./vendor/bin/pint -v
```

如果你只想 Pint 檢查程式碼中風格是否有錯誤，而不實際更改檔案，則可以使用 `--test` 選項：

```shell
./vendor/bin/pint --test
```

如果你希望 Pint 根據 Git 僅修改未提交更改的檔案，你可以使用 `--dirty` 選項：

```shell
./vendor/bin/pint --dirty
```

## 組態 Pint

如前面所述，Pint 不需要任何組態。但是，如果你希望自訂預設、規則或檢查的資料夾，可以在項目的根目錄中建立一個 `pint.json` 檔案：

```json
{
  "preset": "laravel"
}
```

此外，如果你希望使用特定目錄中的 `pint.json`，可以在呼叫 Pint 時提供 `--config` 選項：

```shell
pint --config vendor/my-company/coding-style/pint.json
```

### Presets(預設)

Presets 定義了一組規則，可以用來修復程式碼風格問題。默認情況下，Pint 使用 laravel preset，通過遵循 `Laravel` 的固定編碼風格來修復問題。但是，你可以通過向 Pint 提供 `--preset` 選項來指定一個不同的 preset 值：

```shell
pint --preset psr12
```

如果你願意，還可以在項目的 `pint.json` 檔案中設定 preset ：

```json
{
  "preset": "psr12"
}
```

Pint 目前支援的 presets 有：`laravel`、`psr12` 和 `symfony`。

### 規則

規則是 Pint 用於修復程式碼風格問題的風格指南。如上所述，presets 是預定義的規則組，適用於大多數 PHP 項目，因此你通常不需要擔心它們所包含的單個規則。

但是，如果你願意，可以在 `pint.json` 檔案中啟用或停用特定規則：

```json
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "braces": false,
        "new_with_braces": {
            "anonymous_class": false,
            "named_class": false
        }
    }
}
```

Pint是基於 [PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) 建構的。因此，您可以使用它的任何規則來修復項目中的程式碼風格問題： [PHP-CS-Fixer Configurator](https://mlocati.github.io/php-cs-fixer-configurator).

### 排除檔案/資料夾

默認情況下，Pint將檢查項目中除 `vendor` 目錄以外的所有 `.php` 檔案。如果您希望排除更多資料夾，可以使用 `exclude` 組態選項:

```json
{
    "exclude": [
        "my-specific/folder"
    ]
}
```

如果您希望排除包含給定名稱模式的所有檔案，則可以使用 `notName` 組態選項:

```json
{
    "notName": [
        "*-my-file.php"
    ]
}
```

如果您想要通過提供檔案的完整路徑來排除檔案，則可以使用 `notPath` 組態選項:

```json
{
    "notPath": [
        "path/to/excluded-file.php"
    ]
}
```

