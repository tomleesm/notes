# PHP 物件導向語法

參考 [PHP 官方文件](https://www.php.net/manual/en/language.oop5.php) ，沒看過、記不起來，或覺得有必要的才在這裡寫筆記。

## Properties
class 中的變數，至少要有一個修飾詞  [Visibility](https://www.php.net/manual/en/language.oop5.visibility.php)、 [Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) 或是 [readonly](https://www.php.net/manual/en/language.oop5.properties.php#language.oop5.properties.readonly-properties) (PHP 8.1.0 以上)。宣告時的初始值必須是常數(基本型別或 constant )，表示必須在編譯時期確定其值，而不是在 runtime 時。沒有 Visibility 的屬性，預設是 public，所以 `static $var = 1` ，則此屬性預設是 public

PHP 7.4.0 開始，可以有 [Type declarations](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations) 的屬性，必須在存取前初始化，否則會丟出 [Error](https://www.php.net/manual/en/class.error.php) ，不會自動給預設值 0, 0.0F, 或空字串。沒有  [Type declarations](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations) 的屬性 ( 例如 `public $var`)，則會給預設值 `null`

## Class Constants

一個類別建立一個 class constant，不是一個物件建立一個。

``` php
<?php
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

## Object Inheritance

子類別必須和父類別相同，或比父類別更開放或更多。參考 [Signature compatibility rules](https://www.php.net/manual/en/language.oop5.basic.php#language.oop.lsp)

- 父類別方法 `protected`；子類別方法必須是 `protected` 或 `public`
- 父類別方法有一個參數，而且參數沒有預設值；子類別方法必須有一個參數以上，而且參數可以有預設值
- 父類別方法沒有 return  [Type declarations](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations)，子類別方法可以有 return  [Type declarations](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations)`
- 如果 return  [Type declarations](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations) 是類別，爲了遵守基本的多型機制(里式替換原則)，所以子類別可以比父類別的 return type 更緊縮，方法的參數型別則是子類別可以比父類別更開放。參考 [Covariance and Contravariance](https://www.php.net/manual/en/language.oop5.variance.php "Covariance and Contravariance")

## Traits
方法名稱衝突時

``` php
<?php
trait A {
    public function smallTalk() {
        echo 'a';
    }
    public function bigTalk() {
        echo 'A';
    }
    public function otherTalk() {
        echo '+';
    }
}
trait B {
    public function smallTalk() {
        echo 'b';
    }
    public function bigTalk() {
        echo 'B';
    }
    public function otherTalk() {
        echo '-';
    }
}
trait C {
    public function smallTalk() {
        echo 'c';
    }
    public function bigTalk() {
        echo 'C';
    }
    public function otherTalk() {
        echo '*';
    }
}
class Talker
{
    // 三個 trait 方法名稱都一樣，在 use {} 中指定用哪個
    use A, B, C {
        // 使用 B 的 smallTalk() 而不是 A 或 C 的
        // 注意是 B::smallTalk，沒有小括號！沒有小括號！沒有小括號！
        B::smallTalk insteadof A, C;
        A::bigTalk insteadof B, C;
        C::otherTalk insteadof A, B;
        // C 的 otherTalk 可以改名 talk
        // 所以 $t->talk() 和 $t->otherTalk() 都可以
        C::otherTalk as talk;
    }
}

$t = new Talker();
$t->smallTalk(); // b
$t->bigTalk(); // A
$t->otherTalk(); // *
$t->talk(); // *
?>
```

trait 可以定義靜態屬性，如果類別使用這個 trait，則此靜態屬性是隨著物件生成的，不是隨著 trait，也就是說以下的 trait Counter，如果被類別 C1 和 C2 使用，則會有兩個靜態屬性 `$c` 分屬 C1 和 C2，不是只有一個

``` php
<?php
trait Counter {
  private static $c = 0;
  
  public function inc() {
    self::$c++;
    echo self::$c . "\n";
  }
}
```

## Overloading

一般程式語言的 overloading 是指 function 可以有不同的參數種類和數量，例如 [Java String](https://docs.oracle.com/javase/7/docs/api/java/lang/String.html) Constructors 有 15 個(Java 使用和類別相同名稱的 method 作爲建構式)， Java 程式依照參數數量(0 ~ 4 個)和參數型別決定要呼叫哪一個建構式。

但是 PHP 的 overloading 指的是動態產生類別的屬性和方法。下表的屬性 `name` 在類別中沒有定義或不是設爲 `public` ，所以是在類別定義外面直接設定和存取的

| magic function | 何時執行 | 參數意義 | 回傳值 |
| --------------- | -------- | ---------- | ------- |
| `__set(string, $name, mix $value): void` | `$object->name = 123` | `$name`：屬性的名稱(字串 name)，`$value`：屬性被設定的值(123) | 沒有回傳值 |
| `__get(string $name): mix` | 存取 `$object->name` | `$name`：屬性的名稱(字串 name) | 在`__get()` 中回傳的值，決定 `$object->name` 屬性的值 |
| `__isset(string $name): bool` | `isset($object->name)` | `$name`：屬性的名稱(字串 name) | 在 `__isset()` 中回傳的值，決定 `isset()` 的值。回傳值如果不是 bool 型別，會自動轉型 |
| `__unset(string $name): void` | `unset($object->name)` | `$name`：屬性的名稱(字串 name) | 沒有回傳值 |

下表的方法 `run` 在類別中沒有定義或不是設爲 `public`，所以是在類別定義外面直接呼叫

| magic function | 何時執行 | 參數意義 | 回傳值 |
| --------------- | -------- | ---------- | ------ |
| `public __call(string $name, array $arguments): mix` | `object->run('arg 1', 123)` | `$name`：呼叫的方法名稱(字串 run)，`$arguments`：呼叫方法時所有的參數( `['arg 1', 123]` ) | 呼叫方法的回傳值 |
| `public __callStatic(string $name, array $arguments): mix` | `MyClass::run('arg 2', 456)` | `$name`：呼叫的靜態方法名稱(字串 run)，`$arguments`：呼叫靜態方法時所有的參數( `['arg 2', 456]` ) | 呼叫靜態方法的回傳值 |

## Final

final 只能用在類別和方法。顧名思義爲「最後」，表示不會改變。

- 屬性不能設爲 `final`，而是使用 `const` 代表屬性是不能改變的常數
- 方法設爲 `final`，則不能被子類別 `override` ，改變成另一種方法
- 類別設爲 `final`，則不能被繼承，改變成另一種類別

## Object Cloning

使用 `clone $obj` 或 `clone($obj)` 複製物件的所有屬性和值，包含 `private` 和 `protected` 屬性，如果有定義 `__clone()`，則會在複製屬性後執行它

## Object Serialization

`serialize()` 會把物件轉成 byte-stream 字串，`unserialize()` 根據此字串還原成物件。所以可以把物件 Serialization 後存成檔案放到其他地方，之後再 Unserialization 還原成物件後繼續執行

``` php
<?php
// classa.inc:
  
  class A {
      public $one = 1;
    
      public function show_one() {
          echo $this->one;
      }
  }
  
// page1.php:

  include("classa.inc");
  
  $a = new A;
  $s = serialize($a);
  // store $s somewhere where page2.php can find it.
  file_put_contents('store', $s);

// page2.php:
  
  // this is needed for the unserialize to work properly.
  include("classa.inc");

  $s = file_get_contents('store');
  $a = unserialize($s);

  // now use the function show_one() of the $a object.  
  $a->show_one();
?>
```

執行 `php page1.php` 會在同一目錄新增檔案 store，內容是字串 `O:1:"A":1:{s:3:"one";i:1;}`，然後 `php page2.php` 會依據此字串還原成物件，繼續執行

官方文件的讀者留言

> Note that static members of an object are not serialized.

但是實測結果，新增一個 `public static $two = 2` ，結果完全可以存取

官方文件寫說如果在 Object Serialization 前沒有類別定義，PHP 會把它變成 `__PHP_Incomplete_Class_Name` 類別的物件，基本上沒什麼用。實測會產生 PHP Fatal error，無法繼續執行