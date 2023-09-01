``` php
// 寫入檔案
Storage::put('avatars/1', $fileContents);
// 指定磁碟
Storage::disk('local')->put('file.txt', 'Contents');
Storage::get('file.jpg');
Storage::exists('file.jpg');
Storage::download('file.jpg',  $name,  $headers);
Storage::url('file.jpg');
Storage::temporaryUrl('file.jpg', now()->addMinutes(5));
Storage::size('file.jpg');
Storage::lastModified('file.jpg');
// 自動為檔案名稱生成唯一的ID...  
Storage::putFile('photos',  new  File('/path/to/photo'));
// 手動指定檔案名稱...  
Storage::putFileAs('photos', new  File('/path/to/photo'), 'photo.jpg');
Storage::prepend('file.log', 'Prepended Text');
Storage::append('file.log', 'Appended Text');
Storage::copy('old/file.jpg', 'new/file.jpg');
Storage::move('old/file.jpg', 'new/file.jpg');
Storage::putFileAs('avatars', $request->file('avatar'), Auth::id());
// 「可見性」是對多個平台的檔案權限的抽象
Storage::getVisibility('file.jpg');
Storage::setVisibility('file.jpg',  'public')
Storage::delete('file.jpg');
Storage::delete(['file.jpg',  'file2.jpg']);
// 獲取目錄下的所有的檔案
Storage::files($directory);
Storage::allFiles($directory);
// 獲取目錄下的所有目錄
Storage::directories($directory);
Storage::allDirectories($directory);
// 建立目錄
Storage::makeDirectory($directory);
// 刪除目錄
Storage::deleteDirectory($directory);
```
