# 資料庫正規化

魔法學校選課資料
| 姓名 | 課程 | 必選修 | 及格 | 教師 |
| ----- | ----- | ----- | ----- | ----- |
| 妙麗 | 飛行學, 魔藥學, 符咒學 | 必選, 必選, 必選 | 及格, 及格, 及格 | 胡奇, 石內卜, 孚立維 |
| 湯姆 | 黑魔法防禦術, 古代神秘文字研究,  占卜學 | 必選, 選修, 選修 | 及格, 不及格, 不及格 | 鄧不利多, 戴華德, 勒梅 |

## 1NF：去除重複
一格內不能重複放多筆資料，例如欄位課程放了不只一筆資料，如果要過濾出有科目不及格的學生及其科目，很難下 SQL 語法，所以一格只能放一筆資料。

| 姓名 | 課程 | 必選修 | 及格 | 教授 |
| ----- | ----- | ----- | ----- | ----- |
| 妙麗 | 飛行學 | 必修 | 及格 | 胡奇 | 
| 妙麗 | 魔藥學 | 必修 | 及格 | 石內卜 |
| 妙麗 | 符咒學 | 必修 | 及格 | 孚立維 |
| 湯姆 | 黑魔法防禦術 | 必選 | 及格 | 鄧不利多 |
| 湯姆 | 古代神秘文字研究 | 選修 | 不及格 | 戴華德 |
| 湯姆 | 占卜學 | 選修 | 不及格 | 勒梅 |

這樣要過濾出有科目不及格的學生及其科目，可以用以下的 SQL

``` sql
SELECt 姓名, 課程, 必選修, 教授
FROM 魔法學校選課資料
WHERE 及格 = '不及格'
```

如果魔法學校有相同姓名的學生在同一年級，例如有 2 個哈利，則會出現重複的資料，所以加上絕不會重複的編號，名爲主鍵，就能用 SQL 語法找出要的是哪一列資料

| 編號 | 姓名 | 課程 | 必選修 | 及格 | 教授 |
| :-----: | ----- | ----- | ----- | ----- | ----- |
| 1 | 哈利 | 飛行學 | 必修 | 及格 | 胡奇 | 
| 2 | 哈利 | 魔藥學 | 必修 | 及格 | 石內卜 |
| 3 | 哈利 | 符咒學 | 必修 | 及格 | 孚立維 |
| 4 | 哈利 | 飛行學 | 必修 | 及格 | 胡奇 | 
| 5 | 哈利 | 魔藥學 | 必修 | 及格 | 石內卜 |
| 6 | 哈利 | 符咒學 | 必修 | 及格 | 孚立維 |

選擇 6 號資料

``` sql
SELECt *
FROM 魔法學校選課資料
WHERE 編號 = 6
```

## 2NF：去除部分相依

- 相依性：某個欄位的值是跟著另一欄位的值改變，例如必選修與否是跟著課程決定。
- 完全相依：非主鍵欄位相依而且只相依於主鍵，例如姓名要相依於編號
- 部分相依：必選修相依於課程，而不是主鍵編號

去除的方法是把部分相依欄位依照相依性另成一張表，例如課程、必選修

## 3NF：去除遞移相依

## B-C NF：主鍵不相依於非主鍵

## 4NF

## 5NF

## 6NF