---
title: 真的需要在 Gradle 裡引用機密資料但是不想上傳至 GitHub 怎麼辦？
categories:
  - Academic
tags:
  - GitHub
  - Gradle
  - Android
abbrlink: 1438358789
date: 2024-07-11 14:02:35
---

真的需要在 Gradle 裡引用機密資料但是不想上傳至 GitHub 怎麼辦？就不想把機密資料放上 GitHub！

<!-- more -->

## 前言

生成的 APK 都是有上鑰匙加密的，但是希望平常用 Debug 版本時，可以直接無縫安裝，不需要移除再安裝，因此我會選擇把金鑰的訊息直接放在 Gradle 裡面。

```kotlin
android {
    signingConfigs {
        getByName("debug") {
            keyAlias = "KEY"
            keyPassword = "12345678"
            storePassword = "12345678"
            storeFile = file("F:\\Key")
        }
    }
    ......
}
```

但是這種東西直接上傳到 GitHub 實在是太不安全了！如果流出去了，那麼別人就可以用這把金鑰上傳簽署過的 APK 至 PlayStore 上面。

## 作法

因此，我們希望額外創造一個檔案，放入機密資料後再額外從原本的檔案中引用，並且把額外的檔案加入 .gitignore 裡，防止機密檔案被 git 上去。

### 準備

我是新創 `apikeys.properties` 於同份專案裡，我是放在跟 `.gitignore` 同個層級

### 放入資料

接著在 `apikeys.properties` 放入機密資料

```text
keyAlias = KEY
keyPassword = 12345678
storePassword = 12345678
storeFile = F:\\Key
```

這裡的概念是 Map，因此 Key 的部分就自己命名。

### 引用

接著取代本來的 Gradle 裡放置帳密的地方。

```Kotlin
import org.jetbrains.kotlin.konan.properties.Properties

.......

android{
    signingConfigs {
        getByName("debug") {
            val keyStoreFile = project.rootProject.file("apikeys.properties")
            val properties = Properties().apply {
                load(keyStoreFile.inputStream())
            }
            keyAlias = properties.getProperty("keyAlias")
            keyPassword = properties.getProperty("keyPassword")
            storePassword = properties.getProperty("storePassword")
            storeFile = file(properties.getProperty("storeFile"))
        }
    }
    ......
}
```

### 排除上傳

記得要排除 `apikeys.properties` 於同步之外。也就是要加入 `.gitignore`。

## 結語

用這個方法，防止上傳機密資料。如果同事需要用的話，我想就額外再傳遞機密的檔案即可。至少不要上傳至別人的伺服器裡吧！
