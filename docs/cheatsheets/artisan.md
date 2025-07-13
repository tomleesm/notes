---
hide:
  - toc
---

```shell
// 針對命令顯示幫助資訊
php artisan --help OR -h
// 抑制輸出資訊
php artisan --quiet OR -q
// 列印 Laravel 的版本資訊
php artisan --version OR -V
// 不詢問任何互動性的問題
php artisan --no-interaction OR -n
// 強制輸出 ANSI 格式
php artisan --ansi
// 禁止輸出 ANSI 格式
php artisan --no-ansi
// 顯示當前命令列運行的環境
php artisan --env
// -v|vv|vvv 通過增加 v 的個數來控制命令列輸出內容的詳盡情況: 1 個代表正常輸出, 2 個代表輸出更多消息, 3 個代表偵錯
php artisan --verbose
// 移除編譯最佳化過的檔案 (storage/frameworks/compiled.php)
php artisan clear-compiled
// 顯示當前框架運行的環境
php artisan env
// 顯示某個命令的幫助資訊
php artisan help
// 顯示所有可用的命令
php artisan list
// 進入應用互動模式
php artisan tinker
// 配合 dump() 函數偵錯資料
php artisan dump-server
// 進入維護模式
php artisan down
// 退出維護模式
php artisan up
// 最佳化框架性能
 // --force    強制編譯已寫入檔案 (storage/frameworks/compiled.php)
 // --psr      不對 Composer 的 dump-autoload 進行最佳化
php artisan optimize [--force] [--psr]
// 更改前端預設
// type_name (可以是 none, bootstrap, vue, react)
php artisan preset [options] [--] type_name
// 啟動內建伺服器
php artisan serve
// 更改默認連接埠
php artisan serve --port 8080
// 使其在本地伺服器外也可正常工作
php artisan serve --host 0.0.0.0
// 更改應用命名空間
php artisan app:name namespace
// 清除過期的密碼重設令牌
php artisan auth:clear-resets

// 清空應用快取
php artisan cache:clear
// 移除 key_name 對應的快取
php artisan cache:forget key_name [<store>]
// 建立快取資料庫表 migration
php artisan cache:table

// 合併所有的組態資訊為一個，提高載入速度
php artisan config:cache
// 移除組態快取檔案
php artisan config:clear

// 程序內部呼叫 Artisan 命令
$exitCode = Artisan::call('config:cache');
// 運行所有的 seed 假資料生成類
 // --class      可以指定運行的類，默認是: "DatabaseSeeder"
 // --database   可以指定資料庫
 // --force      當處於生產環境時強制執行操作
php artisan db:seed [--class[="..."]] [--database[="..."]] [--force]

// 基於註冊的資訊，生成遺漏的 events 和 handlers
php artisan event:generate
// 羅列所有事件和監聽器
php artisan event:list
// 快取事件和監聽器
php artisan event:cache
// 清除事件和監聽器快取
php artisan event:clear

// 生成新的處理器類
 // --command      需要處理器處理的命令類名字
php artisan handler:command [--command="..."] name
// 建立一個新的時間處理器類
 // --event        需要處理器處理的事件類名字
 // --queued       需要處理器使用佇列話處理的事件類名字
php artisan handler:event [--event="..."] [--queued] name

// 生成應用的 key（會覆蓋）
php artisan key:generate

// 發佈本地化翻譯檔案到 resources 檔案下
// locales: 逗號分隔，如 zh_CN,tk,th [默認是: "all"]
php artisan lang:publish [options] [--] [<locales>]

// 建立使用者認證腳手架
php artisan make:auth
// 建立 Channel 類
php artisan make:channel name
// 在默認情況下, 這將建立未加入佇列的自處理命令
 // 通過 --handler 標識來生成一個處理器, 用 --queued 來使其入佇列.
php artisan make:command [--handler] [--queued] name
// 建立一個新的 Artisan 命令
 //  --command     命令被呼叫的名稱。 (默認為: "command:name")
php artisan make:console [--command[="..."]] name
// 建立一個新的資源控製器
 // --plain      生成一個空白的控製器類
php artisan make:controller [--plain] name
php artisan make:controller App\\Admin\\Http\\Controllers\\DashboardController
// 建立一個新的事件類
php artisan make:event name
// 建立異常類
php artisan make:exception name
// 建立模型工廠類
php artisan make:factory name
// 建立一個佇列任務檔案
php artisan make:job 
// 建立一個監聽者類
php artisan make:listener name
// 建立一個新的郵件類
php artisan make:mail name
// 建立一個新的中介軟體類
php artisan make:middleware name
// 建立一個新的遷移檔案
 // --create     將被建立的資料表.
 // --table      將被遷移的資料表.
php artisan make:migration [--create[="..."]] [--table[="..."]] name
// 建立一個新的 Eloquent 模型類
php artisan make:model User
php artisan make:model Models/User
// 新建一個消息通知類
php artisan make:notification TopicRepliedNotification
// 新建一個模型觀察者類
php artisan make:observer UserObserver
// 建立授權策略
php artisan make:policy PostPolicy
// 建立一個新的服務提供者類
php artisan make:provider name
// 建立一個新的表單請求類
php artisan make:request name
// 建立一個 API 資源類
php artisan make:resource name
// 新建驗證規則類
php artisan make:rule name
// 建立模型腳手架
// <name> 模型名稱，如 Post
// -s, --schema=SCHEMA 表結構如：--schema="title:string"
// -a, --validator[=VALIDATOR] 表單驗證，如：--validator="title:required"
// -l, --localization[=LOCALIZATION] 設定本地化資訊，如：--localization="key:value"
// -b, --lang[=LANG] 設定本地化語言 --lang="en"
// -f, --form[=FORM] 使用 Illumintate/Html Form 來生成表單選項，默認為 false
// -p, --prefix[=PREFIX] 表結構前綴，默認 false
php artisan make:scaffold  [options] [--] <name>
// 生成資料填充類
php artisan make:seeder
// 生成測試類
php artisan make:test

// 資料庫遷移
 // --database   指定資料庫連接（下同）
 // --force      當處於生產環境時強制執行，不詢問（下同）
 // --path       指定單獨遷移檔案地址
 // --pretend    把將要運行的 SQL 語句列印出來（下同）
 // --seed       Seed 任務是否需要被重新運行（下同）
php artisan migrate [--database[="..."]] [--force] [--path[="..."]] [--pretend] [--seed]
// 建立遷移資料庫表
php artisan migrate:install [--database[="..."]]
// Drop 所有資料表並重新運行 Migration
php artisan migrate:fresh
// 重設並重新運行所有的 migrations
 // --seeder     指定主 Seeder 的類名
php artisan migrate:refresh [--database[="..."]] [--force] [--seed] [--seeder[="..."]]
// 回滾所有的資料庫遷移
php artisan migrate:reset [--database[="..."]] [--force] [--pretend]
// 回滾最最近一次運行的遷移任務
php artisan migrate:rollback [--database[="..."]] [--force] [--pretend]
// migrations 資料庫表資訊
php artisan migrate:status

// 為資料庫消息通知建立一個表遷移類
php artisan notifications:table
// 清除快取的 bootstrap 檔案
php artisan optimize:clear
// 擴展包自動發現
php artisan package:discover

// 為佇列資料庫表建立一個新的遷移
php artisan queue:table
// 監聽指定的佇列
 // --queue      被監聽的佇列
 // --delay      給執行失敗的任務設定延時時間 (默認為零: 0)
 // --memory     記憶體限制大小，單位為 MB (默認為: 128)
 // --timeout    指定任務運行超時秒數 (默認為: 60)
 // --sleep      等待檢查佇列任務的秒數 (默認為: 3)
 // --tries      任務記錄失敗重試次數 (默認為: 0)
php artisan queue:listen [--queue[="..."]] [--delay[="..."]] [--memory[="..."]] [--timeout[="..."]] [--sleep[="..."]] [--tries[="..."]] [connection]
// 查看所有執行失敗的佇列任務
php artisan queue:failed
// 為執行失敗的資料表任務建立一個遷移
php artisan queue:failed-table
// 清除所有執行失敗的佇列任務
php artisan queue:flush
// 刪除一個執行失敗的佇列任務
php artisan queue:forget
// 在當前的佇列任務執行完畢後, 重啟佇列的守護處理程序
php artisan queue:restart
// 對指定 id 的執行失敗的佇列任務進行重試(id: 失敗佇列任務的 ID)
php artisan queue:retry id
// 指定訂閱 Iron.io 佇列的連結
 // queue: Iron.io 的佇列名稱.
 // url: 將被訂閱的 URL.
 // --type       指定佇列的推送類型.
php artisan queue:subscribe [--type[="..."]] queue url
// 處理下一個佇列任務
 // --queue      被監聽的佇列
 // --daemon     在後台模式運行
 // --delay      給執行失敗的任務設定延時時間 (默認為零: 0)
 // --force      強制在「維護模式下」運行
 // --memory     記憶體限制大小，單位為 MB (默認為: 128)
 // --sleep      當沒有任務處於有效狀態時, 設定其進入休眠的秒數 (默認為: 3)
 // --tries      任務記錄失敗重試次數 (默認為: 0)
php artisan queue:work [--queue[="..."]] [--daemon] [--delay[="..."]] [--force] [--memory[="..."]] [--sleep[="..."]] [--tries[="..."]] [connection]

// 生成路由快取檔案來提升路由效率
php artisan route:cache
// 移除路由快取檔案
php artisan route:clear
// 顯示已註冊過的路由
php artisan route:list

// 運行計畫命令
php artisan schedule:run

// 為 session 資料表生成遷移檔案
php artisan session:table
// 建立 "public/storage" 到 "storage/app/public" 的軟連結
php artisan storage:link

// 從 vendor 的擴展包中發佈任何可發佈的資源
 // --force        重寫所有已存在的檔案
 // --provider     指定你想要發佈資原始檔的服務提供者
 // --tag          指定你想要發佈標記資源.
php artisan vendor:publish [--force] [--provider[="..."]] [--tag[="..."]]
php artisan tail [--path[="..."]] [--lines[="..."]] [connection]

// 快取檢視檔案以提高效率
php artisan view:cache
// 清除檢視檔案快取
php artisan view:clear
```
