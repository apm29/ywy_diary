---
title: Navigation初探
date: 2019-12-11 21:11:20
tags: 
    - Android
---
## Navigation初探

#### Usage
> Gradle引入
* 写的时候版本还是1.0.0-alpha1

```
   implementation "android.arch.navigation:navigation-fragment:$versions.navigation"
```

```
   implementation "android.arch.navigation:navigation-ui:$versions.navigation"
```

> 修改Layout文件
  * 新增一个Fragment,指定道航文件的定义`app:navGraph="@navigation/nav_graph"`
  * 指定Fragment的name `android:name="androidx.navigation.fragment.NavHostFragment"` 固定的
```xml
<?xml version="1.0" encoding="utf-8"?>
  <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
        <fragment
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/my_nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        app:navGraph="@navigation/nav_graph"
        app:defaultNavHost="true"
        />
  </FrameLayout>
```

> 新建刚才指定的naviGraph文件
* 在res目录下新建一个navigation文件夹,新建一个navi_graph.xml文件
* 根节点navigation,指定初始的fragment`  app:startDestination="@+id/launcher_home"`
* 子节点`fragment`定义页面,`id`就是导航的页面标的,`name`是Fragment的全路径
* `action`是`fragment`的子节点,定义了跳转的动作,通过action的`id`来寻找页面并导航到id标定的fragment,


```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
~ Copyright (C) 2017 The Android Open Source Project
~ Licensed under the Apache License, Version 2.0 (the "License");
~ you may not use this file except in compliance with the License.
~ You may obtain a copy of the License at
~      http://www.apache.org/licenses/LICENSE-2.0
~ Unless required by applicable law or agreed to in writing, software
~ distributed under the License is distributed on an "AS IS" BASIS,
~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
~ See the License for the specific language governing permissions and
~ limitations under the License.
-->
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:app="http://schemas.android.com/apk/res-auto"
          app:startDestination="@+id/launcher_home">

            <fragment android:id="@+id/launcher_home"
              android:label="Home"
              android:name="com.shirly.apm29.shirlyq.ui.main.MainFragment" >

              <action android:id="@+id/end_action" app:destination="@id/end_dest"/>

            </fragment>

            <fragment android:id="@+id/end_dest"
              android:label="End"
              android:name="com.shirly.apm29.shirlyq.ui.end.EndFragment" >
              <action android:id="@+id/start_action" app:destination="@id/launcher_home"/>
            </fragment>
</navigation>
```

> 最后在代码中进行导航
* 通过Navigation.findNavController方法找到导航控制器,navigate对应的action的id就会导航到对应的Fragment

```kotlin
class MainFragment : Fragment() {


    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
                              savedInstanceState: Bundle?): View {
        return inflater.inflate(R.layout.main_fragment, container, false)
    }

    override fun onActivityCreated(savedInstanceState: Bundle?) {
        super.onActivityCreated(savedInstanceState)

        button.setOnClickListener {
            // Navigate to the login destination
            view?.let { Navigation.findNavController(it).navigate(R.id.end_action) }
        }

    }
}

```
