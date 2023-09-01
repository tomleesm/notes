``` php
// 自動處理分頁邏輯
Model::paginate(15);
Model::where('cars', 2)->paginate(15);
// 使用簡單範本 - 只有 "上一頁" 或 "下一頁" 連結
Model::where('cars', 2)->simplePaginate(15);
// 手動分頁
Paginator::make($items, $totalItems, $perPage);
// 在頁面列印分頁導覽列
$variable->links();

// 獲取當前頁資料數量。
$results->count()
// 獲取當前頁頁碼。
$results->currentPage()
// 獲取結果集中第一條資料的結果編號。
$results->firstItem()
// 獲取分頁器選項。
$results->getOptions()
// 建立分頁 URL 範圍。
$results->getUrlRange($start, $end)
// 是否有多頁。
$results->hasMorePages()
// 獲取結果集中最後一條資料的結果編號。
$results->lastItem()
// 獲取最後一頁的頁碼（在 `simplePaginate` 中無效）。
$results->lastPage()
// 獲取下一頁的 URL 。
$results->nextPageUrl()
// 當前而是否為第一頁。
$results->onFirstPage()
// 每頁的資料條數。
$results->perPage()
// 獲取前一頁的 URL。
$results->previousPageUrl()
// 資料總數（在 `simplePaginate` 無效）。
$results->total()
// 獲取指定頁的 URL。
$results->url($page)
```
