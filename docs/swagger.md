# 使用 Swagger 寫文件

Swagger 使用 YAML 或 JSON 的語法來寫 RESTful API 文件，主要使用 YAML，官方文件也都用 YAML

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

參考 [Swagger v2 官方文件](https://swagger.io/docs/specification/2-0/basic-structure/)。

之前RESTful API：請求與回應的 API 文件可以用 Swagger 寫成如下格式：

``` yaml
swagger: "2.0"
info:
  title: 文章 API
  description: 文章的 API 文件
  version: "2022-01-06"

host: api.example.com
basePath: /
schemes:
  - http

produces:
  - application/json

paths:
  /posts:
    post:
      summary: 新增文章
      description: 新增一篇文章
      consumes:
        - application/json
      produces:
       - text/palin
      parameters:
        - in: body
          name: aPost
          description: 一篇要新增的文章
          schema:
            $ref: '#/definitions/onePostContent'
      responses:
        201:
          description: Created
          headers:
            Location:
              type: string
              description: 新增後的 URL，例如 /posts/1
    get:
      summary: 回傳全部文章
      description: 回傳全部文章
      parameters:
        - in: query
          name: page
          description: 頁碼
          type: integer
          required: false
          default: 1
        - in: query
          name: per_page
          description: 每一頁有幾篇文章
          type: integer
          required: false
          default: 10
      responses:
        200:
          description: 回傳全部文章的標題、內容和分頁資料
          schema:
            type: object
            properties:
              data:
                type: array
                items:
                  $ref: '#/definitions/onePostContent'
              pagination:
                $ref: '#/definitions/pagination'
  /posts/{post}:
    parameters:
      - in: path
        name: post
        description: 文章的 id
        type: integer
        required: true
    get:
      summary: 回傳指定的文章
      responses:
        200:
          description: 回傳一篇指定的文章
          schema:
            $ref: '#/definitions/onePost'
    put:
      summary: 修改指定的文章所有欄位
      produces:
       - text/palin
      parameters:
        - in: body
          name: aPost
          description: 一篇要修改的文章
          schema:
            $ref: '#/definitions/onePost'
      responses:
        204:
          description: No Content
    patch:
      summary: 修改指定的文章部分欄位
      produces:
       - text/palin
      parameters:
        - in: body
          name: aPost
          description: 一篇要修改的文章
          schema:
            $ref: '#/definitions/onePostContent'
      responses:
        204:
          description: No Content
    delete:
      summary: 刪除指定的文章
      produces:
        - text/palin
      responses:
        204:
          description: No Content

definitions:
  onePostContent:
    type: object
    properties:
      title:
        type: string
        description: 文章標題
        example: 文章標題
      content:
        type: string
        description: 文章內容
        example: 文章內容
  onePost:
    type: object
    properties:
      id:
        type: integer
        description: 文章的 id
        example: 1
      title:
        type: string
        description: 文章標題
        example: 文章標題
      content:
        type: string
        description: 文章內容
        example: 文章內容
      createdAt:
        type: string
        description: 文章新增時間，格式為 %Y-%M-%DT%h:%m:%s.%u%Z:%z
        example: "2022-01-20T18:39:27.774000+08:00"
      updatedAt:
        type: string
        description: 文章修改時間，格式為 %Y-%M-%DT%h:%m:%s.%u%Z:%z
        example: "2022-01-20T18:39:27.774000+08:00"
  pagination:
    type: object
    properties:
      total:
        type: integer
        example: 1000
      count:
        type: integer
        example: 10
      perPage:
        type: integer
        example: 10
      currentPage:
        type: integer
        example: 2
      totalPages:
        type: integer
        example: 100
      nextUrl:
        type: string
        example: /posts?page=2&per_page=10
```

用 Swagger UI 轉換後產生 [Swagger Posts API](/swagger-posts-api.png)
