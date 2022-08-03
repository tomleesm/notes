# Blade

## 顯示值

``` blade
# 顯示變數的值，$name 使用 htmlentities() 處理，以避免 XSS
{{ $name }}

# 可以將任何 PHP 程式碼放入 {{ }} 中
{{ time() }}

# 不使用 htmlentities() 處理
{!! $name !!}

{{-- 註解 --}}

# 如果 $array = [ 'name' => 'Tom' ]
# json = { "name" => "Tom" }，方便設定 JavaScript 變數初始值
# 結尾的分號不能省略
var json = @json($array);
# 整齊的縮排
json = @json($array, JSON_PRETTY_PRINT);
```

Vue 之類的 JavaScript Framework 同樣使用 `{{ }}`  顯示變數的值，Blade 使用 `@{{ }}` 或 `@verbatim` 輸出 `{{ }}`

``` blade
# 使用 @{{ }} 輸出 {{ }}
<h1>@{{ title }}</h1>
<h2>@{{ author }}</h2>
<h3>@{{ message }}</h3>

# 或者包在 @verbatim 內
@verbatim
<h1>{{ title }}</h1>
<h2>{{ author }}</h2>
<h3>{{ message }}</h3>
@endverbatim

# 以上都是輸出
<h1>{{ title }}</h1>
<h2>{{ author }}</h2>
<h3>{{ message }}</h3>
```

## 指令

`@` 開頭是指令，例如：

``` blade
@if(count($array) === 1)
...
@endif
```

等於

``` php
<?php
if(count($array) === 1) {
  ...
}
```

### include

``` blade
# 執行原始的 php code
@php
  $a = 'a';
@endphp

# 包含其他的 blade.php，並傳遞 $i = '123' 給 resources/views/partials/sidebar.blade.php
# 上面的 $a = 'a' 可以在 partials.sidebar 中存取
# 但是在 partials.sidebar 定義的變數無法給 include 它的 view 存取
@include('partials.sidebar', [ 'i' => '123' ])
```

## 繼承

resources/views/layouts/app.blade.php

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- @yield() 的第二個參數是預設值 -->
    <title>@yield('title', 'default title')</title>
</head>
<body>
    <!-- @show 表示定義了 @section 後立刻顯示出來 -->
    @section('sidebar')
        Sidebar
    @show

    <div class="cintainer">
        @yield('content')
    </div>
</body>
</html>
```

``` blade
# 繼承 resources/views/layouts/app.blade.php
@extends('layouts.app')

# 尋找繼承的 view 中的 @yield('title') 或 section('title') 然後替換成 Laravel blade
@section('title', 'Laravel blade')

# 尋找繼承的 view 中的 @yield('sidebar') 或 @section('sidebar')
# 然後替換成用 @endsection 包起來的這兩行
# @parent 會用繼承的 view 的 @yield('sidebar') 或 @section('sidebar') 內容取代，
# 也就是 Sidebar
@section('sidebar')
    @parent
    <p>This is sidebar from child</p>
@endsection

# 尋找繼承的 view 中的 @yield('content') 或 section('content')
# 然後替換成 <p>content</p>
@section('content')
    <p>content</p>
@endsection
```

## slot

定義在很多地方都出現的 HTML，例如以下的 alert，然後用 `@component()` 填入 alert 中需要的變數

``` html
<!-- resources/views/alert.blade.php -->
<!-- -->
<div class="alert alert-danger">
    <div class="alert-title">{{ $title }}</div>
    {{ $slot }}
</div>
```

`@component()` 會載入上面的 alert.blade.php，`@slot('title')` 命名 slot 定義了 `$title` 的值 Forbidden，然後其他全部都是 `$slot` 的值。

`@component()` 第二個參數傳入 `$other = '其他額外的變數'` 給 alert。注意，它會 escape，所以 `<h2>` 會變成 `&lt;h2&gt;`

``` blade
# @component(要載入的 view, 額外的變數)
@component('alert', ['other' => '其他額外的變數'])
    # 命名 slot：定義 $title = 'Forbidden' (注意！不會 escape)
    @slot('title')
        Forbidden
    @endslot
    # 其他的都是 $slot 的值。注意不會 escape
    # 所以 $slot = 'You are not allowed to access this resource!'
    You are not allowed to access this resource!
@endcomponent
```

`@componentFirst()` 第一個參數是 2 個 view 名稱的陣列，先載入第一個 view，如果沒有才載入第二的 view。其餘都和 `@component()` 一樣

``` blade
# @componentFirst( [ 第一個 view, 預設的 view ], 額外的變數)
@componentFirst([ 'custom.alert', 'alert' ], ['h1' => 'This is <h1> title'])
    @slot('title')
        Forbidden
    @endslot
    You are not allowed to access this resource!
@endcomponent
```

### component 別名

``` php
<?php
// app/Providers/AppServiceProvider.php
use Illuminate\Support\Facades\Blade;

public function boot()
{
    // custom.alert 設定別名 alert
    Blade::component('custom.alert', 'alert');
}

// @alert 等於 @component('custom.alert')
// 沒有額外變數時，可簡寫成 @alert ... @endalert
@alert( ['other' => '其他額外的變數'] )
    @slot('title')
        Forbidden
    @endslot
    You are not allowed to access this resource!
@endalert
```

``` blade
# 清除 Blade cache view，讓 component 別名生效
php artisan view:clear
```