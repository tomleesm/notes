# 使用 Swagger 寫文件

臺灣業界好像比較常用 [Swagger](https://swagger.io/) 寫 API 文件，雖然我不喜歡這樣多層巢狀的 YAML ，還是沒辦法要研究它，所以在這裡筆記重點，以免之後又要再看一次官方文件。

## 工具

使用 Swagger Editor 來編輯並預覽，在 Debian 11 中安裝方法如下：

首先要安裝 Node.js

``` bash
sudo apt install curl build-essential
# 安裝 node.js v16，請參考 https://nodejs.org/en/ 選擇目前推薦的版本
curl -sL https://deb.nodesource.com/setup_16.x | sudo bash -
sudo apt install nodejs

# 檢查 node.js 和 npm 的版本
node -v
npm -v
```

下載 Swagger-Editor ，並解壓縮。[查看目前最新版本](https://github.com/swagger-api/swagger-editor/releases)

``` bash
wget https://github.com/swagger-api/swagger-editor/archive/refs/tags/v4.0.4.tar.gz -O swagger-editor.tar.gz
tar zxvf swagger-editor.tar.gz
mv swagger-editor-4.0.4 swagger-editor
```

安裝 http-server，因為 swagger-editor 沒有內建伺服器軟體

``` bash
sudo npm install -g http-server
```
用 http-server 服務 swagger-editor

``` bash
# 切換到 swagger-editor 目錄的上一層
http-server -a 0.0.0.0 -p 8115 swagger-editor
```

用瀏覽器打開 http://192.168.56.10:8115/

改用 nginx

``` bash
sudo vim /etc/nginx/sites-available/swagger-editor.conf

# 把以下的設定複製貼上到 vim 存檔
# vm 使用 host-only，IP 為 192.168.56.10
# root 設定是 swagger-editor 目錄所在，必須使用絕對路徑
server {
  listen 8115;
  server_name 192.168.56.10;
  index index.html  index.htm;
  client_max_body_size 50M;
  root  /home/tom/apps/swagger-editor;
}
```

Nginx 啓用設定，並重新啓動 Nginx 讓設定生效
``` bash
sudo ln -s /etc/nginx/sites-available/swagger-editor.conf /etc/nginx/sites-enabled/swagger-editor.conf
sudo service nginx restart
```
使用瀏覽器打開 http://192.168.56.10:8115/

## 語法

以下筆記參考自 [Swagger v2 官方文件](https://swagger.io/docs/specification/2-0/basic-structure/)。建議新手可以先去 [YAML 的維基百科](https://zh.wikipedia.org/wiki/YAML) 看一下基本語法。Swagger Editor 安裝後會提供一個寵物店 API 作爲範例，提供[在此作爲參考](pet-store-api.yaml)。

### 開始

``` yaml
# 指定 Swagger 的版本是 2.0，每一個 Swagger API 文件都必須要有這一行，
# 放在第幾行倒是無所謂。
# 有趣的是，2.0 需要用引號包起來，否則無法產生文件
# YAML 會自動依照值決定型別，所以 swagger: 2.0 會自動把鍵 swagger 的值
# 設為浮點數 2.0，用引號包起來表示強制轉型成字串2.0
swagger: "2.0"
# info 表示這個文件的相關資訊
info:
  # 描述：可以使用 markdown 語法，可以省略
  description: "This is a sample server Petstore server.
  You can find out more about Swagger at [http://swagger.io](http://swagger.io)
  or on [irc.freenode.net, #swagger](http://swagger.io/irc/). For this sample,
  you can use the api key `special-key` to test the authorization filters."
  # version 是指這個文件的版本，不是 Swagger 的版本，
  # 所以它的值只是字串，可以用任何你喜歡的格式，例如數字 1.0-beta，日期 2016.11.15
  version: "1.0.0"
  title: "Swagger Petstore"
  termsOfService: "http://swagger.io/terms/"
  contact:
    email: "apiteam@swagger.io"
  license:
    name: "Apache 2.0"
    url: "http://www.apache.org/licenses/LICENSE-2.0.html"
```

### API Host and Base URL

``` yaml
host: petstore.swagger.io
basePath: /v2
schemes:
  - https
  - http
```

在此借用官方文件上的圖片
![Swagger URL Structure](images/swagger-url-structure.png)

host 是網站的網址或 IP 位址和port，不能包含通訊協定，例如 `http://`，因為改在 schemes 設定。

basePath 的值一定要以斜線開頭，例如 `/v2`。如果沒有 basePath，預設是 `/`。

schemes 可以使用的通訊協定有 http 和 https，以及 WebSocket 的 ws 和 wss，使用 YAML 的清單語法，也就是開頭為橫線的方式 `- https`，或是陣列實字語法 `schemes: [http, https]`。

如果沒有設定 host 或 schemes，預設值是文件所在的網站網址和通訊協定。如果文件放在 http://192.168.56.10:8115 ，則 host 預設值為 192.168.56.10:8115，schemes 預設值為 `- http`

### 標籤

``` yaml
# 標籤把路由設定分組
tags:
# 標籤有 pet 和 store (name 的值)
- name: "pet"
  # 參考下方的圖片
  description: "Everything about your Pets"
  externalDocs:
    description: "Find out more"
    url: "http://swagger.io"
- name: "store"
  description: "Access to Petstore orders"
```
產生效果如圖

![Swagger tags](images/swagger-tags.png)

### 請求與回應

``` yaml
# 所有 RESTful 路由都定義在 path
path:
  # RESTful 資源(URI)
  /pet/findByStatus:
    # HTTP methods GET, POST, PUT 等緊接著放在 URI 之下，
    # 注意必須用小寫的 get，不能用大寫 GET
    get:
      # 可以加上標籤，路由會自動歸類在這個群組，沒有標籤的話，預設是 default 群組
      tags:
      - "pet"
      # 可選的簡介會顯示在 /pet/findByStatus 的右邊
      summary: "Finds Pets by status"
      # 可選的描述則顯示在展開的內容
      description: "Multiple status values can be provided with comma separated strings"
      # operationId 有些工具會用到，官方文件解釋：
      # Some code generators use this value to name the corresponding methods in code.
      # 不過 Swagger Editor 沒有用到
      operationId: "findPetsByStatus"
      # consumes：請求的 body Content-Type
      # produces：回應的 body Content-Type 和請求的 Accept
      # 如果定義在 path 外，此爲整個 API 的預設值。定義在 path 內的則會覆蓋預設值。
      produces:
      - "application/xml"
      - "application/json"
      # parameters 定義請求的 body, header 和網址
      # - in: path：定義 /users/{id} 網址的參數 id
      # - in: query：定義網址中的查詢參數，例如 /search?q=keyword 的 q
      #   所以路由網址不能有查詢參數，例如 /search?q=keyword，而要改成 /search
      # - in: body：定義 body
      # - in: header：自訂標頭，例如 X-RateLimit
      parameters:
      # 所以是定義 /pet/findPetsByStatus?status=
      - name: "status"
        in: "query"
        description: "Status values that need to be considered for filter"
        # 是否爲必填，in: path 一定是必填，因爲 /users 和 /users/{id} 對於 Swagger 是不同的 path
        required: true
        # 型別有 boolean, number, integer, string, array 和 object
        # 型別是陣列
        type: "array"
        # 定義陣列元素
        items:
          type: "string"
          # enum：列舉，所以只能是以下這三種
          enum:
          - "available"
          - "pending"
          - "sold"
          # default 預設值，用在 required: false，而且沒有輸入時使用，
          # 因爲上面設定 required: true，所以這裡其實不用定義 default
          default: "available"
        # 相隔方式
        # csv: 逗號相隔，例如 foo,bar,baz
        # ssv: 空格相隔，例如 foo bar baz
        # tsv: Tab 相隔，例如 "foo\tbar\tbaz"
        # pipes: 用 | 相隔，例如 foo|bar|baz
        # multi: 重複出現，例如 foo=value&foo=another_value
        collectionFormat: "multi"
      # 回應都在這裡定義
      responses:
        # 必須先寫 HTTP 狀態碼，然後再描述內容
        "200":
          # 一般來說 description 是可有可無，但是在回應卻是必填，不然會產生錯誤
          description: "successful operation"
          # schema：使用 YAML 定義 JSON 或 XML 之類的樹狀結構
          # 而且定義的是欄位的型別、最大最小值之類的抽象結構，不是 { name: "Tom" } 這樣的具體內容
          schema:
            # 和 parameters 一樣，指定型別爲陣列
            type: "array"
            # 定義陣列元素
            items:
              # 定義放在不遠處的 definitions 中，然後使用 $ref 引用
              $ref: "#/definitions/Pet"
        "400":
          # 只有 description 表示回應沒有 body
          description: "Invalid status value"

# 定義共用的結構
definitions:
  # 以下的 schema 名稱爲 Pet
  Pet:
    # 型別有 boolean, number, integer, string, array 和 object
    type: "object"
    # 必填的屬性 properties 有 name 和 photoUrls
    required:
    - "name"
    - "photoUrls"
    # 定義 schema 的屬性，所以以下可以轉成 JSON
    # [
    #   {
    #     "id": 0,
    #     "category": {
    #       "id": 0,
    #       "name": "string"
    #     },
    #     "name": "doggie",
    #     "photoUrls": [
    #       "string"
    #     ],
    #     "tags": [
    #       {
    #         "id": 0,
    #         "name": "string"
    #       }
    #     ],
    #     "status": "available"
    #   }
    # ]
    properties:
      id:
        # 型別是整數
        type: "integer"
        # format 是可以自己隨意輸入的字串，用來更準確說明型別格式，int64 表示 64 位元整數
        format: "int64"
      category:
        # 可以引用其他的 schema
        $ref: "#/definitions/Category"
      name:
        type: "string"
        # Swagger schema 只有抽象的結構，example 用來定義具體的範例，
        # 如果沒有 example，Swagger Editor 會自動給予 0, "string" 的預設值
        # 如果有 example，則使用它
        example: "doggie"
      photoUrls:
        type: "array"
        items:
          type: "string"
        # xml：好像是專屬於 XML 的設定，目前都用 JSON，所以跳過
        xml:
          name: "photoUrl"
          wrapped: true
      tags:
        type: "array"
        items:
          $ref: "#/definitions/Tag"
        xml:
          name: "tag"
          wrapped: true
      status:
        type: "string"
        description: "pet status in the store"
        enum:
        - "available"
        - "pending"
        - "sold"
    # xml：好像是專屬於 XML 的設定，目前都用 JSON，所以跳過
    xml:
      name: "Pet"
```

type 和 format 可用的清單參考 [Swagger v2.0 規格](https://swagger.io/specification/v2/) 的 Data Types 小節

``` yaml
path:
  # 有 {petId}，所以在 parameters in: path 有定義
  /pet/{petId}/uploadImage:
    # HTTP method 爲 POST
    post:
      tags:
      - "pet"
      summary: "uploads an image"
      description: ""
      operationId: "uploadFile"
      # consumes 指定請求的 body Content-Type 是 multipart/form-data
      consumes:
      - "multipart/form-data"
      # 回應的 body Content-Type 是 application/json
      produces:
      - "application/json"
      parameters:
      # 定義 /pet/{petId}/uploadImage 的參數 petId
      # 所以 name 一定要是 petId，有區分大小寫
      - name: "petId"
        in: "path"
        description: "ID of pet to update"
        # in: path 的 required 一定是 true
        required: true
        type: "integer"
        format: "int64"
      - name: "additionalMetadata"
        # in: formData 定義請求的 body 使用傳統的字串格式，例如 foo=bar&tar=bar
        in: "formData"
        description: "Additional data to pass to server"
        required: false
        type: "string"
      - name: "file"
        in: "formData"
        description: "file to upload"
        required: false
        # 上傳檔案時，使用 in: formData 而且 type: file
        # Swagger Editor 會產生選擇檔案的按鈕，點選按鈕會有對話框
        type: "file"
      responses:
        "200":
          description: "successful operation"
          schema:
            $ref: "#/definitions/ApiResponse"
```

``` yaml
  parameters:
  - name: "orderId"
    in: "path"
    description: "ID of pet that needs to be fetched"
    required: true
    type: "integer"
    # 設定參數的最大和最小值範圍
    maximum: 10.0
    minimum: 1.0
    format: "int64"
  - name: "api_key"
    # 使用 parameters in: header 定義請求的自訂 header
    in: "header"
    required: false
    type: "string"
  # ...
  responses:
    "200":
      description: "successful operation"
      schema:
        type: "string"
      # 回應的自訂 header 則是直接列在 headers 中
      headers:
        X-Rate-Limit:
          type: "integer"
          format: "int32"
          description: "calls per hour allowed by the user"
        X-Expires-After:
          type: "string"
          format: "date-time"
          description: "date in UTC when token expires"
    "400":
      description: "Invalid username/password supplied"
```
