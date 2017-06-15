---
title: Java 8 之 Lambda 表达式
date: 2017-02-17 18:19:10
categories:
  - Java
tags:
  - Java
  - Android
---

## Lambda
在介绍 Lambda 表达式之前，我们先来看只有单个方法的 Interface（通常我们称之为回调接口）：

```Java
public interface OnClickListener {
    void onClick(View v);
}
```
我们是这样使用它的：

```Java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        v.setText("lalala");
    }
  });
```
这种回调模式在 Android 中非常流行,但是像上面这样的匿名内部类并不是一个好的选择，因为：
- 语法冗余
- 匿名内部类中的 this 指针和变量容易产生误解
- 无法捕获非 final 局部变量
- **非静态内部类默认持有外部类的引用，部分情况下会导致外部类无法被 GC 回收，导致内存泄露**

<!-- more -->

而 Java 8 中引入的 Lambda 表达式就解决了以上的问题，先来看一个简单的 Lambda 用法：

```Java
 //多行注释中为不使用Lambda的写法
        
/**
Runnable run = new Runnable() {
    @Override
    public void run() {
        System.out.println("Test")
    }
};
*/

Runnable run = () -> System.out.println("Test");
```
我们来看一下Runnable接口：

```Java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
### 函数式接口
Java 8 引入了 `FunctionalInterface` 注解来表明一个接口打算成为一个函数式接口。
> 函数式接口: 只定义了一个抽象方法的接口称之为函数式接口

在实际使用中不管 `FunctionalInterface` 注解是否存在，Java 编译器都会将所有满足该定义的接口看作是函数式接口。

下面我们自己写一个接口看一下能否如Runnable一般：

```Java
public interface TestLambdaInterface {
    void testInterface();
}
```
我们如下使用该接口，程序能正常编译并输出了结果，说明 Java 编译器都会将所有满足该定义的接口看作是函数式接口：

```Java
TestLambdaInterface test = ()->System.out.println("Test");

test.testInterface();
```
当我们将方法的返回值改成String时，程序也能正常编译并输出了结果：

```Java
TestLambdaInterface test = ()->"test";

test.testInterface();
```
> **说明函数式接口与接口中定义方法的返回值无关**

下面我们在接口中增加一个方法
```Java
public interface TestLambdaInterface {
    void testInterface();
    void testInterface2();
}
```

程序不能正常通过编译

```bash
D:\Work\LearnJavaFX\src>javac -encoding utf-8 LambdaTest.java
LambdaTest.java:20: 错误: 不兼容的类型: TestLambdaInterface 不是函数接口
        TestLambdaInterface test = () -> System.out.println("Test");
                                   ^
    在 接口 TestLambdaInterface 中找到多个非覆盖抽象方法
1 个错误
```
> 说明函数式接口只能有一个抽象方法

我们再来看看Java中提供的BinaryOperator接口：

```Java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {

    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```

其实现的接口 BiFunction 的代码如下:

```Java
@FunctionalInterface
public interface BiFunction<T, U, R> {

    R apply(T t, U u);

    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
}
```

该接口可以使用如下方法生成实例:

```Java
BinaryOperator<Long> add = (x, y) -> x + y;
```
> 说明函数式接口与接口中是否存在 default 与 static 方法无关

**以上充分说明能成为函数式接口【可以使用 Lambda 来简化接口操作】的条件为：只定义了一个抽象方法的接口。**

### Lambda 书写方式

```Java

        /**
         * 对于只有一个抽象方法且返回值为void可以使用下面的方法快捷的生成接口的实例
         */
        Runnable voidFunc = () -> System.out.println("没有参数的函数式接口");
        /**
         * 上面的声明其实完整写法应该为：
         *  Runnable voidFunc = () -> {System.out.println("没有参数的函数式接口")};
         * 对于只有一行语句时，可以省略{}，
         * 下面是多行语句的情况，{}不能省略
         */
        Runnable muitiStatement = () -> {
            System.out.println("第一行");
            System.out.println("第二行");
            //多行语句....
        };

        /**
         * 对于有接口中有参数哦的方法我们可以使用
         * ActionListener oneParam = (event) -> System.out.println("一个参数");
         * 当只有一个参数时，可以省略()如下：
         */
        ActionListener oneParam = event -> System.out.println("一个参数");

        /**
         * 多参数就必须将参数放入到()中调用，按照参数顺序写入
         */
        BinaryOperator<Long> add = (x, y) -> x + y;

        /**
         * 上面几种情况中，接口的参数在编译时进行类型的推断，我们亦可以显式声明一下【这样可以避免很多错误】
         */
        BinaryOperator<Long> add2 = (Long x, Long y) -> x + y;
```
Lambda 表达式语法由`参数列表`、`->` 和`函数体`组成。函数体既可以是一个表达式也可以是一个代码块。
- **表达式**：表达式会被执行然后返回结果。它简化掉了 `return` 关键字
- **代码块**：顾名思义就是一坨代码，和普通方法中的语句一样

### 目标类型
通过前面的例子我们可以看到，lambda 表达式没有名字，那我们怎么知道它的类型呢？答案是通过上下文推导而来的。例如，下面的表达式的类型是 OnClickListener

```Java
ClickListener listener = (View v) -> {v.setText("lalala");};
```

这就意味着同样的lambda表达式在不同的上下文里有不同的类型

```Java
Runnable runnable = () -> doSomething();  //这个表达式是 Runnable 类型的
Callback callback = () -> doSomething();  //这个表达式是 Callback 类型的
```
编译器利用 lambda 表达式所在的上下文所期待的类型来推导表达式的类型，这个被期待的类型被称为目标类型。lambda 表达式只能出现在目标类型为函数式接口的上下文中。

Lambda 表达式的类型和目标类型的方法签名必须一致，编译器会对此做检查，一个 lambda 表达式要想赋值给目标类型 `T` 则必须满足下面所有的条件：
- `T` 是一个函数式接口
- lambda 表达式的参数必须和 `T` 的方法参数在数量、类型和顺序上一致（一一对应）
- lambda 表达式的返回值必须和 `T` 的方法的返回值一致或者是它的子类
- lambda 表达式抛出的异常和 `T` 的方法的异常一致或者是它的子类

由于目标类型是知道 lambda 表达式的参数类型，所以我们没必要把已知的类型重复一遍。也就是说 lambda 表达式的参数类型可以从目标类型获取：

```Java
//编译器可以推导出s1和s2是String类型
Comparator<String> c = (s1, s2) -> s1.compareTo(s2);

button.setOnClickListener(v -> v.setText("lalala"));

```
### 作用域
在内部类中使用变量名和this非常容易出错。内部类通过继承得到的成员变量（包括来说 object 的）可能会把外部类的成员变量覆盖掉，未做限制的 this 引用会指向内部类自己而非外部类。

而 lambda 表达式的语义就十分简单：它不会从父类中继承任何变量，也不用引入新的作用域。lambda 表达式的参数及函数体里面的变量和它外部环境的变量具有相同的语义（this 关键字也是一样）。

```Java
public class HelloLambda {

    Runnable r1 = () -> System.out.println(this);
    Runnable r2 = () -> System.out.println(toString());

    @Override
    public String toString() {
        return "Hello, lambda!";
    }

    public static void main(String[] args) {
        new HelloLambda().r1.run();  
        new HelloLambda().r2.run();
    }
}
```
上面的代码最终会打印两个 `Hello, lambda!`，与之相类似的内部类则会打印出类似 `HelloLambda$1@32a890` 和 `HelloLambda$1@6b32098` 这种出乎意料的字符串。

> 基于词法作用域的理念，lambda表达式不可以掩盖任何其所在上下文的局部变量。

### 变量
在 JDK8 对匿名内部类中调用类之外变量必须为final的限制进行了一定的放宽：

```Java
String name = "123";

Runnable run = new Runnable() {
    @Override
    public void run() {
      //在JDK8之前下面的语句是会报错的，只有name是final时才可以使用，但是JDK8中，这样的语句就不会报错
       System.out.println(name);
    }
};
```

虽然 JDK8 中上面的语句时可以的，但这只是减少了我们代码中的编写，编译时还是会将 name 当做是 final 所以当有以下几种情况时，编译还是会报错：

```Java
          
        //1.对name进行了第二次赋值，这样编译器就会认为name不是final类型
        String name = "123";
        name="";
        Runnable run = () -> {
            System.out.println(name);
        };
        
        //2.在匿名类方法中对变量进行赋值也是不允许的
        String name = "123";  
        Runnable run = () -> {
            name="";
            System.out.println(name);
        };
```

> 如果真的需要在Lambda中或者匿名中对变量赋值，那么应该将其放入数组，或者当做一个类的属性来传递**对象final时可以设置其属性，不能设置其引用**

## 参考
- [Java8新特性第1章(Lambda表达式)](http://www.jianshu.com/p/6e400da4a239)
- [Java Lambda](https://my.oschina.net/coderknock/blog/716246)