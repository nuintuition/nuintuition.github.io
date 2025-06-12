---
layout: post
title: "Azure App Service 中部署 Bitbucket 多站台方案的設定方法"
date: 2025-06-12T11:18:00+08:00
categories: [Azure, Bitbucket, Deployment]
tags: [azure, app-service, bitbucket, deployment, project-setting, multi-site]
---

在使用 Bitbucket 版控時，有可能會遇到在一個方案中存放多個站台，而 Azure App Service 只支援一個站台，這時候會需要到能夠指定部署特定站台的功能，要做到這點，可以透過 Azure 後台的應用程式設定來達到。

## 應用程式設定 (App Settings)

設定 App Settings 可以到 Azure 後台的組態 → 應用程式設定中設定，新增一個 key: `PROJECT` 與 value: 要部署的專案路徑，這個專案路徑需要指定到 csproj 檔。設定方法如下:

### 範例:

![Azure App Service PROJECT 設定範例](/assets/images/azure/app-service/project-setting-example.png)

只要透過設定 `PROJECT` 對應的路徑，就可以控制 Azure App Service 要跑哪個站台，不過在設定時記得要在專案 Build 之前設定好，這樣才會依照設定的站台來部署。

## 參考資料

- [Kudu - Customizing deployments](https://github.com/projectkudu/kudu/wiki/Customizing-deployments)
