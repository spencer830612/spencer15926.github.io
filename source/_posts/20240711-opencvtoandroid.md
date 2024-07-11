---
title: 把 OpenCv 匯入至 Android 專案的過程與使用事項
categories:
  - Academic
tags:
  - Android
  - OpenCv
abbrlink: 209776317
date: 2024-07-11 10:43:13
---

目前考慮要在 Android 專案匯入 OpenCv，所以這篇文章在記錄這個過程。

<!-- more -->

## 前言

目前是想要做在 Android 手機上的影像辨識。前面的文章有提出幾條路，後來我發現：在 Python 上執行的程式碼其實都是 OpenCv on Pytohn 的。接著我又發現 OpenCv 上其實有 OpenCv4Android 的專案，那代表：我其實把一樣的過程模仿至 Android 上，就不用處理在 Android 上執行 Python 的效能問題了！

## 作法

### 環境

- Android Studio Koala
- build.gradle in Kotlin
- OpenCV 4.10

### 下載

首先去 OpenCv 的 [GitHub 頁面](<https://github.com/opencv/opencv/releases>) 去下載他的 SDK。
![Image](https://i.imgur.com/w9R8VhG.png)
當然你也可以依據需求下載別的版本。

### 解壓縮與探勘

解壓縮後會看到兩個資料夾名為 `sdk` 與 `samples`，內容與下圖對應於左邊與右邊。左邊是他的原始檔案，後續會教如何匯入；右邊是他的示範專案，可以看看他是如何應用在 Android Project 裡面。

![Image](https://i.imgur.com/XpML3ZQ.png)

這是我少數一開啟就能執行的示範專案，也覺得他裡面管理 flavors 的 gradle 寫得很詳細，也是一個學習的好專案。

### 把 SDK 匯入至現有專案

1. 匯入 SDK
![Image](https://i.imgur.com/syCkIjE.png)

2. 找到你剛剛下載的地方，選取 sdk
![Image](https://i.imgur.com/3aCqSSj.png)

3. 選好後，要為這個 SDK 命名。因為我已經先匯入了，他會提醒我名字重複；
![Image](https://i.imgur.com/YL1l0WN.png)

4. 按下 Finish

### 調整匯入的 SDK 的 Gradle

點進去匯入的 SDK 的 Gradle。這裡是命名成 OpenCV，Android Studio 也會提醒你。
![Image](https://i.imgur.com/YZoggMy.png)

裡面有幾個需要更改的地方：
![Image](https://i.imgur.com/8WLNtuR.png)

1. 註解掉 `apply plugin: 'kotlin-android'`
2. 註解掉 `targetSdkVersion 33`
3. `compileSdk` 與 `minSdkVersion` 我是直接對齊我原本專案的版本
4. 我也有對齊 `compileOptions` 的 `JavaVersion`

### 調整原本的 Gradle

在 dependencies 區塊匯入 OpenCV 的 SDK

```Kotlin
dependencies {
    implementation (project(":OpenCV"))
}
```

### 編譯與執行測試

在使用 OpenCV 的功能前，一定要先初始化：

```Kotlin
if (OpenCVLoader.initLocal()) {
    Log.i("TAG", "OpenCV loaded successfully")
} else {
    Log.e("TAG", "OpenCV initialization failed!")
}
```

再經過一分多鐘的編譯，成功！

## 結語

OpenCV 的匯入感覺是已經算比較容易的。如果你打包 APK 時想縮減一些容量，可以看看要不要去除 javadoc 與 licenses 的內容。前者是說明文件，後者是引用開源套件的文件說明。
