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

以下筆記參考自官方文件的[範例](https://github.com/apiaryio/api-blueprint/tree/master/examples)

### 開始

``` markdown
FORMAT: 1A

# The Simplest API
This is one of the simplest APIs written in the **API Blueprint**. One plain
resource combined with a method and that's it! We will explain what is going on
in the next installment -
[Resource and Actions](02.%20Resource%20and%20Actions.md).
```
開頭的 `FORMAT: 1A` 表示文件格式是 markdown，不寫也可以

接著一定要有一個 h1 標題（`# The Simplest API`）作爲文件主標題，一般是說明這是哪一個 API 文件，標題下方可以加上摘要描述，寫法和一般的markdown 一樣。

``` markdown
# GET /message
+ Response 200 (text/plain)

        Hello World!
```

這是一個簡單的例子，說明 HTTP 請求與回應。`#` 之後的是 HTTP method `GET` 和 RESTful 資源 `/message`，回應則寫成 `+ Response`，使用了 markdown 的無序清單語法 `+`（清單可用 `+`、`-` 或 `*` 開頭），Response 後面接著是 HTTP 狀態碼 200 和 Content-Type text/plain（要寫在小括號內），最後回應的 body 則用 markdown 的 code 區塊語法表示，所以 Hello World 前面要有 8 個空格或是 2 個 Tab，或是用成對的三個 ` 包起來，不過一般都是用空格或 Tab，之後會看到，那是因爲 body 會和 Headers 等其它設定構成巢狀結構。

### 資源與動作

上一個例子中，定義了一個路由 `GET /message` 放在 h1 標題裡，如果有另一個路由 `PUT /message`，可以放在另一個 h1 標題裡，但是 `/message` 重複了，所以更好的方式是定義一個資源 `/message`，裏面包含 2 個動作 `GET` 和 `PUT`。如果日後需要修改資源，只要修改一個地方就好，減少產生 bug 的機會。

``` markdown
# /message
在這裡定義了一個資源 `/message`

## GET
裡面定義了動作 `GET`，所以等同於路由 `GET /message`

+ Response 200 (text/plain)

        Hello World!

## PUT
這裡是另一個動作 PUT，等同於路由 `PUT /message`，用來修改資料。

+ Request (text/plain)

        All your base are belong to us.

+ Response 204
```

動作 `PUT` 中定義了 HTTP 請求，和回應一樣使用 markdown 的無序清單語法 `+`，後面接著 `Request`，括號內是請求的標題欄位 Accept text/plain，表示可接受的回應 body 類型是純文字。請求的 body 和回應一樣使用 markdown 的 code 區塊語法，所以 All your base are belong to us. 的前面要有 8 個空格或 2 個 Tab，最後是回應 `+ Response 204`，狀態碼 204 表示 No Content，表示修改完成，回應的 body 沒有內容。

### 命名資源與動作

``` markdown
# 訊息 [/message]
資源 `/message` 取名爲訊息

## 接收訊息 [GET]
`GET /message` 的名稱是接收訊息。

## 更新訊息 [PUT]
`PUT /message` 的名稱是更新訊息。
```
可以爲資源取一個名稱，方便識別，則名稱在`#` 之後，資源和動作改成放在中括號裡。名稱可以用中文。

### 群組

可以把多個資源和動作包含在一個群組裡。

``` markdown
# Group 訊息
定義一個訊息群組

## 訊息 [/message]

### 接收訊息 [GET]

### 更新訊息 [PUT]

# Group Users
```
關鍵字 `Group` 或 `group` 後面接著群組名稱（訊息），則之後的資源和動作都包含在這個群組之內，直到出現另一個群組。資源和動作的 `#` 可以維持原來的 `# 訊息 [/message]` 和 `## 接收訊息 [GET]`，改成如上這樣是爲了閱讀 markdown 時了解階層關係，是比較好的做法。

### 回應

定義標頭和回應的方式如下：

``` markdown
# GET /message

+ Response 200 (text/plain)
這個是純文字回應

    + Headers

            X-My-Message-Header: 42

    + Body

            Hello World!

+ Response 200 (application/json)
這個是 JSON 回應

    + Headers

            X-My-Message-Header: 42

    + Body

            { "message": "Hello World!" }

```

一個路由有兩個回應，指定 body 是純文字或 JSON。回應的格式是 `+ Response 狀態碼 (Content-Type)`，標頭 Headers 和內容 Body 是回應的內含資訊，所以用巢狀無序清單，必須向內縮至少一個空格，慣例是 4 個空格或 1 個 Tab；`X-My-Message-Header: 42` 和 `Hello World!` 都使用 code 區塊，必須向內縮止至少 8 個空格或 2 個 Tab，再加上 Header 或 Body 的縮排，所以是 12 個空格或 3 個 Tab。

### 請求

定義請求的方式如下：

``` markdown
## 訊息 [/message]

### 接收訊息 [GET]

+ Request 純文字
名稱是純文字的請求

    + Headers

            Accept: text/plain

+ Response 200 (text/plain)

    + Headers

            X-My-Message-Header: 42

    + Body

            Hello World!

+ Request JSON 訊息
名稱是 JSON 訊息的請求

    + Headers

            Accept: application/json

+ Response 200 (application/json)

    + Headers

            X-My-Message-Header: 42

    + Body

            { "message": "Hello World!" }

### 更新訊息 [PUT]
2 種不同的更新請求：純文字和 JSON

+ Request 更新文字訊息 (text/plain)

        All your base are belong to us.

+ Request 更新 JSON 訊息 (application/json)

        { "message": "All your base are belong to us." }

+ Response 204
```
使用無序清單，例如 `+` 後面接著關鍵字 Request，然後標頭欄位 Accept 可以放在小括號內 (text/plain) 或是放在 Headers 內。
可以爲請求取名，名稱放在 `+ Request` 和小括號之間，可以用中文，但是回應不能取名。

所以請求和回應很類似，都用 `+` 開頭，都有 `+ Headers` 和 `+ Body` 定義標頭和內容，都用小括號定義 Accept 或 Content-Type。

不同的是請求可以取名，回應不行；回應要有 HTTP 狀態碼。

### 參數 Parameters

資源通常有 id 或關鍵字包含其中，例如 `/users/123` 或 `/search?q=關鍵字`，這時使用參數來定義。

``` markdown
## 訊息 [/message/{id}]

+ Parameters

    + id: 1 (number) - 訊息的識別 ID

## 所有訊息 [/messages{?limit}]

### Retrieve all Messages [GET]

+ Parameters

    + limit (number, optional) - 回傳訊息的最大數量
        + Default: `20`
```
資源 `/message/{id}` 定義了一個名稱爲 id 的[URI Template variable](http://tools.ietf.org/html/rfc6570)，使用大括號包起來，然後在底下的 `+ Parameters` 定義 id 。

同樣使用 markdown 的無序清單表示每一個項目，所以用 `+` 開頭，向內縮4 個空格或 1 個 Tab，然後格式是
```
+ Parameters
    + 名稱: 範例值 (型別, optional 或 required) - 描述(可用 markdown 語法)

        額外的描述(可用 markdown 語法)

        + Default: 預設值
```

- 名稱：參數的名稱，這是唯一必須要有的，其它都是可選的。
- 範例值在 aglio 會放在 URI 中，顯示成例如 GET /messages/1
- 型別：number, string, 或 boolean。預設是 string
- optional 或 required：表明這個參數是可選的或是必須要有的，預設是 required
- 描述和額外的描述：描述參數的用途等資訊，可用 markdown 語法。額外的描述要向內縮 4 個空格或 1 個 Tab
- `+ Default`：預設值，沒有指定參數值時所用的值，只在參數設定爲 optional 時可用，可用 ` 包起值，確保可以正確轉換成 HTML，尤其是字串。要向內縮 4 個空格或 1 個 Tab

### 屬性 Attributes

請求或回應的 body 常常是重複的，可以用屬性 Attributes 來定義 body，然後在請求或回應中引用它。

``` markdown
## 折價卷 [/coupons/{id}]
折價卷資料。屬性定義在資源中如下：

+ Parameters
    + id (string)

        折價卷 ID

+ Attributes (object)
    + id: 250FF (string, required)
    + created: 1415203908 (number) - 新增時間戳
    + percent_off: 25 (number)

        打折的百分比，從 1 到 100

    + redeem_by (number) - 兌換有效期限，此爲時間戳

### 收到一張折價卷 [GET]

+ Response 200 (application/json)
    + Attributes (折價卷)

### 新增一個折價卷 [POST]
屬性也可以定義在請求和回應中：

+ Request

    + Attributes (object)

        + percent_off: 25 (number)
        + redeem_by (number)

### 列出所有折價卷 [GET /coupons]

+ Response 200 (application/json)
    + Attributes (array[折價卷, 折價卷])
```

屬性 Attributes 可以定義在資源或動作中，上面的例子裡，在資源折價卷中定義了一個 Attributes，有 4 個欄位，欄位的定義方式和 Parameters 一樣，然後在動作 GET 的回應中使用 `+ Attributes (折價卷)` 引用，會自動依照回應的 Content-Type 轉成對映的格式，在此是 application/json，所以 body 會是 JSON 如下所示：

``` json
{
  "id": "250FF",
  "created": 1415203908,
  "percent_off": 25,
  "redeem_by": 0
}
```

不過在 apiary 實測，只能轉成 JSON，其它都不行。

動作「新增一個折價卷」的請求定義了一個屬性作爲 body，所以屬性可以定義在請求和回應中作爲 body，但是寫法是如上所示，不是在 `+ Body` 中內含 `+ Attributes`，如下所示：

```
+ Response 200 (application/json)

    + Body

        + Attributes (object)

            + percent_off: 25 (number)
            + redeem_by (number)
```

然後在列出所有折價卷的回應中，`+ Attributes (array[折價卷, 折價卷])` 表示屬性折價卷是陣列的元素，總共有 2 個元素，所以回應的 body 會是這樣：

``` json
[
  {
    "id": "250FF",
    "created": 1415203908,
    "percent_off": 25,
    "redeem_by": 0
  },
  {
    "id": "250FF",
    "created": 1415203908,
    "percent_off": 25,
    "redeem_by": 0
  }
]
```

不能引用自己內部定義的屬性。如下所示，在動作 get users 內定義的屬性，不能用在自己的回應中引用。

``` markdown
# users [/users]

## get users [GET]

+ Attributes (object)
  + name (string)
  + email (string)

+ Response 200 (application/json)
  + Attributes (get users)
```

### 資料結構 Data Structures

屬性 Attributes 不一定要屬於某個資源或動作，可以單獨放到外面，名爲資料結構 Data Structures 的區塊中。

``` markdown
### 收到折價卷 [GET /coupons]

+ Response 200 (application/json)
    + Attributes (折價卷)

# Data Structures

## 折價卷 (object)

+ id: 250FF (string, required)
+ created: 1415203908 (number) - 新增時間戳
+ percent_off: 25 (number)

    打折的百分比，從 1 到 100

+ redeem_by (number) - 兌換有效期限，此爲時間戳
```

上述放在折價卷資源中的屬性，改成放在資料結構中，則 `+ Attributes (object)` 改成 `## 折價卷 (object)`，內含的欄位 id, created, percent_off 和 redeem_by 都不變，只是沒有向內縮排

請注意，使用 aglio 轉換時，如果在資源或動作中定義 Attributes，會沒有 body，但是在 Data Structures 定義就沒有這個問題。

總結：屬性可以定義在資源、動作、請求與回應和資料結構，但是只能引用定義在其他資源和資料結構的屬性。

### 繼承屬性

屬性之間可以有類似類別的繼承關係，例如定義折價卷屬性基礎的 2 個欄位 percent_off 和 redeem_by，然後在需要時繼承它，再新增 2 個欄位

``` markdown
### 收到折價卷 [GET /coupons]

+ Response 200 (application/json)
    + Attributes (折價卷)

# Data Structures

## 折價卷基礎 (object)

+ percent_off: 25 (number)

    打折的百分比，從 1 到 100

+ redeem_by (number) - 兌換有效期限，此爲時間戳

## 折價卷 (折價卷基礎)

+ id: 250FF (string, required)
+ created: 1415203908 (number) - 新增時間戳
```

折價卷基礎先定義 2 個欄位 percent_off 和 redeem_by，然後折價卷屬性在小括號中指定繼承它，再新增 2 個欄位 id 和 created，資源中的屬性也能這樣繼承。

``` json
{
  "percent_off": 25,
  "redeem_by": 0,
  "id": "250FF",
  "created": 1415203908
}
```
子屬性折價卷的欄位會加在親屬性折價卷基礎的下方

只有定義在資源和資料結構的屬性可以被引用，所以也只有這 2 個地方的屬性可以被繼承。

### 資源模型 Resource Model

可以把請求或回應的內容獨立出來，重複引用，名爲資源模型。

``` markdown
## My Resource [/resource]

+ Model (text/plain)

        Hello World

## My Message [/message]

+ Model (application/vnd.siren+json)

    一個 `application/vnd.siren+json` 訊息資源範例

    + Headers

            Location: http://api.acme.com/message

    + Body

            {
              "class": [ "message" ],
              "properties": {
                    "message": "Hello World!"
              },
              "links": [
                    { "rel": "self" , "href": "/message" }
              ]
            }

### Retrieve a Message [GET]
在回應和請求中引用

+ Request 接收訊息的請求

    [My Resource][]

+ Response 200

    [My Message][]

```

上述有兩個資源模型，格式是 `+ Model (Content-Type)`，底下必須要向內縮 4 個空格或 1 個 Tab，表示這些都隸屬於這個資源模型。第一個的內容是簡寫，表示 body 的內容是 Hello World，第二個定義在資源 My Message 中，內含說明、`Headers` 和 `Body`，然後在 `+ Request` 或 `+ Response` 向內縮 4 個空格或 1 個 Tab，使用 `[資源名稱][]` 引用它。

資源模型中的小括號在請求或回應中都代表 Content-Type，所以 Accept 需要在 `+ Headers` 中明確指出才行。不過即使如此，在請求和回應中，Accept 也不會覆蓋 Content-Type，而是兩者併呈。

一個資源模型只能在資源中定義，而且一個資源只能有一個資源模型，所以上面的範例只有一個資源模型。模型本身不能像請求一樣命名，它使用資源的名稱作爲引用的名稱，所以沒有 `+ Model A (application/json)` 這樣的寫法。

引用使用的是 Markdown 的[參考語法](https://daringfireball.net/projects/markdown/syntax#link)，不過第二個中括號要空著，否則無法引用。

### JSON Schema

[JSON Schema](https://json-schema.org)用來描述 JSON 的資料格式，例如欄位 name 的型別是字串，欄位 quantity 的型別是整數，方便自動測試。使用屬性 Attributes 時，會自動產生 Schema，你也可以手動輸入 Schema。

``` markdown
+ Response 200 (application/json)

    + Body

            {
                "id": "abc123",
                "title": "This is a note",
                "content": "This is the note content."
                "tags": [
                    "todo",
                    "home"
                ]
            }

    + Schema

            {
                "type": "object",
                "properties": {
                    "id": {
                        "type": "string"
                    },
                    "title": {
                        "type": "string"
                    },
                    "content": {
                        "type": "string"
                    },
                    "tags": {
                        "type": "array",
                        "items": {
                            "type": "string"
                        }
                    }
                }
            }
```

使用 `+ Schema` 手動輸入 JSON Schema，如上所示。apiary 自動產生的 JSON Schema 如下所示，所以 `+ Schema` 手動輸入的內容會覆蓋掉屬性自動產生的 JSON Schema。

``` json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "title": {
      "type": "string"
    },
    "content": {
      "type": "string"
    },
    "tags": {
      "type": "array"
    }
  }
}
```
