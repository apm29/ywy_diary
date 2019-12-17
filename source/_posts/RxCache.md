---
title: RxCache
date: 2019-12-11 21:11:20
tags: 
    - Android
---
# RxCache
```
/**
 * 此为RxCache官方Demo
 */
public interface CacheProviders {

    @LifeCache(duration = 2, timeUnit = TimeUnit.MINUTES)
    Observable<Reply<List<Repo>>> getRepos(Observable<List<Repo>> oRepos, DynamicKey userName, EvictDynamicKey evictDynamicKey);

    @LifeCache(duration = 2, timeUnit = TimeUnit.MINUTES)
    Observable<Reply<List<User>>> getUsers(Observable<List<User>> oUsers, DynamicKey idLastUserQueried, EvictProvider evictProvider);

    Observable<Reply<User>> getCurrentUser(Observable<User> oUser, EvictProvider evictProvider);
}

作者：JessYan
链接：https://www.jianshu.com/p/b58ef6b0624b
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### 参数
此Observable的意义为需要将你想缓存的Retrofit接口作为参数传入(返回值必须为Observable),RxCache会在没有缓存,或者缓存已经过期,或者EvictProvider为true时,通过这个Retrofit接口重新请求最新的数据,并且将服务器返回的结果包装成Reply返回,返回之前会向内存缓存和磁盘缓存中各保存一份

值得一提的是,如果需要知道返回的结果是来自哪里(本地,内存还是网络),是否加密,则可以使用Observable<Reply<List<Repo>>>作为方法的返回值,这样RxCache则会使用Reply包装结果,如果没这个需求则直接在范型中声明结果的数据类型Observable<List<Repo>>

#### @LifeCache
保存缓存的事件

#### DynamicKey
保存缓存的key值,比如getRepo这个方法保存时就会根据key分开保存,而DynamicKeyGroup接收一个DynamicKey参数和Object的group参数,可以理解为在Group下再分key保存
