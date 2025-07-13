# Git 設定

使用 Neovim 編輯 commit 訊息，不套用 init.lua 設定，速度快

``` bash
git config --global core.editor 'nvim -u NONE -N'
```

[強制每個 Git Repository 都要設定使用者資訊](https://blog.gslin.org/archives/2020/03/29/9459)

``` bash
git config --global user.useConfigOnly true
git config --global --unset user.name
git config --global --unset user.email
git config --global --unset user.signingkey
```

不允許 有 LF 與 CRLF 混合的檔案

``` bash
git config --global core.safecrlf true
```

方便的縮寫

``` bash
# git s 在 git status 只顯示檔名，不顯示分支名稱
git config --global alias.s "status -bs"
# git br 等於 git branch
git config --global alias.br "branch"
# git l 簡短顯示 git log 圖形：hash、新增或更新、commit 訊息
git config --global alias.l "log --oneline --graph"
# git l 簡短顯示 git log 圖形：hash、多久前、誰、新增或更新、commit 訊息
git config --global alias.lt 'log --oneline --graph --pretty=format:"%h [%ar] %an %s"'
```

git init 預設分支名稱爲 main，否則新版本的 Git 會跑出警告

``` bash
git config --global init.defaultBranch main
```

全域  `.gitignore`，不用每個專案都要設定 git ignore .DS_Store

``` bash
git config --global core.excludesfile ~/.gitignore
```

然後編輯 `~/.gitignore` ，列出 `.DS_Store` 和 `*.swp` 等一般不放進 Git 的東西

git status 正常顯示中文檔名
 
``` bash
git config --global core.quotepath false
```