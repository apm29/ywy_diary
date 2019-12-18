---
title: LiveData
tags:
  - Android
categories: []
toc: false
date: 2019-12-17 06:13:22
---

* # 简介
  * `LiveData`是androidx.lifecycle-livedata包下的一个抽象类,实现了一种关联了生命周期的简单观察者模式,主要的功能就是用于视图层与数据间的单向一对多通知,一个`LiveData`会持有一个可观察的Data对象,一开始是处于`NOT_SET`状态,当用户通过setValue方法更新后,`LiveData`会遍历所有的observer(处于Active状态的),通知他们Data的更新.`LiveData`的行为会在`Lifecycle`范围内,避免很多Activity Leak和空指针 **本文代码样例都是kotlin,LiveData的源码是java**
* # LiveData的使用
  ```kotlin
    //声明一个LiveData
    val data:MutableLiveData<String> = MutableLiveData()
    //在需要的地方订阅,observer的onChange方法会回调
    data.observe(lifecycleOwner,observer)
    //可以和retrofit配合使用
   interface ApiService {
      @GET("/api/get")
      fun getChannels():LiveData<ApiResponse<Page<Media>>>
    }
    //可以和room配合使用
    @Dao
   interface BorrowModelDao {

        @Query("select * from s_borrow_moel")
       fun getAllBorrowedItems(): LiveData<List<BorrowModel>>

    }
    //配合ViewModel保存数据/监听数据更改
    class MyViewModel:ViewModel(){
      val data:MutableLiveData<String> = MutableLiveData()
    }
  ```
* # LiveData Tricks
  * Transformations.map方法转换LiveData的数据类型  
  比如我们用Retrofit获取了一个User类数据:LiveData<User>,但是我们只想把User.name暴露给外部观察者,那么我们可以这样操作
  ```java
  private val userLiveData:MutableLiveData<User> = MutableLiveData()
  val userNames:LiveData<String> = Transformations
  .map(userLiveData){
      user ->
      user.name
  }
  ```
  > Transformations在androidx.lifecycle包下

  * MediatorLiveData  
    MediatorLiveData可以同时观察多个LiveData的变化
    ```kotlin
      //声明
      val mediatorLiveData:MediatorLiveData<String> = MediatorLiveData()
      //其他合适的地方添加source
      mediatorLiveData.addSource(stringFromNetwork){
          mediatorLiveData.value = it
      }
      mediatorLiveData.addSource(stringFromDatabase){
          mediatorLiveData.value = it
      }
    ```
* # 配合ViewModel使用
  * 我们知道`ViewModel`可以在一个`LifecycleOwner`的`onCreate`到`OnDestroy`之间保存实例不受到屏幕旋转等ConfigChange的影响  
所以可以通过`Activity/Fragment`持有`ViewModel`,而`ViewModel`持有各类`LiveData`,`Activity/Fragment`注册观察需要的数据,实现数据与UI的同步,而且不会因此发生`Activity`泄漏,甚至可以用`ViewModel`来进行`Activity/Fragment`之间的通讯
* # 关于Lifecycle
  * ## Lifecycle组成
    * `LifecycleOwner`: `Activity/Fragment`都是`LifecycleOwner`(support包下的AppCompatActivity和对应的Fragment),他是androidx.lifecycle:lifecycle-common包下的一个借口,只有getLifecycle()一个方法,返回一个Lifecycle对象
    * `Lifecycle`: 而这个`Lifecycle`就是维护生命周期state的主体,默认实现是`LifecycleRegistry`,这个说起来也挺长的,以后有空再新的文章里仔细说明,反正现在读者只要知道我们常用的`AppCompatActivity/Fragment`都是可以拿到生命周期Lifecycle的就行了
    * `LifecycleObserver`: 也是一个接口,不过它一个抽象方法也没有,如果我们需要一个新的自定义的观察
     者,只要实现这个接口,然后在需要对应生命周期方法回调的方法上添加  
     `OnLifecycleEvent`注解并添加对应注解就好了,`LiveData`有一个
     `LifecycleBoundObserver`内部类实现了这个接口,不过继承关系是这样的
     `LifecycleObserver`  <--`GenericLifecycleObserver`  <- `LifecycleBoundObserver`,  
    `LifecycleBoundObserver`会在`onStateChange`收到回调,不依赖于注解,不过读
     者现在只要知道在`Lifecycle`中`state`发生改变的时候,`LifecycleObserver`会收
     到通知(`void onStateChanged(LifecycleOwner source, Lifecycle.Event event);`)  

![xianyu.gif](https://upload-images.jianshu.io/upload_images/5133402-e6115a9f98a3da14.gif?imageMogr2/auto-orient/strip)
* ## LiveData类分析
  * 简单的类结构  
     有几个内部类,都是ObserverWrapper的子类,用于普通的观察者(带生命周期)的`LifecycleBounderObserver`,和不带的`AlwaysActiveObserver`..

  ```java
    abstract class LiveDate{

      class LifecycleBounderObserver extends ObserverWrapper implements GenericLifecycleObserver{

      }

      private abstract ObserverWrapper{

      }

      private class AlwaysActiveObserver extends ObserverWrapper {

      }
    }
  ```
* ## observe入手分析
  * 传入参数: `LifecycleOwner` , `Observer`类(简单的回调通知接口)
  * 首先通过`LifecycleOwner`获取`Lifecycle`,判断时候是`DESTROYED`状态,是的话直接return
将`Observer`和`LifecycleOwner`包装为`LifecycleBoundObserver`对象
  * LiveData维护了一个`SafeIterableMap<Observer<? super T>, ObserverWrapper>` 对象mObservers,保存了Observer和对应的Wrapper,存入LifecycleBoundObserver对象后判断是否已存在并且是否对应当前LifecycleOwner,做出相应的处理,当mObservers中没有时,直接给LifecycleOwner的Lifecycle注册该LifecycleBoundObserver
  * 所以说实际上是我们传入的LifecycleOwner保存的Lifecycle注册了一个观察者,LifecycleBoundObserver会在state 等于 DESTROYED的时候移除该观察者,在其他状态的时候调用activeStateChange()方法
  * 先比较当前mActive和新active,赋值给`mActive`,然后通过LiveData类的mActivieCount是否为0来判断是不是完全没有通知过,如果是进入非active状态mActiveCount就-1,否则+1,原先未active现在active了,调用`onActive`方法,新的`mActivieCount == 0 `并且 新active的状态为false就调用`onInactive`方法,然后判断`mActive`,true则会调用dispatchValue方法,基本上mActivieCount只会有0 -> 1 ,1 -> 0两个状态变化,所以`onActive`是LiveData观察者数量从无到有时回调,`onInactive`反之
    ```java
    private abstract class ObserverWrapper {
     final Observer<? super T> mObserver;
     boolean mActive;
     int mLastVersion = START_VERSION;
     void activeStateChanged(boolean newActive) {
         if (newActive == mActive) {
             return;
         }
         // immediately set active state, so we'd never dispatch anything to inactive
         // owner
         mActive = newActive;
         boolean wasInactive = LiveData.this.mActiveCount == 0;
         LiveData.this.mActiveCount += mActive ? 1 : -1;
         if (wasInactive && mActive) {
             onActive();
         }
         if (LiveData.this.mActiveCount == 0 && !mActive) {
             onInactive();
         }
         if (mActive) {
             dispatchingValue(this);
         }
     }
 }
    ```
  * 上面罗里吧嗦一大推读者可能一脸懵逼,反正结论就是,LiveData状态由没有观察者到有观察者会走`onActive`方法,反之会走`onInactive`方法,不管原先状态如何,只要新状态是活跃就会走`dispatchValue`
  * `onInactive/onActive` 是`LiveData`里两个open protected的方法,只在MediatorLiveData里有实现,我还没有仔细研究MediatorLiveData,大家可以去看看源码
  * 所以LifecycleBoundObserver新状态为活跃时,会调用外部类`LiveData`的`dispatchValue`方法,传入的参数为`ObserverWrapper`,也就是LifecycleBoundObserver的父类(`LifecycleBoundObserver`继承了`ObserverWrapper`,实现了`GenericLifecycleObserver`),在这要注意下内部类(非静态)是可以拿到外部类实例的,LiveData.this.XXX,忘记的同学复习下,我是好久没用java,突然看到有点懵逼..
  ```java
  //传入参数是Observer,有的话就分发给它,为null就对所有Observers分发数据
  void dispatchingValue(@Nullable ObserverWrapper initiator) {
    //如果正在分发数据,就标记为分发无效,分发中的会因此跳出循环
     if (mDispatchingValue) {
       //标记为分发无效
         mDispatchInvalidated = true;
         return;
     }
     //标记为分发中
     mDispatchingValue = true;
     do {
         mDispatchInvalidated = false;
         if (initiator != null) {
             considerNotify(initiator);
             initiator = null;
         } else {
             for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                     mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                  //尝试分发
                 considerNotify(iterator.next().getValue());
                 //标记为无效时会跳出循环,在while循环中等待一次mDispatchInvalidated再次变为true时再次进入for循环
                 if (mDispatchInvalidated) {
                     break;
                 }
             }
         }
         //没有被标记无效时while循环就执行一次完事,否则会再次执行
     } while (mDispatchInvalidated);
     mDispatchingValue = false;
 }
  ```
  * `LiveData`中的`mDispatchingValue/mDispatchInvalidated`在此处保证不会重复分发或者分发旧的value,当`setValue`分发调用`dispatchValue(null)`时会遍历所有的Observer分发新值,否则只分发给传入的`ObserverWrapper`,这里说明下:`LiveData`维护了`mDispatchingValue/mDispatchInvalidated`,而Observer可能会有多个,都持有LiveData对象(因为是内部类),所以等待的observer对应的`LiveData`会获得`mDispatchInvalidated = true`,中断进行中的分发,让旧数据不再分发
  * `considerNotify`: 第一步检查`ObserverWrapper`是否`mActive`,第二步检查`shouldBeActive()`,第三步检查`ObserverWrapper`的版本`observer.mLastVersion >= mVersion`,最后才通知更新,调用Observer的onChange方法
  * ObserverWrapper中的mLastVersion /和LiveData的mVersion用来防止再入活跃状态后重复分发
  * `ObserverWrapper`中的`mActive`用来控制该观察者是否需要分发(`activeStateChanged(boolean)`方法的作用下)
* ## considerNotify方法
  先上代码
  ```java
  private void considerNotify(ObserverWrapper observer) {
       if (!observer.mActive) {
           return;
       }
       // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.
       //
       // we still first check observer.active to keep it as the entrance for events. So even if
       // the observer moved to an active state, if we've not received that event, we better not
       // notify for a more predictable notification order.
       if (!observer.shouldBeActive()) {
           observer.activeStateChanged(false);
           return;
       }
       if (observer.mLastVersion >= mVersion) {
           return;
       }
       observer.mLastVersion = mVersion;
       //noinspection unchecked
       observer.mObserver.onChanged((T) mData);
   }

  ```  
  很简单的逻辑,就是
    1. 第一步判断是否active,这个mActive属性在几种情况下会为true  
      a. Observer是通过observeForever注册的  
      b. Observer的shouldBeActive返回true
            我们看看方法代码,只有绑定的Lifecycle的State大于STARTED才会为true
```java
            @Override
            boolean shouldBeActive() {
                return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
            }
```
    mActive会在生命周期变化时会被多次检查
    2. 第二步再次检查`shouldBeActive()`状态
    3. 第三步检查`mLastVersion`和`mVersion`  
        这个version是在setValue时被+1的,检查一下防止重复分发,完成后修改lastVersion然后回调`onChange(T)`

* ## setValue/postValue方法的分析
    按惯例先看看代码,postValue的代码也好理解,只是给赋值操作加了一个锁,并把setValue放到主线程执行,使多个postValue只有最后一个的value生效  
    ```java
    private final Runnable mPostValueRunnable = new Runnable() {
       @Override
       public void run() {
           Object newValue;
           synchronized (mDataLock) {
               newValue = mPendingData;
               mPendingData = NOT_SET;
           }
           //noinspection unchecked
           setValue((T) newValue);
       }
   };
    protected void postValue(T value) {
       boolean postTask;
       synchronized (mDataLock) {
           postTask = mPendingData == NOT_SET;
           mPendingData = value;
       }
       if (!postTask) {
           return;
       }
       ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
   }

    @MainThread
    protected void setValue(T value) {
        assertMainThread("setValue");
        mVersion++;
        mData = value;
        dispatchingValue(null);
    }
    ```
    想象一下如果两个线程`T-A`,`T-B` 里都执行了postValue传入不同的值A,B,先会在方法内的mDataLock锁住的代码块竞争执行,比如A先执行,`mPendingData`先被赋值为A,`postTask` => true,然后赋值为B,也就是说第一次的赋值才会通过同步块之后的if判断,执行分发runnable,线程`T-B`只是修改了`mPendingValue`的值

    然后会有一个主线程上的同步代码块(也是同一个锁),将新值执行`setValue`,`mPendingValue`重设为初始值,也就是说取得值实际上是后一个线程的B,后面分发完成重设为`NOT_SET`,后面的PostValue又进入原来的逻辑了
    > postValue的注释也很清楚了:  If you called this method multiple times before a main thread executed a posted task, only
the last value would be dispatched.

* ## 基本流程
  * observe() -> 新建Observer -> Observer的StateChange -> dispatchValue -> considerNotify  
  * setValue/postValue -> dispatchValue -> considerNotify

* # 总结
  LiveData的结构是比较简单的,使用时注意不要多次注册观察者,一般如果在Activity中使用,就在OnCreate方法注册,Activity被关闭或回收后观察者会自动解除订阅,所以不会发生泄漏,我们也可以自己继承LiveData做更多的定制来实现自己的业务逻辑,配合Tranformations工具类和MediatorLiveData可以实现很多比较复杂的逻辑