---
layout: post
title: "使用 Fetch API 下載檔案的完整解決方案"
date: 2025-06-12T10:59:00+08:00
categories: [Web API, JavaScript, ASP.NET Core]
tags: [fetch, download, file, api, blob, objecturl, content-disposition, asp.net-core]
---

一般在下載檔案時都會使用 html tag `<a>` 來執行，但有很多情況需要控制執行流程，像是下載檔案前後的執行動作，因此需要透過 api 來執行，在這邊會透過 fetch 來執行 api 並把回傳資料轉成檔案格式。

## Web API

Web API 我是使用 ASP.NET Core 的 API，將本地端的檔案回傳給 client。

```csharp
public IActionResult Export()
{
    string fileName = "testfile.csv";
    var testfile = System.IO.File.ReadAllBytes($"./{fileName}");
    var file = File(testfile, "text/csv", fileName);
    return file;
}
```

## Download File

前端在下載資料時是使用 fetch 來執行 API，回傳值的處理上，是透過將資料先轉型為 blob，接著再把 blob 轉成 ObjectURL，有了 ObjectURL 後就可以再透過自行建立 html tag `<a>` 來將檔案下載下來，這樣我們就可以自行控制執行流程，另外下載後的檔案名稱我們也需要自行處理，檔案名稱目前主要的通訊協定是會放在 header: Content-Disposition 來傳送，因此我們可以從 header 解析出來，並在下載時使用。

```javascript
function downloadFile(){
     fetch("./Home/Export")
    .then(response =>{
        response.blob().then( b =>{
            let file = window.URL.createObjectURL(b);
            let a = document.createElement('a');
            a.href = file;
            let filename = getfilename(response);
            a.download = filename;
            a.click();
        });
    });
}

function getfilename(res){
    var disposition = res.headers.get('Content-Disposition');
    if (disposition && disposition.indexOf('attachment') !== -1) {
        var filenameRegex = /filename[^;=\n]*=((['"]).*?\2|[^;\n]*)/;
        var matches = filenameRegex.exec(disposition);
        if (matches != null && matches[1]) {
            return filename = matches[1].replace(/['"]/g, '');
        }
    }
}
```

## 結論

要達到使用 API 來下載檔案，需要處理回傳值轉型成檔案，以及檔案名稱的部分，而這些部分都可以透過既有的方法 `createObjectURL` 配合 html tag `<a>` 與 headers: Content-Disposition 來完成即可。

## Source Code

- [GitHub 專案範例](https://github.com/nuspaceline/downloadfile)

## 參考資料

- [How can I download a file using window.fetch](https://newbedev.com/how-can-i-download-a-file-using-window-fetch)
- [How to get file name from content-disposition](https://stackoverflow.com/questions/40939380/how-to-get-file-name-from-content-disposition)
- [HTML5 神奇的 Object URL：不用後端，前端便能產生獲取指定物件的網址](https://medium.com/@kf99916/html5-%E7%A5%9E%E5%A5%87%E7%9A%84-object-url-%E4%B8%8D%E7%94%A8%E5%BE%8C%E7%AB%AF-%E5%89%8D%E7%AB%AF%E4%BE%BF%E8%83%BD%E7%94%A2%E7%94%9F%E7%8D%B2%E5%8F%96%E6%8C%87%E5%AE%9A%E7%89%A9%E4%BB%B6%E7%9A%84%E7%B6%B2%E5%9D%80-6df283d58505)
