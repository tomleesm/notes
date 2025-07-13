# HTTP 通訊協議簡介

原文在 2023-04-24 完成

以下內容是我閱讀書籍「深入淺出 Servlets 與 JSP」後所做的筆記，用來複習 session 和 cookie 的基本觀念。

## HTTP 請求與回應

使用者點選一個網站連結，連到 `/products.php?id=123`，則瀏覽器會產生一個 HTTP 請求類似這樣：

- `GET /products.php?id=123`：請給我一個檔案或資源，它是在 `/products.php`，參數是 `id=123`
- HTTP/1.1：這個 HTTP 請求的協議的版本是 1.1
- Host：伺服器主機所在的網址
- User-Agent：產生 HTTP 請求的瀏覽器名稱
- Accept：告訴伺服器瀏覽器可以接受的資料是什麼類型

``` http
GET /products.php?id=123 HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html
Accept-Language: en-us,zh-tw
Accept-Encoding: gzip
Accept-Charset: utf-8
Keep-Alive: 300
Connection: keep-alive
```

瀏覽器把上述的 HTTP 請求傳送到伺服器，伺服器發現這個 HTTP 請求需要用 PHP 才能處理，所以丟給 PHP 程式。PHP 處理後產生 HTML，交給伺服器。伺服器把 HTML 處理後產生 HTTP 回應類似這樣：

- 200 OK：回應的 HTTP 狀態碼和狀態碼的對應文字
- Content-Type：告訴瀏覽器要接收的資料是什麼類型
- Server：產生 HTTP 回應的伺服器軟體是 nginx 1.18.0 版
- HTML 放在 body 區塊，只顯示一部分

``` http
HTTP/1.1 200 OK
Cache-Control: no-cache, private
Connection: keep-alive
Content-Encoding: gzip
Content-Type: text/html; charset=UTF-8
Date: Mon, 06 Jun 2022 07:28:34 GMT
Server: nginx/1.18.0
Set-Cookie: session=eyJpdiI6IkY5MTltbm1nSE1cL29ZbDE3b01OdXlBPT0iLCJ2YWx1ZSI6ImtYbEFN;expires=Mon, 06-Jun-2022 09:28:34 GMT; Max-Age=7200; path=/products.php?id=123; httponly
Transfer-Encoding: chunked
```

``` html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
```

伺服器把上述的 HTTP 回應傳送回瀏覽器，瀏覽器把 HTML 處理後呈現在螢幕上，讓使用者看到

## session 與 cookie

如果使用 session 和 cookie 儲存使用者的購物清單，會是這樣的：

1. 使用者點選一個連結，把產品加到購物車中
2. 伺服器產生一個 session 檔案，儲存購物清單，並給 session 一個編號 ABC123
3. 伺服器在 HTTP 回應的 header 加上 `Set-Cookie: session_id=ABC123`
4. 瀏覽器收到 HTTP 回應後，產生一個 cookie 檔案，儲存 `session_id=ABC123` 
5. 瀏覽器讀取 cookie，產生新的 HTTP 請求，在 header 有 `Cookie: session_id=ABC123`
6. 伺服器收到 HTTP 請求後，依照 header `Cookie: session_id=ABC123` 就知道要找哪一個 session
7. 伺服器找到編號 ABC123 的 session，讀取購物清單

如果想要的是使用者登入後，一直維持已登入狀態，則只要在第 2 點儲存購物清單改成儲存 `login=true` 之類的已登入標示，接著就是第 3 點之後那樣。

當然也可以用 cookie 儲存購物清單：

1. 使用者點選一個連結，把產品 productA 和 productB 加到購物車中
2. 伺服器在 HTTP 回應的 header 加上 `Set-Cookie: shopping_cart=productA,productB`
3. 瀏覽器收到 HTTP 回應後，產生一個 cookie 檔案，儲存 `shopping_cart=productA,productB`
4. 瀏覽器讀取 cookie，產生新的 HTTP 請求，在 header 有 `Cookie: shopping_cart=productA,productB`
5. 伺服器收到 HTTP 請求後，依照 header  `Cookie: shopping_cart=productA,productB` 產生購物清單

## 過期

cookie 和 session 會設定一段時間後就過期被刪除，避免很久沒用的 cookie 和 session 佔用磁碟空間。重新產生 session 和 cookie 的過程如下：

1. 儲存著 `session_id=ABC123` 的 cookie 過期，瀏覽器刪除這個 cookie
2. 瀏覽器發出 HTTP 請求，header 沒有 `Cookie: session_id=ABC123`
3. 伺服器收到 HTTP 請求，因爲 header 沒有 `Cookie: session_id=ABC123`，不會尋找這個 session
4. 使用者做了一些事情，需要用 session 儲存資料
5. 伺服器產生新的 session，用 HTTP 回應傳送新的 session_id 給瀏覽器
6. `session_id=ABC123` 的 session 過了一段時間後過期，被伺服器刪除