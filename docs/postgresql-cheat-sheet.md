# PostgreSQL 懶人包

之前都在用 MySQL，好奇隔壁棚的 PostgreSQL 有什麼不一樣，所以想玩玩看。講 PostgreSQL 的書相比 MySQL 很少，幸運的是最近發現一本「SQL 語法查詢入門」，使用的是 PostgreSQL，所以以下是這本書的筆記。

資料庫、資料表和資料欄名稱如果要用引號，必須用雙引號，資料欄的值如果要用引號，必須用單引號。

在 Linux 執行以下指令，用 PostgreSQL 預設的管理員帳號執行 psql 管理程式

``` bash
sudo -u postgres psql
```

新增使用者和資料庫，並設定權限

``` sql
-- 新增使用者，同時設定密碼
CREATE USER 使用者名稱 WITH PASSWORD '密碼';
-- 新增使用者和擁有者
CREATE DATABASE 資料庫名稱 OWNER 使用者名稱;
-- 設定使用者對資料庫有完整權限
GRANT ALL PRIVILEGES ON DATABASE 資料庫名稱 to 使用者名稱;
```

# 新增資料表

``` sql
CREATE TABLE 資料表名稱 (
  id serial,
  name varchar(255) NOT NULL DEFAULT '',
  資料表名稱 資料形態 選項
);
```

### 資料形態

| 語法 | 說明 |
| -------------- | -------------------- |
| **字元** |  |
| char(n) | 長度固定爲 n 的字元，少於 n 則自動以空格填滿，別名 character(n) |
| varchar(n) | 最大長度爲 n 的字元 |
| text | 長度不固定，適合大量的字元，PostgreSQL 最大爲 1GB|
| **整數** | |
| smallint | -32,768 到 +32,767 |
| integer | -2,147,483,648 到 +2,147,483,647 |
| bigint | -9,223,372,036,854,775,808 到 +9,223,372,036,854,775,807|
| smallserial | 1 到 32,767 |
| serial | 1 到 2,147,483,647 |
| bigserial | 1 到 9,223,372,036,854,775,807 |
| **小數** |  |
| numeric(precision, scale) 或 decimal(precision, scale) | 例如 numeric(3, 1) 或 decimal(4, 2) |
| real | 6 位數 precision |
| double precision | 15 位數 precision |
| 日期和時間 |  |
| timestamp (with timezone) 或 timestamptz | 日期與時間，西元前 4,713 年到 294,276 年 |
| date | 只記錄日期，西元前 4,713 年到 5,874,897 年 |
| time with timezone| 只記錄時間，0000:00:00 到 24:00:00 |
| interval | 時間區隔，例如 1 day 或 2 weeks，範圍是正負 178,000,000 年 |

### 字元
3 種沒有明顯效能差異，不用爲了效能改用 char(n)

### 整數

serial 是 PostgreSQL 獨有的資料形態，通常用於主鍵

### 小數
* precision：小數點左右兩側最多一共有幾位數
* scale：小數點右邊的位數 (預設爲0，即設爲整數)，不足會自動補 0
* precision 和 scale 都省略，則自動依值決定直到上限爲止
* 上限是小數點前 131,072位數，小數點後 16,383 位數
* 超過設定的 precision 或 scale，則用下一位數決定 4 捨 5 入
* 注意：如果需要精確數學計算，要使用 numeric 或 decimal。real 和 double precision 是浮點數，相加減是不準確的

### 時區

timestamp 輸入時通常要有時區，例如 '2018-12-31 01:00 EST'，時區可用縮寫 EST、和 UTC 的時差 +8，或時區資料庫 Asia/Taipei

# 新增資料

``` sql
-- 指定新增的欄位，PostgreSQL 使用單引號包住字串值
INSERT INTO 資料表 (欄位1, 欄位2) VALUES (123, 'text'), (456, 'text');
-- 依照資料表欄位順序新增資料
INSERT INTO 資料表 VALUES (123, 'text');
```

# 探索資料

``` sql
-- 顯示所有欄位
SELECT * FROM table_name;
-- 顯示欄位 t1, t2
SELECT t1, t2 FROM table_name;
-- 只顯示欄位 t1 不重複的值
SELECT DISTINCT t1 FROM table_name;
-- 每個選區的候選人，只顯示不重複的欄位組合
SELECT DISTINCT 選區, 候選人姓名 FROM 候選人名冊;
-- DESC 遞減，ASC 遞增（預設）
SELECT * FROM table_name ORDER BY t1 DESC, t2 ASC
-- 顯示 server 設定
-- lc_collate：locale for collation 會影響排序順序
SHOW ALL;
```

字元表對映著數字，排序依據此數字排序。例如 Unicode 字元表 A 對應到 65，a 對應到 97，所以遞增時 A 排在 a 前面。

## 過濾條件

``` sql
-- a <= x <= b
WHERE x BETWEEN a AND b
-- LIKE 區分大小寫
-- ILIKE 不區分大小寫(PostgreSQL 獨有 ILIKE 語法)
-- % 一或多個字元，%abc 表示結尾是 abc，開頭是一或多個字元的字串
WHERE 欄位 LIKE '%abc'
-- _ 單獨一個字元， _abc 表示結尾是 abc，開頭是一個字元的字串
WHERE 欄位 LIKE '_abc'
```

## 數學計算

``` sql
SELECT 2 + 2; -- 4 加
SELECT 9 - 1; -- 8 減
SELECT 3 * 4; -- 12 乘
SELECT 11 / 6; -- 1 除
SELECT 11 % 6; -- 5 取餘數
SELECT 11.0 / 6; -- 1.83333 小數除
-- 把整數 11 轉成 numeric 再除以 6
-- 如果 11 來自整數欄位，無法手動改成 11.0，就要用 CAST() 轉型
SELECT CASE(11 AS numeric(3, 1)) / 6;
SELECT 3 ^ 4; -- 次方，3 的 4 次方等於 81
SELECT |/ 10; -- 平方根
SELECT sqrt(10); -- 平方根
SELECT ||/ 10; -- 立方根
SELECT 4!; -- 階乘，4! = 4 x 3 x 2 x 1 = 24
-- 優先順序：指數和根 --> 乘除餘數 --> 加減

-- 彙總函式
SELECT sum(欄位) FROM table_name; -- 加總
SELECT avg(欄位) FROM table_name; -- 平均
-- 4 捨 5 入到小數點第幾位，scale：小數點右邊的位數 (預設爲0，即取到整數)
SELECT round(欄位, scale) FROM table_name;
-- 百分位數，取平均數。例如 1, 2, 3, 4, 5, 6 中會自動取平均 3.5
SELECT percentile_cont(0.5) FROM table_name;
-- 百分位數，取前 50% 的最後一個數。例如 1, 2, 3, 4, 5, 6 中會自動取前 50% 的最後一個數 3
SELECT percentile_disc(0.5) FROM table_name; --
```

## ETL 工具 COPY

PostgreSQL 可以用 COPY 指令，從 CSV 等文字檔匯入到資料庫，或匯出成文字檔

``` sql
-- 匯入 CSV 檔到資料庫，資料表必須自行新增
-- (欄位1, 欄位2)：可以指定匯入的欄位名稱，省略的話則依照資料欄順序
COPY table_name (欄位1, 欄位2)
-- csv 檔位址，必須用絕對位址
FROM 'C:\your_dir\your_file.csv'
WITH (FORMAT CSV, -- 指定格式是 CSV，另一常見格式是 TEXT
	  HEADER, -- 第一列是標題，不匯入
	  DELIMITER ',', -- 用逗號分隔欄位，CSV 檔預設
	  QUOTE '文字限定符' -- 如有欄位值包含逗號，需用文字限定符包起來，CSV 檔預設是雙引號
	  );

-- 匯出資料到 CSV 檔，選項同匯入
COPY table_name (欄位1, 欄位2)
TO 'C:\your_dir\your_file.csv'
WITH (FORMAT CSV,
	  HEADER,
	  DELIMITER ',',
	  QUOTE '文字限定符'
	  );

-- 可以把 SELECT 查詢結果匯出到 CSV 檔
COPY (SELECT * FROM table_name)
TO 'C:\your_dir\your_file.csv'
WITH ...
```