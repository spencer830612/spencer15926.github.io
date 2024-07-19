---
title: 沒有崩潰記錄，可是 APP 還是被終止
categories:
  - Academic
tags:
  - Android
  - WakeLock
  - No Exception
abbrlink: 733064438
date: 2024-07-18 17:31:14
updated: 2024-07-18 17:31:14
---

沒有輸出錯誤訊息，也看不到崩潰記錄，阿我是要怎麼辦？

<!-- more -->

## 前言

目前 APP 發現會有閃退的情況，但是遇到很麻煩的事情：Crashlytics 無回報，也就是在回報區都看不到任何記錄，所以不知道錯誤的內容，因此我就去掉出手機本地的 dump 記錄檔。

但是內容非常多，而且搜尋 APP 的名字也沒有看到被終止的字眼，因此一籌莫展。

## 進展

因為真的快瘋掉，於是我就問 ChatGPT「有沒有可能 APP 沒有 crash，是因為系統的某些原因造成 APP 被結束，因此在 log 中找不到相關的 crash 記錄？」於是 GPT 回覆我：

>1. 過熱（Thermal Shutdown）：
>
>    - 系統會記錄溫度相關的信息，並可能顯示應用程式因過熱被強制終止。
>    - 搜尋 thermal、overheat、temperature 等關鍵字。
>
>2. 內存不足（Out of Memory, OOM）：
>
>    - 系統在內存不足時，會殺死低優先級的進程以釋放內存。可以檢查 ActivityManager 的記錄，特別是 ActivityManager: Killing 這類訊息。
>    - 搜尋 low memory、ActivityManager: Killing 等關鍵字。
>
>3. 電池管理（Battery Management）：
>
>    - 某些情況下，系統會因電池管理策略而終止應用程式。檢查電池管理相關的日誌。
>    - 搜尋 battery、power、thermal 等關鍵字。
>
>4. 其他系統原因：
>
>    - 應用程式被系統強制終止可能出現在其他情況下，如設備過載等。可以檢查系統資源的狀態。
>

死馬當活馬醫，於是我搜尋 `ActivityManager: Killing`，看到了：

```text
07-18 09:05:48.619  1000  1740  4365 I ActivityManager: Process com.OOO.XXX (pid 8631) has died: fg  TOP (1924,346)
```

於是 GPT 說：

>這條記錄表明應用程式進程 com.OOO.XXX 被系統終止，原因可能不是因為崩潰，而是由於其他系統級別的原因。從這條記錄看，它被標記為 fg TOP，表示它是前台的頂部應用程式，這通常不是由於內存不足引起的。

接著我就上網搜尋 `has died: fg TOP`，看到一個很棒的[網站](<https://blog.csdn.net/xiaoyantan/article/details/118489613>)，敘述 APP 可能會被哪些方式結束，而不太會有報錯通知。

突然看見一個[搜尋結果](<https://forum.cocos.org/t/topic/137554>)，裡面在詢問關於 WAKE_LOCK 權限導致 APP 常常在小米上被殺掉的問題。我就在我 APP 裡面搜尋。

果不其然，裡面有些地方被放置了 WakeLock，並且如果開啟藍牙，那麼就永遠跑不到關閉 WakeLock 的程式碼。但以目前這個 APP 的使用情形，關閉螢幕時完全不需要做事，那自然也不需要 WakeLock 功能了。

而且為了防止 APP 進入後台，我們在 `Activity.onCreated()` 就加上了

```java
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON, WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);

```

因此就不太需要用 WAKE_LOCK 了。

## 結語

其實要用 APP 開發一個永不關機的門禁系統，以 Android 來說是不太可行的。超過一個禮拜就會容易當機，但至少 APP 裡面不需要做的是情就不要去做，至少在新開機後的幾天內不要崩潰。

## 資料列表

1. [https://chatgpt.com/share/a8e98c7b-151a-4bcc-aadd-82fab2bb37950](<https://chatgpt.com/share/a8e98c7b-151a-4bcc-aadd-82fab2bb3795>)
2. [https://blog.csdn.net/xiaoyantan/article/details/118489613](<https://blog.csdn.net/xiaoyantan/article/details/118489613>)
3. [https://forum.cocos.org/t/topic/137554](< https://forum.cocos.org/t/topic/137554>)
4. [https://stackoverflow.com/questions/3723634/how-do-i-prevent-an-android-device-from-going-to-sleep-programmatically](<https://stackoverflow.com/questions/3723634/how-do-i-prevent-an-android-device-from-going-to-sleep-programmatically>)
