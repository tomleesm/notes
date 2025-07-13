---
hide:
  - toc
---

``` blade
// 輸出內容，被轉義過的
{{ $var }}
// 輸出未轉義內容
{!! $var !!}
{{-- Blade 註釋，不會被輸出到頁面中 --}}
// 三元表示式的簡寫，以下相當於「isset($name) ? $name : 'Default'」
{{ $name ?? 'Default' }}
// 等同 echo json_encode($array);
@json($array);
// 停用 HTML 實體雙重編碼
Blade::withoutDoubleEncoding();
// 書寫 PHP 程式碼
@php
@endphp
@csrf // CSRF 域
@method('PUT') // HTML 表單偽造方法 _method
// 服務容器注入，後呼叫 {{ $metrics->monthlyRevenue() }}
@inject('metrics', 'App\Services\MetricsService')
包含和繼承
// 擴展佈局範本
@extends('layout.name')
// 區塊佔位
@yield('name')
// 第一種、直接填入擴展內容
@section('title',  'Page Title')
// 第二種、實現命名為 name 的區塊（yield 佔位的地方）
@section('sidebar')
    // 繼承父範本內容
    @parent
@endsection
// 可繼承內容區塊
@section('sidebar')
@show
// 繼承父範本內容（@show 的區塊內容）
@parent
// 包含子檢視
@include('view.name')
// 包含子檢視，並傳參
@include('view.name', ['key' => 'value']);
@includeIf('view.name',  ['some'  =>  'data'])
@includeWhen($boolean,  'view.name',  ['some'  =>  'data'])
// 包含給定檢視陣列中第一個存在的檢視
@includeFirst(['custom.admin',  'admin'],  ['some'  =>  'data'])
// 載入本地化語句
@lang('messages.name')
@choice('messages.name', 1);
// 檢查片斷是否存在
@hasSection('navigation')
        @yield('navigation')
@endif
// 迭代 jobs 陣列并包含
@each('view.name',  $jobs,  'job')
@each('view.name',  $jobs,  'job',  'view.empty')
// 堆疊
@stack('scripts')
@push('scripts')
    <script src="/example.js"></script>
@endpush
// 棧頂插入
@prepend('scripts')
@endprepend
// 元件
@component('alert', ['foo' => 'bar'])
    @slot('title')
    @endslot
@endcomponent
// 註冊別名 @alert(['type' => 'danger'])...@endalert
Blade::component('components.alert',  'alert');
條件語句
@if (count($records) === 1)
@elseif  (count($records) > 1)
@else
@endif
// 登錄情況下
@unless (Auth::check())
@endunless
// $records 被定義且不是  null...
@isset($records)
@endisset
// $records 為空...
@empty($records)
@endempty
// 此使用者身份已驗證...
@auth // 或 @auth('admin')
@endauth
// 此使用者身份未驗證...
@guest // 或 @guest('admin')
@endguest
@switch($i)
    @case(1)
        @break
    @default
        // 默認
@endswitch
循環
// for 循環
@for ($i = 0; $i < 10; $i++)
@endfor
// foreach 迭代
@foreach ($users as $user)
@endforeach
// 迭代如果為空的話
@forelse ($users as $user)
@empty
@endforelse
// while 循環
@while (true)
@endwhile
// 終結循環
@continue
@continue($user->type  ==  1) // 帶條件
// 跳過本次迭代
@break
@break($user->number  ==  5) // 帶條件
// 循環變數
$loop->index        // 當前迭代的索引（從 0 開始計數）。
$loop->iteration    // 當前循環迭代 (從 1 開始計算）。
$loop->remaining    // 循環中剩餘迭代的數量。
$loop->count        // 被迭代的陣列元素的總數。
$loop->first        // 是否為循環的第一次迭代。
$loop->last         // 是否為循環的最後一次迭代。
$loop->depth        // 當前迭代的巢狀深度級數。
$loop->parent       // 巢狀循環中，父循環的循環變數
JavaScript 程式碼
// JS 框架，保留雙大括號，以下會編譯為 {{ name }}
@{{ name }}
// 大段 JavaScript 變數，verbatim 裡範本引擎將不解析
@verbatim
        Hello, {{ javascriptVariableName }}.
@endverbatim
```
