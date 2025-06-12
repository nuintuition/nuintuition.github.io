---
layout: post
title: "在 APIM 中自動新增 Azure 專案內的 API 設定方法"
date: 2025-06-12T10:34:51+08:00
categories: [Azure, APIM, API Management]
tags: [azure, apim, api-management, openapi, swagger, api-definition]
---

如果你想在 APIM 中自動新增 Azure 專案內的 API，你可能會漏掉幾個步驟，造成 APIM 找不到你專案中的 API，你可以參考看看以下方法。

## API 規格文件

你需要在專案中設定好 API 規格文件，這個規格文件要符合 OpenAPI 規範，你可以讓它自動產生或是自行建立，一般常見可以使用 Swagger，因為 APIM 是要讀取這些規格文件，才能知道有哪些 API。

## 設定 API 定義

除了設定 API 規格文件外還需要再 Azure 的 App 站台上設定 API 定義，這個是要給它檔案連結的，所以產出 API 規格文件後要把檔案連結設定上去，設定完成後 APIM 就可以使用 Azure resource 將站台上的 API 自動導入到 APIM 中。

API 定義設定位置:

![API 定義設定位置](/assets/images/azure/apim/api-definition-setting.png)

設定完後就可以使用 Azure resource 新增 API

![Azure resource 新增 API](/assets/images/azure/apim/azure-resource-add-api.png)
