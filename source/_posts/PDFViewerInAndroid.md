---
title: PDFViewer on Android
date: 2019-12-11 21:11:20
tags: 
    - Android
---
# Android中显示PDF

iOS的WebView能从线上url直接显示pdf,而Android的WebView不能直接显示,
Android的WebView要显示pdf需要拼接url到google的一个url显示,国内需要翻墙...所以这条路肯定走不通了
~~~
urlWebView.loadUrl("http://docs.google.com/gview?embedded=true&url="  
                   + "YOUR_DOC_URL_HERE");
~~~

---

github上有一个控件形式的PDF显示器PDFView
https://github.com/barteksc/AndroidPdfViewer/issues
可以通过Uri / File / byte数组 /输入流 /DocumentSource 几个形式显示PDf到PDFView上
并且支持许多回调和自定义设置

----

## 可以用HttpURLConnection从url中获取流,再传入PdfView显示
* 在xml中使用
~~~
<com.github.barteksc.pdfviewer.PDFView
        android:id="@+id/pdfView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
~~~
* 通过流读取pdf到View
~~~
url = new URL(url);
            urlConnection = (HttpURLConnection) url.openConnection();
            urlConnection.setConnectTimeout(30000);
            urlConnection.setReadTimeout(30000);
            urlConnection.setRequestMethod("GET");
            urlConnection.connect();
            if (urlConnection.getResponseCode() == 200) {
                PDFView.Configurator configurator = vPdfView.fromStream(urlConnection.getInputStream())
                        .enableSwipe(true) // allows to block changing pages using swipe
                        .swipeHorizontal(false)
                        .enableDoubletap(true)
                        .defaultPage(0)
                        .onLoad(nbPages -> showLoading(false))
                        // allows to draw something on the current page, usually visible in the middle of the screen
                        .enableAnnotationRendering(false) // render annotations (such as comments, colors or forms)
                        .password(null)
                        .scrollHandle(null)
                        .enableAntialiasing(true) // improve rendering a little bit on low-res screens
                        // spacing between pages in dp. To define spacing color, set view background
                        .spacing(15)
                        .load();
~~~

> 注意把网络请求放在子线程中进行,PdfView的load()方法需要放在主线程中进行,否则没法显示pdf,

## PDFView的Configurator设置
~~~
.pages(0, 2, 1, 3, 3, 3) // 选择显示那些page
    .enableSwipe(true) // 是否允许滑动切换page
    .swipeHorizontal(false)//水平滑动
    .enableDoubletap(true)//双击放大
    .defaultPage(0)//默认显示page
    // allows to draw something on the current page, usually visible in the middle of the screen
    .onDraw(onDrawListener)//可以在onDraw中绘制自定义图形到当前页
    // allows to draw something on all pages, separately for every page. Called only for visible pages
    .onDrawAll(onDrawListener)//绘制自定义图形到所有页面
    .onLoad(onLoadCompleteListener) // 在加载完成后调用,调用后开始onRender
    .onPageChange(onPageChangeListener)//page切换监听
    .onPageScroll(onPageScrollListener)//page滚动监听
    .onError(onErrorListener)//错误时回调
    .onPageError(onPageErrorListener)//page加载错误
    .onRender(onRenderListener) // called after document is rendered for the first time 首次渲染后回调
    // called on single tap, return true if handled, false to toggle scroll handle visibility
    .onTap(onTapListener)//单击监听
    .enableAnnotationRendering(false) // render annotations (such as comments, colors or forms) 是否渲染注解:评论/颜色/forms
    .password(null)
    .scrollHandle(null)
    .enableAntialiasing(true) // improve rendering a little bit on low-res screens
    // spacing between pages in dp. To define spacing color, set view background
    .spacing(0) //page间隔
    .linkHandler(DefaultLinkHandler) //pdf中链接处理
    .pageFitPolicy(FitPolicy.WIDTH) //page适应策略,宽度优先还是高度优先 3.0.0才有
    .load()//加载pdf
~~~

> 
注意处理与Activity生命周期相关比如缓存和数据清理
