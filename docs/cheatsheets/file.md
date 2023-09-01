``` php
File::exists($path);
File::get($path, $lock = false);
// 加鎖讀取檔案內容
File::sharedGet($path);
// 獲取檔案內容，不存在會拋出 FileNotFoundException 異常
File::getRequire($path);
// 獲取檔案內容, 僅能引入一次
File::requireOnce($file);
// 生成檔案路徑的 MD5 雜湊
File::hash($path);
// 將內容寫入檔案
File::put($path, $contents, $lock = false);
// 寫入檔案，存在的話覆蓋寫入
File::replace($path, $content);
// 將內容新增在檔案原內容前面
File::prepend($path, $data);
// 將內容新增在檔案原內容後
File::append($path, $data);
// 修改路徑權限
File::chmod($path, $mode = null);
// 通過給定的路徑來刪除檔案
File::delete($paths);
// 將檔案移動到新目錄下
File::move($path, $target);
// 將檔案複製到新目錄下
File::copy($path, $target);
// 建立硬連接
File::link($target, $link);
// 從檔案路徑中提取檔案名稱，不包含後綴
File::name($path);
// 從檔案路徑中提取檔案名稱，包含後綴
File::basename($path);
// 獲取檔案路徑名稱
File::dirname($path);
// 從檔案的路徑地址提取檔案的擴展
File::extension($path);
// 獲取檔案類型
File::type($path);
// 獲取檔案 MIME 類型
File::mimeType($path);
// 獲取檔案大小
File::size($path);
// 獲取檔案的最後修改時間
File::lastModified($path);
// 判斷給定的路徑是否是檔案目錄
File::isDirectory($directory);
// 判斷給定的路徑是否是可讀取
File::isReadable($path);
// 判斷給定的路徑是否是可寫入的
File::isWritable($path);
// 判斷給定的路徑是否是檔案
File::isFile($file);
// 尋找能被匹配到的路徑名
File::glob($pattern, $flags = 0);
// 獲取一個目錄下的所有檔案, 以陣列類型返回
File::files($directory, $hidden = false);
// 獲取一個目錄下的所有檔案 (遞迴).
File::allFiles($directory, $hidden = false);
// 獲取一個目錄內的目錄
File::directories($directory);
// 建立一個目錄
File::makeDirectory($path, $mode = 0755, $recursive = false, $force = false);
// 移動目錄
File::moveDirectory($from, $to, $overwrite = false);
// 將資料夾從一個目錄複製到另一個目錄下
File::copyDirectory($directory, $destination, $options = null);
// 刪除目錄
File::deleteDirectory($directory, $preserve = false);
// 遞迴式刪除目錄
File::deleteDirectories($directory);
// 清空指定目錄的所有檔案和資料夾
File::cleanDirectory($directory);
```
