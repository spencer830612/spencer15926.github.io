---
title: 嘗試能讓官方的 keras_ocr 能在我的 Android Studio Koala 跑起來
date: 2024-06-28 09:52:53
tags:
- Keras
- Ocr
- Android Studio Koala
- gradle
---

## 前情提要

在發現 ML KIT 可能沒辦法滿足我要對七段顯示器處理光學文字辨識後，我發現 TensorFlow 其實有給個範例，是關於 ocr 訓練出來的 tensorFlowLite 如何應用到手機上面。不過，當把專案匯入至我的 Android Studio 時，發現因為這個範例專案的 IDE 版本是 Android Studio 4.2，因此碰到許多問題。我會在下面 po 上我跟 ChatGpt 來回對話的記錄，並在這裡 po 上有更改的 code 以方便以後的我與過來人參考

這是我與 ChatGPT 的[對話](<https://chatgpt.com/share/a5dc6ce6-ea42-462c-9894-69ed70242feb>)，可以去看看。

---

### 更改檔案 1：app:build.gradle

主要的變更有

- 添加 namespace
- 手動設置 java 版本：因為我的 IDE 版本設置是 17，因此我就改成 17。
- 加上 viewBinding true：其實我覺得應該是不用，因為專案內的是用最基本的方式指派物件。

[內文比較的詳細連結](<https://www.diffchecker.com/dotkOibz/>)

---

### 更改檔案 2：gradle.properties

主要的變更是指定 java jdk 的位置。雖然 IDE 有指定，但我不知道為什麼這裡要再指定一次，編譯才會過。

[內文比較的詳細連結](<https://www.diffchecker.com/LkYRSIbs/>)

---

### 更改檔案 3：project: build.gradle

主要的變更有

- 變更 gradle 版本號：不同的 Android Studio 有不同的 gradle 版本指定，因此要改
- 變更 gradle-download-task：我就順勢升級了

[內文比較的詳細連結](<https://www.diffchecker.com/LVuvM6FT/>)

---

### 更改檔案 4：gradle-wrapper.properties

主要的變更是修改 gradle 的版號。

[內文比較的詳細連結](<https://www.diffchecker.com/icExyeaN/>)

大坑大雷，但最後我去更新所有 dependencies 的版本號，也沒有問題了！
