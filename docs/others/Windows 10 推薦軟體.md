# Windows 10 推薦軟體

## 影片播放

Linux 我用 mpv 播放影片，Windows 10 也能用 mpv ，用 `scoop` 安裝[^1]，不過需要用指令 `mpv`，所以推薦用 [mpv-hero](https://github.com/stax76/mpv-hero)，啟動速度快，還多了許多功能，例如右鍵選單、[實用的圖形界面](https://github.com/tomasklaen/uosc)、和 YouTube 一樣的影片預覽、最近播放清單

只是安裝有點麻煩：

1. 在 [Releases](https://github.com/stax76/mpv-hero/releases) 下載壓縮檔
2. 在 home 目錄新增 apps 資料夾，解壓縮到裡面。因為安裝後會固定尋找當初安裝的路徑，所以放在 apps
3. 執行資料夾 installer 的 `mpv-install.bat` ，需要用系統管理員權限，所以右鍵 Run as administrator。如果沒有，可能是設定成預設用 Notepad++ 打開
4. 執行完 `mpv-install.bat`，依照指示按鍵會出現 Default apps 設定，或是手動打開 Windows 的設定 > Apps > Default apps ，設定用 mpv 做為預設的音樂和影片播放器

[^1]: [https://mpv.io/installation/](https://mpv.io/installation/)