# GitHub 預設分支 master 改名為 main

原文在 2023-05-08 完成

今天把 GitHub 某個專案的預設分支從 master 改成 main，然後 GitHub 就顯示以下的說明：

If you have a local clone, you can update it by running the following commands.

``` bash
git branch -m master main
git fetch origin
git branch -u origin/main main
git remote set-head origin -a
```

好奇那是什麼意思，所以就查了一下，以後可能會用到

``` bash
git branch -m master main
```
`git branch -m`：把 local 端的分支改名
`git branch -m 舊的分支名稱 新的分支名稱`
可以只有 `git branch -m 新的分支名稱`，這表示把目前的分支改成新的分支名稱

``` bash
git fetch origin
```

`git fetch`：同步遠端的 commit，但是不合併到 local 端
`origin` 代表遠端節點

``` bash
# `git branch (--set-upstream-to=<upstream> | -u <upstream>) [<branchname>]`
git branch -u origin/main main
```

參考 https://git-scm.com/docs/git-branch/#Documentation/git-branch.txt--ultupstreamgt

`git branch -u 遠端分支 local分支`：設定 local 分支要追蹤哪一個遠端分支。如果省略 local 分支，預設為目前的分支

也可以改成 `git branch --set-upstream-to=origin/main main`

``` bash
git remote set-head origin -a
```

參考 https://git-scm.com/docs/git-remote#Documentation/git-remote.txt-emset-headem

說真的，看不太懂。猜測會不會是查詢目前遠端(GitHub)的 HEAD 指向哪一個分支，然後自動設定 local 端的遠端 HEAD 使用相同的分支？