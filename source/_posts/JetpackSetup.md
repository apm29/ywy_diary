---
title: Jetpack
date: 2019-12-11 21:11:20
tags: 
    - Android
---
# Jetpack

### 1. Components Dependencies

---
  * `AppCompat`
    > 依赖于原先的`support v4`包,包含了`ActionBar`,`AppCompatActivity`,`cardview`,`gridlayout`,`recyclerview`等支持

### 2. 迁移到AndroidX
  * 需要AndroidStudio3.2+
  * 在gradle.properties中加入
    ```
    android.useAndroidX=true
    android.enableJetifier=true
    ```
     sync 下Gradle后需要把原有项目中的support库中的类重新import,谷歌的developer网站上有一份迁移的清单,可以看看,顺便学下各种没用过的官方库😭
     [迁移清单](https://developer.android.com/jetpack/androidx/migrate),如果想要用kotlin开发最好尝试下使用ktx
