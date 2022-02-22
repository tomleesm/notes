# 使用 Swagger 寫文件

[Swagger](https://swagger.io) 使用 YAML 或 JSON 的語法來寫 RESTful API 文件，主要使用 YAML，官方文件也都用 YAML

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

下载 Swagger-Editor ，並解壓縮。[查看目前最新版本](https://github.com/swagger-api/swagger-editor/releases)

``` bash
wget https://github.com/swagger-api/swagger-editor/archive/refs/tags/v4.0.4.tar.gz -O swagger-editor.tar.gz
tar zxvf swagger-editor.tar.gz
mv swagger-editor-4.0.4 swagger-editor
```

安裝 http-server，因爲 swagger-editor 沒有內建伺服器軟體

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
# vm 使用 host-only，IP 爲 192.168.56.10
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

以下筆記參考自[Swagger v2 官方文件](https://swagger.io/docs/specification/2-0/basic-structure/)

### 開始

``` yaml
swagger: "2.0"
info:
  title: Sample API
  description: API description in Markdown.
  version: 1.0.0
```
第 1 行的 `swagger: "2.0"` 指定 Swagger 的版本是 2.0，每一個 Swagger API 文件都必須要有這一行，放在第幾行倒是無所謂。有趣的是，2.0 需要用引號包起來，否則無法產生文件。YAML 會自動依照值決定型別，所以 `swagger: 2.0` 會自動把鍵 swagger 的值設爲浮點數 2.0，用引號包起來表示強制轉型成字串2.0

接著 info 表示這個文件的相關資訊，包含 3 個鍵 title、description 和 version，代表文件的標題、描述和版本。description 是這個文件的相關描述，可以用 Markdown 格式，可以沒有這個鍵。version 是指這個文件的版本，不是 Swagger 的版本，所以它的值只是字串，可以用任何你喜歡的格式，例如數字 1.0-beta，日期 2016.11.15。

### API Host and Base URL

host, basePath 和 schemes 指定 API 文件所在的網址

``` yaml
host: petstore.swagger.io
basePath: /v2
schemes:
  - https
  - http
```

在此借用官方文件上的圖片
![Swagger URL Structure](images/swagger-url-structure.png)

host 是網站的網址或 IP 位址，不能包含通訊協定，例如 `http://`，因爲改在 schemes 設定。有 port 的話要指明。

basePath 的值一定要以斜線開頭，例如 `/v2`。如果沒有 basePath，預設是 `/`。

schemes 可以使用的通訊協定有 http 和 https，以及 WebSocket 的 ws 和 wss，使用 YAML 的清單語法，也就是開頭爲橫線的方式 `- https`，或是陣列實字語法 `schemes: [http, https]`。

如果沒有設定 host 或 schemes，預設值是文件所在的網站網址和通訊協定。如果文件放在 http://192.168.56.10:8115 ，則 host 預設值爲 192.168.56.10:8115，schemes 預設值爲 `- http`

### Paths and Operations

如果有一個 RESTful 路由，請求 `GET /message`，回應是字串 Hello World，用 Swagger 寫成文件會是這樣：

``` yaml
swagger: "2.0"
info:
  title: Message API
  version: 1.0.0
paths:
  /message:
    get:
      summary: 回傳字串 Hello World
      description: 可以用 markdown 語法的附加描述
      produces:
        - text/plain
      responses:
        200:
          description: Hello World
```
所有的 RESTful 資源和 HTTP methods 都要定義在 paths 內，上述的例子中，資源 `/message` 寫成 `/message:`，HTTP method `GET` 則在下一層的 `get:`，請注意 get 必須是小寫，不能是大寫的 GET。summary 和 description 是這個路由的簡要描述和附加描述，都是可以省略的。

Swagger 使用 Consumes 代表請求的表頭欄位 Accept，用 Produces 代表回應的表頭欄位 Content-Type，所以下面的例子描述 Accept 和 Content-Type 都是 application/json 或 application/xml

``` yaml
consumes:
  - application/json
  - application/xml
produces:
  - application/json
  - application/xml
```

回應定義在 `responses`，必須先寫 HTTP 狀態碼，例如200，然後是內容。如果內容是單純的字串，寫在 description 即可，如果是 JSON 之類的樹狀結構，則要寫在 `schema` 內，description 則變成單純的附加描述。`GET /message` 的回應只是字串 Hello World，所以寫在 description。

### Parameters


