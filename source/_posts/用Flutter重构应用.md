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



# 项目创建

> 为了省事直接用AndroidStudio创建了

## 命令行创建项目

* 简单创建
```
$ flutter create [项目名称]
```
* `-t [模板名称]`可以指定对应的模板,现在可以创建`app`--应用 `module`--app的模块 `package`--纯Dart代码项目 `plugin`--flutter插件项目 

* `-i [swift/objc]` 设置iOS开发语言,swift或者oc

* `-a [kotlin/java]` 设置Android开发语言

* `--[no-]androidx` 指定要不要使用Androidx支持库

## 设置子模块

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

# 项目框架搭建

从状态管理,路由管理,视图组件,动画,网络组件,本地存储,硬件交互等方面搭建新项目,尽量选择侵入性小的库

## 状态管理
之前的项目在状态管理方面用过`BLoC`和`Provider`,新的项目想试一试`flutter_redux`,`fish_redux`看了下太复杂了,小项目可能不太合适,有种大炮打蚊子的感觉...

### flutter_redux 基本概念

redux的基本概念:
- store 状态管理
- state 应用/数据状态
- action view发出的动作
- reducer 一个接收state,action返回新的state的函数
- middleware 在action和reducer中间执行

### redux的Store
`flutter_redux`依赖于`redux`包,所以我们也要先了解了`redux`里面的概念,`Store`是整个App的状态Owner,**唯一能改变`Store`中状态树的方法就是通过`Store`发送(`dispatch`)一个`Action`**,action会先通过`middleware`,如果没有被`middleware`中断,则会传递至`reducer`,`reducer`根据`action`改变`store`中的`state`完成`state`树的更新,当然具体逻辑可能更复杂一些. 

	- 定义action: `final increment = 'INCREMENT';`
	- 定义reducer: `int counterReducer(int state, action) => state+1` 只可以根据action来决定如何改变state,我这里为了简单就直接写个加一了
	- 创建Store: `Store<int>(counterReducer,initialState:0)`
	- 分发一个action: `store.dispatch(increment);` action可以是任意类型
	- 然后store中的state(原来是0)就会改变
**store提供了两种方式获得state实例:state属性 或者 onChange提供的Stream**

看看构造方法全都明白了:
```
Store(
    this.reducer, {
    State initialState,
    //middleware集合
    List<Middleware<State>> middleware = const [],
    //是否使用同步的Stream控制器(一般就false)
    bool syncStream = false,

    /// 若设置为true,在原来State和reducer处理过的State相同的情况下Store不会发送事件
    bool distinct = false,
  }) : _changeController = StreamController.broadcast(sync: syncStream) {
    _state = initialState;
    //创建分发器
    _dispatchers = _createDispatchers(
      middleware,
      _createReduceAndNotify(distinct),
    );
  }
```
_createDispatchers: 创建一个NextDispatcher集合,放入middleware集合转换来的NextDispatcher,再放入_createReduceAndNotify转化来的NextDispatcher,这个集合就是dispatch方法的调用栈,dispatch方法会先调用第一个NextDispatcher
- 再看看Middleware的定义
```
typedef dynamic Middleware<State>(
  Store<State> store,
  dynamic action,
  NextDispatcher next,
);
```
> 这里的NextDispatcher是对reducer和middleware的简单包装,让他们接受action并返回action(或者被拦截...所以定义里面返回值是dynamic)

因为middleware都在栈顶,所以先调用middleware转换成的NextDispatcher,这个next就是下一个middleware或者reducer对应的NextDispatcher,若你在middleware中调用了next,这个调用链就会一直调用到最后知道reducer调用完成,完成对state的修改.(这个是不是叫责任链设计模式?)
> 来个图 middleware--> middleware(我可以中断也可以往下传球) ---> .... --->reducer

reducer完成了拼图的最后一块,最后的NextDispatcher,往StreamController(`_changeController`)add了一个修改后的state.看到这里恍然大悟,原来又是你:Stream,所以很好猜,下面章节要讲的Widget里用于局部更新的StoreConnector/StoreBuilder应该就是StreamBuilder包装下的东西.
```
 NextDispatcher _createReduceAndNotify(bool distinct) {
    return (dynamic action) {
      final state = reducer(_state, action);

      if (distinct && state == _state) return;

      _state = state;
      _changeController.add(state);
    };
  }
```

flutter_redux的组件:
- StoreProvider store的提供者,类似于Proivder中各种Provider的概念
- StoreConnector 类似于Provider中Consumer,响应变化.包含两个@require的函数:
  - converter,用于将state转化为view_model
  - builder,view_model转为view(widget)
  - onInit:初始化操作
 
### StoreProvider
继承自InheritedWidget,用于给所有子不见提供Store,里面有用的就一个Store对象,和其他状态管理组件例如Provider一样,有个静态方法`static Store<S> of<S>(BuildContext,{bool listen=true})`来获取实例. 几乎一模一样的参数.

- 当该方法传入`listen = true`时
	通过`inheritFromWidgetOfExactType`(这个方法在1.12已经被标记为deprecated,新的api是`dependOnInheritedWidgetOfExactType`,名字与意义更符合一点)获取provider,api文档说的很明显,会把调用者context注册,当该Type对应的InheritedWidget发生改变的时候将会rebuild该context对应的组件

- 当`listen = false`时
	通过`ancestorInheritedElementForWidgetOfExactType`(这个方法在1.12已经被标记为deprecated,新的api是`getElementForInheritedWidgetOfExactType`,)获取provider,文档说的是获取给定的Type最近的`InheritedElement`,然后再通过`InheritedElement`拿到`InheritedWidget`,也就是我们需要的provider.

- 源码大概结构
```
class StoreProvider<S> extends InheritedWidget {
  final Store<S> _store;

  /// 通过Store和Child创建
  const StoreProvider({
    Key key,
    @required Store<S> store,
    @required Widget child,
  })  : assert(store != null),
        assert(child != null),
        _store = store,
        super(key: key, child: child);

  ///用于获取Store实例,在initState中使用时要传listen = false
  static Store<S> of<S>(BuildContext context, {bool listen = true}) {
    final type = _typeOf<StoreProvider<S>>();
    final provider = (listen
        ? context.inheritFromWidgetOfExactType(type)
        : context
            .ancestorInheritedElementForWidgetOfExactType(type)
            ?.widget) as StoreProvider<S>;

    if (provider == null) throw StoreProviderError(type);

    return provider._store;
  }

  // Dart中获取类型的方法,很多地方会用到
  static Type _typeOf<T>() => T;

  ///是否需要通知子Widget的判断方法
  @override
  bool updateShouldNotify(StoreProvider<S> oldWidget) =>
      _store != oldWidget._store;
}
```

### StoreConnector
这个类似于Provider中的Consumer,`class StoreConnector<S, ViewModel> extends StatelessWidget {}`,泛型S表示State类型,ViewModel表示从State类型转化来的model数据,用于生成Widget.这个组件实际只是给已给名叫`__StoreStreamListener`的StatefulWidget包装了下.我们先看看它的State干了什么能监听Store内的State变化.

1.两个重要的属性
  * **Stream<ViewModel> stream;**
  *  **ViewModel latestValue;**

2.initState方法
  * 调用StoreConnector的onInit
  * 在下一帧调用StoreConnector的onInitialBuild
  * 通过StoreConnector的converter吧store转换成viewModel赋值给 latestValue
  * 通过StoreConnector.store.onChange属性获得流(Stream),经过一系列操作转换成ViewModel流赋值给属性`stream`
   ```
	stream = widget.store.onChange
        .where(_ignoreChange)
        .map(_mapConverter)
        // Don't use `Stream.distinct` because it cannot capture the initial
        // ViewModel produced by the `converter`.
        .where(_whereDistinct)
        // After each ViewModel is emitted from the Stream, we update the
        // latestValue. Important: This must be done after all other optional
        // transformations, such as ignoreChange.
        .transform(StreamTransformer.fromHandlers(handleData: _handleChange));

     void _handleChange(ViewModel vm, EventSink<ViewModel> sink) {
    	if (widget.onWillChange != null) {
      	widget.onWillChange(latestValue, vm);
    	}

    	latestValue = vm;

    	if (widget.onDidChange != null) {
      WidgetsBinding.instance.addPostFrameCallback((_) {
        	widget.onDidChange(latestValue);
      	});
    }

    sink.add(vm);
  }
  ```
> 这里有几个Stream操作符:where是筛选,map是映射,transform 方法就是把一个 Stream 作为输入，然后经过计算或数据转换，输出为另一个 Stream。另一个 Stream 中的数据类型可以不同于原类型，数据多少也可以不同

所以其他方法都并不重要了,_createStream方法以及定义好了流中数据转换和过滤的方式,其他方法都是一些生命周期回调方便定制其他功能,最后看看build方法做了些什么事:

```
@override
  Widget build(BuildContext context) {
    return widget.rebuildOnChange
        ? StreamBuilder<ViewModel>(
            stream: stream,
            builder: (context, snapshot) => widget.builder(
              context,
              latestValue,
            ),
          )
        : widget.builder(context, latestValue);
  }
```

没错!!!包装了下StreamBuilder,通过它监听Stream中的新数据,返回builder创建的新Widget.

### StoreBuilder

其实和StoreConnector差不多,懒人版本的StoreConnector,少了个converter,state-->viewModel这部分就省了,直接build完事.其他功能都一毛一样.

### Redux包下提供的工具方法

redux包提供了一些工具类工具方法,让我免去大量switch..case模板代码

比如说我们先定义state和action
```
class AppState {
  final List<Item> items;
  AppState(this.items);
}
class LoadItemsAction {}
class UpdateItemsAction {}
class AddItemAction{}
class RemoveItemAction {}
class ShuffleItemsAction {}
class ReverseItemsAction {}
class ItemsLoadedAction<Item> {
  final List<Item> items;
  ItemsLoadedAction(this.items);
}
```
我们现在有一个state保存一个item集合,有加载,更新,添加,删除等许多action,如果是之前的代码我们可能要写好多switch..case代码,像下面这样👇:
```
final appReducer = (AppState state, action) {
  if (action is ItemsLoadedAction) {
    return new AppState(action.items);
  } else if (action is UpdateItemsAction) {
    return ...;
  } else if (action is AddItemAction) {
    return ...;
  } else if (action is RemoveItemAction) {
    return ...;
  } else if (action is ShuffleItemsAction) {
    return ...;
  } else if (action is ReverseItemsAction) {
    return ...;
  } else {
    return state;
  }
};
```

redux提供了combineReducer方法和TypedReducer类来将多个小的reducer组合为一个reducer:
```
final removeItemReducer = (AppState state, RemoveItemAction action) {
  return ...;
}

///....

final Reducer<AppState> appReducer = combineReducers([
  new TypedReducer<AppState, LoadTodosAction>(loadItemsReducer),
  new TypedReducer<AppState, UpdateItemsAction>(updateItemsReducer),
  new TypedReducer<AppState, AddItemAction>(addItemReducer),
  new TypedReducer<AppState, RemoveItemAction>(removeItemReducer),
  new TypedReducer<AppState, ShuffleItemAction>(shuffleItemsReducer),
  new TypedReducer<AppState, ReverseItemAction>(reverseItemsReducer),
]);
```
middleware当然也有:

```
 final List<Middleware<AppState>> middleware = [
   new TypedMiddleware<AppState, LoadTodosAction>(loadItemsMiddleware),
   new TypedMiddleware<AppState, AddTodoAction>(saveItemsMiddleware),
   new TypedMiddleware<AppState, ClearCompletedAction>(saveItemsMiddleware),
   new TypedMiddleware<AppState, ToggleAllAction>(saveItemsMiddleware),
   new TypedMiddleware<AppState, UpdateTodoAction>(saveItemsMiddleware),
   new TypedMiddleware<AppState, TodosLoadedAction>(saveItemsMiddleware),
 ];
```
## 路由管理

路由管理使用fluro,其实flutter框架自带的路由已经很好了,不过fluro可以往路由路径里加入参数,这个有些项目比较方便,可以自己取舍.

### 引入
```
dependencies:
  fluro: ^1.5.1
```

### 使用

提供一个全局Router实例

```
import 'package:fluro/fluro.dart';

class Application{
 
  static Router router;

  static void init() {
    Router router = Router();
    Routes.configureRoutes(router);
    Application.router = router;
  }
}
```

定义一个路由映射类
```
class Routes {
  static String root = "/";
  static String home = "/home";
  static String login = "/login";
  static String personalSettings = "/personal_settings";

  static void configureRoutes(Router router) {
    router.notFoundHandler = new Handler(handlerFunc: (
      BuildContext context,
      Map<String, List<String>> params,
    ) {
      print("ROUTE WAS NOT FOUND !!!");
      return NotFoundPage();
    });

    /// 第一个参数是路由地址，第二个参数是页面跳转和传参，第三个参数是默认的转场动画，可以看上图
    router.define(root, handler: splashHandler);
    router.define(home, handler: homeHandler);
    router.define(login, handler: loginHandler);
    router.define(personalSettings, handler: personalSettingsHandler);
  }
}
```
可以看到我设置了四个路径,`'/'`为默认第一个页面,`configureRoutes`手动在main方法中调用来初始化Router,我给他设置了一个404页面,然后定义了各个路由的handler:
```
class NotFoundPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text('404 路由未找到'),
      ),
    );
  }
}

/// 跳转到首页Splash
final splashHandler = new Handler(
  handlerFunc: (BuildContext context, Map<String, List<String>> params) {
    return SplashPage();
  },
);

final homeHandler = new Handler(
  handlerFunc: (BuildContext context, Map<String, List<String>> params) {
    return HomePage();
  },
);

final personalSettingsHandler = new Handler(
  handlerFunc: (BuildContext context, Map<String, List<String>> params) {
    return PersonalSettingsPage();
  },
);

final loginHandler = new Handler(
  handlerFunc: (BuildContext context, Map<String, List<String>> params) {
    return LoginPage();
  },
);
```

然后在main.dart中先调用下init:

```
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // 初始化缓存,这个是自己包装的SharedPreferences工具
  await Cache().init();
  // 注册 fluro routes
  Application.init();
  runApp(FlutterReduxApp());
}
```

然后就可以在其他地方使用了,当然你也可以再封装个路由跳转类:
```
Application.router.navigateTo(
      context,
      Routes.personalSettings,
      transition: TransitionType.inFromBottom,
    );
```

路由跳转类:(这边基本没用到参数...只支持String参数并且不能有中文),fluro可以定义转场动画,需要的可以去看看Github:[fluro](https://github.com/theyakka/fluro)
```
class AppRouter {
  static Future toHomeAndReplaceSelf(BuildContext context) {
    return Application.router.navigateTo(context, Routes.home, replace: true);
  }

  static Future toPersonal(BuildContext context) {
    return Application.router.navigateTo(
      context,
      Routes.personalSettings,
      transition: TransitionType.inFromBottom,
    );
  }

  static Future toHome(BuildContext context, ActiveTab activeTab) {
    store.dispatch(TabSwitchAction(activeTab.index, context));
    return Application.router
        .navigateTo(context, Routes.home, replace: true, clearStack: true);
  }

  static Future toLogin(BuildContext context) {
    return Application.router.navigateTo(
      context,
      Routes.login,
      transition
          : TransitionType.inFromBottom,
    );
  }
}
```

定义参数是这样的:
```
var usersHandler = Handler(handlerFunc: (BuildContext context, Map<String, dynamic> params) {
  return UsersScreen(params["id"][0]);
});

void defineRoutes(Router router) {
  router.define("/users/:id", handler: usersHandler);

  // it is also possible to define the route transition to use
  // router.define("users/:id", handler: usersHandler, transitionType: TransitionType.inFromLeft);
}
```

## 国际化

如果您的应用可能会给另一种语言的用户使用，那么您需要“国际化”它。这意味着您在编写应用程序时需要为应用程序支持的每种语言环境， 设置“本地化”的一些值，如文本和布局。Flutter提供一些widgets和类，以帮助实现国际化，而Flutter的库本身也是国际化的。
参考:[Flutter中文网](https://flutterchina.club/tutorials/internationalization/),[Flutter-国际化适配](https://juejin.im/post/5c701379f265da2d9b5e196a)

### 导入flutter_localization包

pubspec.yml文件:
```
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter

```

### 引入

指定`MaterialApp`的`localizationsDelegates`和`supportedLocales`：

```
import 'package:flutter_localizations/flutter_localizations.dart';

new MaterialApp(
 localizationsDelegates: [
   // ... app-specific localization delegate[s] here
   GlobalMaterialLocalizations.delegate,
   GlobalWidgetsLocalizations.delegate,
 ],
 supportedLocales: [
    const Locale('en', 'US'), // English
    const Locale('he', 'IL'), // Hebrew
    // ... other locales the app supports
  ],
  // ...
)
```

### 使用Flutter-i18n插件

如果你已经成功安装插件，打开项目后，会发现自动添加以下两个文件：

lib/generated/i18n.dart 主要的国际化文件，主要使用的类为S
res/values/string_en.arb 该文件主要适配英文语言，内容为json格式
修改:
```
MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.red,
      ),
      localizationsDelegates: const [
        S.delegate,
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate
      ],
      supportedLocales: S.delegate.supportedLocales,
    );
```

- localizationsDelegates本地化委托参数
- S.delegate 我们项目的本地化委托类,这个你不用管,他会根据你的arb文件自动生成对应的函数
GlobalMaterialLocalizations.delegate和GlobalWidgetsLocalizations.delegate为
- flutter_localizations插件包提供的委托，如果你使用MaterialApp这个部件的- - GlobalMaterialLocalizations.delegate这个可以不用
- supportedLocales支持的本地化
- S.delegate.supportedLocales我们项目支持的本地化，这个你不用管，它会在你添加arb文件时自动更新你的支持的本地化

### 声明资源

.arb文件
```
{
	"title":'AppName'
}
```

占位符:
```
{
	"successMessage":"操作成功:$action"
}
```

列表: 支持语法为：key+zero/one/two/few/many/other
```
{
  "selectZero":"没有了",
  "selectOne":"一个",
  "selectTwo":"两个",
  "selectFew":"一些",
  "selectMany":"很多",
  "selectOther":"其它"
}
```

### 使用资源

- 普通:`S.of(context).title`
- 带占位符:`S.of(context).successMessage("登录")`
- 列表
 ```
    S.of(context).select(0);//零个
    S.of(context).select(1);//一个
    S.of(context).select("many");//多个
    S.of(context).select(null);//其它
   ```



# 插件的使用

## camera 0.5.7+2

### 引入

pubspec.yml:
```
camera: ^0.5.7+2
```

### iOS权限
info.plist
```
<key>NSCameraUsageDescription</key>
<string>APP需要您的同意，才能访问相机进行拍照，如禁止将无法拍照更新信息</string>
<key>NSMicrophoneUsageDescription</key>
<string>APP需要您的同意，才能访问麦克风进行录音，如禁止将无法录音发送语音信息</string>
```

### 获取Camera
```
List<CameraDescription> cameras = await availableCameras();
```

### 使用
```
class CameraApp extends StatefulWidget {
  @override
  _CameraAppState createState() => _CameraAppState();
}

class _CameraAppState extends State<CameraApp> {
  CameraController controller;

  @override
  void initState() {
    super.initState();
    controller = CameraController(cameras[0], ResolutionPreset.medium);
    controller.initialize().then((_) {
      if (!mounted) {
        return;
      }
      setState(() {});
    });
  }

  @override
  void dispose() {
    controller?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (!controller.value.isInitialized) {
      return Container();
    }
    return AspectRatio(
        aspectRatio:
        controller.value.aspectRatio,
        child: CameraPreview(controller));
  }
}
```

# Widget使用

## ClipPath
ClipPath传入一个Clipper,和Child,Clipper将会用getClip返回的path来裁剪Child

示例:
```
import 'package:axj_app/generated/i18n.dart';
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';
import 'package:camera/camera.dart';

List<CameraDescription> cameras;

class AuthPage extends StatefulWidget {
  @override
  _AuthPageState createState() => _AuthPageState();
}

class _AuthPageState extends State<AuthPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(S.of(context).authTitle),
        actions: <Widget>[
          IconButton(
              icon: Icon(Icons.switch_camera),
              onPressed: () {
                switchCamera();
              })
        ],
      ),
      body: cameraReady && controllerReady
          ? Stack(
              children: <Widget>[
                Align(
                  alignment: Alignment.center,
                  child: ClipPath(
                    clipBehavior: Clip.antiAlias,
                    clipper: FaceClipper(),
                    child: AspectRatio(
                      aspectRatio: controller.value.aspectRatio,
                      child: CameraPreview(controller),
                    ),
                  ),
                )
              ],
            )
          : Center(
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: <Widget>[
                  CupertinoActivityIndicator(),
                  SizedBox(height: 32),
                  Text(S.of(context).loadingCameraHint)
                ],
              ),
            ),
    );
  }

  CameraController controller;

  bool get cameraReady => cameras != null && cameras.isNotEmpty;

  bool get controllerReady => controller?.value?.isInitialized ?? false;

  int currentCameraIndex = 1;

  @override
  void initState() {
    () async {
      if (!cameraReady) {
        cameras = await availableCameras();
      }
      await initializeCameraController();
    }();
    super.initState();
  }

  @override
  void dispose() {
    controller?.dispose();
    super.dispose();
  }

  void switchCamera() {
    if (!cameraReady) {
      return;
    }
    currentCameraIndex += 1;
    if (currentCameraIndex == cameras.length) {
      currentCameraIndex = 0;
    }
    initializeCameraController();
  }

  Future initializeCameraController() async {
    setState(() {});
    controller =
        CameraController(cameras[currentCameraIndex], ResolutionPreset.high);
    await controller.initialize();
    await Future.delayed(Duration(milliseconds: 400));
    if (!mounted) {
      return;
    }
    setState(() {});
  }
}

class FaceClipper extends CustomClipper<Path> {
  @override
  Path getClip(Size size) {
    var path = Path();

    Rect faceRect =
        Rect.fromLTWH(20, 20, size.width - 40, (size.width - 40) * 1.3);

    path.moveTo(faceRect.topCenter.dx, faceRect.topCenter.dy);

    Offset p1 = faceRect.topRight.translate(0, 0);
    Offset p2 = faceRect.centerRight.translate(-10, 0);

    path.quadraticBezierTo(p1.dx, p1.dy, p2.dx, p2.dy);

    p1 = faceRect.bottomRight.translate(-30, -25);
    p2 = faceRect.bottomCenter.translate(10, 0);

    path.quadraticBezierTo(p1.dx, p1.dy, p2.dx, p2.dy);
    p2 = faceRect.bottomCenter.translate(-10, 0);
    path.lineTo(p2.dx, p2.dy);

    p1 = faceRect.bottomLeft.translate(30, -25);
    p2 = faceRect.centerLeft.translate(10, 0);

    path.quadraticBezierTo(p1.dx, p1.dy, p2.dx, p2.dy);

    p1 = faceRect.topLeft.translate(0, 0);
    p2 = faceRect.topCenter.translate(0, 0);

    path.quadraticBezierTo(p1.dx, p1.dy, p2.dx, p2.dy);

    var rightEarRect = Rect.fromLTWH(
      faceRect.centerRight.dx - 20,
      faceRect.centerRight.dy - 35,
      20,
      60,
    );
    path.addArc(rightEarRect, -pi / 2, pi);

    path.addPolygon([
      rightEarRect.bottomCenter,
      rightEarRect.bottomLeft,
      rightEarRect.topCenter,
    ], true);

    var leftEarRect = Rect.fromLTWH(
          faceRect.centerLeft.dx,
          faceRect.centerLeft.dy - 35,
          20,
          60,
        );
    path.addArc(
        leftEarRect,
        pi / 2,
        pi);

    path.addPolygon([
      leftEarRect.topCenter,
      leftEarRect.bottomRight,
      leftEarRect.bottomCenter,
    ], true);

    path.close();
    return path;
  }

  @override
  bool shouldReclip(CustomClipper oldClipper) {
    return false;
  }
}

```
用ClipPath和自定义的CustomClipper裁剪了个人脸(大概吧.都是凑出来的)



---

## 简单封装

### Dio简单封装
之前的项目使用过Dio作为网络请求的框架,使用起来比较友好,因为是FlutterChina维护的项目,看文档和提issue都比较方便. 现在最新的版本是3.0,比我刚接触的时候成熟了不少,已经支持Multiple-part File,Post二进制流数据等功能.

我们先搞一个单例,方便全局修改dio的配置
```
class Api {
  Api._() {
    init();
  }

  static bool printLog = true;
  static Api _instance;

  static Api getInstance() {
    if (_instance == null) {
      _instance = Api._();
    }
    return _instance;
  }

  factory Api() {
    return getInstance();
  }

  Dio _dio;

  void init() {
    _dio = Dio(BaseOptions(
      method: "POST",
      connectTimeout: 10000,
      receiveTimeout: 20000,
      baseUrl: BaseUrl,
    ));
   _dio.interceptors.add(DioLogInterceptor());
  }
}
```
如图,隐藏主构造函数,提供一个没名字的工厂方法,这样获取实例的方式被简化为`Api()`

`_init`方法只执行一次,初始化Dio实例,给他配好默认方法,超时参数,还有baseUrl,然后加入了一个日志拦截器(自己写的,比较简单),方便Debug,
```
_dio = Dio(BaseOptions(
      method: "POST",
      connectTimeout: 10000,
      receiveTimeout: 20000,
      baseUrl: BaseUrl,
    ));
   _dio.interceptors.add(DioLogInterceptor());
```

再定义一个请求返回值的模板类
```
class BaseResp<T> {
  String status;
  T data;
  String token;
  String text;

  BaseResp(this.status, this.data, this.token, this.text);

  bool get success => status == "1";
}
```

在Api类中加入Post方法的封装:
```
Future<BaseResp<T>> post<T>(
    String path, {
    @required JsonProcessor<T> processor,
    Map<String, dynamic> formData,
    CancelToken cancelToken,
    ProgressCallback onReceiveProgress,
    ProgressCallback onSendProgress,
    useFormData: false,
  }) async {
    assert(processor != null);
    processor = processor ?? (dynamic raw) => null;
    formData = formData ?? {};
    cancelToken = cancelToken ?? CancelToken();
    onReceiveProgress = onReceiveProgress ??
        (count, total) {
          ///默认接收进度
        };
    onSendProgress = onSendProgress ??
        (count, total) {
          ///默认发送进度
        };
    Response resp = await _dio.post(
      path,
      options: RequestOptions(
        responseType: ResponseType.json,
        headers: {
          AuthorizationHeader: Cache().token,
        },
        contentType: useFormData
            ? ContentTypeFormDataValue
            : ContentTypeFormUrlEncodeValue,
        data: useFormData ? FormData.fromMap(formData) : formData,
      ),
      cancelToken: cancelToken,
      onSendProgress: onSendProgress,
      onReceiveProgress: onReceiveProgress,
    );
    dynamic map;
    if (resp.headers
        .value(ContentTypeHeader)
        .contains(ContentTypeTextPlainValue)) {
      map = json.decode(resp.data);
    } else {
      map = resp.data;
    }
    dynamic status = map["status"];
    dynamic text = map["text"];
    dynamic token = map["token"];
    dynamic _rawData = map["data"];
    T data;
    try {
      if (status.toString() == '1') data = processor(_rawData);
    } catch (e, s) {
      print(e);
      print(s);
    }
    return BaseResp<T>(
        status.toString(), data, token.toString(), text.toString());
  }

  Future<BaseResp<T>> get<T>(
    String path, {
    @required JsonProcessor<T> processor,
    Map<String, dynamic> queryMap,
    CancelToken cancelToken,
    ProgressCallback onReceiveProgress,
  }) async {
    assert(processor != null);
    processor = processor ?? (dynamic raw) => null;
    queryMap = queryMap ?? {};
    cancelToken = cancelToken ?? CancelToken();
    onReceiveProgress = onReceiveProgress ??
        (count, total) {
          ///默认接收进度
        };
    Response resp = await _dio.get(
      path,
      queryParameters: queryMap,
      options: RequestOptions(
          responseType: ResponseType.json,
          headers: {
            AuthorizationHeader: Cache().token,
          },
          queryParameters: queryMap,
          contentType: ContentTypeFormUrlEncodeValue),
      cancelToken: cancelToken,
      onReceiveProgress: onReceiveProgress,
    );
    dynamic map;
    if (resp.headers
        .value(ContentTypeHeader)
        .contains(ContentTypeTextPlainValue)) {
      map = json.decode(resp.data);
    } else {
      map = resp.data;
    }
    dynamic status = map["status"];
    dynamic text = map["text"];
    dynamic token = map["token"];
    dynamic _rawData = map["data"];
    T data;
    try {
      if (status.toString() == '1') data = processor(_rawData);
    } catch (e,s) {
      print(e);
      print(s);
    }
    return BaseResp<T>(
        status.toString(), data, token.toString(), text.toString());
  }
```

当然还有定义好json数据处理函数
```
typedef JsonProcessor<T> = T Function(dynamic json);
```


### FutureBuilder与网络请求

FutureBuilder是Flutter Framework中提供的异步构建视图的一个Widget,提供给它一个Future类型的数据,它会执行future任务封装成AsyncSnapshot返回,并在error,success情况下rebuild.我们的网络工具一般都是封装成async方法,也就是返回值为Future的方法,我们可以给网络请求到视图显示这一步加入转场特效还有错误处理/重试功能.

首先我们的FutureBuilder
