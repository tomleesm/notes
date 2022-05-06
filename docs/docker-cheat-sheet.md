# Docker 懶人包

## 安裝 Docker

- Windows：參考[Install Docker Desktop on Windows](https://docs.docker.com/desktop/windows/install/)。WSL 是系統核心 API 轉譯的方式執行(參考[介紹好用工具：WSL (Windows Subsystem for Linux)](https://blog.miniasp.com/post/2019/02/01/Useful-tool-WSL-Windows-Subsystem-for-Linux))，Hyper-V 則是微軟自家的虛擬化技術，所以保守的安裝方式是選擇 Hyper-V backend and Windows containers，以免之後可能有一些奇怪的小問題，或是自己用 VirtualBox 或 VMWare 建立一個 Linux 環境，然後在裡面安裝 Docker
- Linux：參考[Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)或選擇對應的發行套件

## 指令說明

以下語法如果在 Linux 下，應該都要在開頭加上 `sudo`

| 語法 | 說明 | 範例 |
| ---- | ---- | ---- |
| docker pull Image名稱 | 下載 [Docker Hub](https://hub.docker.com/) 公用的 Image | docker pull nginx |
| docker images | 顯示已下載的 Image | |
| docker run -p 對外的port:預設的port -d Image名稱 | 新增並執行一個容器 | docker run --name web 8080:80 -d nginx |
| docker ps | 顯示運作中的容器 | |
| docker start 容器ID或名稱 | 啓動容器 | docker start ut37 |
| docker stop 容器ID或名稱 | 容器執行正常關機程序 | docker stop my_nginx |
| docker kill 容器ID或名稱 | 中斷容器執行 | docker kill my_nginx |
|docker restart 容器ID或名稱 | 重新啓動容器 | docker restart my_nginx |
| docker rm 容器ID或名稱 | 刪除已停止的容器 | docker rm my_nginx |
| docker rmi 容器ID或名稱 | 刪除 Image | docker rmi hello-world |
| docker inspect 容器ID或名稱 | 顯示容器資訊 | docker inspect my_nginx |
| docker cp 檔案位址 容器ID或名稱:容器內的絕對位址 | 複製檔案到容器內的指定位置 | docker cp ./some_file CONTAINER:/work |
| docke stats 容器ID或名稱 | 顯示容器運作時的 CPU 等使用情況 | docker stats my_nginx |
| docker logs 容器ID或名稱 | 顯示容器內程式的 log (伺服器的 access log) | docker logs my_nginx |
| docker exec 容器ID或名稱 指令 | 連線到運作中的容器內執行指令 | docker exec -it my_mysql mysql -u root -p 用 root 連進 MySQL |


- 容器ID或名稱可以一次接多個，例如 `docker start php mysql nginx`
- `ps` 加上 `-a` 顯示運作中和停止的容器，`-s` 顯示佔用的磁碟空間


- `run` 可加上 `--name 名稱` 自訂容器名稱，方便之後操作，省略的話，docker 會自動給一個名稱
- `run`  可加上 `-d` 使用 Detach 方式(在背景)執行。省略的話，則在前景執行，你會看到 nginx 的 access log
- `run` 的參數 `-p` 可以指定 IP 位址。`-p 192.168.1.1:80:80 -d nginx`。上面表格的 `-p 8080:80` 指定的 IP 位址是 0.0.0.0，所以如要限制只有本機可以連線，要用 `-p 127.0.0.1:8080:80`
- `run` 可加上 `-e 變數名稱=值` 作爲新增容器時的初始設定值，例如 `docker run --name my_mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3307:3306 -d mysql` 設定 MySQL root 的密碼是123456
- `run` 可加上 `-v` 外掛儲存空間
    - 可以是主機目錄：`-v /home/tom/:/usr/share/nginx/html` 把 /home/tom 目錄掛載到容器內的 /usr/share/nginx/html
    - 也可以是 Docker Data Volume：
- `run` 可以加上 `--restart` 自動啓動，有三種
    - `--restart=always`：系統啓動 Docker 時，自動啓動容器，所以 web server 程式等可以用它來自動啓動
    - `--restart=unless-stopped`：容器在 Exited(0) 時自動啓動，但系統啓動 Docker 時不會啓動
    - `--restart=on-failure:5`：容器中斷時啓動，最多嘗試 5 次

- `create` 和 `run` 的可用參數一樣，只是 `create` 只有新增容器，沒有執行
- `inspect` 加上參數 `--format='{{json .State.Status}}'` 則會只顯示 State 中的 Status 欄位值
- `cp` 後的第一個參數複製到第二個參數，所以只要把參數位置對調，就是複製容器內的檔案到外面
- `stats` 如果沒有參數，會像 `top` 一樣持續更新，使用 `Ctrl + C` 離開。`-a` 顯示運作中和沒有運作的容器
- `logs` 可用參數 `--tail` 只顯示最尾端的記錄，`--since 2m` 顯示 2 分鐘之前的記錄，`-f` 持續監看記錄
- `exec` 可用 `-i` 進入互動模式，`-t` 顯示執行結果，通常會合併成 `-it`

### Docker 網路

容器之間的內部網路

``` bash
# 新增網路
docker network create 網路名稱
# 啓動容器並加入網路
docker run --name --net=網路名稱
# 刪除網路，要先確定加入網路的容器已停止
# 刪除網路後，加入此網路的容器會因爲找不到網路而無法啓動
# 需要重新加入同名的網路才行
docker network rm 網路名稱
# 顯示網路資訊，可在 Containers 查到加入此網路的容器 ID
docker network inspect 網路名稱
```

### Docker Data Volume

建立一個資料夾，給多個容器共用，存放資料，參考[Use volumes](https://docs.docker.com/storage/volumes/)

## Docker Compose

