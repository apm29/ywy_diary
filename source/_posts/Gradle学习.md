---
title: Gradle与Groovy
date: 2019-12-11 21:11:20
tags: Gradle , Groovy
---
# Gradle与Groovy
> Gradle是用来构建的一个框架

## Gradle的编译周期
>
* 每个项目的编译至少需要一个Project,一个Build.Gradle就代表一个Project
 里面包含多个Task,Task中包含很多Action,Action是一个代码块,里面是需要执行的代码
* 在编译过程中， Gradle 会根据 build 相关文件，聚合所有的project和task，执行task 中的 action。因为 build.gradle文件中的task非常多，先执行哪个后执行那个需要一种逻辑来保证。这种逻辑就是依赖逻辑，几乎所有的Task 都需要依赖其他 task 来执行，没有被依赖的task 会首先被执行。所以到最后所有的 Task 会构成一个 有向无环图（DAG Directed Acyclic Graph）的数据结构。

https://www.jianshu.com/p/9df3c3b6067a

## AndroidStudio3.0
* productFlavor需要指定一个或多个flavorDimension
```
  flavorDimensions "default"
  productFlavors {
      official { dimension "default" }
      samsung { dimension "default" }
      oppo { dimension "default" }
    }
```
* 遍历productFlavor为每种flavor添加设置(就在productFlavor这个task内遍历)
```
productFlavors.all { flavor ->
            //定义一个map
            def map = [
                    CHANNEL_VALUE     : name,
                    //这里的name就是productFlavor的变量名oppo/360这样的
                    APP_NAME          : "APP默认Name",
                    JPUSH_PKGNAME     : "com.example.app",
                    JPUSH_APPKEY      : "XXXXXXJPUSHKEY",
                    UMENG_APPKEY_VALUE: "XXXXXAPP_KEY_VALUE",
            ]
            switch (name) {
                case "oppo":
                    map.APP_NAME = "Oppo渠道APPName"
                    break
                case "samsung":
                    map.APP_NAME = "samsung渠道APPName"
                    break
            }
            //给每个flavor的manifestplaceholders赋值,对应的值可以在Manifest文件中以${APP_NAME}这样的形式获取
            flavor.manifestPlaceholders = map
        }
```

## 在android Project下设置签名文件

* 从local.properties文件读取文件流
* 获取local.properties中定义的键值对(定义在本地防止打包时上传)
* 对应key赋值设置等
```
signingConfigs {
        release {
            //加载资源
            Properties properties = new Properties()
            InputStream inputStream = project.rootProject.file('local.properties').newDataInputStream()
            properties.load(inputStream)

            //读取字段
            def store_file = properties.getProperty('storeFile')
            def key_keyAlias = properties.getProperty('keyAlias')
            def key_keyPassword = properties.getProperty('keyPass')
            def key_storePassword = properties.getProperty('storePass')
            storeFile file(store_file)//签名文件的path
            storePassword key_storePassword //签名文件的密码
            keyAlias key_keyAlias //别名
            keyPassword key_keyPassword //别名的密码
        }
    }
```

## android Project下的构建类型设置
* 可以设置多个构建类型:debug release stage product等,可以自己随意定义
* 构建类型下的可配置Config
  * buildConfigField 可以声明BuildConfig变量,比如自己设置一个叫BASE_URL,类型为String的值
  ```
  release{
      buildConfigField("String", "BASE_URL", "\"http://abc.com:9999/\"")  
  }
  ```
  * applicationIdSuffix 设置包名后缀,比如在测试包加一个 .test : applicationIdSuffix ".test"
  * signingConfig signingConfigs.release 定义签名文件设置(引用signingConfigs设置的release签名设置,如上一节所述的)
  * manifestPlaceholders 定义Manifest清单文件中占位字段,定义好后可以以${APP_NAME}这样的形式获取

  ## android Project下的输出Apk文件名设置
  * 在android{}内遍历获取output文件路劲/productFlavor的name 拼接到apk文件名中
    ```
    applicationVariants.all { variant ->
        variant.outputs.all { output ->
            def outputFile = output.outputFile//文件路径
            def versionCode = defaultConfig.versionCode
            def versionName = defaultConfig.versionName
            def flavor = variant.productFlavors[0].name//渠道名称
            if (outputFile != null && outputFile.name.endsWith('.apk')) {
                if (variant.buildType.name.equals('release')) {
                    outputFileName = "appname-" + flavor + "-" + versionCode + "-" + versionName + "-release.apk";
                } else if (variant.buildType.name.equals('debug')) {
                    outputFileName = "appname_test-" + "-" + versionCode + "-" + versionName + "-debug.apk";
                }
            }
        }
    }
    ```
## 其他一些设置(来自简书)
```
android {
    defaultConfig {
        //默认配置项，defaultConfig就是程序的默认配置，注意，如果在   AndroidMainfest.xml里面定义了与这里相同的属性，会以这里的为主。
    }

    buildTypes {
      // 编译配置，release或debug版本的内容
    }

    compileOptions {
      // Java 的版本配置
    }

    sourceSets {
        //源码设置(项目目录结构的设置)
    }

    packagingOptions {
       //打包时的相关配置  
    }

    lintOptions {
        //编译的 lint 开关，程序在buid的时候，会执行lint检查，有任何的错误或者警告提示，都会终止构建，我们可以将其关掉。
        //abortOnError false  
    }

    productFlavors {
        //产品发布的一些东西，比如渠道、包名等
        flavor1 {
        }

        flavor2 {
        }
    }

    signingConfigs {
        //签名的配置
        release {
        }
    }

    testOptions{
        //测试配置，TestOptions类型
    }
    aaptOptions{
      //aapt配置，AaptOptions类型
    }
     lintOptions{
       //lint配置，LintOptions类型
    }
    dexOptions{
       //dex配置，DexOptions类型
    }
    compileOptions{
     // 编译配置，CompileOptions类型
    }
    packagingOptions{
       // PackagingOptions类型
    }
    jacoco{
       //JacocoExtension类型。 用于设定 jacoco版本
    }
    splits{
       //Splits类型。
    }
}

链接：https://www.jianshu.com/p/ac02cebd279e

```
