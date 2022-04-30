# PostgreSQL 懶人包

之前都在用 MySQL，好奇隔壁棚的 PostgreSQL 有什麼不一樣，所以想玩玩看。講 PostgreSQL 的書相比 MySQL 很少，幸運的是最近發現一本「SQL 語法查詢入門」，使用的是 PostgreSQL，所以以下是這本書的筆記。

資料庫、資料表和資料欄名稱如果要用引號，必須用雙引號，資料欄的值如果要用引號，必須用單引號。

## 新增和管理資料庫

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

### psql

``` bash
# 測試使用者能否登入
psql -U 使用者名稱 -d 資料庫名稱 -h 127.0.0.1
# 使用指定的資料庫和使用者執行 test.sql 儲存的 SQL，然後離開 psql
psql -U 使用者名稱 -d 資料庫名稱 -h 127.0.0.1 -f test.sql
# 顯示 psql 所有指令
\?
# 更改連線的資料庫、使用者
\c[onnect] 資料庫名稱 使用者名稱
# 離開 psql
\q
```

## 新增資料表和 View

``` sql
-- 最常見的方式
CREATE TABLE 資料表名稱 (
  id serial,
  name varchar(255) NOT NULL DEFAULT '',
  資料表名稱 資料形態 選項
);

-- 用 SELECT 查詢結果新增資料表。AS 不能省略
-- 注意：只有複製新增資料表時的資料
-- 主鍵、外鍵、檢查、NOT NULL、UNIQUE 和 serial 等約束條件不會複製到新的資料表
CREATE TABLE 資料表 AS
  SELECT 欄位1, 欄位2 FROM 資料表名稱;

-- 主鍵等約束條件需要新增資料表後再設定
ALTER TABLE 資料表 ADD COLUMN id serial, ADD PRIMARY KEY (id);

-- 複製來源資料表的欄位名稱、資料型別和 NOT NULL，建立新的資料表
-- LIKE 來源資料表要用括號包起來
-- 注意：新的資料表只有複製 NOT NULL，沒有主鍵、外鍵、檢查、UNIQUE 和 serial 等約束條件
CREATE TABLE 新的資料表 (LIKE 來源資料表);

-- 新增 VIEW
CREATE VIEW VIEW名稱 AS SELECT ...;
DROP VIEW VIEW名稱;

-- 修改資料表
ALTER TABLE 資料表名稱 ADD COLUMN 欄位名稱 資料形態;
ALTER TABLE 資料表名稱 DROP COLUMN 欄位名稱;
ALTER TABLE 資料表名稱 ALTER COLUMN 欄位名稱 SET DATA TYPE 資料形態;
ALTER TABLE 資料表 RENAME TO 新的資料表名稱;
-- 把 SELECT 查詢結果存到新增的資料表，不會原查詢的索引
CREATE TABLE 資料表名稱 AS SELECT 查詢;

-- 刪除資料表
DROP TABLE 資料表名稱;
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
| numeric(precision, scale) 或 decimal(precision, scale) | 例如 numeric(3, 1) 或 decimal(4, 2) (fixed-point) |
| real | 6 位數 precision (floating-point) |
| double precision | 15 位數 precision (floating-point) |
| 日期和時間 |  |
| date | September 21, 2018、9/21/2018 或 2018-09-21，從西元前 4,713 年到 5,874,897 年 |
| time (with timezone) | 15:37 +8 或 12:24:06 PST，從 0000:00:00 到 24:00:00 |
| timestamp (with timezone) 或 timestamptz | 2018-09-21 15:06:27 Asia/Taipei，從西元前 4,713 年到 294,276 年 |
| interval | 時間區隔，例如 1 hour, 2 day 或 3 weeks，範圍是正負 178,000,000 年 |

### 字元

3 種沒有明顯效能差異，不用爲了效能改用 char(n)

| 語法 | 說明 |
|---------------------------------------- | -------------------------------------- |
| upper(字串) | 轉成大寫 |
| lower(字串) | 轉成小寫 |
| initcap(字串) | 每個單字第一個字母轉大寫，其餘字母轉小寫(PostgreSQL 獨有) |
| char_length(字串) | 字元數，注意：一個字中文回傳 1 |
| length(字串) | 字元數，注意：一個字中文回傳 1 (PostgreSQL 獨有) |
| position(', ' in 'Tan, Bella') | 回傳 ', ' 在 'Tan, Bella' 中的第一次出現的起始位置 (4)，注意：不是從 0 開始 |


### 整數

serial 是 PostgreSQL 獨有的資料形態，通常用於主鍵

### 小數

* precision：小數點左右兩側最多一共有幾位數
* scale：小數點右邊的位數 (預設爲0，即設爲整數)，不足會自動補 0
* precision 和 scale 都省略，則自動依值決定直到上限爲止
* 上限是小數點前 131,072位數，小數點後 16,383 位數
* 超過設定的 precision 或 scale，則用下一位數決定 4 捨 5 入
* 注意：如果需要精確數學計算，要使用 numeric 或 decimal。real 和 double precision 是浮點數，數學計算是不準確的

### 時區

timestamp 輸入時通常要有時區，例如 '2018-12-31 01:00 EST'，時區可用縮寫 EST、和 UTC 的時差 +8，或時區資料庫 Asia/Taipei

`SELECT * FROM pg_catalog.pg_timezone_names ORDER BY utc_offset;` 查詢時區資料庫清單，欄位 is_dst 代表是否使用日光節約時間

時間戳記欄位例如 `'2019-12-01 18:37:12 EST'`

| 語法 | 說明 |
| -------------------------------- | --------------------------------------- |
| date_part('year', cast(時間戳記欄位 AS timestamptz)) return 2019 | 擷取型別 date, time 或 timestamptz 的資料值 |
| extract('year' from cast(時間戳記欄位 AS timestamptz)) return 2019 | 擷取型別 date, time 或 timestamptz 的資料值 (SQL 標準) |
| make_date(2018, 2, 22) | 回傳 date 型別資料 2018-02-22 |
| make_time(18, 4, 30.3) | 回傳 time 型別資料 18:04:30.3，沒有時區 |
| make_timestamptz(2018, 2, 22, 18, 4, 30.3, 'Asia/Tokyo') | 回傳 timestamp with time zone 型別資料 2018-02-22 17:04:30.3+8，系統時區設定爲 Asia/Taipei，所以是 17 點 |

## 約束條件

### 主鍵

``` sql
CREATE TABLE 資料表 (
  id serial,
  CONSTRAINT 主鍵名稱 PRIMARY KEY (id)
);
CREATE TABLE 資料表 (
  column_1 integer,
  column_2 integer,
  CONSTRAINT 主鍵名稱 PRIMARY KEY (column_1, column_2)
);
CREATE TABLE 資料表 ( id serial CONSTRAINT 主鍵名稱 PRIMARY KEY );
CREATE TABLE 資料表 ( id serial PRIMARY KEY );

ALTER TABLE 資料表 ADD CONSTRAINT 主鍵名稱 PRIMARY KEY (column_1, column_2);
ALTER TABLE 資料表 DROP CONSTRAINT 主鍵名稱;
```

### 外鍵

``` sql
CREATE TABLE 資料表 (
  foreign_key_name serial
    CONSTRAINT 外鍵名稱 FOREIGN KEY REFERENCES 資料表 (資料欄)
    ON DELETE CASCADE -- 自動刪除外鍵參考的主鍵資料
);
```

### 檢查

``` sql
CREATE TABLE 資料表 (
  user_role varchar(50),
  salary integer,
  -- CHECK () 括號內的語法和 WHERE 子句相同
  CONSTRAINT 檢查條件名稱 CHECK ( user_role IN ('admin', 'staff') ),
  CONSTRAINT 檢查條件名稱 CHECK ( salary > 0 )
);
```

### UNIQUE

PostgreSQL 認爲 NULL 和 NULL 是無法比較的，所以欄位是 UNIQUE 時，可以儲存多筆資料是 NULL

``` sql
CREATE TABLE 資料表 (
  email varchar(255),
  CONSTRAINT 名稱 UNIQUE (email)
);
```

### NOT NULL

``` sql
CREATE TABLE 資料表 (
  欄位名稱 varchar(20) NOT NULL
);
ALTER TABLE 資料表 ALTER COLUMN 欄位名稱 SET NOT NULL;
ALTER TABLE 資料表 ALTER COLUMN 欄位名稱 DROP NOT NULL;
```

## 新增修改刪除資料

``` sql
-- 指定新增的欄位，PostgreSQL 使用單引號包住字串值
INSERT INTO 資料表 (欄位1, 欄位2) VALUES (123, 'text'), (456, 'text');
-- 依照資料表欄位順序新增資料
INSERT INTO 資料表 VALUES (123, 'text');
-- 把 SELECT 查詢的結果新增到資料表
INSERT INTO 資料表 SELECT * FROM ...;
-- 用 SELECT 查詢結果新增到指定欄位
INSERT INTO 目標資料表 (欄位1, 欄位2)
  SELECT 欄位A, 欄位B FROM 欄位資料表;
  
UPDATE 資料表
SET 欄位名稱1 = 值1,
    欄位名稱2 = 值2
WHERE id = 1

DELETE FROM 資料表 WHERE id = 1;
-- 清空資料表
TRUNCATE TABLE 資料表;
```

## 交易

``` sql
START TRANSACTION;
COMMIT;
ROLLBACK;
```

## 探索資料

``` sql
-- 顯示所有欄位
SELECT * FROM 資料表;
-- 顯示欄位 t1, t2
SELECT t1, t2 FROM 資料表;
-- 只顯示欄位 t1 不重複的值
SELECT DISTINCT t1 FROM 資料表;
-- 每個選區的候選人，只顯示不重複的欄位組合
SELECT DISTINCT 選區, 候選人姓名 FROM 候選人名冊;
-- DESC 遞減，ASC 遞增（預設）
SELECT * FROM 資料表 ORDER BY t1 DESC, t2 ASC;
-- 只顯示 1 筆資料
SELECT * FROM 資料表 LMIT 1;
-- 從索引值 0 的資料開始，只顯示 5 筆（注意，第 1 筆資料的索引值是 0，不是 1）
SELECT * FROM 資料表 LMIT 0, 5;

-- 顯示 server 設定
-- lc_collate：locale for collation 會影響排序順序
SHOW ALL;
-- 顯示系統設定的時區
SHOW timezone;
```

字元表對映著數字，排序依據此數字排序。例如 Unicode 字元表 A 對應到 65，a 對應到 97，所以遞增時 A 排在 a 前面。

## 過濾條件

``` sql
-- a <= x <= b。a 一定要大於或等於 b，所以如果 BETWEEN 5 AND 3，會回傳空值
WHERE x BETWEEN a AND b
WHERE tags IN ('A', 'B', 'C')
-- LIKE 區分大小寫
-- ILIKE 不區分大小寫(PostgreSQL 獨有 ILIKE 語法)
-- % 一或多個字元，%abc 表示結尾是 abc，開頭是一或多個字元的字串
WHERE 欄位 LIKE '%abc'
-- _ 單獨一個字元， _abc 表示結尾是 abc，開頭是一個字元的字串
WHERE 欄位 LIKE '_abc'
```

## CASE WHEN

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
SELECT sum(欄位) FROM 資料表; -- 加總
SELECT avg(欄位) FROM 資料表; -- 平均
-- 4 捨 5 入到小數點第幾位，scale：小數點右邊的位數 (預設爲0，即取到整數)
SELECT round(欄位, scale) FROM 資料表;
-- 百分位數，取平均數。例如 1, 2, 3, 4, 5, 6 中會自動取平均 3.5
SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY 欄位) FROM 資料表;
-- 百分位數，取前 50% 的最後一個數。例如 1, 2, 3, 4, 5, 6 中會自動取前 50% 的最後一個數 3
SELECT percentile_disc(0.5) WITHIN GROUP (ORDER BY 欄位) FROM 資料表;
-- unnest() 把陣列多筆資料，array[]：建立一個陣列
SELECT unnest(array['25%', '50%', '75%']) AS "百分比";
-- percentile_test 只有一個資料欄 numbers，內含6筆資料 1, 2, 3, 4, 5, 6
-- percentile_cont() 和 percentile_disc() 參數可以給它陣列，它回傳的也是陣列，
-- 所以用 unnest() 轉成多筆資料
SELECT unnest(array['25%', '50%', '75%']) AS "百分比",
       unnest(
         percentile_cont(array[.25, .50, .75])
           WITHIN GROUP (ORDER BY numbers)
       ) AS "百分位數(連續)",
       unnest(
         percentile_disc(array[.25, .50, .75])
           WITHIN GROUP (ORDER BY numbers)
       ) AS "百分位數(離散)"
FROM percentile_test;

-- 依照欄位 price 排名
-- rank(): 排名，平手時留白，dense_rank(): 排名，平手時不留白(名次不會跳下一位)
SELECT name, price,
       rank() OVER (ORDER BY price DESC)
         AS "依照欄位 price 的總排名",
       dense_rank() OVER (ORDER BY price DESC)
         AS "依照欄位 price 的總排名(不留白)",
       rank() OVER (PARTION BY category ORDER BY price DESC)
         AS "依照欄位 category 分組後，欄位 price 在分組內的排名",
       dense_rank() OVER (PARTION BY category ORDER BY price DESC)
         AS "依照欄位 category 分組後，欄位 price 在分組內的排名(不留白)",
FROM products;
```

## ETL 工具 COPY

PostgreSQL 可以用 COPY 查詢，從 CSV 等文字檔匯入到資料庫，或匯出成文字檔。缺點是匯入或匯出的檔案必須和 PostgreSQL 在同一台機器上，但是用 `\copy` 就沒有這樣的限制

``` sql
-- 匯入 CSV 檔到資料庫，資料表必須自行新增
-- (欄位1, 欄位2)：可以指定匯入的欄位名稱，省略的話則依照資料欄順序
COPY 資料表 (欄位1, 欄位2)
-- csv 檔位址，必須用絕對位址
FROM 'C:\your_dir\your_file.csv'
WITH (FORMAT CSV, -- 指定格式是 CSV，另一常見格式是 TEXT
	  HEADER, -- 第一列是標題，不匯入
	  DELIMITER ',', -- 用逗號分隔欄位，CSV 檔預設
	  QUOTE '文字限定符' -- 如有欄位值包含逗號，需用文字限定符包起來，CSV 檔預設是雙引號
	  );

-- 匯出資料到 CSV 檔，選項同匯入
COPY 資料表 (欄位1, 欄位2)
TO 'C:\your_dir\your_file.csv'
WITH (FORMAT CSV,
	  HEADER,
	  DELIMITER ',',
	  QUOTE '文字限定符'
	  );

-- 可以把 SELECT 查詢結果匯出到 CSV 檔
COPY (SELECT * FROM 資料表)
TO 'C:\your_dir\your_file.csv'
WITH ...

-- 基本上和上面的 COPY 查詢一樣，只是 COPY 改成 \copy
\copy 資料表 (欄位1, 欄位2) FROM 'C:\your_dir\your_file.csv' WITH (...);
\copy 資料表 (欄位1, 欄位2) TO 'C:\your_dir\your_file.csv' WITH (...);
\copy (SELECT * FROM 資料表)
TO 'C:\your_dir\your_file.csv'
WITH ...
```

## 備份與還原

``` bash
# 備份到 SQL 文字檔
pg_dump -U 使用者名稱 -d 資料庫名稱 -h 127.0.0.1 --blobs > 備份.sql
# 用 SQL 文字檔還原資料庫
psql -U 使用者名稱 -d 資料庫名稱 -h 127.0.0.1 -f 備份.sql
# 備份到壓縮檔
pg_dump -U 使用者名稱 -d 資料庫名稱 -h 127.0.0.1 --blobs -Ft > 備份.tar.gz
# 用壓縮檔還原資料庫
pg_restore -U 使用者名稱 -d 資料庫名稱 -h 127.0.0.1 備份.tar.gz
```

## 判斷查詢效能

``` sql
-- 顯示 PostreSQL 查詢計畫：如何掃描資料表、是否用到索引等
EXPLAIN SELECT ...;
-- 除了顯示 PostreSQL 查詢計畫，還會執行查詢，顯示實際執行時間
EXPLAIN ANALYZE SELECT ...;
```

## 索引

``` sql
CREATE INDEX 索引名稱 ON 資料表名稱 (欄位名稱);
-- 排除索引 NULL，增加搜尋速度 (PostgreSQL 才能在新增索引時用 WHERE 子句)
CREATE INDEX 索引名稱 ON 資料表名稱 (欄位名稱) WHERE 欄位名稱 IS NOT NULL;
```
