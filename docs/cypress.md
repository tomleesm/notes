# 使用 Cypress 測試網站

以前知道有 Selenium 這種東西可以操作網站，因爲官方文件不齊全，學得很辛苦，後來發現 Laravel 有 Dusk，根本不用辛苦的研究 Selenium 的 Java 語法，真是令人傻眼。沒過多久發現網路延遲時，Dusk 不會自動等候，所以表單送出後，沒辦法檢查是否已跳轉頁面，最後才發現 Cypress 這個好東西，可以用很直白的語法去做瀏覽器測試，而且沒有 Selenium 的種種問題，真是相見恨晚。

想要學 Cypress，最好直接下載[Cypress 桌面版本](https://docs.cypress.io/guides/getting-started/installing-cypress#Direct-download)，解壓縮後開啓程式，選好一個目錄後會自動產生一堆範例，包含一堆註解，全部看完後大部分的操作都會了，[官方文件](https://docs.cypress.io/guides/core-concepts/introduction-to-cypress)則是用來補充觀念的。

## 常用指令

### 導覽

`cy.visit(String URL)`：連線到指定的網址或相對位址

### selector

### assertion

## 觀念
