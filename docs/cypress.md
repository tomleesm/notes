# 使用 Cypress 測試網站

以前知道有 Selenium 這種東西可以操作網站，因爲官方文件不齊全，學得很辛苦，後來發現 Laravel 有 Dusk，根本不用辛苦的研究 Selenium 的 Java 語法，真是令人傻眼。沒過多久發現網路延遲時，Dusk 不會自動等候，所以表單送出後，沒辦法檢查是否已跳轉頁面，最後才發現 Cypress 這個好東西，可以用很直白的語法去做瀏覽器測試，而且沒有 Selenium 的種種問題，真是相見恨晚。

想要學 Cypress，最好直接下載[Cypress 桌面版本](https://docs.cypress.io/guides/getting-started/installing-cypress#Direct-download)，解壓縮後開啓程式，選好一個目錄後會自動產生一堆範例在子目錄 cypress/integration/，包含一堆註解，全部看完後大部分的操作都會了，[官方文件](https://docs.cypress.io/guides/core-concepts/introduction-to-cypress)則是用來補充觀念的。

## 常用指令

### 分組

`describe()`, `it()` 和 `context()` 把指令分組，可以隨意搭配，只是語義不同而已，官方文件使用 `it()` 代表一個測試，用 `describe()` 包起一個個 `it()`，代表一組相關的測試，如果有其他需求，則用 `context()`

``` javascript
describe('名稱', () => {
  it('測試名稱', () => {
    // Cypress 指令
  })
  context('準備特別的需求', () => {

  })
})
```

### 測試前和測試後

``` javascript
describe('名稱', () => {
  before(() => {
    // 在這個區塊中的第一個測試執行前，執行 before()
  })

  after(() => {
    // 在這個區塊中的最後一個測試執行後，執行 after()
  })

  beforeEach(() => {
    // 在這個區塊中的每一個測試執行前，執行 beforeEach()
  })

  afterEach(() => {
    // 在這個區塊中的每一個測試執行後，執行 afterEach()
  })
})
```

### 導覽

- [visit()](https://docs.cypress.io/api/commands/visit)：連結到指定的網址或相對位址。在 `cypress.json` 設定 `{ "baseUrl": "http://www.example.com" }`，則 `cy.visit('about')` 會連結到 http://www.example.com/about

### 選取元素

- [get()](https://docs.cypress.io/api/commands/get)：使用 CSS Selector 或 alias 選取 DOM 元素
- [contains()](https://docs.cypress.io/api/commands/contains)：選取符合文字的 DOM 元素
- [fiirst()](https://docs.cypress.io/api/commands/first)：選取多個 DOM 元素的第一個
- [last()](https://docs.cypress.io/api/commands/last)：選取多個 DOM 元素的最後一個
- [parent()](https://docs.cypress.io/api/commands/parent)：選取目前 DOM 元素的上一層元素
- [parents()](https://docs.cypress.io/api/commands/parents)：選取目前 DOM 元素的多個上層元素
- [find()](https://docs.cypress.io/api/commands/find)：選取目前 DOM 元素內層的多個元素

### 判斷

- [should()](https://docs.cypress.io/api/commands/should)：產生一個判斷，自動重試直到超過時間

### 輸入資料與操作元素

- [type()](https://docs.cypress.io/api/commands/type)：輸入資料到 DOM 元素
- [click()](https://docs.cypress.io/api/commands/click)：點選 DOM 元素
- [check()](https://docs.cypress.io/api/commands/check)：核取 input checkbox 或 radio 元素

## 觀念
