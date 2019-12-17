---
title: DataBinding in kotlin
date: 2019-12-11 21:11:20
tags:  
    - Kotlin
    - DataBinding
    - Android 
---
# DataBinding in kotlin

### DataBinding 引入
* Gradle引入
```
apply plugin: 'kotlin-kapt'//需要使用kapt作为注解 处理器
kapt {
    generateStubs = true
}
  android{
    ....
    dataBinding {
        enabled = true
    }
  }
  dependencies{
    ///...
    kapt "com.android.databinding:compiler:3.0.1"//dataBinding需要的编译处理工具
  }
```

### 使用

* 使用普通的xml定义,以`layout`标签作为顶层标签,次级定义`import`/`variable`等
```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    >
    <data>
        <variable
            name="bean"
            type="bean类的全路径,例如com.example.app.bean.MyBean"/>
    </data>
    <!-- 这里是原来的layout -->
    <TextView
      android:text="@{bean.name}"
      />
</layout>
```

* 简单的绑定（单向绑定）

  * 1.先编写一个kotlin的bean类(可以直接用data class)
    ```kotlin
      data class MyBean(
          var name:String
      )
    ```
  * 2.在xml中使用`@{bean.field}`
  * 3.在Activity/Fragment中设置：
  ```kotlin
  val applicantInfoBinding = DataBindingUtil.setContentView<ActivityMyBeanBinding>(this, R.layout.activity_my_bean)
  ```
  其中ActivityMyBeanBinding是Databinding生成的类,继承自`android.databinding.ViewDataBinding `
  返回值就是一个ActivityMyBeanBinding对象,会在之后用到,这个类中我们会用到它保存的View的引用,以及我们引入的Bean对象,也就是
  `variable`定义的Bean,当然还有他的父类的方法
    * `public abstract boolean setVariable(int variableId, Object value);`//设置field值
    * `public void executePendingBindings() `//立即把bindings刷新
    * 还有一些其他方法
* 双向绑定
  * 1.改造下kotlin类
    * 首先,需要双向绑定的类需要继承`BaseObservable`
    * 双向绑定属性的get方法需要添加@Bindable注解,这样在set方法调用`notifyPropertyChanged`方法时UI会重新获取值
    * 属性的set方法最后调用`notifyPropertyChanged`方法
    * 如果一个属性需要逻辑处理为其他类型比如Int->String,可以写setXXX/getXXX方法,XXX可以自己定义但是get/set方法要对应,然后我们在xml中可以以`@={bean.XXX}`的形式双向绑定该XXX
  * 2.普通的属性
  ```kotlin
  @Bindable//这个bindable可以只放在get方法
   var id: Int = 0
       set(value) {
           field = value
           println("profile id set $field")
           notifyPropertyChanged(BR.id)
       }
       get() {
           println("profile id get $field")
           return field
       }
  ```
  * 3.自定的属性
  ```kotlin
    var repayment_type: Int = 0//back field

   @Bindable
   fun getRepaymentTypeString(): String {
       return "XXXXXXXXXXX"
   }

   fun setRepaymentTypeString(str: String) {
     //设置backfield的值
       if(str=="XXXXXXXXXXX")
        repayment_type = 0
      else {
        repayment_type = 10
      }
       notifyPropertyChanged(BR.repaymentTypeString)//通知更新
   }
   ```
