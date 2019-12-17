---
title: RxLifecycle
date: 2019-12-11 21:11:20
tags: 
    - Android
---
# RxLifecycle:
基于第二个事件流(生命周期)来帮助我们自动完成一个序列(sequence)

## Usage

You must start with an `Observable<T>` representing a lifecycle stream. Then you use `RxLifecycle` to bind
a sequence to that lifecycle.

You can bind when the lifecycle emits anything:

```java
myObservable
    .compose(RxLifecycle.bind(lifecycle))
    .subscribe();
```

Or you can bind to when a specific lifecyle event occurs:

```java
myObservable
    .compose(RxLifecycle.bindUntilEvent(lifecycle, ActivityEvent.DESTROY))
    .subscribe();
```

Alternatively, you can let RxLifecycle determine the appropriate time to end the sequence:

```java
myObservable
    .compose(RxLifecycleAndroid.bindActivity(lifecycle))
    .subscribe();
```

It assumes you want to end the sequence in the opposing lifecycle event - e.g., if subscribing during `START`, it will
terminate on `STOP`. If you subscribe after `PAUSE`, it will terminate at the next destruction event (e.g.,
`PAUSE` will terminate in `STOP`).
