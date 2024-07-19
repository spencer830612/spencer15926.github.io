---
title: ExceptionInInitializerError 之怪事件
categories:
  - Academic
tags:
  - Android
  - Exception
  - ExceptionInInitializerError
  - Static
abbrlink: 105329673
date: 2024-07-17 17:55:28
updated: 2024-07-17 17:59:28
---

第一次看過 `ExceptionInInitializerError`，好好的記錄下來。

<!-- more -->

## 前言

因為原本公司的 APP 有太多的缺陷，例如太多的靜態變數，導致結束 APP 後常常沒有好好的釋放，因此現在就需要補上許多措施，例如在 onDestroy() 階段手動釋放這些靜態變數。

## 過程

此 APP 有個類別 `BBB`，裡面放了一堆靜態的 view，因此我打算去釋放他們，因此我在 `BBB` 裡面創一個靜態的函式 `clear()` 函式去釋放他們。

```java
public static void clear(){
    LockCheckBox = null;
    connectCheckBox = null;
    deviceName = null;
    serviceMessenger = null;
}
```

當下測試完後發現有釋放成功，所以就去搞別項。結果別項在測試時，就看到了此錯誤：

```logcat
FATAL EXCEPTION: Timer-6 (Ask Gemini)
Process: com.XXXXX.OOOOOO, PID: 3391
java.lang.ExceptionInInitializerError
    at XXXXX.com.mainpage.ZZZZZZZZ.destroyAsDepart(ZZZZZZZZ.java:874)
    at XXXXX.com.mainpage.ZZZZZZZZ.-$$Nest$mdestroyAsDepart(Unknown Source:0)
    at XXXXX.com.mainpage.ZZZZZZZZ$3.run(ZZZZZZZZ.java:841)
    at java.util.TimerThread.mainLoop(Timer.java:563)
    at java.util.TimerThread.run(Timer.java:513)
Caused by: java.lang.RuntimeException: Can't create handler inside thread Thread[Timer-6,5,main] that has not called Looper.prepare()
    at android.os.Handler.<init>(Handler.java:227)
    at android.os.Handler.<init>(Handler.java:129)
    at XXXXX.pub.BBB$1.<init>(BBB.java:176)
    at XXXXX.pub.BBB.<clinit>(BBB.java:176)
    at XXXXX.com.mainpage.ZZZZZZZZ.destroyAsDepart(ZZZZZZZZ.java:874) 
    at XXXXX.com.mainpage.ZZZZZZZZ.-$$Nest$mdestroyAsDepart(Unknown Source:0) 
    at XXXXX.com.mainpage.ZZZZZZZZ$3.run(ZZZZZZZZ.java:841) 
    at java.util.TimerThread.mainLoop(Timer.java:563) 
    at java.util.TimerThread.run(Timer.java:513) 
```

我就覺得非常奇怪，為什麼會出現此錯誤？而且測試當下沒錯，測試別項後就出錯，但是因為是錯在結束程式後，因此也不確定會造成什麼巨大的影響。

然後反覆測試後，發現：當有執行過 `BBB` 後，就不會出現此錯誤；如果沒有執行過，那麼就會出現此錯誤。

我後來查到了 [https://blog.csdn.net/xie_xiansheng/article/details/50831623](<https://blog.csdn.net/xie_xiansheng/article/details/50831623>)，裡面有提到：

1. 靜態類別也有載入順序問題：以本次為例，可能是因為已進入 onDestroy 階段，又再進入 `clear()`，因此開始初始化 `BBB`，結果錯亂之下，裡面的 Handler 物件就出錯。
2. 主要造成的錯誤應該是 `RuntimeException`，但因為跟靜態類別的初始化有關，因此最後拋出的是 `ExceptionInInitializerError`。

我考慮過後，覺得既然這個錯誤只會發生在程式結束時，因此應當是不用處理 `BBB` 是否還是要先初始化的問題，因此我就用 try-catch 解決。

```java
try {
    BBB.clear();
} catch (ExceptionInInitializerError ignore) {
    // 在還沒執行到 BBB 時，不會載入任何 BBB 的靜態部分。因此沒進入該類別後退出 APP
    // 時，會跑出 RuntimeException，最後成為 ExceptionInInitializerError。
    // ignore.printStackTrace();
}
```

## 結語

這份程式碼原本的人就是把所有的變數與函式都用成靜態，不管有沒有需要公開，一律設置成 `public static`，所以導致大部分的程式碼引用都是建立在 `static` 隨處可引用的概念，因此結束時就需要多花心力去釋放這些資源。
