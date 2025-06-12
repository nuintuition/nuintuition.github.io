---
layout: post
title: "Visual Studio 開發與Debug實用工具介紹"
date: 2025-06-12T10:27:49+08:00
categories: [Visual Studio, IDE, Debug]
tags: [visual-studio, debug, watch, tabs, scrollbar, 開發工具]
---

Visual Studio 有提供我們很多能快速檢視程式碼及Debug的工具，我在這整理一些剛開始使用時就能感覺到方便開發的小功能，也建議可以自己探索一下，找出更多能加快開發的工作模式。

## Watch

Watch是提供我們Debug用的非常方便的小工具，你可以在這個地方預覽程式變數的內容，並且也可以做簡易的運算。

### 開啟方式

要在程式執行後到 **Debug → Windows → Watch → Watch1**，其中watch可以選擇不只一個。

![Watch開啟方式](/assets/images/visual-studio/debug-tools/watch-open.png)

### 變數的預覽與運算

![Watch變數預覽](/assets/images/visual-studio/debug-tools/watch-variables.png)

## 分頁釘選

我們可以把常用的分頁釘選起來，除了可以快速看到我們要的項目外，也可以方便清除已經沒再關注的分頁。

釘選分成獨立一排顯示與和非釘選放在同一排顯示，獨立顯示可以更容易注意到現在正在編輯的項目。

### 設定方式

修改位置可以到 **Tools → Options → Environment → Tabs and Windows → Pinned Tabs** 修改。

![分頁釘選設定](/assets/images/visual-studio/debug-tools/pinned-tabs-settings.png)

![分頁釘選選項](/assets/images/visual-studio/debug-tools/pinned-tabs-options.png)

### 顯示效果

**單排顯示釘選**

![單排釘選顯示](/assets/images/visual-studio/debug-tools/single-row-pinned.png)

**獨立顯示釘選**

![獨立釘選顯示](/assets/images/visual-studio/debug-tools/separate-row-pinned.png)

### 快速清除釘選以外的項目

![清除非釘選分頁](/assets/images/visual-studio/debug-tools/close-unpinned-tabs.png)

## 捲軸

捲軸有提供一般與可預覽程式的模式，可預覽程式的模式除了可以快速看程式內容外，如果你有設書籤或是中斷點，上面也會有一個小點作為提示，你可以快速找到你所需要的程式碼。

### 設置位置

**Tools → Options → Text Editor → Scroll Bars → Behavior**

![捲軸設定](/assets/images/visual-studio/debug-tools/scrollbar-settings.png)

### 快速預覽程式碼及找尋中斷點位置

![捲軸預覽功能](/assets/images/visual-studio/debug-tools/scrollbar-preview.png)

## 總結

這些Visual Studio的內建工具能夠大幅提升開發和除錯的效率：

- **Watch**：讓你在Debug時能夠即時監控變數狀態
- **分頁釘選**：幫助管理多個檔案，保持工作流程的整潔
- **捲軸預覽**：快速導航和定位程式碼位置

善用這些工具可以讓你的開發工作更加順暢，建議多多探索Visual Studio的其他功能，找出適合自己的工作模式。
