# 檔案上傳

``` php
<?php
# 路由
Route::view('file/upload', 'upload');
```

`view('upload')`

``` html
<form method="post" action="/file/upload" enctype="multipart/form-data">
    @csrf
    <input type="file" name="photo">
    <button type="submit">Upload Image</button>
</form>
```

上傳檔案的設定在 config/filesystems.php
``` php
<?php
Route::post('file/upload', function(Request $request) {
    # 抓取上傳的檔案
    $file = $request->file('photo');
    # 或是 $file = $request->photo;

    # hasFile() 檢查檔案在請求中是否存在
    # isValid() 檢查檔案是否上傳成功
    if($request->hasFile('photo') && $file->isValid()) {
        # store('相對於根目錄的資料夾', 'local 或 s3')
        # 所以檔案儲存在 storage/app/photo
        # 回傳相對於 storage/app 的檔案位置
        # 類似 photo/Jp6jI226FiF90tEI5wzH6Mb1KMoTEI3mf7Q6OJx7.jpg
        # store() 自動產生唯一的檔名
        # storeAs('相對於根目錄的資料夾', '指定的檔名', 'local 或 s3')
        $path = $file->store('photo');
        return 'file upload path: ' . $path;
    } else {
        return 'file upload fail';
    }

});
```