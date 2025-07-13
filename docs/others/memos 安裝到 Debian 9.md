---
hide:
  - toc
---

# memos 安裝到 Debian 9

[memos](https://www.usememos.com/) 官網文件只有寫怎麼用 Docker 安裝，但是產生的 SQlite 檔案是 root 權限，所以試著編譯原始碼產生執行檔

``` bash
# 安裝 Node.js
sudo apt install curl build-essential
curl -sL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt install nodejs
node -v
npm -v

cd /tmp
git clone https://github.com/usememos/memos.git
cd memos/web
sudo corepack enable
```

輸入 Y 和 Enter 安裝 pnpm 和 package.json 所列套件

``` bash
pnpm i --frozen-lockfile
```

編輯 `package.json`，加上 `NODE_OPTIONS='–max_old_space_size=1024'`
設定使用 1 GB，否則 pnpm build 會丟出 GC exception。假設主機是 2GB VM

``` javascript
"scripts": {
  "build": "NODE_OPTIONS='–max_old_space_size=1024' tsc && vite build"
}
```

然後執行 `pnpm build`

參考

- [Memos Dockerfile 設定](https://github.com/usememos/memos/blob/main/Dockerfile)
- https://nodejs.org/api/cli.html#cli_max_old_space_size_size_in_megabytes

複製 docker 容器內的 memos 執行檔到 host 端執行

新增檔案 docker-compose.yaml

``` yaml
services:
  memos:
    container_name: memos
    image: ghcr.io/usememos/memos:0.21.0
    ports:
      - 127.0.0.1:5230:5230
    restart: always
    volumes:
      - ./data/:/var/opt/memos
```

執行以下指令

``` bash
docker compose up -d
docker compose exec memos /bin/sh
cp -r dist/ /var/opt/memos
cp memos /var/opt/memos
exit
docker compose down -v
docker image rm ghcr.io/usememos/memos:0.21.0

# 假設目前使用者是 tom
sudo chown tom:tom -R *
```

執行 `./memos`，用瀏覽器瀏覽 http://127.0.0.1:8081 ，應該能看到 Memos 執行

### 開機執行 memos

建立一個新的 Systemd 服務單位設定檔，儲存於 `/etc/systemd/system/memos.service`（需要 `sudo`）

```
[Unit]
Description=Memos

[Service]
Type=simple
WorkingDirectory=/home/tom/Dropbox/apps/memos/
ExecStart=/home/tom/Dropbox/apps/memos/memos
Restart=always
User=tom
Group=tom

[Install]
WantedBy=multi-user.target
```

執行以下指令

``` bash
# 資料儲存在我的 Dropbox
chmod +x /home/tom/Dropbox/apps/memos/memos
sudo chmod 644 /etc/systemd/system/memos.service
sudo systemctl daemon-reload
sudo systemctl start memos
systemctl status memos
sudo systemctl stop memos
sudo systemctl enable memos
sudo systemctl disable memos
```

參考

- [Linux 建立自訂 Systemd 服務教學與範例](https://blog.gtwang.org/linux/linux-create-systemd-service-unit-for-python-echo-server-tutorial-examples/)
