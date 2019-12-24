---
title: 用Flutter重构应用
tags:
  - Android
  - Flutter
  - Dart
  - Kotlin
  - Gradle
categories:
  - 笔记
toc: false
date: 2019-12-23 14:09:19
---

# 前言

# 开发环境
* macOS Catalina10.15.2
* AndroidStudio 3.5.3
* Xcode 11.3
* 看看doctor
`
$ flutter doctor -v
`
```
[✓] Flutter (Channel master, v1.13.6-pre.16, on Mac OS X 10.15.2 19C57, locale zh-Hans-CN)
    • Flutter version 1.13.6-pre.16 at /Users/ciih/flutter
    • Framework revision fcaf9c4070 (34 hours ago), 2019-12-21 14:03:01 -0800
    • Engine revision 33813929e3
    • Dart version 2.8.0 (build 2.8.0-dev.0.0 886615d0f9)

 
[✓] Android toolchain - develop for Android devices (Android SDK version 28.0.3)
    • Android SDK at /Users/ciih/Library/Android/sdk
    • Android NDK location not configured (optional; useful for native profiling support)
    • Platform android-29, build-tools 28.0.3
    • Java binary at: /Applications/Android Studio.app/Contents/jre/jdk/Contents/Home/bin/java
    • Java version OpenJDK Runtime Environment (build 1.8.0_202-release-1483-b49-5587405)
    • All Android licenses accepted.

[✓] Xcode - develop for iOS and macOS (Xcode 11.3)
    • Xcode at /Applications/Xcode.app/Contents/Developer
    • Xcode 11.3, Build version 11C29
    • CocoaPods version 1.9.0.beta.2

[✓] Chrome - develop for the web
    • Chrome at /Applications/Google Chrome.app/Contents/MacOS/Google Chrome

[✓] Android Studio (version 3.5)
    • Android Studio at /Applications/Android Studio.app/Contents
    • Flutter plugin version 42.1.1
    • Dart plugin version 191.8593
    • Java version OpenJDK Runtime Environment (build 1.8.0_202-release-1483-b49-5587405)

[✓] IntelliJ IDEA Community Edition (version 2019.1.3)
    • IntelliJ at /Applications/IntelliJ IDEA CE.app
    • Flutter plugin version 39.0.3
    • Dart plugin version 191.8593

[✓] VS Code (version 1.39.2)
    • VS Code at /Applications/Visual Studio Code.app/Contents
    • Flutter extension version 3.6.0

[✓] Connected device (3 available)
    • 钱水利的 iPhone • 46e7963e6f0f38edf5c1c78f2f782a814f1edb34 • ios            • iOS 13.3
    • Chrome      • chrome                                   • web-javascript • Google Chrome 79.0.3945.88
    • Web Server  • web-server                               • web-javascript • Flutter Tools

• No issues found!
```



# 1.项目创建

> 为了省事直接用AndroidStudio创建了

## 1.1 命令行创建项目

* 简单创建
```
$ flutter create [项目名称]
```
* `-t [模板名称]`可以指定对应的模板,现在可以创建`app`--应用 `module`--app的模块 `package`--纯Dart代码项目 `plugin`--flutter插件项目 

* `-i [swift/objc]` 设置iOS开发语言,swift或者oc

* `-a [kotlin/java]` 设置Android开发语言

* `--[no-]androidx` 指定要不要使用Androidx支持库

## 1.2 设置子模块

运行`flutter create --project-name [你的项目名称] -i swift -a kotlin --androidx` 之后项目根目录大概是这个样子滴

- **android** android端代码
- **ios** ios端代码
- **web** web端代码
- **test** 测试代码
- **lib** 存放Dart代码
- **pubspec.lock** 
- **pubspec.yaml** 依赖文件,类似于Android的gradle或者Vue.js里面的package.json

完成之后需要将项目纳入版本控制,但是ios/web/android三个文件夹需要作为子模块(git submodule)
首先在github或者其他git网站新建子模块的repo(空的仓库),然后分别进入ios/web/android目录,
`$ git init` 初始化本地git仓库
`$ git remote add origin [你的子模块仓库地址]` 设置远程仓库地址
`$ git commit -ma "init"` 提交所有文件到本地仓库
`$ git push -u origin master` 推送本地代码到远程仓库

然后在git网站新建你的总repo,在根目录下初始化git仓库
然后添加子模块`git submodule add [子模块仓库链接] [名称]` ,注意名称要改成ios/android/web等,和本地目录名称对应起来,然后也是提交
`$ git remote add origin [你的仓库地址]` 
`$ git commit -ma "init"` 
`$ git push -u --recurse-submodules=on-demand
 origin master` 推送本地代码到远程仓库
不出意外的话你的git仓库看起来是这样的
![image.png](/images/2019/12/23/48c205d0-255e-11ea-a317-cbe164904c80.png)

> 子模块设置完成,如果需要更新子模块/拉取服务端改动等,可以参考git子模块那篇文章

# 2.项目框架搭建

从状态管理,路由管理,视图组件,动画,网络组件,本地存储,硬件交互等方面搭建新项目,尽量选择侵入性小的库

## 2.1 状态管理
之前的项目在状态管理方面用过`BLoC`和`Provider`,新的项目想试一试`flutter_redux`,`fish_redux`看了下太复杂了,小项目可能不太合适,有种大炮打蚊子的感觉...

### flutter_redux 基本概念

redux的基本概念:
- store 保存数据
- state 应用/数据状态
- action view发出的动作
- reducer 一个接收state,action返回新的state的函数
- middleware 在action和reducer中间执行,一般用thunk_redux

flutter_redux的组件:
- StoreProvider store的提供者,类似于Proivder中各种Provider的概念
- StoreConnector 类似于Provider中Consumer,响应变化.包含两个@require的函数:
  - converter,用于将state转化为view_model
  - builder,view_model转为view(widget)
  - onInit:初始化操作



