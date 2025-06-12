---
layout: post
title: "在 Azure 上部署 SPA 網站爬蟲 API 的解決方案"
date: 2025-06-12
categories: [Azure, Node.js, Puppeteer]
tags: [azure, spa, crawler, api, puppeteer, headless-chrome, deployment]
---

你如果想要在 Azure 上部署一個能讀取 SPA 網站的爬蟲 API 時，沒特別做任何額外設定，你會發現怎樣都無法執行，這是因為一般的部署方案，是不會幫你下載 headless Chromium，另外如果使用 docker 部署也是會有一樣的結果，docker 本身也不會幫你下載。

目前測試了幾個方法，如果要能夠成功執行，比較簡單的方案，需要在幾個限定的環境下比較容易處理。

## 執行環境

1. **node.js**
   - 會使用 node.js 主要是因為 puppeteer 是 node.js 的套件，也可以確保 headless Chromium 是可以執行得起來的。

2. **Linux**
   - 選用 Linux 主要就是要確保 headless Chromium 可以執行，如果是使用 Windows，目前實驗起來 headless Chromium 是跑不起來的。

3. **Azure 方案類型**
   - 目前測試下來至少要選擇應用程式服務方案然後是 B1 等級，雖然在查相關文獻的時候有發現，有人是可以使用使用量(無伺服器)的類型，但在實驗時不知是資源不足還是什麼原因無法成功啟動

![Azure 應用程式服務方案](/assets/images/azure/azure-app-service-plan.png)

## 專案設定

1. **專案建置這方面可以透過 Azure CLI 或是 VS Code 的 Azure 套件來建置這邊使用 VS Code 作為範例**
   - **建立專案**
