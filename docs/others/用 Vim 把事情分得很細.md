---
hide:
  - toc
---

# 用 Vim 把事情分得很細

在「內向心理學」這本書(P. 250)提到：

> 著手工作時將任務分解成小部份。對性格內向者來說，這是最有益的方式。它有助於你減輕焦慮、頭腦不清和無助的感覺。

「工作大解放: 這樣做事反而更成功」提到把大計劃分成小計劃，會改變你的人生

> 創造速贏：別耗太久，沒有人喜歡追逐看不到的終點線。養成一路上達成許多小勝利的習慣，如此便能保有氣勢、增強動機。
> 
> ...解決的辦法就是把大事化分成數件小事。要辦的事情規模愈小，就愈容易預測
> 
> 優先的事情總是一大堆。這種做法其實不是排序。你應該做的是視覺上的優先順序。把最重要的事情列在最上面，當你做完一件事，在清單上的下一件事就是下一個最重要的事

「為什麼他接的案子比我多？」第35章如何吃下一頭大象則是說：

> 細分成「口服」份量...我們把任務仔細的分解
> ...工作細分的大小，應該跟案子大小成反比：案子愈大，分的愈細

所以把事情分解成更小的事項，可以保持速贏、容易預測時間、減輕無助與焦慮，好處多多。一直以來我參考 XDite 的建議使用 Redmine 這個專案管理軟體分解工作事項。參考[淺談軟體專案管理，如何挑選工具與方法](http://xdite-ld.logdown.com/posts/2015/02/18/introduction-project-management)
> 我們怎用 Redmine ? 會用 Redmine 是因為試到一個功能其他專案管理軟體「都沒有」。就是「無限的巢狀票」... [範例圖片](https://www.flickr.com/photos/xdite/6469521821/sizes/o/in/photostream/)

但是 Redmine 有一些缺點，讓我受不了：

1. 不好安裝和備份: 用 Docker 安裝方便很多，但是用 Dropbox 備份 SQLite 資料庫，我老是覺得檔案太大
2. 字太小：不熟 Redmine 的 CSS 設定，而且每次更新不就都要改 CSS？
3. 不方便調整 issue 順序：常常工作到一半發現需要調整 issue 順序，到上一層或下一層，調到其他 issue 成為 sub-issue 等

改用 Vim 就方便許多：

1. 就只是一個文字檔，備份很方便
2. Vim 字型大小遵照終端機設定，一般用滑鼠按一按即可找到。更新 Vim，設定也不會變
3. 用 Vim 折疊功能，打開來就是 issue 內容，折起來又能看整份大綱
4. 設定向右二個空格為一層，和 Redmine issue 一樣都是向右縮排視為子票
5. 能用 Vim 本身的功能快速操作：
   - 用 `:set foldlevel=2` 只打開到第二層
   - `>>` 移動到下一層，`<<` 移動到上一層
   - 用 Visual 模式選好範圍，用 `d` 和 `p` 剪下貼上即可移動整個 issue 
   - 一點小小設定就能用 `Ctrl + 向上/向下` 把整行上下移動，比一般 todo list 軟體用滑鼠更準確、方便

不過也是有缺點的：

1. 不能附加檔案，例如圖片
2. 需要熟悉 Vim

在 home 目錄的檔案 `.vimrc` 加入以下設定，一個雙引號開頭是註解，可省略
``` vim
" 依照縮排折疊
set foldmethod=indent
" 設定一層是向右縮排幾個空格
set shiftwidth=2
" 折疊到剩下幾層。這裡是打開檔案時不折疊，因為一般不會向右縮排到十層吧？
set foldlevel=10
" 大於多少行才要折疊，預設是 1
set foldminlines=0
" 啟用折疊
set foldenable

" Ctrl + J / K 向下/向上一行移動
nnoremap <C-j> :move +1<CR>
nnoremap <C-k> :move -2<CR>
```

nnoremap 表示在 `Normal` 模式下執行，`<C-j>` 表示 `Ctrl + J`，Ex 指令 `:move +1` 是把目前游標的那一行向下一行移動， `:move -2` 則是向上一行移動，`<CR>` 是 Enter。

`<C-j>` 的 C 可以改成 S (Shift)、A (Alt)、或 M (Meta)，方向鍵是 UP (向上) 和 DOWN (向下)，所以 `Ctrl + 向上` 是 `<C-UP>`

以下是 Vim 的折疊相關指令

* `zo`：打開游標所在的那一層折疊
* `zc`：關閉游標所在的那一層折疊
* `za`：切換游標所在的那一層折疊
* `zA`：切換游標所在的每一層折疊
* `zO`：打開游標所在的每一層折疊
* `zC`：關閉游標所在的每一層折疊
* `zr`：`foldlevel + 1`
* `zm`：`foldlevel - 1`
* `zR`：打開每一層折疊
* `zM`：等於 `set foldlevel=0`
* `zi`：切換啟用折疊功能
* `zx`：更新折疊。取消手動打開或關閉的折疊，重新套用 foldlevel
* `zX`：取消手動打開或關閉的折疊，重新套用 foldlevel
* `[z`：到目前開啟的折疊中，和游標同一層的第一個摺疊
* `]z`：到目前開啟的折疊中，和游標同一層的最後一個摺疊
* `zj`：向下移動。到達下一層摺疊的第一個。關閉的摺疊也被計入
* `zk`：向上移動。到達下一層摺疊的最後一個。關閉的摺疊也被計入
