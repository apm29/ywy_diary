---
title: Android Text measure/draw
tags:
  - Android
categories:
  - 笔记
toc: false
date: 2019-12-19 04:52:46
---

# 字符串绘制的字符高度以及行高
![fontMetrics.png](https://upload-images.jianshu.io/upload_images/5133402-4faa3bb2e82dab41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![实例](http://upload-images.jianshu.io/upload_images/5133402-73c2905b827cb70d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 

# 字符高度
> 我们在绘制文字的时候,需要确定绘制的字符高度以免发生重叠等问题,最简单的方式是通过Paint的fontMetrics来确定,如上图所示,一个文字有五个属性

* top 文字`绘制行`顶部相对baseLine的y值
* ascent `单个文字`顶部相对baseLine的y值
* baseLine 文字绘制的y轴基点,用`canvas.drawText(String text, float x, float y, Paint paint)`方法绘制的时候,参数`y`就是绘制文字的baseLine
* bottom 文字`绘制行`底部相对baseLine的y值
* descent `单个文字`底部相对baseLine的y值
* 还有一个leading属性,是文字行间距

> 所以很明显的,字符高度就是`descent` - `ascent`

# 行高
> 行高就是`bottom` - `top` + `leading`

# 绘制
![progress.jpg](https://upload-images.jianshu.io/upload_images/5133402-ce58c7ee8f01f41e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 用`canvas.drawText(String text, float x, float y, Paint paint)`方法绘制的时候
需要确定baseLine,比如我需要绘制一个垂直进度图,需要计算高度生成`MeasureSpec`,从上往下计算:`textY`
*  `textY += paddingTop`
*  text高度=` bottom-top + leading`
*  自己定义的进度文字之间的间距`D`*M个
*  text高度*N个...
*  `textY += paddingBottom`

> 绘制的时候
计算baseLine的位置abs(ascent)