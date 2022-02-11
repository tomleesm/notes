# 使用 API Blueprint 寫文件

[API Blueprint](https://apiblueprint.org) 是一種 markdown 語法的擴充，用來寫 RESTful API 文件

## 工具

API Blueprint 官網有列出許多工具，我推薦 Aglio 和 apiary。Aglio 是用 javascript 寫的工具，可以把 API Blueprint markdown 檔案轉成 html，安裝指令如下：

```
# 使用 yarn 全域安裝 Aglio
sudo yarn global add aglio
# 或使用 npm
sudo npm install -g aglio
```

接著編輯好 markdown 檔案，例如 example.md，然後執行 `aglio -i example.md -o example.html` ，轉換成 example.html。可以執行 `aglio` 指令看看參數有哪些。可以使用 `aglio -i example.md -s -h 0.0.0.0 -p 8000` 產生即時預覽。但是實測的結果發現更新不是很可靠，所以我都是在 [apiary](https://apiary.io) 編輯好再使用 aglio

## 語法

沒有學過 API Blueprint 的人，推薦官方文件的[範例](https://github.com/apiaryio/api-blueprint/tree/master/examples)，雖然是英文的，但是每一個篇幅都不長，循序漸進很好懂。

之前[RESTful API：請求與回應](/posts/restful-api-request-response/)的 API 文件可以用 API Blueprint 寫成如下格式：

``` markdown
FORMAT: 1A

# 文章 API

文章的 API 文件

# Group 文章

文章群組

## 新增文章 [POST /posts]

+ Request (application/json)

    + Attributes (一篇文章內容)

+ Response 201

    + Headers

            Location: /posts/1

## 回傳全部文章 [GET /posts?page={page}&per_page={per_page}]

+ Parameters

    + page: 1 (number, optional) - 頁碼
        + Default: 1
    + per_page: 20 (number, optional) - 每一頁有幾篇文章
        + Default: 10

+ Response 200 (application/json)

        {
          "data": [
            {
              "title": "文章標題 1",
              "content": "文章內容 1"
            },
            {
              "title": "文章標題 2",
              "content": "文章內容 2"
            },
            {
              "title": "文章標題 3",
              "content": "文章內容 3"
            },
            // 以下省略
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

## 一篇文章 [/posts/{post}]

+ Parameters

    + post: 1 (number, required) - 文章的 id

### 回傳指定的文章 [GET]

+ Response 200 (application/json)
    + Attributes (一篇文章)

### 修改指定的文章所有欄位 [PUT]

+ Request (application/json)
    + Attributes (一篇文章)

+ Response 204

### 修改指定的文章部分欄位 [PATCH]

+ Request (application/json)

    + Attributes (一篇文章內容)

+ Response 204

### 刪除指定的文章 [DELETE]

+ Response 204

# Data Structures

## 一篇文章內容 (object)

+ title: 文章標題 (string) - 文章標題
+ content: 文章內容 (string) - 文章內容

## 一篇文章 (一篇文章內容)

+ id: 1 (number) - 文章的 id
+ createdAt: `2022-01-20T18:39:27.774000+08:00` (string)

    文章新增時間，格式爲 %Y-%M-%DT%h:%m:%s.%u%Z:%z

+ updatedAt: `2022-01-20T18:39:27.774000+08:00` (string)

    文章修改時間，格式爲 %Y-%M-%DT%h:%m:%s.%u%Z:%z

```

用 aglio 轉換後產生 [api-blueprint-example.html](/api-blueprint-example.html)

請注意，使用 aglio 轉換時，如果在資源或動作中定義 Attributes，最好使用英文名稱，因爲中文名稱無法在 `+Attributes ()` 中使用，會沒有 body，但是在 Data Structures 定義就沒有這個問題。此外， attribute 在 `+ Body` 區塊中無法使用。
