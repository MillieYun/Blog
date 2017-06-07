---
title: Hexo 常用指令
tags: hexo
date: 2017-05-23 12:17:32
---

參考來源:
[Hexo command](https://hexo.io/zh-tw/docs/commands.html)

## 網站

### 發佈或佈署網站
```cli
hexo deploy
hexo deploy --generate
```
-g, --generate	部署網站前先產生靜態檔案


### 產生靜態檔案
```cli
hexo generate
```

### 本機運行
```cli
hexo server
```
-p, --port	覆蓋連接埠設定
-s, --static	只使用靜態檔案
-l, --log	啟動記錄器，或覆蓋記錄格式

## 文章
參考來源:
[Hexo Writing](https://hexo.io/zh-tw/docs/writing.html)

### 新文章
```cli
hexo new [layout] <title>
```
`[layout]`: post(source/_posts), page(source), draft(source/_drafts)

### 完成草稿後，移動草稿到公開資料夾
```
hexo publish [layout] <title>
```