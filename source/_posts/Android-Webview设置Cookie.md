---
title: Android Webview设置Cookie
tags:
  - Android
originContent: ''
categories:
  - 笔记
toc: false
date: 2019-12-18 20:53:48
---

Android中的webview相当于在App中新开了一个浏览器客户端，所以cookie不会和App的普通网络请求同步，需要我们手动吧cookie设置到webview中（如果需要用到cookie的话）

首先我们要从App的普通请求的返回中获取cookie：

不同的请求方式取cookie的方式可能有所不同，项目中以volley为例：

在volley请求中一般需要定义一个Request对象继承自com.android.volley.Request

其中需要覆写多个方法，包括com.zhaosha.zsnetservice.util.CookiePostRequest#parseNetworkResponse

这个方法返回一个NetworkResponse对象，我们需要从这个对象中取出cookie保存到sp或者文件中

~~~

//首先拿到response.data，和header

String jsonString =

new String(response.data, HttpHeaderParser.parseCharset(response.headers));

mHeader = response.headers.toString();

Log.w("LOG", "get headers in parseNetworkResponse " + response.headers.toString());

//使用正则表达式从reponse的头中提取cookie内容的子串

Pattern pattern = Pattern.compile("Set-Cookie.*?;");

Matcher m = pattern.matcher(mHeader);

if (m.find()) {

cookieFromResponse = m.group();

cookieFromResponse = cookieFromResponse.substring(11,                  cookieFromResponse.length() - 1);

if(cookieFromResponse.contains("COOKIE_ID_HERE")){//如果包含自己设定cookie_ID就保存

//保存Cookie，保存到sp或者文件都可以，自己实现

saveCookie(context,cookieFromResponse);

}

}

//将cookie字符串添加到jsonObject中，该jsonObject会被deliverResponse递交，调用请求时则能在onResponse中得到

JSONObject jsonObject = new JSONObject(jsonString);

return Response.success(jsonObject,HttpHeaderParser.parseCacheHeaders(response));

~~~

然后在需要用到cookie的webview中把保存的cookie同步

一般是在设置的WebviewClient中覆写shouldInterceptRequest方法，这个方法会在webview加载url之前调用之前调用

~~~

//获取cookie，自己实现

String cookie= getCookie(this);

if(Build.VERSION.SDK_INT< Build.VERSION_CODES.LOLLIPOP) {

CookieSyncManager.createInstance(this);

}

CookieManager cookieManager = CookieManager.getInstance();

cookieManager.removeAllCookie();

if(!cookieManager.hasCookies()){

cookieManager.setCookie(url,cookie);//如果没有特殊需求，这里只需要将session id以"key=value"形式作为cookie即可

}

~~~

这样webview就会用新的sessionid和服务器进行通信了，