---
layout: post
title: "Docker 中使用 Express 執行 Puppeteer 的解決方案"
date: 2025-06-12T10:55:00+08:00
categories: [Docker, Node.js, Puppeteer]
tags: [docker, express, puppeteer, linux, chrome, sandbox, dockerfile]
---

Docker 想使用 Express 來執行 Puppeteer 時我遇到了幾個問題，我在這紀錄一下如何一步一步建構專案並說明會發生什麼問題及如何解決。

## 專案設置

### 建立新專案

使用指令安裝並建立一個基本專案

```bash
npx express-generator
```

### 設置 API 執行 Puppeteer

這邊我修改了 Express 的 users 這支 API 來執行 puppeteer 的功能，簡單建立取得網站的 title 來測試用。

這邊要注意的是如果你是在 linux 環境下執行會有 sandbox 設定的問題，因為 chromium 在執行時會在 sandbox 中執行，如果無法在 sandbox 執行則會發生 crash，這部分需要另行設定，如果你要取得的網站是可以信任的話，可以使用 `args: --no-sandbox` 設定即可。

```javascript
var express = require('express');
var router = express.Router();
const puppeteer = require('puppeteer');

/* GET users listing. */
router.get('/', async function (req, res, next) {

  const browser = await puppeteer.launch({
    args: ['--no-sandbox', '--disable-setuid-sandbox'],
  });
  const page = await browser.newPage();
  await page.goto('https://example.com');
  let htmlTitle = await page.title();
  await browser.close();
  res.send('respond with a resource :' + htmlTitle);
});

module.exports = router;
```

## Docker 設置

### Dockerfile

我使用的 base image 是 `node:latest`，由於它執行的 OS 是 linux，而 Chrome 無法直接在 linux 上執行，會缺少很多執行套件，因此在設定上需要額外下載，不過這部分官方有提供下載的指令，所以根據官方提供的方案可以解決這個問題。

```dockerfile
FROM node:latest

COPY . /workspace
WORKDIR /workspace

## 下載Linux缺少的chrome執行套件
RUN apt-get update \
    && apt-get install -y wget gnupg \
    && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
    && apt-get update \
    && apt-get install -y google-chrome-stable fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-kacst fonts-freefont-ttf libxss1 \
      --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

RUN npm install

EXPOSE 3000

CMD npm start
```

### .dockerignore

建議設定一個 dockerignore 檔案避免將 node_modules 一起帶到 image 中，因為如果不小心帶到，docker build 在執行 `npm install` 時 puppeteer 的設置是會有問題的，你需要移除再重裝，因此記得設定忽略掉不必要帶到 image 的檔案

```
node_modules
npm-debug.log
```

## 總結

docker 要執行 puppeteer 主要會發生的問題都是因為 chrome 要在 linux 上執行是無法直接執行的，要做不少額外設置，只要確保專案中 puppeteer 套件正確安裝，並將 linux 作業系統要執行 chrome 的相關套件下載完成，最後設置好 sandbox，就能正常執行 puppeteer 了。

## Source code

- [GitHub 專案範例](https://github.com/nuspaceline/testexpresspuppeteer)

## 參考資料

- [Puppeteer 疑難排解官方文件](https://github.com/puppeteer/puppeteer/blob/main/docs/troubleshooting.md)
- [Stack Overflow - Puppeteer error chromium revision is not downloaded](https://stackoverflow.com/questions/53997175/puppeteer-error-chromium-revision-is-not-downloaded)
- [CSDN - Docker 中使用 Puppeteer 相關問題](https://blog.csdn.net/a1368783069/article/details/87262041)
