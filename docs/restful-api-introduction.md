# RESTful API 簡介

## 規劃

先思考名詞，再來是如何操作名詞

Posts 文章 (名詞)
- Create 新增
- Read 讀取
- Update 修改
- Delete 刪除
- List 列出全部清單

## 請求

RESTful API 使用 HTTP 動詞和網址配成一對，決定 API 路由規則，例如 GET /posts 回傳所有文章，但是 POST /posts 是新增一篇文章。哪個 HTTP 動詞搭配怎樣的網址有約定俗成的慣例如下：

| HTTP 動詞 | 網址                       | 說明                     | 冪等             |
| --------- | ---------------------------| ------------------------ | ---------------- |
| POST      | /posts                     | 新增文章                 | X                |
| GET       | /posts/create              | 顯示新增文章頁面         | O                |
| GET       | /posts/{post}              | 回傳指定的文章           | O                |
| PUT/PATCH | /posts/{post}              | 修改指定的文章           | 不一定           |
| GET       | /posts/{post}/edit         | 顯示修改文章頁面         | O                |
| DELETE    | /posts/{post}              | 刪除指定的文章           | O                |
| GET       | /posts                     | 回傳全部文章             | O                |

{post} 代表文章 id（例如 123）

一般來說保持 HTTP 動詞搭配網址的形式，才符合 RESTful

- 路由中的 URL 都是名詞
- 動詞用 HTTP 動詞表示

但是非 CRUD 操作就不一定會依照上面表格的慣例使用 HTTP 動詞表示操作動作，有三種做法：

1. 用 JSON 放在 HTTP request body，例如 `{ "lock": true }`
1. 放在網址中當做資源向後連接，例如 /posts/123/lock
1. 放在網址，但是使用傳統的格式，最常見的是搜尋，例如 /search?q=abc&location=home

網址 posts 被稱爲資源，資源可以向後連接，代表更進一步的意義，例如：

- GET /posts/123/comments 回傳 id 爲 123 的文章的所有留言
- PUT /posts/123/comments/456 修改 id 爲 123 的文章中，id 爲 456 的留言

### 冪等 idempotence

冪：音同「蜜」。冪等：無論執行多少次，結果都一樣

| HTTP 動詞       | 冪等   | 修改資料 | 說明                                                         |
| --------------- | :----: | :------: | ------------------------------------------------------------ |
| GET             | O      | X        | 查詢 meta 和 body 資料                                       |
| HEAD            | O      | X        | 查詢 meta 資料                                               |
| PUT             | O      | O        | 修改所有欄位                                                 |
| DELETE          | O      | O        | 刪除。不論刪除多少次，結果都是沒有資料                       |
| POST            | X      | O        | 新增。即使欄位資料一樣，每次新增的主鍵也不一樣，所以不是冪等 |
| PATCH           | X      | O        | 修改部分欄位                                                 |

PATCH 不用一定要冪等，當然有冪等也可以，舉例：

| 之前                     | 操作              | 之後                     | 冪等         |
| ------------------------ | ----------------- | ------------------------ | :----------: |
| `{name: "Tom", age: 32}` | PATCH age 改成 33 | `{name: "Tom", age: 33}` | O            |
| `{name: "Tom", age: 32}` | PATCH age 增加 1  | `{name: "Tom", age: 33}` | X            |

## 回應

回傳的結果通常是 JSON，所以 Content-Type 請指定爲 application/json，推薦回應的 JSON 格式如下，不管是一筆或多筆資料，都是一致的：

``` json
這是一筆資料
{
  "data": {
    "name": "Tom",
    "email": "tom@email.com"
  }
}
這是多筆資料，即使巢狀結構也能保持一致
{
  "data": [
    {
      "name": "Tom",
      "email": "tom@email.com"
    },
    {
      "name": "John",
      "email": "john@email.com"
    },
    {
      "name": "Alice",
      "email": "alice@email.com",
      "posts": {
        "data": [
          {
            "title": "title of post 1",
            "content": "content of post 1"
          },
          {
            "title": "title of post 2",
            "content": "content of post 2"
          }
        ]
      }
    }
  ],
  "pagination": {
    "total": 1000,
    "count": 10,
    "perPage": 10,
    "currentPage": 2,
    "totalPages": 100,
    "nextUrl": "/posts?page=2&per_page=10"
  }
}
(以上只列出 3 筆資料)
```
