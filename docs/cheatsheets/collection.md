``` php
// 建立集合
collect([1, 2, 3]);
// 返回該集合所代表的底層陣列：
$collection->all();
// 返回集合中所有項目的平均值：
collect([1, 1, 2, 4])->avg() // 2
$collection->average();
// 將集合拆成多個給定大小的較小集合：
collect([1, 2, 3, 4, 5])->chunk(2); // [[1,2], [3,4], [5]]
// 將多個陣列組成的集合折成單一陣列集合：
collect([[1],  [4,  5]])->collapse(); // [1, 4, 5]
// 將一個集合的值作為鍵，再將另一個集合作為值合併成一個集合
collect(['name', 'age'])->combine(['George', 29]);
// 將給定的 陣列 或集合值追加到集合的末尾
collect(['PHP'])->concat(['Laravel']); // ['PHP', 'Laravel']
// 用來判斷該集合是否含有指定的項目：
collect(['name' => 'Desk'])->contains('Desk'); // true
collect(['name' => 'Desk'])->contains('name',  'Desk'); // true
// 返回該集合內的項目總數：
$collection->count();
// 交叉連接指定陣列或集合的值，返回所有可能排列的笛卡爾積
collect([1, 2])->crossJoin(['a', 'b']); // [[1, 'a'],[1, 'b'],[2, 'a'],[2, 'b']]
// dd($collection) 的另一個寫法
collect(['John Doe', 'Jane Doe'])->dd();
// 返回原集合中存在而指定集合中不存在的值
collect([1,  2,  3])->diff([2, 4]); // [1, 3]
// 返回原集合不存在與指定集合的鍵 / 值對
collect(['color' => 'orange', 'remain' =>  6])->diffAssoc(['color' => 'yellow', 'remain' => 6, 'used' => 6]);  // ['color' => 'orange']
// 返回原集合中存在而指定集合中不存在鍵所對應的鍵 / 值對
collect(['one' => 10, 'two' => 20])->diffKeys(['two' => 2, 'four' => 4]); // ['one' => 10]
// 類似於 dd() 方法，但是不會中斷
collect(['John Doe', 'Jane Doe'])->dump();
// 遍歷集合中的項目，並將之傳入給定的回呼函數：
$collection = $collection->each(function ($item, $key) {});
// 驗證集合中的每一個元素是否通過指定的條件測試
collect([1,  2])->every(function  ($value,  $key)  { return $value > 1; }); // false
// 返回集合中排除指定鍵的所有項目：
$collection->except(['price', 'discount']);
// 以給定的回呼函數篩選集合，只留下那些通過判斷測試的項目：
$filtered = $collection->filter(function ($item) {
    return $item > 2;
});
// 返回集合中，第一個通過給定測試的元素：
collect([1, 2, 3, 4])->first(function ($key, $value) {
    return $value > 2;
});
// 將多維集合轉為一維集合：
$flattened = $collection->flatten();
// 將集合中的鍵和對應的數值進行互換：
$flipped = $collection->flip();
// 以鍵自集合移除掉一個項目：
$collection->forget('name');
// 返回含有可以用來在給定頁碼顯示項目的新集合：
$chunk = $collection->forPage(2, 3);
// 返回給定鍵的項目。如果該鍵不存在，則返回 null：
$value = $collection->get('name');
// 根據給定的鍵替集合內的項目分組：
$grouped = $collection->groupBy('account_id');
// 用來確認集合中是否含有給定的鍵：
$collection->has('email');
// 用來連接集合中的項目
$collection->implode('product', ', ');
// 移除任何給定陣列或集合內所沒有的數值：
$intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);
// 假如集合是空的，isEmpty 方法會返回 true：
collect([])->isEmpty();
// 以給定鍵的值作為集合項目的鍵：
$keyed = $collection->keyBy('product_id');
// 傳入回呼函數，該函數會返回集合的鍵的值：
$keyed = $collection->keyBy(function ($item) {
    return strtoupper($item['product_id']);
});
// 返回該集合所有的鍵：
$keys = $collection->keys();
// 返回集合中，最後一個通過給定測試的元素：
$collection->last();
// 遍歷整個集合併將每一個數值傳入給定的回呼函數：
$multiplied = $collection->map(function ($item, $key) {
    return $item * 2;
});
// 返回給定鍵的最大值：
$max = collect([['foo' => 10], ['foo' => 20]])->max('foo');
$max = collect([1, 2, 3, 4, 5])->max();
// 將合併指定的陣列或集合到原集合：
$merged = $collection->merge(['price' => 100, 'discount' => false]);
// 返回給定鍵的最小值：
$min = collect([['foo' => 10], ['foo' => 20]])->min('foo');
$min = collect([1, 2, 3, 4, 5])->min();
// 返回集合中指定鍵的所有項目：
$filtered = $collection->only(['product_id', 'name']);
// 獲取所有集合中給定鍵的值：
$plucked = $collection->pluck('name');
// 移除並返回集合最後一個項目：
$collection->pop();
// 在集合前面增加一個項目：
$collection->prepend(0);
// 傳遞第二個參數來設定前置項目的鍵：
$collection->prepend(0, 'zero');
// 以鍵從集合中移除並返回一個項目：
$collection->pull('name');
// 附加一個項目到集合後面：
$collection->push(5);
// put 在集合內設定一個給定鍵和數值：
$collection->put('price', 100);
// 從集合中隨機返回一個項目：
$collection->random();
// 傳入一個整數到 random。如果該整數大於 1，則會返回一個集合：
$random = $collection->random(3);
// 會將每次迭代的結果傳入到下一次迭代：
$total = $collection->reduce(function ($carry, $item) {
    return $carry + $item;
});
// 以給定的回呼函數篩選集合：
$filtered = $collection->reject(function ($item) {
    return $item > 2;
});
// 反轉集合內項目的順序：
$reversed = $collection->reverse();
// 在集合內搜尋給定的數值並返回找到的鍵：
$collection->search(4);
// 移除並返回集合的第一個項目：
$collection->shift();
// 隨機排序集合的項目：
$shuffled = $collection->shuffle();
// 返回集合從給定索引開始的一部分切片：
$slice = $collection->slice(4);
// 對集合排序：
$sorted = $collection->sort();
// 以給定的鍵排序集合：
$sorted = $collection->sortBy('price');
// 移除並返回從指定的索引開始的一小切片項目：
$chunk = $collection->splice(2);
// 返回集合內所有項目的總和：
collect([1, 2, 3, 4, 5])->sum();
// 返回有著指定數量項目的集合：
$chunk = $collection->take(3);
// 將集合轉換成純 PHP 陣列：
$collection->toArray();
// 將集合轉換成 JSON：
$collection->toJson();
// 遍歷集合併對集合內每一個項目呼叫給定的回呼函數：
$collection->transform(function ($item, $key) {
    return $item * 2;
});
// 返回集合中所有唯一的項目：
$unique = $collection->unique();
// 返回鍵重設為連續整數的的新集合：
$values = $collection->values();
// 以一對給定的鍵／數值篩選集合：
$filtered = $collection->where('price', 100);
// 將集合與給定陣列同樣索引的值合併在一起：
$zipped = $collection->zip([100, 200]);
//把集合放到回呼參數中並返回回呼的結果:
collect([1, 2, 3])->pipe(function ($collection) {
    return $collection->sum();
});//6
```
