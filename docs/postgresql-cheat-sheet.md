# PostgreSQL 懶人包

之前都在用 MySQL，好奇隔壁棚的 PostgreSQL 有什麼不一樣，所以想玩玩看。講 PostgreSQL 的書相比 MySQL 很少，幸運的是最近發現一本「SQL 語法查詢入門」，使用的是 PostgreSQL，所以以下是這本書的筆記。

``` sql
CREATE DATABASE 資料庫名稱;
USE 資料庫名稱;
```

## 新增資料表

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
| **整數** | **數值可用範圍** |
| smallint | -32,768 到 +32,767 |
