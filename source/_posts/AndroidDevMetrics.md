---
title: AndroidDevMetrics
date: 2019-12-13 22:13:22
tags: 
    - Android
    - Tools
---


# AndroidDevMetrics
> 项目地址  <href>https://github.com/apm29/AndroidDevMetrics</href>

* Usage

  ```
  public class ExampleApplication extends Application {

    @Override
    public void onCreate() {
      super.onCreate();
      //Use it only in debug builds
      if (BuildConfig.DEBUG) {
         AndroidDevMetrics.initWith(this);
       }
     }
 }
 ```
## Activity生命周期方法计时的原理
* AndroidDevMetrics.java

> 1.AndroidDevMetrics实例创建,通过Builder模式设置参数
可配置的参数:
* dagger2WarningLevel1/2/3 三个等级设置 默认是30/50/100ms
* enableAcitivtyMetrics = true;//是否记录Activity生命周期耗时
* enableDagger2Metrics = true;//dagger注入耗时
* showNotification = true;//在通知栏显示
* autoCancelNotification = false;//是否可以点击后取消
* intervalMillis = FRAME_DROPS_DEFAULT_INTERVAL_MS;//帧率分析间隔
* maxFpsForFrameDrop = FRAME_DROPS_FPS_LIMIT;//最大掉帧限制
* List<UIInterceptor> interceptors;//UI拦截器

> 2.开始分析的主入口,AndroidDevMetrics实例的方法setupMetrics()

```
private void setupMetrics() {
    Dagger2GraphAnalyzer.setEnabled(enableDagger2Metrics);
    InitManager.getInstance().initializedMetrics.clear();

    ActivityLifecycleAnalyzer.setEnabled(true);
    if (enableAcitivtyMetrics) {
        MethodsTracingManager.getInstance().init(context);
        ActivityLaunchMetrics activityLaunchMetrics = ActivityLaunchMetrics.getInstance();
        ((Application) context.getApplicationContext()).registerActivityLifecycleCallbacks(activityLaunchMetrics);
        ChoreographerMetrics.getInstance().setMaxFpsForFrameDrop(maxFpsForFrameDrop);
        ChoreographerMetrics.getInstance().setIntervalMillis(intervalMillis);
        ChoreographerMetrics.getInstance().start();
    }

    if (showNotification) {
        showNotification();
    }
}
```

* Dagger2GraphAnalyzer.setEnabled(enableDagger2Metrics);Dagger分析器的,一个带@Aspect注解的类,这个类里有许多@Pointcut @Around 修饰的方法,先略过,AOP相关的可以看看[Android基于AOP的非侵入式监控之——AspectJ实战](https://blog.csdn.net/woshimalingyi/article/details/51476559)

* InitManager.getInstance().initializedMetrics.clear();
  InitManager单例维护了几个LinkedHashMap对象和一个Set

```
  public final Set<OnMetricsDataListener> dataListeners = new HashSet<>();

  public final LinkedHashMap<String, InitMetric> initializedMetrics = new LinkedHashMap<>();
  public final LinkedHashMap<String, Integer> initCounter = new LinkedHashMap<>();
```
> initializedMetrics应该是比较重要的,其中InitMetric这个类在许多地方都有涉及,类中成员的意义应该很明显了,看命名
```
   public Class<?> cls; //实例类class
   public long initTimeMillis = 0;//初始化时间
   public int instanceNo = 0;//实例个数
   public String threadName = "";//线程名称?
   public Set<InitMetric> args = new HashSet<>(); //初始化这个类需要的其他类的集合? 大概
   public StackTraceElement[] traceElements;//堆栈记录
```
> InitManager一上来先把initializedMetrics清空了.

* 然后是ActivityLifecycleAnalyzer.setEnabled(true);应该和Dagger的分析器一样,是一个带@Aspect注解的类,有许多@Pointcut注解,
 其实是AspectJ的方式对onCreate/onStart/onResume三个生命周期方法进行监听,我们来看看监听的过程(在一个@Around注解的方法中,熟悉AOP的应该都晓得)
 ```
 @Around("onCreateMethod() || onStartMethod() || onResumeMethod()")
   public Object logAndExecute(ProceedingJoinPoint joinPoint) throws Throwable {
       if (!enabled) return joinPoint.proceed();

       MethodSignature signature = (MethodSignature) joinPoint.getSignature();
       String methodName = signature.getMethod().getName();

       final Object result;
       if (METHOD_ON_RESUME.equals(methodName)) {
           ActivityLifecycleMetrics.getInstance().logPreOnResume((Activity) joinPoint.getTarget());
           result = executeWithTracingIfEnabled(joinPoint, methodName);
           ActivityLifecycleMetrics.getInstance().logPostOnResume((Activity) joinPoint.getTarget());
       } else if (METHOD_ON_START.equals(methodName)) {
           ActivityLifecycleMetrics.getInstance().logPreOnStart((Activity) joinPoint.getTarget());
           result = executeWithTracingIfEnabled(joinPoint, methodName);
           ActivityLifecycleMetrics.getInstance().logPostOnStart((Activity) joinPoint.getTarget());
       } else if (METHOD_ON_CREATE.equals(methodName)) {
           ActivityLifecycleMetrics.getInstance().logPreOnCreate((Activity) joinPoint.getTarget());
           result = executeWithTracingIfEnabled(joinPoint, methodName);
           ActivityLifecycleMetrics.getInstance().logPostOnCreate((Activity) joinPoint.getTarget());
       } else {
           result = null;
       }

       return result;
   }
```
> 在启用监听的情况下首先
  * 获取方法的签名来区分是OnResume还是其他什么的
  * 获取了ActivityLifecycleMetrics实例在onResume执行前后分别调用logOreXXX 和 logPostXXX方法,很明显是打log用的
  * ActivityLifecycleMetrics自己维护了一个LinkedhashMap存放已经记录过得ActivityMetric对象,而ActivityMetric对象保存了一个Activity的 1. 状态 2. 各个生命周期耗时 3.是否实现了各个生命周期方法 4.各个状态的timeMillis,logPreXXX logPostXXX方法会把对应的ActivityMetric对象的状态进行对应的改变

* MethodTrackingManager,应该是追踪Activity中具体方法的一个管理类,有两个Set集合保存ScheduledMethods 和 TrackedMethod ,setupMetrics里只做了一些初始化工作

* ActivityLaunchMetrics,获取到Application之后为之注册了一个ActivityLifecycle监听,这个ActivityLaunchMetrics里面也在对应的生命周期里调用了ActivityLifecycleMetrics实例的logPostXXX方法,也就是说postXXX在 ActivityLaunchMetrics和ActivityLifecycleAnalyzer都被调用了一次

* ChoreographerMetrics,应该是帧率帧数统计的管理类,使用Choreographer来统计的帧数这些,可以看看Choreographer的官方文档,了解下原理
