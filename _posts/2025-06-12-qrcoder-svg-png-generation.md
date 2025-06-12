---
layout: post
title: "使用 QRCoder 在 .NET Core 中產生 SVG 和 PNG 格式的 QR Code"
date: 2025-06-12T11:01:00+08:00
categories: [.NET Core, QR Code, API]
tags: [qrcoder, qr-code, svg, png, asp.net-core, bitmap, error-correction]
---

剛好有需要使用到 SVG 檔的 QR Code，花了一點時間搜尋 .Net Core 的 QR Code 相關套件，找到了 QRCoder 可以符合這個需求，在這裡紀錄一下如何執行產出的方法。

## 用法

QRCoder 的使用方法蠻簡單的基本上只要五行就可以了，不過需要注意的是，如果要產生的是 SVG 那在建立時所使用到的物件有所不同，要調整一下作法才可以。基本範例可以透過[QRCoder](https://github.com/codebude/QRCoder) 查閱。

以下透過 API 的應用範例來示範，如何產生 QR Code 並回傳回去。

### PNG 格式

PNG 作法蠻簡單的只要先產生出 QR Code 的 Bitmap 格式，在轉型成想要的圖形格式即可。

```csharp
public IActionResult GetQRCodePNG(string text)
{
    var qrGenerator = new QRCodeGenerator();
    QRCodeData qrCodeData = qrGenerator.CreateQrCode(text, QRCodeGenerator.ECCLevel.H);
    var qrCode = new QRCode(qrCodeData);
    Bitmap qrCodeImage = qrCode.GetGraphic(20);
    using var stream = new MemoryStream();
    qrCodeImage.Save(stream, ImageFormat.Png);
    return File(stream.ToArray(), "image/jpeg");
}
```

### SVG 格式

SVG 作法與 PNG 差不多，不過不能像 PNG 一樣產生成 Bitmap 格式後在選擇要轉型的圖片格式，而是要使用 QRCoder 提供給 SVG 專用的 `SvgQRCode` 物件來生成 SVG 圖片，而且在產生成圖片時實際上會是紀錄成一個 SVG String，所以使用時要先把這個 String 轉換成 Byte Array，最後再把這個 Byte Array 轉成檔案。

```csharp
public IActionResult GetQRCodeSVG(string text)
{
    var qrGenerator = new QRCodeGenerator();
    QRCodeData qrCodeData = qrGenerator.CreateQrCode(text, QRCodeGenerator.ECCLevel.H);
    var svgQRCode = new SvgQRCode(qrCodeData);
    var qrCodeImage = svgQRCode.GetGraphic(20);
    byte[] byteArray = Encoding.UTF8.GetBytes(qrCodeImage);
    return File(byteArray, "image/svg+xml");
}
```

## 容錯率

QR Code 在使用時還有一個常常發生的情境，就是會在上面加入一些客製的圖片，而要加上圖片後又要能夠讀取的到，在這樣的應用上就是要調整 QR Code 的容錯率。QRCoder 也有提供這樣的功能，只要調整 `QRCodeGenerator.ECCLevel` 即可，分為 L、M、Q、H 四個等級，容錯幅度約在 7~30% 之間。

調整位置如下：

```csharp
QRCodeData qrCodeData = qrGenerator.CreateQrCode(text, QRCodeGenerator.ECCLevel.H);
```

## 總結

以上就是我簡單紀錄的 QRCoder 範例，QRCoder 可以產生 SVG 與 PNG 檔也能控制容錯率，這在基本的應用上可以符合大多數的需求，是一個蠻好用的套件。

## Source Code

- [GitHub 專案範例](https://github.com/nuspaceline/GenerateQRCode)

## 參考資料

- [QRCoder](https://github.com/codebude/QRCoder)
- [How to easily implement QRCoder in ASP.NET Core using C#](https://www.freecodecamp.org/news/how-to-easily-implement-qrcoder-in-asp-net-core-using-c-10c4aa857e84/)
