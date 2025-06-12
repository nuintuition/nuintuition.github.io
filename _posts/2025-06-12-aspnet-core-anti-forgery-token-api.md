---
layout: post
title: "ASP.NET Core MVC 中API 調用的Anti-Forgery Token 驗證"
date: 2025-06-12
categories: [ASP.NET Core, Security, CSRF]
tags: [asp.net-core, mvc, anti-forgery, csrf, api, ajax]
---


如果你是使用ASP.NET Core MVC當作前端框架，因為為了防範CSRF攻擊，你在執行API時沒有夾帶驗證的token那anti-forgery驗證是會把你擋下來的。為了解決這個問題，你需要取得這個token並夾帶在API中來完成驗證。以下我會使用幾個作法來達到使用API通過驗證。

## Anti-Forgery驗證的做法

首先我們要先了解anti-forgery驗證是如何運作的，這個驗證的作法也很簡單，ASP.NET Core MVC如果你在前端有寫入`@Html.AntiForgeryToken()`的程式碼，則會在HTML夾帶一個隱藏的程式碼：

```html
<input name="__RequestVerificationToken" type="hidden" value="CfDJ8NrAkS...s2-m9Yw">
```

當表單要發送的時候，會去取得裡面的value夾帶在封包中發送，我們知道這個原理後就可以效仿相同的機制來處理達到驗證的效果。

## 使用Form資料型態API

如果要使用Form來當資料型態傳送，要將token包在Form之中來傳送才能通過驗證，範例如下。另外一點提醒如果你是直接用已經存在的form來傳送，只要記得加`@Html.AntiForgeryToken()`，那form裡面就已經有token了，就不需要額外夾帶。

```javascript
function test() {
    var token = $("input[name='__RequestVerificationToken']").val();
    let input = {};
    input["test"] = "testdata";
    input["__RequestVerificationToken"] = token;
    let postData = $.param(input);
    $.ajax({
        type: "POST",
        url: `/Home/PostApi`,
        data: postData,
        success: function (result) { }
    });
}
```

## multipart/form-data

`multipart/form-data`是使用header在驗證，因此需要在API request中加上header才能通過驗證，範例如下：

```javascript
function test() {
    var form = $("#TestForm"); // multipart/form-data是要傳FormData格式，所以你如果要用既有的Form來用是要轉型的，如下方Data中的資料。
    var token = $("input[name='__RequestVerificationToken']").val();
    $.ajax({
        type: "POST",
        url: "/Home/PostApi",
        headers: { "RequestVerificationToken": token },
        data: new FormData(form[0]),
        mimeType: 'multipart/form-data',
        contentType: false,
        processData: false,
        dataType: "json",
        success: function (data) { }
    });
}
```

## JSON

JSON也是在header上驗證，因此需要將token放在header上驗證，範例如下：

```javascript
function test() {
    var token = $("input[name='__RequestVerificationToken']").val();
    let postData = { test: "TestData" };
    let jsonData = JSON.stringify(postData);
    $.ajax({
        type: "POST",
        url: `/Home/PostApi`,
        headers: { "RequestVerificationToken": token },
        dataType: "json",
        contentType: "application/json; charset=utf-8",
        data: jsonData,
        success: function (result) { }
    });
}
```

## 參考資料

- [ASP.NET Core 中的防偽要求功能](https://docs.microsoft.com/zh-tw/aspnet/core/security/anti-request-forgery?view=aspnetcore-6.0)
- [Sending an Anti-Forgery Token with ASP.NET Core MVC AJAX Requests](https://plbonneville.com/blog/sending-an-anti-forgery-token-with-asp.net-core-mvc-ajax-requests/)
