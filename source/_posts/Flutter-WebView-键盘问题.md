---
title: Flutter WebView 键盘问题
tags:
  - Android
  - Flutter
categories:
  - 笔记
toc: false
date: 2019-12-19 04:49:49
---

Flutter_Webview 键盘弹出问题

---

`webview_flutter ^0.3.7+1` [pub链接](https://pub.flutter-io.cn/packages/webview_flutter)

> webview_flutter在Android上没有办法弹出键盘,github上的issue已经提了很久,但是官方的milestone还要到19年的十月 [issue #19718](https://github.com/flutter/flutter/issues/19718#issuecomment-499756410),(截止发稿时已经有一个PR到master分支了,但是stable分支的同学可能就还要等一哈了),但是PR的解决方案在AndroidN之前并没有用...[comment](https://github.com/flutter/flutter/issues/19718#issuecomment-499759204)



#### 1.来自其他同学的启发

> [隐藏TextField方案](https://github.com/flutter/flutter/issues/19718#issuecomment-480902121),这个方案的简单思路就是在`onPageFinish`时给Webview注入一段js代码,监听input/textarea的focus事件,然后通过JSChannel发送给隐藏在WebView之后的TextField,通过TextField间接唤起软键盘然后,通过监听TextField的onChange事件给input/textarea设置新的值

下面是他的JS代码实现:他用js监听某个网站的几个需要弹出输入法的element,focus事件/focusout事件

```javascript
inputs = document.getElementsByClassName('search-bar');
                            header=document.getElementsByClassName('site-header');
                            header[0].style.display = 'none';
buttons = document.getElementsByClassName('icon');
                            buttons[0].focus();
                            if (inputs != null) {
                              input = inputs[0];
                              InputValue.postMessage(input.value);
                              input.addEventListener('focus', (_) => {
                                Focus.postMessage('focus');
                              }, true);
                              input.addEventListener('focusout', (_) => {
                                Focus.postMessage('focusout');
                              }, true)
                            }
```

JSChannel:

```dart
 javascriptChannels: Set.from(
                          [
                            // Listening for Javascript messages to get Notified of Focuschanges and the current input Value of the Textfield.
                            JavascriptChannel(
                              name: 'Focus',
                              // get notified of focus changes on the input field and open/close the Keyboard.
                              onMessageReceived: (JavascriptMessage focus) {
                                print(focus.message);
                                if (focus.message == 'focus') {
                                  FocusScope.of(context)
                                      .requestFocus(_focusNode);
                                } else if (focus.message == 'focusout') {
                                  _focusNode.unfocus();
                                }
                              },
                            ),
                            JavascriptChannel(
                              name: 'InputValue',
                              // set the value of the native input field to the one on the website to always make sure they have the same input.
                              onMessageReceived: (JavascriptMessage value) {
                                _textController.value =
                                    TextEditingValue(text: value.toString());
                              },
                            )
                          ],
                        ),
```

接收事件更改TextField的text,通过focusNode来唤起/隐藏 软键盘

这个方案看起来很好,但是手写监听所有Classname可太笨了...而且如果用户在弹出键盘后手动关闭键盘(按软键盘上的关闭按钮),软键盘就再也弹不出来了...



#### 2.自己的方案升级

 1. 用两个TextField交替接收事件来解决无法在手动关闭键盘后重新唤起的问题
 2. 注入JS监听所有input/textarea element,不监听focus/focusout事件,监听click事件,有几个好处
      	1. 可以获取用户当前文字选择/光标位置
      	2. 可以判断再次点击事件
 3. 记录当前focus的element,用于判断是否需要重新唤起新的键盘

下面的简单的代码实现:

```javascript
var inputs = document.getElementsByTagName('input');
var textArea = document.getElementsByTagName('textarea');
var current;

for (var i = 0; i < inputs.length; i++) {
  console.log(i);
  inputs[i].addEventListener('click', (e) => {
    var json = {
      "funcName": "requestFocus",
      "data": {
        "initText": e.target.value,
        "selectionStart":e.target.selectionStart,
        "selectionEnd":e.target.selectionEnd
      }
    };
    json.data.refocus = (document.activeElement == current);
    
    current = e.target;
    var param = JSON.stringify(json);
    console.log(param);
    UserState.postMessage(param);
  })
  
}
for (var i = 0; i < textArea.length; i++) {
  console.log(i);
  textArea[i].addEventListener('click', (e) => {
    var json = {
      "funcName": "requestFocus",
      "data": {
        "initText": e.target.value,
        "selectionStart":e.target.selectionStart,
        "selectionEnd":e.target.selectionEnd
      }
    };
    
    json.data.refocus = (document.activeElement == current);
    
    current = e.target;
    var param = JSON.stringify(json);
    console.log(param);
    
    UserState.postMessage(param);
  })
};
console.log('===JS CODE INJECTED INTO MY WEBVIEW===');
```



UserState是定义好的jsChannel,接收一个jsonString形式的参数,data.initText是初始文字,selectEnd selectStart是文字选择位置,refocus是判断是否重新点击focus的Element的标记,element被点击时:

	*  先判断是不是refocus,发送给channel

接下来要处理JSChannel收到的数据了,其中showAndroidKeyboard是自己编写的一个PlatformChannel,用来显示软键盘,代码还可以简化,我这里就不写了,有改进的请和作者联系…也是在踩坑中.

* 关键是TextField交替获取焦点,当TextField - A获取焦点,进行一番编辑之后,用户手动关闭了软键盘,这个时候FocusScop.of(context).requestFocus(focusNodeA)是无法唤起键盘的,所以需要另一个TextField-B来请求焦点,
* showAndroidKeyboard()是为了在某些情况下备用focusNode也没获取焦点,需要我们手动唤起焦点
* 隐藏键盘的事件还没有很好的解决方案

```dart
bool refocus = data['refocus'] as bool;
    if (_focusNode2.hasFocus) {
      if (!refocus) FocusScope.of(context).requestFocus(_focusNode1);
      //FocusScope.of(context).requestFocus(_focusNode);
      if (!_focusNode1.hasFocus) {
        //hideAndroidKeyboard();
        showAndroidKeyboard();
      }
      //把初始文本设置给隐藏TextField
      String initText = data['initText'];
      var selectionStart = data['selectionStart'];
      var selectionEnd = data['selectionEnd'];

      int end = initText.length;
      int start = initText.length;
      if (selectionEnd is int) {
        end = selectionEnd;
      }
      if (selectionStart is int) {
        start = selectionEnd;
      }
      print(selectionEnd);
      textController1.value = TextEditingValue(
        text: initText,
        selection: TextSelection(baseOffset: start, extentOffset: end),
      );
      //TextField请求显示键盘
      FocusScope.of(context).requestFocus(_focusNode1);
    } else {
      if (!refocus) FocusScope.of(context).requestFocus(_focusNode2);
      //FocusScope.of(context).requestFocus(_focusNode);
      if (!_focusNode2.hasFocus) {
        //hideAndroidKeyboard();
        showAndroidKeyboard();
      }
      //把初始文本设置给隐藏TextField
      String initText = data['initText'];
      var selectionStart = data['selectionStart'];
      var selectionEnd = data['selectionEnd'];

      int end = initText.length;
      int start = initText.length;
      if (selectionEnd is int) {
        end = selectionEnd;
      }
      if (selectionStart is int) {
        start = selectionEnd;
      }
      print(selectionEnd);
      textController2.value = TextEditingValue(
        text: initText,
        selection: TextSelection(baseOffset: start, extentOffset: end),
      );
      //TextField请求显示键盘
      FocusScope.of(context).requestFocus(_focusNode2);
    }
```

webview后面隐藏的TextField



```dart
return Stack(
                  children: <Widget>[
                    TextField(
                      autofocus: false,
                      focusNode: _focusNode1,
                      controller: textController1,
                      onChanged: (text) {
                        controller.evaluateJavascript('''
                         if(current != null){
                          current.value = '${textController1.text}';
                          current.selectionStart = ${textController1.selection.start};
                          current.selectionEnd = ${textController1.selection.end};
                          current.dispatchEvent(new Event('input'));
                         }
                        ''');
                      },
                      onSubmitted: (text) {
                        controller.evaluateJavascript('''
                        if(current != null)
                          current.submit();
                        ''');
                        _focusNode1.unfocus();
                      },
                    ),
                    TextField(
                      autofocus: false,
                      focusNode: _focusNode2,
                      controller: textController2,
                      onChanged: (text) {
                        controller.evaluateJavascript('''
                         if(current != null){
                          current.value = '${textController2.text}';
                          current.selectionStart = ${textController2.selection.start};
                          current.selectionEnd = ${textController2.selection.end};
                          current.dispatchEvent(new Event('input'));
                         }
                        ''');
                      },
                      onSubmitted: (text) {
                        controller.evaluateJavascript('''
                        if(current != null)
                          current.submit();
                        ''');
                        _focusNode2.unfocus();
                      },
                    ),
                    WebView(....)
                  ]
  );
```



* 在TextField的onChange中改编当前focus的Element的text/selection,然后发送一个input事件来触发Element改变`current.dispatchEvent(new Event('input'));`,这里的大家应该都很好理解了.



> 不得不说谷歌太坑了,webview竟然有这样的大缺陷,不过也没办法毕竟版本号还没到1.0.0呢…本文的方法只是一个临时的解决方案,比如点击空白隐藏软键盘等功能还没实现,只使用一些简单的文本输入的webview还是可以的,还有一些input标签作为按钮的可能还要手动隐藏下键盘,以上.