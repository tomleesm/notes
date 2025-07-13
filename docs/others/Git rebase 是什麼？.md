# Git rebase 是什麼？

原文在 2023-04-26 完成

首先新增一個目錄 test，並在目錄中新增文字檔 c，第一行新增數字 0，存檔後執行 `git commit -a -m 'C0'`，接著第二行新增數字 1，存檔後執行 `git commit -a -m 'C1'`，第三行新增數字 2，存檔後執行 `git commit -a -m 'C2'`。

以上，如果有分支 master 和 experiment 如下，C2 到 C3 的改變是檔案 c 第 4 行新增數字 3，C2 到 C4 的改變是檔案 c 第 1 行改成 5

```
                C4 -- [master]
               /
C0 <-- C1 <-- C2
               \
               C3 -- [experiment]
```

切換分支到 experiment

`git merge master`：以 C3 和 C4 的共同祖先 C2 為基礎，比較 C2 到 C3 的改變 (第 4 行新增 3)，然後 C2 和 C4 的改變(第 1 行改成 5)，於是產生 C5，檔案 c 的內容和分支會是以下所示：

```
5
1
2
3
```

```
                  C4 -- [master]
                /    \
C0 <-- C1 <-- C2      C5 -- [experiment]
                \    /
                  C3
```

`git rebase master`：

> 回到兩個分支最近的共同祖先，根據當前分支（也就是要進行衍合的分支 `experiment`）後續的歷次提交物件（這裡只有一個 C3），生成一系列檔補丁，然後以基底分支（也就是主幹分支 `master`）最後一個提交物件（C4）為新的出發點，逐個應用之前準備好的補丁檔，最後會生成一個新的合併提交物件（C3'），從而改寫 `experiment` 的提交歷史，使它成為 `master` 分支的直接下游 （參考[分支的衍合](https://iissnan.com/progit/html/zh-tw/ch3_6.html)）

以 C4 為基礎，把 C2 到 C3 的改變 (第 4 行新增 3) 改成移到 C4 上做(此時 C4 檔案內容是 5 1 2)，於是產生 C3'。重點是改寫歷史，C3 落單了，產生 C3'，分支 experiment 從指向 C3 改成指向 C3'。C3 會在 git 執行 gc 時清除掉。檔案 c 的內容和上面的 C5 是一樣的(5 1 2 3)。分支結果如下，為了方便和上面比較所以畫成這樣，一般會顯示成一直線。

```
                 C4 -- [master]
               /   \
C0 <-- C1 <-- C2   C3' -- [experiment]

                C3
```

rebase 指令的一般形式是 `git rebase [基礎分支] [被合併的分支]`，所以上述的 rebase 等於指令 `git rebase master experiment`。如果省略被合併的分支，則使用當前分支。