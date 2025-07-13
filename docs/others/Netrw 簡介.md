---
hide:
  - toc
---

# Netrw 簡介

原文在 2024-04-19 完成

Vim 內建檔案瀏覽器 Netrw，是像 VS Code 左側的檔案清單，所以其實不需要安裝其它外掛，例如 [nvim-neo-tree/neo-tree.nvim](https://github.com/nvim-neo-tree/neo-tree.nvim)、[prichrd/netrw.nvim](https://github.com/prichrd/netrw.nvim) 或 [nvim-tree/nvim-tree.lua](https://github.com/nvim-tree/nvim-tree.lua) 。不過需要一些設定才行。

在終端機執行 vim 時，如果開啟的是目錄，則自動使用 Netrw 。或者在 Vim 中執行指令 `:Explore` 或縮寫 `:E` ，也能開啟 Netrw。請把 Netrw 視為 buffer，所以關閉 Netrw 的方法是指令 `:bdelete` 或縮寫 `:bd`。不過用 `:ls` 不會列出 Netrw

如何產生類似 VS Code 預設的，左側 sidebar，開啟檔案後維持 sidebar 顯示？

首先在 `.vimrc` 輸入以下設定

- `let g:netrw_liststyle = 3` 顯示為樹狀結構
- `let g:netrw_banner = 0` 不顯示上方的 banner
- `let g:netrw_winsize = 30` 分割視窗的寬度為 30%

然後執行指令 `:Lexplore`，會在左邊分割視窗開啟 Netrw，選擇檔案開啟後 Netrw 仍會顯示。再次輸入 `Lexplore` 會關閉 Netrw

在 `.vimrc` 設定輸入 `<Leader>f` 會切換顯示 Netrw（`<Leader>` 鍵設定為 `,` 逗號）

``` vim
nnoremap <LEADER>f :Lexplore<CR>
```

常用指令

- Enter：開啟目錄或檔案
- `-`：進入上一層目錄
- `u`：返回歷史記錄中的上一個目錄
- `<C-L>`：更新目錄清單
- `p`：在下方水平分割視窗顯示預覽。曾經預覽過的檔案，關閉 vim 後會在下次預覽時直接打開檔案，而且是唯讀、沒有進到 buffer
- `<C-W>z`：關閉預覽視窗
- gh：切換隱藏檔案
- `%`：詢問新的檔案名稱，並新增和編輯它
- R：重新命名或移動檔案
- d：建立目錄
