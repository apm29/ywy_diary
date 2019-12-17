---
title: Flutter
date: 2019-12-11 21:11:20
tags: 
    - Flutter
---
## Flutter

# Get Started: Install on Windows
### INFO
> 大部分来自 https://flutter.io
### Get the Flutter SDK
> [SDKv0.2.8下载地址](https://storage.googleapis.com/flutter_infra/releases/beta/windows/flutter_windows_v0.2.8-beta.zip)
### Update your path
> 在window环境变量PATH中添加 `flutter\bin`,
   记得重启电脑后生效
### Run flutter doctor
> 在CMD中执行`flutter doctor`命令,检测哪些 工具/环境 没有安装
### Android setup
> 一般是AndroidStudio
### Install Android Studio
> [AndroidStudio](https://developer.android.com/studio/index.html)
### Set up your Android device
Download and install Android Studio.

Start Android Studio, and go through the ‘Android Studio Setup Wizard’. This will install the latest Android SDK, Android SDK Platform-Tools, and Android SDK Build-Tools, which are required by Flutter when developing for Android.

Set up your Android device
To prepare to run and test your Flutter app on an Android device, you’ll need an Android device running Android `4.1 (API level 16)` or higher.

Enable Developer options and USB debugging on your device. Detailed instructions are available in the Android documentation.
Using a USB cable, plug your phone into your computer. If prompted on your device, authorize your computer to access your device.
In the terminal, run the flutter devices command to verify that Flutter recognizes your connected Android device.
Start your app by running flutter run.
By default, Flutter uses the version of the Android SDK where your adb tool is based. If you want Flutter to use a different installation of the Android SDK, you must set the `ANDROID_HOME` environment variable to that installation directory.

### Set up the Android emulator
To prepare to run and test your Flutter app on the Android emulator, follow these steps:

Enable VM acceleration on your machine.
Launch Android `Studio>Tools>Android>AVD` Manager and select Create Virtual Device.
Choose a device definition and select Next.
Select one or more system images for the Android versions you want to emulate, and select Next. An x86 or x86_64 image is recommended.
Under Emulated Performance, select Hardware - GLES 2.0 to enable hardware acceleration.
Verify the AVD configuration is correct, and select Finish.

For details on the above steps, see Managing AVDs.

In Android Virtual Device Manager, click Run in the toolbar. The emulator starts up and displays the default canvas for your selected OS version and device.
Start your app by running flutter run. The connected device name is Android SDK built for <platform>, where platform is the chip family, such as x86.


### 1. 最简单的Flutter APP
编辑新建项目中的 main.dart 文件
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(
    new Center(
      child: new Text(
        'Hello, world!',
        textDirection: TextDirection.ltr,
      ),
    ),
  );
}
```
main方法是app的主入口,runApp方法接收一个Widget对象,这里的Center相当于Android中RelativeLayout加了gravity-center(大概吧),Text应该对应的TextView这种,点击run或者在terminal中`flutter run`运行程序,`crtl+s`可以热加载程序,随时修改代码.

这里的Center/Text都是 Widget的子类,所有的UI控件都继承自Widget,比较常用的Widget 有
* Text
* Stack(有点像FrameLayout)
* Row/Column (类似LineLayout)
* Container(有Margin/padding/border/background,可以通过Matrix变成三维空间?!)


> 下面的代码是一些控件的使用,因为使用Material控件,需要在pubspec.yaml中申明
`name: my_app
flutter:
  uses-material-design: true`

```dart
import 'package:flutter/material.dart';

class MyAppBar extends StatelessWidget {
  MyAppBar({this.title});

  // Fields in a Widget subclass are always marked "final".

  final Widget title;

  @override
  Widget build(BuildContext context) {
    return new Container(
      height: 56.0, // in logical pixels
      padding: const EdgeInsets.symmetric(horizontal: 8.0),
      decoration: new BoxDecoration(color: Colors.blue[500]),
      // Row is a horizontal, linear layout.
      child: new Row(
        // <Widget> is the type of items in the list.
        children: <Widget>[
          new IconButton(
            icon: new Icon(Icons.menu),
            tooltip: 'Navigation menu',
            onPressed: null, // null disables the button
          ),
          // Expanded expands its child to fill the available space.
          new Expanded(
            child: title,
          ),
          new IconButton(
            icon: new Icon(Icons.search),
            tooltip: 'Search',
            onPressed: null,
          ),
        ],
      ),
    );
  }
}

class MyScaffold extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Material is a conceptual piece of paper on which the UI appears.
    return new Material(
      // Column is a vertical, linear layout.
      child: new Column(
        children: <Widget>[
          new MyAppBar(
            title: new Text(
              'Example title',
              style: Theme.of(context).primaryTextTheme.title,
            ),
          ),
          new Expanded(
            child: new Center(
              child: new Text('Hello, world!'),
            ),
          ),
        ],
      ),
    );
  }
}

void main() {
  runApp(new MaterialApp(
    title: 'My app', // used by the OS task switcher
    home: new MyScaffold(),
  ));
}
```

> Widget的目录在这个网址上有([WidgetCatalog](https://flutter.io/widgets/))


## Widget

### Container
* 没有父布局的时候填充整个屏幕
* color和decoration只可有一个
* padding: child与边界的距离
* contraint:让文字有足够的垂直空间,并且水平延伸来适应父控件
* 旋转偏移等transfromation之后:container占据的位置不变的



### Dismissible
* 滑动删除组件(类似iOS)
* 可以设定滑动方向
* 设定滑动成功事件
* 代码例子
```dart
Widget _buildBody() {
    print(_list.length);
    scrollController.addListener(() {
      print("position:" + scrollController.position.pixels.toString());
      print("offset:" + scrollController.offset.toString());
    });
    return new ListView.builder(
      controller: scrollController,
      physics: new BouncingScrollPhysics(),
      itemBuilder: (context, index) {
        final item = _list[index];
        return new Dismissible(
          //key用于复用
          key: new Key(item),
          child: new Container(
            height: 200.0,
            child: new Align(child: new Text(item)),
          ),
          //background决定了背景(滑动露出来的部分)
          background: new Container(
            color: Colors.red,
            padding: new EdgeInsets.fromLTRB(0.0, 0.0, 10.0, 0.0),
            alignment: Alignment.centerRight,
            child: new Text(
              'Delete',
              textAlign: TextAlign.center,
              style: new TextStyle(color: Colors.white, fontSize: 20.0),
            ),
          ),
          //滑动完成回调
          onDismissed: (direction) {
            Scaffold.of(context).showSnackBar(new SnackBar(
                  content: new Text(
                    'Item $item Removed',
                    textAlign: TextAlign.center,
                    style: new TextStyle(color: Colors.white, fontSize: 20.0),
                  ),
                  duration: new Duration(seconds: 3),
                ));
          },
        );
      },
      itemCount: _list.length,
    );
  }
```

### AnimatedList
* 在创建的时候带入一个GlobalKey,用于InsertItem 和 RemoveItem


### margin/padding

> EdgeInsets is a class that will generate the appropiate margins. It has some fancy constructors:

* `EdgeInsets.only(left, top, right, bottom)`: allows us to define different margins per side. All them are optional, so you can specify, for example, only left and top.
* `EdgeInsets.fromLTRB(left, top, right, bottom)`: similar to previous, but, you have to specify the four margins with positional parameters. The LTRB is a nmemonic rule (Left, Top, Right, Bottom).
* `EdgeInsets.all(value`): sets the same margin for all four sides.
* `EdgeInsets.symmetric(vertical, horizontal)`: allows us to specify top/bottom and/or left/right with a single value.


### Enter the `Slivers`
Despite our list is now fully working as intended, I want to introduce you to the use of `Slivers`.

`Slivers` are a very powerful tool, and the scrollable Widgets are based on internal use of `Slivers`.

A `Sliver` is a piece of scrollable content. `Slivers` should be placed inside a `ScrollView` widget. The basic class to create a `ScrollView` is `CustomScrollView` which allows a list of `Slivers` as children. There are several `ScrollView` classes, and reviewing all them goes beyond this article.

There are also several `Slivers`, these are some of them:

`SliverAppBar`: used to create a collapsable material AppBar.
`SliverList`: a linear list of items.
`SliverFixedExtentList`: similar to the previous one, but for items with fixed height.
`SliverToBoxAdapter`: a sliver with a single child with a defined size.
`SliverPadding`: a simple sliver that contains antoher Sliver and allows us to apply a padding.

### Route
```java
return new GestureDetector(
   onTap: () => Navigator.of(context).push(new PageRouteBuilder(
     pageBuilder: (_, __, ___) => new DetailPage(planet),
   )),
   child: new Container(
     height: 120.0,
     margin: const EdgeInsets.symmetric(
       vertical: 16.0,
       horizontal: 24.0,
    ),
    child: new Stack(
      children: <Widget>[
        planetCard,
        planetThumbnail,
      ],
    ),
  )
);
```
> Now, instead of recovering the `PageRoute` from the table of routes with pushNamed we are creating a new one using the class `PageRoutBuilder`, and it has a parameter named pageBuilder that should return a new Widget to show as page, in this case, DetailPage receiving as a parameter the planet to show.

### Hero动画

```java
new Hero(
      tag: "planet-hero-${planet.id}",
      child: new Image(
      image: new AssetImage(planet.image),
     height: 92.0,
      width: 92.0,
    ),
  ),
```
> 给两个不同页面的Widget包裹Hero,打上相同的tag,在页面跳转时(Navigator.push...等)会执行hero动画

# 隐藏keyboard
* 方法一

```dart
//把一个新的FocusNode作为新焦点,这样其他的控件(比如TextField TextFormField)会失去焦点,从而隐藏键盘
FocusScope.of(context).requestFocus(new FocusNode());
```
> 相应的,请求焦点也可以用这个方法

* 方法二

> 比较Hacky的方法

```dart
SystemChannels.textInput.invokeMethod('TextInput.clearClient');
```
