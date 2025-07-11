---
layout: post
title: "Visual Studio 常用快捷鍵與實用功能整理"
date: 2025-06-12T10:27:49+08:00
categories: [Visual Studio, IDE, 開發工具]
tags: [visual-studio, keyboard-shortcuts, 快捷鍵, 開發效率, ide]
---

我在這裡整理了一些我常用的快捷鍵與你可能忽略的一些方便特殊功能，如果你是不想改快捷鍵，又想盡量不使用到滑鼠的話，這些快捷鍵應該會對你有些幫助。

## 游標操作

### Ctrl + 上下左右
- **上下 + Ctrl**：會把游標鎖在原地然後畫面上下移動。
- **左右 + Ctrl**：則會一次跳動一個文字，這個在橫向移動的時候能夠快速不少，因此蠻推薦使用的。

另外非常推薦配合 **Shift** 鍵使用，在圈選文字上真的非常快速。

### Ctrl + G
可以快速跳往特定行，這個剛開始用可能不習慣，不過用習慣後一次到達，視覺比較可以不受到畫面游標跳動的影響，會感覺比較簡潔。

### Home 與 End
在處理游標橫向移動上，實在沒有太多快捷的方案，因此 **Home** 與 **End** 就成了重要的功能，當程式過長時我會用 **Home** 或 **End** 配合 **Ctrl + 左右** 來快速移動到要修改的位置。

## 操作功能

### Alt + Enter, Ctrl + .
這是個重構快捷鍵，如果你的程式碼有出現燈泡，或是底下有三個小點的提示，則可以使用這個快捷鍵來讓Visual Studio協助你重構程式碼。

**提示有可以修改的部分**

![重構提示](/assets/images/visual-studio/refactor-tip.png)

**Alt + Enter 查看建議項目**

![建議項目](/assets/images/visual-studio/alt-enter-suggestions.png)

**自動重構**

![自動重構](/assets/images/visual-studio/auto-refactor.png)

### F12、Ctrl + F12、Shift + F12
到了開發都以物件導向為主後，**F12** 就成了使用率非常高的快捷鍵，非常建議習慣使用他。

- **F12**：到達定義所在位置。
- **Ctrl + F12**：是到達實作的位置，他與F12的差別在於，如果你使用了interface，因為可能有好幾個實作的位置，使用Ctrl + F12則會顯示有多少個實作，讓你選擇你要前往的位置。
- **Shift + F12**：是列出所有參考位置，你實作的這個方法，有多少地方參考，用這個快捷鍵會將它們全部顯示出來。

### F5、Shift + F5
分別為開始執行與停止執行，這個使用上非常方便也很好記。

### F8、Shift + F8
分別為前往下一個錯誤，或是回到上一個錯誤，開發Debug時非常好用。

### Ctrl + K, Ctrl + D
這是一個組合鍵，用來重新編排程式碼，記得程式碼寫完都要重新排版，這讓自己與其他人都好閱讀。

### Ctrl + K, Ctrl + C
這是註解的組合鍵，執行會將程式註解。

### Ctrl + K, Ctrl + U
這是解除註解，主要是配合註解一起使用。

### Ctrl + D
這應該是大家比較少用到的快捷鍵，他會新增一行與現在相同的程式碼，他的方便性在於，如果我現在有複製了一段程式碼，但是又想利用某段程式碼修改，這時候就可以用 **Ctrl + D**，他並不會把之前複製的部分蓋掉。

### Ctrl + X
這是剪下的快捷鍵，我推薦的用法是當刪除使用，如果你沒選取任何文字，執行時會直接剪取一行，因此你就不用要按好幾個 **Delete** 來刪除了。

### Ctrl + M, Ctrl + M
這個組合鍵是摺疊與展開程式碼，當程式碼寫的很多時，你可能會想專注在特定段落，就可以把程式碼摺疊起來，這樣在移動游標時也會比較方便。

### F9
設置中斷點，這可以快速設置或取消一個中斷點，在Debug非常方便，通常會在Debug模式配合這 **F10**、**F5** 使用。

- **F10**：一行一行執行
- **F5**：跳往下一個中斷點

### Ctrl + Shift + F
這是全域搜尋，如果你要在整個專案搜尋，請一定要記起來，會常常用到。

### Ctrl + Delete
**Ctrl + Delete** 是一次往後刪除一個字，可以用來快速刪除文字，不過我這裡還會推薦一個用法，就是如果下方有一段程式碼是要接續到上面來的，這時候就可以在這行程式碼最後直接執行 **Ctrl + Delete** 這樣就可以快速把下面的程式碼接上來了。

## 總結

這些快捷鍵能夠大幅提升開發效率，減少對滑鼠的依賴。建議從最常用的幾個開始練習，逐漸培養使用習慣，相信會對你的開發工作有很大的幫助。
