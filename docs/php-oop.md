# PHP 物件導向語法

參考 [PHP 官方文件](https://www.php.net/manual/en/language.oop5.php) ，沒看過、記不起來，或覺得有必要的才在這裡寫筆記。

## Properties
class 中的變數，至少要有一個修飾詞  [Visibility](https://www.php.net/manual/en/language.oop5.visibility.php)、 [Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) 或是 [readonly](https://www.php.net/manual/en/language.oop5.properties.php#language.oop5.properties.readonly-properties) (PHP 8.1.0 以上)。宣告時的初始值必須是常數(基本型別或 constant )，表示必須在編譯時期確定其值，而不是在 runtime 時。沒有 Visibility 的屬性，預設是 public，所以 `static $var = 1` ，則此屬性預設是 public

PHP 7.4.0 開始，可以有 [Type declarations](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations) 的屬性，必須在存取前初始化，否則會丟出 [Error](https://www.php.net/manual/en/class.error.php) ，不會自動給預設值 0, 0.0F, 或空字串。沒有  [Type declarations](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations) 的屬性 ( 例如 `public $var`)，則會給預設值 `null`

## Class Constants

一個類別建立一個 class constant，不是一個物件建立一個。

``` php
class A
{
  const C = 'C';
  
  public function getConstant() {
    return self::C;
  }
}
```

class constant C 只有一個，歸屬於類別 A，即使是 `new` 三個物件 A，分給變數 `$a1`, `$a2` 和 `$a3`，也只有一個 class constant C，所以使用 `$a1::C` 存取，不是 `$a1->C`，在內部存取是用 `self::C`，不過用 `$this->C` 也可以，但是那樣的話用 `$a1->getConstant()` 會產生錯誤。總之，把 class constant 當做 static final 變數來使用

## Autoloading Classes

`new` 一個不屬於標準程式庫的類別，或是定義類別， 會自動執行 `spl_autoload_register()` 第一個參數的 callable，一般是一個函式。通常用來自動載入類別定義，省去一堆的 `require_once()`。目前只需要學會用 composer 就好，不用自己呼叫 `spl_autoload_register()` 。

## Constructor 和 Destructor

如果子類別有定義建構式，必須在第一行用 `parent::__construct()` 先呼叫父類別的建構式。而且子類別不用遵守 [Signature compatibility rules](https://www.php.net/manual/en/language.oop5.basic.php#language.oop.lsp) ，也就是說建構式的參數沒有限制數量，沒有限制要和父類別方法的參數一樣提供預設值等。

PHP 沒有和 Java 一樣，可以對建構式 overloading，一個類別只有建構式，所以推薦使用 [Static creation methods](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.static)。經過測試 (在 PHP 7.4.28 中)，範例中可以改成用 `new Product(1, 'milk')`。

## Scope Resolution Operator (::)

兩個冒號可以有以下語法：

- `self::$staticVar`：同一個類別的靜態變數
- `self::constant`：同一個類別的常數
- `parent::$staticVar`：父類別靜態變數
- `parent::constant`：父類別常數
- `parent::__construct()`：父類別建構式
- `parent::myFunction()`：父類別的靜態和「一般」方法

但是 `parent::$var` 不是存取父類別的一般變數，而是父類別靜態變數。而且沒有 `parent->$var` 這樣的語法