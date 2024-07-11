---
title: 有可能利用 MATLAB 生成 C++ 程式碼給 Android 用嗎？
tags:
  - Image process
  - NDK
  - MATLAB
categories:
  - Academic
abbrlink: 2876724007
date: 2024-07-04 16:54:07
---

關於對 MATLAB 不太熟，以及對 MATLAB 這條路感到幻滅的嘗試過程。

<!-- more -->

## 前言

前面有在研究 Python 做影像前置處理的部分。但我突然想到：MATLAB 似乎有把程式碼轉成 C++ 的功能，那我是否可以先用 MATLAB 的 ImageProcess Tool 先自己進行一遍提取所需區域的內容，再把他生成的程式碼轉換成 C++ 丟到 Android 上執行，可以這樣嗎？

### 前提

因為 Android Studio 其實有開發 NDK 的功能，也就是可以在上面執行 C 或 C++ 語言的功能，因此我才會考慮用這種方式轉換 MATLAB 的程式碼。一方面是 Android 上不太可能能跑 Matlab 的程式碼，另一方面是我認為 C++ 語言執行的效率夠快，或許更適合處理影像的場景

## 嘗試過程

其實也蠻順利的，因為你在學生時代申請的學術帳號還可以繼續用，我就可以再下載到公司的電腦用，等到不用時再記得移除即可。

不過在嘗試輸出時就發現問題了！

### 什麼是 Entry-Point Functions？

首先我上次寫 C 語言已經是八年前的事情，我剩下的印象就是「恐怖」、「指標很可怕」。不過 MATLAB 的好處，就是有提供詳細的說明，大家可以去看看。

連結：[https://www.mathworks.com/help/rtw/ug/configure-c-code-generation-for-model-entry-point-functions.html](<https://www.mathworks.com/help/rtw/ug/configure-c-code-generation-for-model-entry-point-functions.html>)

### 決定輸入參數的資料型態

在 MATLAB 裡，這個函式吃進去的參數是 RGB 色版的陣列，所以應該是類似於 3 維陣列的 tensor 形式。但是呢......

![Image](https://i.imgur.com/YiZEdYw.png)
![Image](https://i.imgur.com/QlVini6.png)

他只有 Matrix 形式耶……那我就會被卡在這裡了。好就算我這裡先用成 2 維型式，但接下來還是有別的事情。

### 選擇編譯的 compiler

首先會要你選擇你是要在哪類裝置上進行

![Image](https://i.imgur.com/fjfQEWI.png)

如果選擇 None 的話，接下來會繼續選裝置內容

![Image](https://i.imgur.com/Qdu45Qe.png)

以及選擇你要的 Compiler，這可能問題相對沒那麼大。

![Image](https://i.imgur.com/XvjXhhu.png)

這邊我根本不知道怎麼選，於是最後編譯時就過不去了。

### 反思

這邊非常吃你對於硬體的理解。那我覺得麻煩的是：如果是跟硬體有關係，那麼我想放到手機上，手機我記得是 arm 吧？可是會不會不同手機會有不同的差異呢？會不會中國那邊習慣用的手機又是不一樣的？然後 ImageProcess Tool 又是 MATLAB 獨有的工具，如果程式碼裡有用到他的函式，那有可能會全部輸出讓我們用嗎？可惜後者無法驗證，因為光是前者就過不去了，唉唉。
