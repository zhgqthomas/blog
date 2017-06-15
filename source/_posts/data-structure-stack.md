---
title: 数据结构之栈
date: 2017-02-15 18:24:21
categories:
  - 数据结构
tags: 
	- 数据结构
---

## 堆栈
堆栈是一种特殊的线性表，堆栈的数据元素以及数据元素间的逻辑关系和线性表完全相同，其差别是线性表允许在任意位置进行插入和删除操作，而堆栈只允许在固定一端进行插入和删除操作。因此也被成为 LIFO(Last In, First Out, 后进先出),

堆栈中允许进行插入和删除操作的一端称为栈顶，另一端称为栈底。堆栈的插入和删除操作通常称为进栈或入栈，堆栈的删除操作通常称为出栈或退栈。

常见的实现方式包括数组实现，容器实现和链表实现。

### 基本算法
1. 进栈（PUSH）算法
    1. 若 TOP > n 时，则给出溢出信息，作出错处理(进栈前首先检查栈是否已满，满则溢出；不满则继续)
    2. 置 TOP=TOP+1 (栈指针加 1，指向进栈地址)
    3. S(TOP)=X，结束( X 为新进栈的元素)
2. 退栈（POP）算法
    1. 若TOP < 0，则给出下溢信息，作出错处理(退栈前先检查是否已为空栈， 空则下溢；不空则作②)
    2. X=S(TOP)，（退栈后的元素赋给X）
    3. TOP=TOP-1，结束（栈指针减1，指向栈顶)

### 顺序栈
顺序存储结构的堆栈称作顺序堆栈。其存储结构示意图如下图所示：

![](http://upload-images.jianshu.io/upload_images/854027-f72938d07787fe69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<!-- more -->

#### 数组实现
```Java
public class ArrayStack {

    private int[] stack;
    private int maxSize;
    private int top;

    public ArrayStack(int maxSize) {
        this.maxSize = maxSize;
        stack = new int[maxSize];
        top = -1;
    }

    public void push(int data) {
        if (isFull()) {
            throw new StackOverflowError();
        }

        // 先让 top + 1 后赋值
        stack[++top] = data;
    }

    public int pop() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }

        // 先返回值，再 top - 1
        return stack[top--];
    }

    public int peek() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }
        return stack[top];
    }

    public boolean isEmpty() {
        return top == -1;
    }

    public boolean isFull() {
        return top == maxSize;
    }
}
```
#### 容器实现
```Java
public class ListStack<T> {

    private List<T> stack;

    public ListStack() {
        stack = new ArrayList<>();
    }

    public boolean isEmpty() {
        return stack.size() == 0;
    }


    public void push(T data) {
        stack.add(data);
    }

    public T pop() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }

        return stack.remove(stack.size() - 1);
    }

    public T peek() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }

        return stack.get(stack.size() - 1);
    }
    
}
```

### 链式堆栈
链式存储结构的堆栈称作链式堆栈。

与单链表相同，链式堆栈也是由一个个结点组成的，每个结点由两个域组成，一个是存放数据元素的数据元素域 data，另一个是存放指向下一个结点的对象引用（即指针）域 next。

堆栈有两端，插入数据元素和删除数据元素的一端为栈顶，另一端为栈底。链式堆栈都设计成把靠近堆栈头head的一端定义为栈顶。

依次向链式堆栈入栈数据元素a0, a1, a2, ..., an-1后，链式堆栈的示意图如下图所示：　

![](http://upload-images.jianshu.io/upload_images/854027-7f646d4b855ad809.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```Java
public class LinkedStack<T> {

    class Node {
        T data;
        Node next;

        public Node() {
        }

        public Node(T data) {
            this.data = data;
        }

        public Node(T data, Node next) {
            this.data = data;
            this.next = next;
        }
    }

    private Node top;

    public void push(T data) {
        top = new Node(data, top);
    }

    public T pop() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }

        T data = top.data;
        top = top.next;
        return data;
    }

    public T peek() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }

        return top.data;
    }

    public boolean isEmpty() {
        return null == top;
    }
}
```
### Java 中栈与堆的区别
#### 栈(stack):（线程私有）
是一个先进后出的数据结构，通常用于保存方法(函数)中的参数，局部变量。在java中,所有基本类型和引用类型的引用都在栈中存储。栈中数据的生存空间一般在当前scopes内(就是由{...}括起来的区域)。
#### 堆(heap):（线程共享）
是一个可动态申请的内存空间(其记录空闲内存空间的链表由操作系统维护)，C中的malloc语句所产生的内存空间就在堆中。在java中，所有使用new xxx()构造出来的对象都在堆中存储，当垃圾回收器检测到某对象未被引用，则自动销毁该对象。所以，理论上说java中对象的生存空间是没有限制的，只要有引用类型指向它，则它就可以在任意地方被使用。
### hashCode 与对象之间的关系
如果两个对象的hashCode不相同，那么这两个对象肯定也不同。

如果两个对象的hashCode相同，那么这两个对象有可能相同，也有可能不同。

总结一句：**不同的对象可能会有相同的hashCode；但是如果hashCode不同，那肯定不是同一个对象。**
```Java
public class StringTest {

    public static void main(String[] args) {

        //s1 和 s2 其实是同一个对象。对象的引用存放在栈中，对象存放在方法区的字符串常量池
        String s1 = "china";
        String s2 = "china";

        //凡是用new关键创建的对象，都是在堆内存中分配空间。
        String s3 = new String("china");

        //凡是new出来的对象，绝对是不同的两个对象。
        String s4 = new String("china");

        System.out.println(s1 == s2);  //true
        System.out.println(s1 == s3);
        System.out.println(s3 == s4);
        System.out.println(s3.equals(s4));

        System.out.println("\n-----------------\n");
      /*String很特殊，重写从父类继承过来的hashCode方法，使得两个
       *如果字符串里面的内容相等，那么hashCode也相等。
       **/

        //hashCode相同
        System.out.println(s3.hashCode());  //hashCode为94631255
        System.out.println(s4.hashCode());  //hashCode为94631255

        //identityHashCode方法用于获取原始的hashCode
        //如果原始的hashCode不同，表明确实是不同的对象

        //原始hashCode不同
        System.out.println(System.identityHashCode(s3)); //2104928456
        System.out.println(System.identityHashCode(s4)); //2034442961

        System.out.println("\n-----------------\n");

        //hashCode相同
        System.out.println(s1.hashCode());  //94631255
        System.out.println(s2.hashCode());  //94631255

        //原始hashCode相同：表明确实是同一个对象
        System.out.println(System.identityHashCode(s1));  //648217993
        System.out.println(System.identityHashCode(s2));  //648217993
    }
}
```
上面的代码中，注释已经标明了运行的结果。通过运行结果我们可以看到，s3和s4的字符串内容相同，但他们是两个不同的对象，由于String类重写了hashCode方法，他们的hashCode相同，但原始的hashCode是不同的。

## 参考
[数据结构Java实现05----栈：顺序栈和链式堆栈
](http://www.cnblogs.com/smyhvae/p/4789699.html)