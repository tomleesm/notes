## 安裝

本站使用 [Material for Mkdocs](https://squidfunk.github.io/mkdocs-material/)。[Mkdocs](https://www.mkdocs.org/) 是用 Python 寫的網站產生器，所以先安裝 Python3 和 pip

``` bash
sudo apt install python3 python3-pip
```

然後用 pip 安裝佈景主題 [Material for Mkdocs](https://squidfunk.github.io/mkdocs-material/)
```
pip install mkdocs-material
```

## 指令

* `mkdocs new [dir-name]` - 新增專案
* `mkdocs serve` - 預覽網站
* `mkdocs build` - 產生網站的 HTML
* `mkdocs -h` - 顯示說明

## 網站結構

所有設定和文件導覽結構都放在 `mkdocs.yml`，markdown 檔案放在目錄 docs

使用 GitHub Actions 部署到 GitHub Pages

新增檔案 .github/workflows/deploy.yml，輸入以下設定：
``` yaml
name: deploy
on:
  push:
    branches:
      - master
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
        with:
          python-version: 3.x
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```

## 設定


