---
title: 解剖 RxJava 之变换操作符
date: 2017-03-27 18:08:19
categories:
  - RxJava
tags:
  - RxJava
  - Java
  - Android
---
## 介绍

此文章结合 [Github AnalyseRxJava](https://github.com/zhgqthomas/AnalyseRxJava) 项目，给 Android 开发者带来 RxJava 详细的解说。参考自 [RxJava Essential](https://rxjava.yuxingxin.com/) 及书中的例子

关于 RxJava 的由来及简介，这里就不在重复了，感兴趣的请阅读 `RxJava Essential`。

### 相关文章链接
- [解剖 RxJava 之过滤操作符](http://www.jianshu.com/p/bf39b9c3155a)
- [解剖 RxJava 之变换操作符](http://www.jianshu.com/p/ae5cdb6579a8)
- 解剖 RxJava 之组合操作符(未完待续)

## 变换操作符
### map 家族
RxJava 提供了几个 mapping 函数：`map()`, `flatMap()`, `concatMap()`, `flatMapIterable()` 以及 `switchMap()` .所有这些函数都作用于一个可观测序列，然后变换它发射的值，最后用一种新的形式返回它们。

<!-- more -->

#### map()
此函数的示意图为:

![](http://upload-images.jianshu.io/upload_images/854027-83e9a6b3670a2795.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

RxJava的 map 函数接收一个指定的 Func 对象然后将它应用到每一个由 Observable 发射的值上, 比如获取 App 列表的方法，就是将原有数据源的 `ResolveInfo` map 为 `AppInfo` 

```Java
    private void performMap() {
        mAppAdapter.clear();

        Observable.from(mApps)
                .take(3)
                .map(new Func1<AppInfo, AppInfo>() {
                    @Override
                    public AppInfo call(AppInfo appInfo) {
                        String name = appInfo.getName();
                        appInfo.setName(name + " map");
                        LogUtils.d(TAG, "map -- 1 -- " + appInfo.getName());
                        return appInfo;
                    }
                })
                .subscribe(new Action1<AppInfo>() {
                    @Override
                    public void call(AppInfo appInfo) {
                        mAppAdapter.add(appInfo);
                        LogUtils.d(TAG, "map -- 2 -- " + appInfo.getName());
                    }
                });
    }
```

> `map()` 被订阅时每传递一个事件执行一次 onNext 方法  
map 返回的是结果集  
map只能单一转换，单一只的是只能一对一进行转换，指一个对象可以转化为另一个对象但是不能转换成对象数组（map返回结果集不能直接使用from/just再次进行事件分发，一旦转换成对象数组的话，再处理集合/数组的结果时需要利用for一一遍历取出，而使用 RxJava 就是为了剔除这样的嵌套结构，使得整体的逻辑性更强。）

#### flatMap()
此函数的示意图为:

![](http://upload-images.jianshu.io/upload_images/854027-ed6e1c260254869a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在复杂的场景中，我们有一个这样的 Observable：它发射一个数据序列，这些数据本身也可以发射 Observable。RxJava 的 `flatMap()` 函数提供一种铺平序列的方式，然后合并这些 Observables 发射的数据，最后将合并后的结果作为最终的 Observable

当我们在处理可能有大量的 Observables 时，重要是记住任何一个 Observables 发生错误的情况，`flatMap()` 将会触发它自己的 `onError()` 函数并放弃整个链。
重要的一点提示是关于合并部分：它允许交叉。正如上图所示，这意味着 `flatMap()` 不能够保证在最终生成的 Observable 中源 Observables 确切的发射顺序。

```Java
    private void performFlatMap() {
        mAppAdapter.clear();

        getApps().take(3)
                .flatMap(new Func1<AppInfo, Observable<AppInfo>>() {
                    @Override
                    public Observable<AppInfo> call(AppInfo appInfo) {
                        String name = appInfo.getName();
                        appInfo.setName(name + " flatMap");
                        LogUtils.d(TAG, "flatMap -- 1 -- " + appInfo.getName());
                        return Observable.just(appInfo)
                                .delay((long) (Math.random() * 2 + 0.5), TimeUnit.SECONDS);
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Action1<AppInfo>() {
                    @Override
                    public void call(AppInfo appInfo) {
                        mAppAdapter.add(appInfo);
                        LogUtils.d(TAG, "flatMap -- 2 -- " + appInfo.getName());
                    }
                });
    }
```
log 结果

```xml
8622-8622/com.zhgqthomas.rxjava D/github_TransformObserv: flatMap -- 1 -- Contacts flatMap
8622-8622/com.zhgqthomas.rxjava D/github_TransformObserv: flatMap -- 1 -- Phone flatMap
8622-8622/com.zhgqthomas.rxjava D/github_TransformObserv: flatMap -- 1 -- Settings flatMap
8622-8622/com.zhgqthomas.rxjava D/github_TransformObserv: flatMap -- 2 -- Settings flatMap
8622-8622/com.zhgqthomas.rxjava D/github_TransformObserv: flatMap -- 2 -- Contacts flatMap
8622-8622/com.zhgqthomas.rxjava D/github_TransformObserv: flatMap -- 2 -- Phone flatMap
```
**通过 log 可以发现 `flatMap` 合并结果是允许交叉的**

> `flatMap` 被订阅时将所有数据传递完毕汇总到一个Observable然后一一执行onNext方法（执行顺序不同）>>>>(如单纯用于一对一转换则和map相同)  
flatmap返回的是包含结果集的Observable（返回结果不同）  
flatmap多用于多对多，一对多，再被转化为多个时，一般利用from/just进行一一分发  
flatmap既可以单一转换也可以一对多/多对多转换，flatmap要求返回Observable，因此可以再内部进行from/just的再次事件分发，一一取出单一对象（转换对象的能力不同）

#### concatMap()
此函数的示意图为:

![](http://upload-images.jianshu.io/upload_images/854027-52ed01150e77424e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```Java
    private void performConcatMap() {
        mAppAdapter.clear();

        getApps().take(3)
                .concatMap(new Func1<AppInfo, Observable<AppInfo>>() {
                    @Override
                    public Observable<AppInfo> call(AppInfo appInfo) {
                        String name = appInfo.getName();
                        appInfo.setName(name + " concatMap");
                        LogUtils.d(TAG, "concatMap -- 1 -- " + appInfo.getName());
                        return Observable.just(appInfo)
                                .delay((long) (Math.random() * 2 + 0.5), TimeUnit.SECONDS);
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Action1<AppInfo>() {
                    @Override
                    public void call(AppInfo appInfo) {
                        mAppAdapter.add(appInfo);
                        LogUtils.d(TAG, "concatMap -- 2 -- " + appInfo.getName());
                    }
                });
    }
```
log 信息

```xml
2637-2637/com.zhgqthomas.rxjava D/github_TransformObserv: concatMap -- 1 -- Contacts concatMap
2637-2637/com.zhgqthomas.rxjava D/github_TransformObserv: concatMap -- 1 -- Phone concatMap
2637-2637/com.zhgqthomas.rxjava D/github_TransformObserv: concatMap -- 2 -- Contacts concatMap
2637-3048/com.zhgqthomas.rxjava D/github_TransformObserv: concatMap -- 1 -- Settings concatMap
2637-2637/com.zhgqthomas.rxjava D/github_TransformObserv: concatMap -- 2 -- Phone concatMap
2637-2637/com.zhgqthomas.rxjava D/github_TransformObserv: concatMap -- 2 -- Settings concatMap
```

> 通过 log 可以发现 RxJava 的 `concatMap()` 函数解决了 `flatMap()` 的交叉问题，提供了一种能够把发射的值连续在一起的铺平函数，而不是合并它们, 如上述示意图所示

#### flatMapIterable()
以下为该函数的示意图：

![](http://upload-images.jianshu.io/upload_images/854027-b97d4eb3cf4d084f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

作为 map 家族的一员，`flatMapInterable()`和 `flatMap()` 很像。仅有的本质不同是它将源数据生成 Iterable，而不是原始数据项和生成的 Observables。

#### switchMap()
以下为该函数的示意图：

![](http://upload-images.jianshu.io/upload_images/854027-0ba07e9b985b3be0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`switchMap()` 和 `flatMap()` 很像，除了一点：每当源 Observable 发射一个新的数据项（Observable）时，它将取消订阅并停止监视之前那个数据项产生的 Observable，并开始监视当前发射的这一个。

```Java
    private void performSwitchMap() {
        mAppAdapter.clear();

        getApps().take(3)
                .switchMap(new Func1<AppInfo, Observable<AppInfo>>() {
                    @Override
                    public Observable<AppInfo> call(AppInfo appInfo) {
                        String name = appInfo.getName();
                        appInfo.setName(name + " switchMap");
                        LogUtils.d(TAG, "switchMap -- 1 -- " + appInfo.getName());
                        return Observable.just(appInfo)
                                .delay(1, TimeUnit.SECONDS);
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Action1<AppInfo>() {
                    @Override
                    public void call(AppInfo appInfo) {
                        mAppAdapter.add(appInfo);
                        LogUtils.d(TAG, "switchMap -- 2 -- " + appInfo.getName());
                    }
                });
    }
```
log 打印结果

```xml
32196-32196/com.zhgqthomas.rxjava D/github_TransformObserv: switchMap -- 1 -- Contacts switchMap
32196-32196/com.zhgqthomas.rxjava D/github_TransformObserv: switchMap -- 1 -- Phone switchMap
32196-32196/com.zhgqthomas.rxjava D/github_TransformObserv: switchMap -- 1 -- Settings switchMap
32196-32196/com.zhgqthomas.rxjava D/github_TransformObserv: switchMap -- 2 -- Settings switchMap
```
> 通过 log 可以看出当源 Observable 发射一个新的数据项（Observable）时，它将取消订阅并停止监视之前那个数据项产生的 Observable，并开始监视当前发射的这一个。  
关于 `switchMap` 巧妙应用也可以查看[这篇文章](http://www.jianshu.com/p/33c548bce571)

### scan()
以下是该函数的示意图:

![](http://upload-images.jianshu.io/upload_images/854027-caf764e50502f40f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

RxJava 的 `scan()` 函数可以看做是一个累积函数。`scan()` 函数对原始 Observable 发射的每一项数据都应用一个函数，计算出函数的结果值，并将该值填充回可观测序列，等待和下一次发射的数据一起使用。

```Java
    private void performScan() {
        mAppAdapter.clear();

        getApps().scan(new Func2<AppInfo, AppInfo, AppInfo>() {
                    @Override
                    public AppInfo call(AppInfo appInfo, AppInfo appInfo2) {
                        if (appInfo.getName().length() > appInfo2.getName().length()) {
                            return appInfo;
                        } else {
                            return appInfo2;
                        }
                    }
                })
                .distinct()
                .subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Action1<AppInfo>() {
                    @Override
                    public void call(AppInfo appInfo) {
                        mAppAdapter.add(appInfo);
                    }
                });
    }
```
有一个 `scan()` 函数的变体，它用初始值作为第一个发射的值，方法签名看起来像：`scan(R,Func2)`

### groupBy()
以下为此函数的示意图：

![](http://upload-images.jianshu.io/upload_images/854027-47c302ae29edefaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过示意图可以看出，RxJava 使用 `groupBy`从列表中按照指定的规则来分组元素。这个函数将源 Observable 变换成一个发射 Observables 的新的 Observable。它们中的每一个新的 Observable 都发射一组指定的数据。

```Java
    private void performGroupBy() {
        mAppAdapter.clear();

        getApps()
                .groupBy(new Func1<AppInfo, String>() {
                    @Override
                    public String call(AppInfo appInfo) { // 根据第一次安装时间进行分组
                        SimpleDateFormat formatter = new SimpleDateFormat("MM/yyyy", Locale.CHINA);
                        return formatter.format(new Date(appInfo.getFirstInstallTime()));
                    }
                })
                .subscribe(new Action1<GroupedObservable<String, AppInfo>>() {
                    @Override
                    public void call(final GroupedObservable<String, AppInfo> result) {
                        // 只显示 key 为 09/2016 的 Apps
                        if (!result.getKey().equalsIgnoreCase("09/2016")) {
                            return;
                        }

                        result.toList()
                                .flatMap(new Func1<List<AppInfo>, Observable<AppInfo>>() {
                                    @Override
                                    public Observable<AppInfo> call(List<AppInfo> appInfos) {
                                        return Observable.from(appInfos);
                                    }
                                })
                                .subscribe(new Action1<AppInfo>() {
                                    @Override
                                    public void call(AppInfo appInfo) {
                                        appInfo.setName(appInfo.getName() + " " + result.getKey());
                                        mAppAdapter.add(appInfo);
                                    }
                                });
                    }
                });
    }
```
以上代码，创建了一个新的 Observable，它将会发射一个带有 `GroupedObservable` 的序列。`GroupedObservable` 是一个特殊的 Observable，它源自一个分组的 key。在这个例子中，key就是String，代表的意思是 Month/Year 格式化的最近更新日期。

> 在研究这个函数的用法的时候，通过打 log 一度进入了迷惑的状态，后通过 google 搜索找到了[该篇文章](http://stackoverflow.com/questions/30817281/groupby-operator-items-from-different-groups-interleaved)来更形象的描述了其实现原理，强烈推荐阅读此[文章](http://stackoverflow.com/questions/30817281/groupby-operator-items-from-different-groups-interleaved)。

### buffer()
`buffer()` 函数将源 Observable 变换一个新的 Observable，这个新的 Observable 每次发射一组列表值而不是一个一个发射。

`buffer(count = ?) `以下为该函数的示意图:

![](http://upload-images.jianshu.io/upload_images/854027-8a7247a20e93dc86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图中展示了 `buffer()` 如何将 count 作为一个参数来指定有多少数据项被包在发射的列表中。实际上，`buffer()` 函数有几种变体。其中有一个是允许你指定一个 skip 值：此后每 skip 项数据，然后又用 count 项数据填充缓冲区。如下图所示：

![](http://upload-images.jianshu.io/upload_images/854027-e310270120293a65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`buffer()` 带一个 timespan 的参数，会创建一个每隔 timespan 时间段就会发射一个列表的 Observable。

![](http://upload-images.jianshu.io/upload_images/854027-f960604b0b6dffe6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```Java
    private void performBuffer() {
        mAppAdapter.clear();

        getApps().take(8)
                .doOnNext(new Action1<AppInfo>() {
                    @Override
                    public void call(AppInfo appInfo) {
                        mAppAdapter.add(appInfo);
                    }
                })
                .flatMap(new Func1<AppInfo, Observable<String>>() {
                    @Override
                    public Observable<String> call(AppInfo appInfo) {
                        long time = (long) (Math.random() * 3 + 0.5);
                        LogUtils.d(TAG, "app: " + appInfo.getName() + " time: " + time);
                        return Observable.just(appInfo.getName())
                                .delay(time, TimeUnit.SECONDS);
                    }
                })
                .buffer(2, TimeUnit.SECONDS)
                .subscribe(new Action1<List<String>>() {
                    @Override
                    public void call(List<String> strings) {
                        LogUtils.d(TAG, "values: " + Arrays.toString(strings.toArray()));
                    }
                });
    }
```
log 信息为:

```xml
30843-30843/com.zhgqthomas.rxjava D/github_TransformObserv: app: Phone time: 0
30843-30843/com.zhgqthomas.rxjava D/github_TransformObserv: app: Email time: 2
30843-30843/com.zhgqthomas.rxjava D/github_TransformObserv: app: Contacts time: 0
30843-30843/com.zhgqthomas.rxjava D/github_TransformObserv: app: Settings time: 2
30843-30843/com.zhgqthomas.rxjava D/github_TransformObserv: app: Messages time: 2
30843-30843/com.zhgqthomas.rxjava D/github_TransformObserv: app: i Music time: 2
30843-30843/com.zhgqthomas.rxjava D/github_TransformObserv: app: i Theme time: 3
30843-30843/com.zhgqthomas.rxjava D/github_TransformObserv: app: Recorder time: 2
30843-30983/com.zhgqthomas.rxjava D/github_TransformObserv: values: [Phone, Contacts]
30843-30988/com.zhgqthomas.rxjava D/github_TransformObserv: values: [Email, Settings, Messages, i Music, Recorder, i Theme]
```

### window()

`window()` 函数和 `buffer()` 很像，但是它发射的是 Observable 而不是列表。下图展示了 `window()` 如何缓存3个数据项并把它们作为一个新的 Observable 发射出去。


![](http://upload-images.jianshu.io/upload_images/854027-86ca60733e23ab4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```Java
    private void performWindow() {
        mAppAdapter.clear();

        getApps().take(8)
                .doOnNext(new Action1<AppInfo>() {
                    @Override
                    public void call(AppInfo appInfo) {
                        mAppAdapter.add(appInfo);
                    }
                })
                .flatMap(new Func1<AppInfo, Observable<String>>() {
                    @Override
                    public Observable<String> call(AppInfo appInfo) {
                        long time = (long) (Math.random() * 3 + 0.5);
                        LogUtils.d(TAG, "app: " + appInfo.getName() + " time: " + time);
                        return Observable.just(appInfo.getName())
                                .delay(time, TimeUnit.SECONDS);
                    }
                })
                .window(2, TimeUnit.SECONDS)
                .subscribe(new Action1<Observable<String>>() {
                    @Override
                    public void call(Observable<String> result) { // 与 buffer 的不同在于返回的是个 Observable
                        result.toList()
                            .subscribe(new Action1<List<String>>() {
                                @Override
                                public void call(List<String> strings) {
                                    LogUtils.d(TAG, "values: " + Arrays.toString(strings.toArray()));
                                }
                            });
                    }
                });
    }
```
log 信息为

```xml
28277-28277/com.zhgqthomas.rxjava D/github_TransformObserv: app: Phone time: 1
28277-28277/com.zhgqthomas.rxjava D/github_TransformObserv: app: Email time: 0
28277-28277/com.zhgqthomas.rxjava D/github_TransformObserv: app: Contacts time: 1
28277-28277/com.zhgqthomas.rxjava D/github_TransformObserv: app: Settings time: 2
28277-28277/com.zhgqthomas.rxjava D/github_TransformObserv: app: Messages time: 0
28277-28277/com.zhgqthomas.rxjava D/github_TransformObserv: app: i Music time: 3
28277-28277/com.zhgqthomas.rxjava D/github_TransformObserv: app: i Theme time: 2
28277-28277/com.zhgqthomas.rxjava D/github_TransformObserv: app: Recorder time: 1
28277-28442/com.zhgqthomas.rxjava D/github_TransformObserv: values: [Email, Messages, Phone, Contacts, Recorder]
28277-28445/com.zhgqthomas.rxjava D/github_TransformObserv: values: [Settings, i Theme, i Music]
```

### cast()
以下为该函数的示意图:

![](http://upload-images.jianshu.io/upload_images/854027-eb845660efa834e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`cast()` 函数是 map()操作符的特殊版本。不同的地方在于map操作符可以通过自定义规则，把一个值A1变成另一个值A2，A1和A2的类型可以一样也可以不一样；而cast操作符主要是做类型转换的，传入参数为类型class，**如果源Observable产生的结果不能转成指定的class，则会抛出ClassCastException运行时异常。**

## Github
- [Github AnalyseRxJava](https://github.com/zhgqthomas/AnalyseRxJava)