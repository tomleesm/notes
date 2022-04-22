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
| **字元** | **以下 3 種沒有明顯效能差異** |
| char(n) | 長度固定爲 n 的字元，少於 n 則自動以空格填滿，別名 character(n) |
| varchar(n) | 最大長度爲 n 的字元 |
| text | 長度不固定，適合大量的字元，PostgreSQL 最大爲 1GB|
| **整數** | |
| smallint | -32,768 到 +32,767 |
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

```