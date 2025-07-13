---
hide:
  - toc
---

# 解決 VTE 為基礎的終端機程式游標問題

這幾天改用 Windows 10，用 VirtualBox 建立 Alpine Linux 虛擬環境，用來跑 docker。其中用 ssh 連到 Linux，再用 Vim 編輯檔案，結果發現游標不是我習慣的那樣。正常情況下，在 normal 模式，游標會選取字元並反白，按 i 會進入 insert 模式，字元不再選取，`|` 游標會在左邊，然後可以開始輸入，如果按 a 則 `|` 游標會在右邊。但是現在卻變成不管是哪個模式，游標都一樣，例如全部都是 `|`，即使是 normal 模式。Neovim 則無此問題。

Google 後發現 Vim 用以下的設定就能恢復正常，其中最後三行的 set 讓游標切換沒有大約一秒的延遲。

``` vim
let &t_SI = "\<Esc>[6 q" " I beam cursor for insert mode
let &t_SR = "\<Esc>[4 q" " underline cursor for replace mode
let &t_EI = "\<Esc>[2 q" " block cursor for normal mode
set ttimeout
set ttimeoutlen=1
set ttyfast
```

VTE 是 Virtual Terminal Emulator（虛擬終端模擬器）的縮寫，為 Gnome 的一部分，它提供 API 以建立各種終端機程式，功能強大。會有游標問題，似乎是因為 VTE 可以自訂游標，Vim 如果沒有特別設定，便會依照終端機的設定全部模式都用相同的游標

資料參考

* [How to change the cursor between Normal and Insert modes in Vim?](https://stackoverflow.com/questions/6488683/how-to-change-the-cursor-between-normal-and-insert-modes-in-vim)
* [help I've tried to change my cursor shape for hours](https://www.reddit.com/r/vim/comments/ar1xer/help_ive_tried_to_change_my_cursor_shape_for_hours/)
* [Change cursor shape in different modes](https://vim.fandom.com/wiki/Change_cursor_shape_in_different_modes)
* [探索GNOME的VTE库：一个强大的终端模拟器引擎](https://blog.csdn.net/gitblog_00046/article/details/136756787)
