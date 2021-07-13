---
title: JetPack-Navigation中Fragment的返回键拦截
tags:
  - Android
  - Fragment
  - JetPack
categories:
  - 笔记
toc: false
date: 2020-10-28 15:54:26
---

# 返回键拦截
Activity中默认的返回键拦截方式是override Activity.onBackPressed(),当我们使用Navigation库或者其他一些全部使用Fragment作为主要View组件的情况下,我需要Fragment自己拦截返回键,典型的使用案例就是使用Jetpack-Navigation库的一个WebView页面,WebView包含在一个Fragment之内,我们需要在用户按下返回键时判断:webview是否可以goBack,可以的话就goBack,否则调用默认的back时间处理方法.

# 来自androidx库的解决方案
ComponentActivity(相信现在大部分的App使用这个Activity的子类,例如:FragmentActivity,AppCompatActivity),它提供了一个OnBackPressedDispatcher(`ComponentActivity.getOnBackPressedDispatcher() `),我可以看看developer网站上的说法:

> OnBackPressedDispatcher 控制将返回按钮事件分派给一个或多个 OnBackPressedCallback 对象的方式。OnBackPressedCallback 的构造函数利用布尔值来指示初始启用状态。仅当回调处于启用状态（即 isEnabled() 返回 true）时，调度程序才会调用回调的 handleOnBackPressed()，以处理返回按钮事件。您可以通过调用 setEnabled() 更改启用状态。

> 回调通过 addCallback 方法添加。强烈建议您使用 addCallback() 方法，该方法采用了 LifecycleOwner。这可以确保仅在 LifecycleOwner 为 Lifecycle.State.STARTED 时添加 OnBackPressedCallback。此外，该 Activity 还会在与注册回调相关联的 LifecycleOwner 被销毁时移除注册回调。这不仅可以防止内存泄露，还可以使其适用于生命周期短于该 Activity 的 Fragment 或其他生命周期所有者。

> 以下是一个回调实现示例：

> 您可以通过 addCallback() 提供多个回调。如果这样做，回调的调用顺序与其添加顺序相反，即最后添加的回调是第一个有机会处理返回按钮事件的回调。例如，如果您按顺序添加了三个名为 one、two 和 three 的回调，则它们的调用顺序分别为 three、two 和 one。

> 回调遵循责任链模式。责任链中的每个回调仅在前面的回调处于未启用状态时调用。这意味着，在前面的示例中，仅当 three 回调处于未启用状态时，系统才会调用 two 回调。仅当 two 回调处于未启用状态时，系统才会调用 one 回调，以此类推。

> 请注意，如果回调是通过 addCallback() 添加的，则在 LifecycleOwner 进入 Lifecycle.State.STARTED 状态之前，它不会添加到责任链中。

> 对于临时性更改，强烈建议您更改 OnBackPressedCallback 的启用状态，因为它会维护上文所述的排序。如果您在多个不同的嵌套生命周期所有者上注册了回调，这一点尤为重要。

> 但是，如果您想要彻底移除 OnBackPressedCallback，则应调用 remove()。不过，这通常是没有必要的，因为回调会在与其关联的 LifecycleOwner 被销毁时自动移除。


所以典型的使用方法就是在需要的Fragment里添加Callback,当然你也可以在自己的Fragment基类里添加公共方法.
```kotlin
 requireActivity().onBackPressedDispatcher.addCallback(
            this,
            object :OnBackPressedCallback(true){
                override fun handleOnBackPressed() {
                    if(webView.canGoBack()){
                        webView.goBack()
                    }else{
 			pop()
                    }
                }
            }
        )
```

# 其他依赖Activity具体实现的方法
既然Jetpack推荐整个App使用一个Activity加多个Fragment来组织视图,我们可以让Fragment实现一个公共BackPress接口,然后在Activity的onBackPressed中处理栈顶Fragment的返回事件,这个方法应该网上有很多例子,大家可以参考:[两步搞定Fragment返回键](https://www.jianshu.com/p/fff1ef649fc0)