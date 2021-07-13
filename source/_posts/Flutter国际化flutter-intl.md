---
title: Flutter国际化flutter_intl
tags:
  - Flutter
  - Tools
  - Dart
originContent: ''
categories:
  - 笔记
toc: false
date: 2020-07-07 16:33:25
---

# 缘起
之前项目一直在用的国际化库flutter_i18n作者跑路不维护了，idea插件也只支持到192.x，只好自己再找个类似的库，然后就发现了Flutter Intl插件

# 使用
默认情况下，Flutter仅提供美国英语本地化。要添加对其他语言的支持，应用程序必须指定其他MaterialApp属性，并包含一个名为的单独包-“flutter_localizations”。

在`pubspec.yaml`中添加`flutter_localizations`依赖并执行`packages get`
```
# 国际化
flutter_localizations:
    sdk: flutter
```

然后在AndroidStudio->Tools->Flutter Intl->initialize for the Project,
初始化之后pubspec，yaml最好会加上一段配置：
```
flutter_intl:
  enabled: true
```

然后就可以在AndroidStudio->Tools->Flutter Intl->Add Locale添加新的语言包
添加后会在`lib/i10n`目录生成对应语言的.arb文件，内容是json格式的通过`"@@locale"："en"`来指定语言，`"appName":"xxxx"`来生成一个文本，保存后插件会帮你生成`lib/generated/intl`和`lib/generated/i10n.dart`


# 配置
给顶部的MaterialApp设置：
```
	MaterialApp(	
            debugShowCheckedModeBanner: Config.AppDebug,
            localizationsDelegates: [
              // ... app-specific localization delegate[s] here
              S.delegate,
              GlobalMaterialLocalizations.delegate,
              GlobalWidgetsLocalizations.delegate,
              GlobalCupertinoLocalizations.delegate,
              DefaultCupertinoLocalizations.delegate
            ],
            locale: locale,
            supportedLocales: S.delegate.supportedLocales,
            home: child,
          )
```
