---
title: 研究如何在 Android 跑 Python
tags: 
- Python
- Android
- Chaquopy
categories:
- Academic
abbrlink: 2386244310
date: 2024-07-05 09:33:21
mathjax: true
---

想在 Android 上面跑 Python 是滿奇特的，但是研究研究也無妨！

<!-- more -->

## 前言

我為了要實行影像辨識的方法，因此在網路上找了許多條路。不過影像辨識常常用 Python，很多教學也是使用 Python，我就想說：我可不可以在 Android 上面執行 Python 呢？因此我就尋找相關的套件，以及一些 Python 執行的注意事項。

## 相關套件 - Chaquopy

目前看到我比較滿意的是 Chaquopy，我覺得有幾點是滿不錯的;

1. 支援知名套件：目前看到他的[支援列表](<https://chaquo.com/pypi-13.1/>)，知名的 numpy、tensorFlow、matplotlib 等等的都有，也說支援超過 90% 以上 PyPl 所列出的套件。

2. 方便整合進去 AndroidL：一開始看到網路上的[採雷心得](<https://medium.com/@k1992313/python-for-android-%E8%B8%A9%E9%9B%B7%E5%BF%83%E5%BE%97-f07ac9c106ac>)，發現要整合是語言是很困難的事情。剛好 Chaquopy 幫我們解決這一哩路，當然就使用 Chaquopy 了。

## 開始安裝

參考資料：

- 官方網站：[https://chaquo.com/chaquopy/doc/current/android.html](<https://chaquo.com/chaquopy/doc/current/android.html>)
- 其餘資料：[https://hackmd.io/@kennethHung/S1mMZThps](<https://hackmd.io/@kennethHung/S1mMZThps>)

### 設定 Project 層級的 build.gradle

新增設定：

```groovy
plugins {
    id("com.chaquo.python") version "15.0.1" apply false
}
```

### 設定 Module 層級的 build.gradle

要在 plugins 地方新增：

```groovy
plugins {
    id ("com.chaquo.python")
}
```

然後你直接編譯後會看到：

```text
A problem occurred configuring project ':app'.
> Variant 'debug': Chaquopy requires ndk.abiFilters: you may want to add it to android.defaultConfig.
```

因此我們還要在 android -> defaultConfig 裡新增 ndk 選項：

```groovy
android {
    ......
    defaultConfig {
        ......
        ndk {
            abiFilters "arm64-v8a"
        }
    }
```

如果有在開發 Android APP，那麼會知道手機 CPU 的支援有分四種：

- armeabi-v7a, 32-bit arch，幾乎涵蓋所有android 裝置。
- arm64-v8a, 64-bit arch，涵蓋2018 後的大部分裝置。
- x86, Android 模擬器使用。
- x86_64, Android 模擬器使用。

要注意：支援的越多，Chaquopy 就會複製相對應的分數，會讓 APK 容量變大。

![Image](https://i.imgur.com/iIjaFWs.png)
可以看到：x86_64 主要是給虛擬器用的。

#### 安裝 Package

接著依據你需要的 package 安裝。除了連上 Chaquopy 的庫，也可以用 whl、requirement 安裝。

```groovy
chaquopy {
    defaultConfig {
        pip {
            // A requirement specifier, with or without a version number:
            install("scipy")
            install("requests==2.24.0")

            // An sdist or wheel filename, relative to the project directory:
            install("MyPackage-1.2.3-py2.py3-none-any.whl")

            // A directory containing a setup.py, relative to the project
            // directory (must contain at least one slash):
            install("./MyPackage")

            // "-r"` followed by a requirements filename, relative to the
            // project directory:
            install("-r", "requirements.txt")
        }
    }
}
```

例如我就這麼設定：

```groovy
python {
    pip {
        install "scipy"
        install "numpy"
        install "opencv-python"
    }
}
```

## 放入 Python code

Python code 放置在 `app\src\main\python`，於是我就用這段 code 做範例：

```python
import numpy as np

def numpy_example():
    a = np.random.rand(3,3)
    mean = np.mean(a)
    std = np.std(a)
    return mean, std
```

基本上返回的是 $2\times 1$ 的陣列，裡面的元素都是 `float` 型態。 接著我在 `MainActivity.kt` 這麼做：

```Kotlin
if(!Python.isStarted()) {
    Python.start(AndroidPlatform(this))
    // 一開始一定要初始化
}
val python = Python.getInstance()
val pyProject = python.getModule("test") // code 的檔案名字
val pyObj = pyProject["numpy_example"] // code 的 function 名字
val result = pyObj!!.call()
val mean: Float = result.asList()[0].toFloat()
val std: Float = result.asList()[1].toFloat()
Log.d("Example", "mean: $mean")
Log.d("Example", "std: $std")
val time2 = Instant.now().toEpochMilli()
Log.d("Example", "Go: $time2")
Log.d("Example", "Pass: ${time2 - time}")
```

看到結果了！

```text
Now: 1720161362600
mean: 0.5501784
std: 0.27493423
Go: 1720161363050
Pass: 450
```

不過我發現：當輸出的情況變複雜時，在 Kotlin 這裡的處理會愈複雜。如果我把 Python 那邊改成這樣：

```Python
import numpy as np

def numpy_example():
    a = np.random.rand(3,3)
    mean = np.mean(a)
    return mean, a
```

這時 Kotlin 這邊就要這樣處理

```Kotlin
val mean: Float = result.asList()[0].toFloat()
val a: Array<FloatArray> = result.asList()[1].asList()
    .map { array -> array.asList()
        .map { it.toFloat() }.toFloatArray()
    }.toTypedArray()
```

好耶已經有點瘋狂了，如果你輸出的會是 Tensor 型式，那麼 map 的地方應該會變的非常醜。而其他輸出的型式，就需要再花時間研究：水來土掩、兵來將擋。

Update: 其實：

```Kotlin
val a: Array<FloatArray> = result.asList()[1].toJava(Array<FloatArray>::class.java)
```

這樣就好，我想得太複雜了。

## 檔案大小

![Image](https://i.imgur.com/CSkaXtd.png)

上面是沒有 Pytohn 的純粹 APK，下面是包了 Python 的 APK，各位可以看到：光是引入 numpy，就多了近 45 MB，所以如果你不節制的控制 Package 的數量，那麼你整個 APK 就會非常肥大。

## 結語

使用 Python 非常方便，隨之而來的代價是 APK 的大小，以及執行速度被拖慢。我是覺得如果有能在 Android 完成的就先使用，如果是 Python 裡面好用的套件，建議先處理完後再傳入。那這樣子，我不如直接在 Android 上面使用 OpenCV 應該更好吧？

謝謝大家。
