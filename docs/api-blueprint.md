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

接著一定要有一個 h1 標題（`# The simplest API`）作爲文件主標題，一般是說明這是哪一個 API 文件，標題下方可以加上摘要描述，寫法和一般的markdown 一樣。

``` markdown
# GET /message
+ Response 200 (text/plain)

        Hello World!
```

這是一個簡單的例子，說明 HTTP 請求與回應。`#` 之後的是 HTTP method `GET` 和 RESTful 資源 `/message`，回應則寫成 `+ Response`，使用了 markdown 的無序清單語法 `+`（清單可用 `+`、`-` 或 `*` 開頭），Response 後面接著是 HTTP 狀態碼 200 和 Content-Type text/plain，最後回應的 body 則用 markdown 的 code 區塊語法表示，所以 Hello World 前面要有 8 個空格或是 2 個 Tab，或是用成對的三個 ` 包起來，不過一般都是用空格或 Tab，之後會看到，那是因爲 body 會和 Headers 等其它設定構成巢狀結構。

請注意，使用 aglio 轉換時，如果在資源或動作中定義 Attributes，最好使用英文名稱，因爲中文名稱無法在 `+Attributes ()` 中使用，會沒有 body，但是在 Data Structures 定義就沒有這個問題。此外， attribute 在 `+ Body` 區塊中無法使用。
