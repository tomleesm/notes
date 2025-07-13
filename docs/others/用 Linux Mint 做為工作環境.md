---
hide:
  - toc
---

# 用 Linux Mint 做為工作環境

之前用 Windows 10，但是現在又改回用 Linux Mint，因為有一些缺點讓我無法忍受：

只要停下來幾分鐘不用電腦，就自動開始下載檔案並更新系統，搞得 CPU 和磁碟滿載，而且還一直安裝失敗。即使設定手動安裝 Windows Update，也還是一樣。

Docker 啟動很慢，即使是啟動 Docker 本身就大約需要 4 分多鐘，還要吃掉 WSL 需要的 2GB 記憶體。

雖然有用 ClearType，字型還是很醜。我需要在瀏覽器同一頁上顯示繁體中文、簡體中文、日文和韓文，但是 Windows 10 設定用 Noto 字型，只能套用一種字型，結果就是簡體中文仍然是鋸齒狀的字，只能用新同文堂自動轉繁體，自動套用正常的繁體中文字型。

如果改用 Linux Mint，自然不需要 Windows 10 更新，可以用 100% 全速執行 Docker，不需要擔心 VM 吃掉記憶體，還能像 CSS 那樣設定套用各種字型（參考 [用 fontconfig 治理 Linux 中的字型](https://catcat.cc/post/2021-03-07/) 和 [fontconfig調整Linux中文預設字體的優先順序，修正字體模糊、Emoji亂碼的問題](https://ivonblog.com/posts/linux-fontconfig/) ）

但是還是有缺點的：

- 牛津英漢字典只能在 Windows 執行，這個可以用 Google 查字典替代
- Linux 看圖軟體功能普通，不過我並不常看圖，所以沒差
