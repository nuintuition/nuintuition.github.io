---
layout: post
title: "在 APIM 中設置 API 參數表格的解決方案"
date: 2025-06-12
categories: [Azure, APIM, JSON Schema]
tags: [azure, apim, json-schema, table, html, api-documentation]
---

在 APIM 中設置 API 參數時，這個參數是以 Json Schema 型式紀錄，因此我想在裡面放入表格時，發生了一些困難，在測試各種狀況後在這邊分享一下，目前測試的結果。

## 資料型態

由於 APIM 的 API 說明欄位使用 markdown 撰寫是可以顯示內容的，因此我也使用 markdown 格式的 table 撰寫，這時才發現 table 是無法正確顯示的。之後我查詢了一下官方說明，我翻了很多個地方，不知道是我漏掉還是如何，我一直沒看到要如何正確加入 table，後來只好試試基本的 html 語法，發現是有支援的，才因此解決。

- **Json Schema 加入表格**
  - 要在 Json Schema 中加入表格基本上就按照 Json Schema 的基本格式，在內容的位置直接輸入 html `<table>` 的語法即可，只是要注意的是符號的部分記得做字串型式的轉換。
  - 範例如下:

```json
{
    "type": "object",
    "required": [
        "Test1",
        "Test2"
    ],
    "properties": {
        "Test1": {
            "type": "integer",
            "description": "描述1"
        },
        "Test2": {
            "type": "string",
            "description": "描述2"
        },       
        "ITEM": {
            "type": "object",
            "items": {
                "type": "object",
                "required": [
                    "Test3",
                    "Table1"
                ],
                "properties": {
                    "Test3": {
                        "type": "string",
                        "description": "描述3"
                    },
                    "Table1": {
                        "type": "string",
                        "description": "表格  \n &lt;table class=\"table\"&gt;&lt;thead&gt;&lt;tr&gt;&lt;th scope=\"col\"&gt;VALUE&lt;/th&gt;&lt;th scope=\"col\"&gt;STATUS&lt;/th&gt;&lt;th scope=\"col\"&gt;Meaning&lt;/th&gt;&lt;th scope=\"col\"&gt;Description&lt;/th&gt;&lt;/tr&gt;&lt;/thead&gt;&lt;tbody&gt;&lt;tr&gt;&lt;td&gt;0&lt;/td&gt;&lt;td&gt;T1&lt;/td&gt;&lt;td&gt;狀態1&lt;/td&gt;&lt;td&gt;說明1&lt;/td&gt;&lt;/tr&gt;&lt;tr&gt;&lt;td&gt;1&lt;/td&gt;&lt;td&gt;T2&lt;/td&gt;&lt;td&gt;狀態2&lt;/td&gt;&lt;td&gt;說明2&lt;/td&gt;&lt;/tr&gt;&lt;/tbody&gt;&lt;/table&gt; \n &lt;font color=red&gt;Status = \"0\" 為初始值&lt;/font&gt;"
                    }
                }
            }
        }
    }
}
```

- **資料內容設定完成後只要添加到 API 參數中的 Definition 中即可顯示**
  - 新增 API 參數說明

    ![新增 API 參數說明](/assets/images/azure/apim/add-api-parameter-description.png)

  - 顯示效果如下

    ![表格顯示效果](/assets/images/azure/apim/table-display-result.png)

## 參考資料

- [JSON Schema 物件參考文件](http://json-schema.org/understanding-json-schema/reference/object.html)
