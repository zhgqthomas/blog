---
title: RxJava 源码解析之观察者模式
date: 2017-03-30 18:03:55
categories:
  - RxJava
tags:
  - RxJava
  - Java
  - Android
---
了解 RxJava 的应该都知道是一个基于事务驱动的库，响应式编程的典范。提到事务驱动和响应就不得不说说，设计模式中观察者模式，已经了解的朋友，可以直接跳过观察者模式的介绍，直接到 RxJava 源码中对于观察者的应用。

## 观察者模式
该部分结合自扔物线的 [《给 Android 开发者的 RxJava 详解》](http://gank.io/post/560e15be2dca930e00da1083)， 强烈推荐刚接触 RxJava 的朋友阅读。

### 传统观察者模式
观察者模式面向的需求是：A 对象（观察者）对 B 对象（被观察者）的某种变化高度敏感，需要在 B 变化的一瞬间做出反应。举个例子，新闻里喜闻乐见的警察抓小偷，警察需要在小偷伸手作案的时候实施抓捕。在这个例子里，警察是观察者，小偷是被观察者，警察需要时刻盯着小偷的一举一动，才能保证不会漏过任何瞬间。程序的观察者模式和这种真正的『观察』略有不同，观察者不需要时刻盯着被观察者（例如 A 不需要每过 2ms 就检查一次 B 的状态），而是采用注册( Register )或者称为订阅( Subscribe )的方式，告诉被观察者：我需要你的某某状态，你要在它变化的时候通知我。 Android 开发中一个比较典型的例子是点击监听器 `OnClickListener` 。对设置 `OnClickListener` 来说， `View` 是被观察者， `OnClickListener` 是观察者，二者通过 `setOnClickListener()` 方法达成订阅关系。订阅之后用户点击按钮的瞬间，Android Framework 就会将点击事件发送给已经注册的 OnClickListener 。采取这样被动的观察方式，既省去了反复检索状态的资源消耗，也能够得到最高的反馈速度。当然，这也得益于我们可以随意定制自己程序中的观察者和被观察者，而警察叔叔明显无法要求小偷『你在作案的时候务必通知我』。

<!-- more -->

OnClickListener 的模式大致如下图：

![](https://user-gold-cdn.xitu.io/2017/3/30/4f0280f858afb51900cd49811c3fefbe)

如图所示，通过 `setOnClickListener()` 方法，`Button` 持有 `OnClickListener` 的引用（这一过程没有在图上画出）；当用户点击时，`Button` 自动调用 `OnClickListener` 的 `onClick()` 方法。另外，如果把这张图中的概念抽象出来（`Button` -> 被观察者、`OnClickListener` -> 观察者、`setOnClickListener()` -> 订阅，`onClick()` -> 事件），就由专用的观察者模式（例如只用于监听控件点击）转变成了通用的观察者模式。如下图：

![](https://user-gold-cdn.xitu.io/2017/3/30/fb1214aa0d931e62784164456218abea)

而 RxJava 作为一个工具库，使用的就是通用形式的观察者模式。

### RxJava 中观察者模式
RxJava 有四个基本概念：`Observable` (可观察者，即被观察者)、 `Observer` (观察者)、 `subscribe` (订阅)、事件。Observable 和 Observer 通过 subscribe() 方法实现订阅关系，从而 Observable 可以在需要的时候发出事件来通知 Observer。

与传统观察者模式不同， RxJava 的事件回调方法除了普通事件 `onNext()` （相当于 onClick() / onEvent()）之外，还定义了两个特殊的事件：`onCompleted()` 和 `onError()`。

- `onCompleted()`: 事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 `onNext()` 发出时，需要触发 `onCompleted()` 方法作为标志。
- `onError()`: 事件队列异常。在事件处理过程中出异常时，`onError()` 会被触发，同时队列自动终止，不允许再有事件发出。
- 在一个正确运行的事件序列中, `onCompleted()` 和 `onError()` 有且只有一个，并且是事件序列中的最后一个。需要注意的是，`onCompleted()` 和 `onError()` 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。**并且只要`onCompleted()` 和 `onError()` 中有一个调用了，都会中止 `onNext()` 的调用。**

RxJava 的观察者模式大致如下图：
![](https://user-gold-cdn.xitu.io/2017/3/30/08a09f6dac11430a1daa840fa9ff61cd)

## 基本实现
基于以上观点， RxJava 的基本实现主要有三点:
### 创建 Observer
Observer 即观察者，它决定事件触发的时候将有怎样的行为。 RxJava 中的 Observer 接口的实现方式：

```Java
Observer<String> observer = new Observer<String>() {
    @Override
    public void onNext(String s) {
        Log.d(tag, "Item: " + s);
    }

    @Override
    public void onCompleted() {
        Log.d(tag, "Completed!");
    }

    @Override
    public void onError(Throwable e) {
        Log.d(tag, "Error!");
    }
};
```

除了 Observer 接口之外，RxJava 还内置了一个实现了 Observer 的抽象类：`Subscriber`。 Subscriber 对 Observer 接口进行了一些扩展，但他们的基本使用方式是完全一样的：

```Java
Subscriber<String> subscriber = new Subscriber<String>() {
    @Override
    public void onNext(String s) {
        Log.d(tag, "Item: " + s);
    }

    @Override
    public void onCompleted() {
        Log.d(tag, "Completed!");
    }

    @Override
    public void onError(Throwable e) {
        Log.d(tag, "Error!");
    }
};
```
不仅基本使用方式一样，实质上，在 RxJava 的 subscribe 过程中，Observer 也总是会先被转换成一个 Subscriber 再使用。

```Java
// Observable.java 源码

    public final Subscription subscribe(final Observer<? super T> observer) {
        if (observer instanceof Subscriber) { // 如果是 Subscriber 的子类，直接转化为 Subscriber
            return subscribe((Subscriber<? super T>)observer);
        }
        if (observer == null) {
            throw new NullPointerException("observer is null");
        }
        
        return subscribe(new ObserverSubscriber<T>(observer));
    }
```
```Java
// ObserverSubscriber.java

public final class ObserverSubscriber<T> extends Subscriber<T> {
    ...
}
```
> 通过源码可以看到，传入的 `Observer` 最终还是会转化为 `Subscriber` 来使用。

所以如果你只想使用基本功能，选择 Observer 和 Subscriber 是完全一样的。它们的区别对于使用者来说主要有两点：
- `onStart()`: 这是 Subscriber 增加的方法。它会在 subscribe 刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行）， onStart() 就不适用了，因为它总是在 subscribe 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用 doOnSubscribe() 方法。

```Java
// Subscriber.java

    public void onStart() {
        // do nothing by default
    }
```

- `unsubscribe()`: 这是 Subscriber 所实现的另一个接口 Subscription 的方法，用于取消订阅。在这个方法被调用后，Subscriber 将不再接收事件。一般在这个方法调用前，可以使用 isUnsubscribed() 先判断一下状态。 `unsubscribe()` 这个方法很重要，因为在 subscribe() 之后， Observable 会持有 Subscriber 的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：要在不再使用的时候尽快在合适的地方（例如 onPause() onStop() 等方法中）调用 `unsubscribe()` 来解除引用关系，以避免内存泄露的发生。

```Java
// Subscriber.java

    @Override
    public final void unsubscribe() {
        subscriptions.unsubscribe();
    }

    
    @Override
    public final boolean isUnsubscribed() {
        return subscriptions.isUnsubscribed();
    }
```
### 创建 Observable
Observable 即被观察者，它决定什么时候触发事件以及触发怎样的事件。例如  `create()` 方法

```Java
Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        subscriber.onNext("Hello");
        subscriber.onNext("Hi");
        subscriber.onNext("Aloha");
        subscriber.onCompleted();
    }
});
```

可以看到，这里传入了一个 `OnSubscribe` 对象作为参数。`OnSubscribe` 会被存储在返回的 `Observable` 对象中，它的作用相当于一个计划表，当 `Observable` 被订阅的时候，`OnSubscribe` 的 **call()** 方法会自动被调用，事件序列就会依照设定依次触发（对于上面的代码，就是观察者`Subscriber` 将会被调用三次 **onNext()** 和一次 **onCompleted()**。这样，由被观察者调用了观察者的回调方法，就实现了由被观察者向观察者的事件传递，即观察者模式。

`create()` 方法是 RxJava 最基本的创造事件序列的方法。基于这个方法， RxJava 还提供了一些方法用来快捷创建事件队列，例如 `just()`, `from()`

### 订阅 Subscribe
创建了 Observable 和 Observer 之后，再用 subscribe() 方法将它们联结起来，整条链子就可以工作了。代码形式很简单：

```Java
observable.subscribe(observer);

// 或者：
observable.subscribe(subscriber);
```
> 有人可能会注意到， `subscribe()` 这个方法有点怪：它看起来是『`observalbe` 订阅了 `observer / subscriber`』而不是『`observer / subscriber` 订阅了 `observalbe`』，这看起来就像『杂志订阅了读者』一样颠倒了对象关系。这让人读起来有点别扭，不过如果把 API 设计成 `observer.subscribe(observable) / subscriber.subscribe(observable)` ，虽然更加符合思维逻辑，但对流式 API 的设计就造成影响了，比较起来明显是得不偿失的。

整个过程中对象间的关系如下图：
![](https://user-gold-cdn.xitu.io/2017/3/30/73ee7227e4e965511c2162e6afe1a014)

## 源码层解析
### 基础原理
```Java
// 例子

Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("Hello");
                subscriber.onNext("Hi");
                subscriber.onNext("Aloha");
                subscriber.onCompleted();
            }
        }).subscribe(new Subscriber<String>() {
            @Override
            public void onCompleted() {
                System.out.println("onCompleted");
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                System.out.println("value: " + s);
            }
        });
```
log 信息

```xml
value: Hello
value: Hi
value: Aloha
onCompleted
```

看到上面代码，可能会有人跟我一样不明白， `create()` 中的 `OnSubscribe` 中 `call()` 的 `Subscriber` 是怎么样最终就变成了 `subscribe()` 中的 `Subscriber`。

下面来一下 `Observable.subscribe(Subscriber)` 的内部实现是这样的（仅核心代码）：

```Java
// 注意：这不是 subscribe() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。

static <T> Subscription subscribe(Subscriber<? super T> subscriber, Observable<T> observable) {

    ...

    // 可以用于做一些准备工作，例如数据的清零或重置, 默认情况下它的实现为空
    subscriber.onStart();

    if (!(subscriber instanceof SafeSubscriber)) {
        // 强制转化为 SafeSubscriber 是为了保证 onCompleted 或 onError 调用的时候会中止 onNext 的调用
        subscriber = new SafeSubscriber<T>(subscriber);
    }

    ...
    // // onObservableStart() 默认返回的就是 observable.onSubscribe
    RxJavaHooks.onObservableStart(observable, observable.onSubscribe).call(subscriber);
        
    // onObservableReturn() 默认也是返回 subscriber
    return RxJavaHooks.onObservableReturn(subscriber);   
    
    ...
}
```
通过源码可以看到，`subscriber()` 实际就做了 4 件事情
- 调用 `Subscriber.onStart()` 。这个方法在前面已经介绍过，是一个可选的准备方法。
- 将传入的 `Subscriber` 转化为 `SafeSubscriber`， 为了保证 onCompleted 或 onError 调用的时候会中止 onNext 的调用。

```Java
// 注意：这不是 SafeSubscriber 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。

public class SafeSubscriber<T> extends Subscriber<T> {

    private final Subscriber<? super T> actual;

    boolean done; // 通过改标志来保证 onCompleted 或 onError 调用的时候会中止 onNext 的调用

    public SafeSubscriber(Subscriber<? super T> actual) {
        super(actual);
        this.actual = actual;
    }

    @Override
    public void onCompleted() {
        if (!done) {
            done = true;

            ...
            
            actual.onCompleted();
            
            ...

            unsubscribe(); // 取消订阅，结束事务
        }
    }
       
    @Override
    public void onError(Throwable e) {
        
        ...

        if (!done) {
            done = true;
            _onError(e);
        }
    }

    @Override
    public void onNext(T t) {
    
        if (!done) { // done 为 true 时，中止传递
            actual.onNext(t);
        }
        
    }

    @SuppressWarnings("deprecation")
    protected void _onError(Throwable e) {
        ...

        actual.onError(e);

        ...

        unsubscribe();
 
        ...
    }
}

```
通过代码可以看出来，通过 `SafeSubscriber` 中的布尔变量 `done` 来做标记保证上文提到的 `onCompleted()` 和 `onError()` 二者的互斥性，即在队列中调用了其中一个，就不应该再调用另一个。**并且只要 `onCompleted()` 和 `onError()` 中有一个调用了，都会中止 `onNext()` 的调用。**

- 调用 Observable 中的 OnSubscribe.call(Subscriber) 。在这里，事件发送的逻辑开始运行。从这也可以看出，**在 RxJava 中， Observable 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当 subscribe() 方法执行的时候。**
- 将传入的 `Subscriber` 作为 Subscription 返回。这是为了方便 `unsubscribe()`.

以上就是 RxJava 最基本的一个通过观察者模式，来响应事件的原理。下面来看看 RxJava 中一些基本操作符的实现原理又是怎样的。
> 为了能更好的理解源码，需要对 RxJava 有基本的使用基础，对 RxJava 不太熟悉的朋友请先一步到[《给 Android 开发者的 RxJava 详解》](http://gank.io/post/560e15be2dca930e00da1083)

### 进阶
```Java
        Observable.interval(1, TimeUnit.SECONDS)
                .map(new Func1<Long, Long>() {
                    @Override
                    public Long call(Long aLong) {
                        return aLong * 5;
                    }
                })
                .subscribe(new Action1<Long>() {
                    @Override
                    public void call(Long aLong) {
                        System.out.println("value: " + aLong);
                    }
                });
```
log 信息

```xml
value: 0
value: 5
value: 10
...
```
上面的列子会每秒生成一个从 0 依次递增的整数，然后通过 `map()` 变换操作符后，变成了 5 的倍数的一个整数列。

第一次看到该例子时，就喜欢上了 RxJava，这种链式函数的交互模式真的很简洁，终于可以从回调地狱里逃出来了。喜欢的同时不免也会想 RxJava 是如何实现的。这种链式的函数流可以算是[建造者模式](http://wiki.jikexueyuan.com/project/java-design-pattern/builder-pattern.html)的一种变形，只不过省去了中间 `Builder` 而直接返回当前对象来实现。 更让我兴奋的是内部这些操作符的实现原理。

上文也已经说过了**在 RxJava 中， Observable 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当 subscribe() 方法执行的时候**。 所以对于上面一段的代码我们要从 `subscribe()` 往前屡，首先看一下 `map()` 这个函数的内部实现。

```Java
    public final <R> Observable<R> map(Func1<? super T, ? extends R> func) {
        // 新建了一个 Observable 并使用新的 OnSubscribeMap 来封装传入的数据
        return unsafeCreate(new OnSubscribeMap<T, R>(this, func));
    }
```

不用说大家也猜到了 `OnSubscribeMap` 是 `OnSubscribe` 的子类

```Java
// 注意：这不是 OnSubscribeMap 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。

public final class OnSubscribeMap<T, R> implements OnSubscribe<R> {

    final Observable<T> source;

    final Func1<? super T, ? extends R> transformer;

    public OnSubscribeMap(Observable<T> source, Func1<? super T, ? extends R> transformer) {
        this.source = source; // 列子中经过 Observable.interval() 函数生成的 Observable
        this.transformer = transformer;
    }

    // 传入的 o 就是例子中 `subscribe()` 出入的 Subscribe 
    // 具体结合 Observable.subscribe() 源码来理解
    @Override
    public void call(final Subscriber<? super R> o) {
        
        // 对传入的 Subscriber 进行再次封装成 MapSubscriber
        // 具体 Observable.map() 的逻辑是在 MapSubscriber 中
        MapSubscriber<T, R> parent = new MapSubscriber<T, R>(o, transformer); 
        o.add(parent); // 加入到 SubscriptionList 中，为之后取消订阅
        
        // Observable.interval() 返回的 Observable 进行订阅(关键点)
        source.unsafeSubscribe(parent); 
    }

    ...
}
```

可以看到 `call()` 方法的逻辑很简单，只是将例子中 `Observable.subscribe()` 传入的 `Subscriber` 进行封装后，再将上流传入的 `Observable` 进行订阅
```Java
// 注意：这不是 MapSubscriber 的源码
// 而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
static final class MapSubscriber<T, R> extends Subscriber<T> {

    final Subscriber<? super R> actual;

    final Func1<? super T, ? extends R> mapper;

    public MapSubscriber(Subscriber<? super R> actual, Func1<? super T, ? extends R> mapper) {
        this.actual = actual; // Observable.subscribe() 传入的 Subscriber
        this.mapper = mapper;
    }

    @Override
    public void onNext(T t) {
        R result;

        ...

        result = mapper.call(t); // 数据进行了变换

        ...

        actual.onNext(result); // 往下流传
    }

    ...
}
```
> 通过以上就完成了 `map()` 对数据的变换，这里最终的就是理解 `OnSubscribeMap` 的 `call()` 中 **source.unsafeSubscribe(parent);** `source` 指的是例子中 `Observable.interval()` 生成的对象。

再来看一下 RxJava 中对 `Observable.interval()` 的实现

```Java
    public static Observable<Long> interval(long initialDelay, long period, TimeUnit unit, Scheduler scheduler) {
        return unsafeCreate(new OnSubscribeTimerPeriodically(initialDelay, period, unit, scheduler));
    }
```
可以看出 `interval()` 和 `map()` 一样都是通过生成新的 `Observable` 并向 `Observable` 中传入与之对应的 `OnSubscribe` 的子类来完成具体操作。

```Java
// 注意：这不是 OnSubscribeTimerPeriodically 的源码
// 而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。

public final class OnSubscribeTimerPeriodically implements OnSubscribe<Long> {
    final long initialDelay;
    final long period;
    final TimeUnit unit;
    final Scheduler scheduler;

    public OnSubscribeTimerPeriodically(long initialDelay, long period, TimeUnit unit, Scheduler scheduler) {
        this.initialDelay = initialDelay;
        this.period = period;
        this.unit = unit;
        this.scheduler = scheduler;
    }

    // 传入的 Subscriber 为上文提到的 OnSubscribeMap.call() 方法中 source.unsafeSubscribe(parent);
    @Override
    public void call(final Subscriber<? super Long> child) {
        final Worker worker = scheduler.createWorker();
        child.add(worker);
        worker.schedulePeriodically(new Action0() {
            long counter;
            @Override
            public void call() {
                ...

                child.onNext(counter++);

                ...
            }

        }, initialDelay, period, unit);
    }
}
```
以上就是 RxJava 整体的逻辑结构，可以看到 RxJava 将观察者模式发挥的淋漓尽致。**整体逻辑的处理有点像递归函数的原理。而 `map()` 则像一种代理机制，通过事件拦截和处理实现事件序列的变换。**

> 总结: 精简掉细节的话，也可以这么说：在 Observable 执行了各种操作符( map, interval 等)之后 方法之后，会返回一个新的 Observable，这个新的 Observable 会像一个代理一样，负责接收原始的 Observable 发出的事件，并在处理后发送给 Subscriber。

## 参考
- [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)