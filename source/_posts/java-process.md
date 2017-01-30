---
title: Shadowsocks Android 源码解读之多进程编程
date: 2017-01-30 22:01:23
categories:
  	- 翻墙
tags:
	- Java
	- Shadowsocks
---

阅读 Shadowsocks Android 源码时，遇到了 `GuardedProcess` 类，该类是用来创建 native 层的 `ss-local`, `tun2socks`, `pdnsd`, `ss-tunnel` 进程。要运行这些进程，就牵扯到了 Java 的多进程编程，以下是关于 Java 多进程编程的实现
## Java 进程的创建
Java 提供了两种方法用来启动进程或其它程序
- 使用 Runtime 的 `exec()` 方法
- 使用 ProcessBuilder 的 `start()` 方法

### ProcessBuilder
 ProcessBuilder类，此类用于创建操作系统进程，它提供一种启动和管理进程（也就是应用程序）的方法。 
 
每个 ProcessBuilder 实例管理一个进程属性集。`start()` 方法利用这些属性创建一个新的 Process 实例。`start()` 方法可以从同一实例重复调用，以利用相同的或相关的属性创建新的子进程。

ProcessBuilder 管理以下进程属性
- 命令
    - 一个字符串列表，它表示要调用的外部程序文件及其参数（如果有）。在此，表示有效的操作系统命令的字符串列表是依赖于系统的。例如，每一个总体变量，通常都要成为此列表中的元素，但有一些操作系统，希望程序能自己标记命令行字符串――在这种系统中，Java 实现可能需要命令确切地包含这两个元素。
- 环境
    - 是从变量到值的依赖于系统的映射。初始值是当前进程环境的一个副本（请参阅 System.getenv()）。工作目录。默认值是当前进程的当前工作目录，通常根据系统属性 user.dir 来命名。
- redirectErrorStream 属性
    - 最初，此属性为 **false**，意思是子进程的标准输出和错误输出被发送给两个独立的流，这些流可以通过 `Process.getInputStream()` 和 `Process.getErrorStream()` 方法来访问。如果将值设置为 **true**，标准错误将与标准输出合并。这使得关联错误消息和相应的输出变得更容易。在此情况下，合并的数据可从 `Process.getInputStream()` 返回的流读取，而从 `Process.getErrorStream()` 返回的流读取将直接到达文件尾。

<!-- more -->

> 修改 ProcessBuilder 的属性将影响后续由该对象的 `start()` 方法启动的进程，但从不会影响以前启动的进程或 Java 自身的进程。大多数错误检查由 start() 方法执行。可以修改对象的状态，但这样 start() 将会失败。例如，将命令属性设置为一个**空列表**将不会抛出异常，除非包含了 `start()`。 
**注意，此类不是同步的。如果多个线程同时访问一个 ProcessBuilder，而其中至少一个线程从结构上修改了其中一个属性，它必须 保持外部同步。**

### Runtime
每个 Java 应用程序都有一个 Runtime 类实例，使应用程序能够与其运行的环境相连接。可以通过 `getRuntime()` 方法获取当前运行时。

应用程序不能创建自己的 Runtime 类实例。但可以通过 `getRuntime()` 方法获取当前 Runtime 运行时对象的引用。一旦得到了一个当前的 Runtime 对象的引用，就可以调用 Runtime 对象的方法去控制 Java 虚拟机的状态和行为。 

### Process
不管通过那种方法启动进程后，都会返回一个 Process 类的实例代表启动的进程，该实例可用来控制进程并获得相关信息。Process 类提供了执行从进程输入、执行输出到进程、等待进程完成、检查进程的退出状态以及销毁（杀掉）进程的方法

## 多进程编程实例
一般我们在 Java 中运行其它类中的方法时，无论是静态调用，还是动态调用，都是在当前的进程中执行的，也就是说，只有一个 Java 虚拟机实例在运行。而有的时候，我们需要通过 Java 代码启动多个 Java 子进程。这样做虽然占用了一些系统资源，但会使程序更加稳定，因为新启动的程序是在不同的虚拟机进程中运行的，如果有一个进程发生异常，并不影响其它的子进程。

### Runtime
在 Java 中我们可以使用两种方法来实现这种要求。最简单的方法就是通过 Runtime中的 `exec` 方法执行 Java classname。如果执行成功，这个方法返回一个 Process 对象，如果执行失败，将抛出一个 IOException 错误。下面让我们来看一个简单的例子。

```Java
// Test1.java文件
import java.io.*;
public class Test {
　public static void main(String[] args) {
　　FileOutputStream fOut = new FileOutputStream("c:\\Test1.txt");
　　fOut.close();
　　System.out.println("被调用成功!");
　}
}

// Test_Exec.java
public class Test_Exec {
　public static void main(String[] args) {
　　Runtime run = Runtime.getRuntime();
　　Process p = run.exec("java test1"); 
　}
}
```
通过 `java Test_Exec` 运行程序后，发现在多了个 `Test1.txt` 文件，但在控制台中并未出现"被调用成功！"的输出信息。因此可以断定，Test 已经被执行成功，但因为某种原因，Test 的输出信息未在 `Test_Exec` 的控制台中输出。这个原因也很简单，因为使用 exec 建立的是 Test_Exec 的子进程，这个子进程并没有自己的控制台，因此，它并不会输出任何信息。

如果要输出子进程的输出信息，可以通过 Process 中的 `getInputStream` 得到子进程的输出流（在子进程中输出，在父进程中就是输入），然后将子进程中的输出流从父进程的控制台输出。具体的实现代码如下如示：

```Java
// Test_Exec_Out.java
import java.io.*;
public class Test_Exec_Out {
　public static void main(String[] args) {
　　Runtime run = Runtime.getRuntime();
　　Process p = run.exec("java test1"); 
　　BufferedInputStream in = new BufferedInputStream(p.getInputStream());
　　BufferedReader br = new BufferedReader(new InputStreamReader(in));
　　String s;
　　while ((s = br.readLine()) != null)
　　　System.out.println(s); 
　}
}
```
从上面的代码可以看出，在 `Test_Exec_Out.java` 中通过按行读取子进程的输出信息，然后在 `Test_Exec_Out` 中按每行进行输出。 上面讨论的是如何得到子进程的输出信息。那么，除了输出信息，还有输入信息。既然子进程没有自己的控制台，那么输入信息也得由父进程提供。我们可以通过 Process 的 `getOutputStream` 方法来为子进程提供输入信息（即由父进程向子进程输入信息，而不是由控制台输入信息）。我们可以看看如下的代码：

```Java
// Test2.java文件
import java.io.*;
public class Test
{
　public static void main(String[] args)
　{
　　BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
　　System.out.println("由父进程输入的信息：" + br.readLine());
　}
}

// Test_Exec_In.java
import java.io.*;
public class Test_Exec_In
{
　public static void main(String[] args)
　{
　　Runtime run = Runtime.getRuntime();
　　Process p = run.exec("java test2"); 
　　BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(p.getOutputStream()));
　　bw.write("向子进程输出信息");
　　bw.flush();
　　bw.close(); // 必须得关闭流，否则无法向子进程中输入信息
　　// System.in.read();
　}
}
```

从以上代码可以看出，Test1 得到由 Test_Exec_In 发过来的信息，并将其输出。当你不加 `bw.flash()` 和 `bw.close()` 时，信息将无法到达子进程，也就是说子进程进入阻塞状态，但由于父进程已经退出了，因此，子进程也跟着退出了。如果要证明这一点，可以在最后加上 `System.in.read()`，然后通过任务管理器（在windows下）查看 java 进程，你会发现如果加上 `bw.flush()` 和 `bw.close()`，只有一个 java 进程存在，如果去掉它们，就有两个 java 进程存在。这是因为，如果将信息传给 Test2，在得到信息后，Test2 就退出了。在这里有一点需要说明一下，exec 的执行是异步的，并不会因为执行的某个程序阻塞而停止执行下面的代码。因此，可以在运行 test2 后，仍可以执行下面的代码。
exec 方法经过了多次的重载。上面使用的只是它的一种重载。它还可以将命令和参数分开，如 exec("java.test2") 可以写成 exec("java", "test2")。exec 还可以通过指定的环境变量运行不同配置的 java 虚拟机。
### ProcessBuilder
除了使用Runtime的exec方法建立子进程外，还可以通过ProcessBuilder建立子进程。ProcessBuilder的使用方法如下：

```Java
// Test_Exec_Out.java
import java.io.*;
public class Test_Exec_Out
{
　public static void main(String[] args)
　{
　　ProcessBuilder pb = new ProcessBuilder("java", "test1");
　　Process p = pb.start();
　　… …
　}
}
```

在建立子进程上，ProcessBuilder 和 Runtime 类似，不同的 ProcessBuilder 使用 `start()` 方法启动子进程，而 Runtime 使用 exec 方法启动子进程。得到 Process 后，它们的操作就完全一样的。

ProcessBuilder 和 Runtime 一样，也可设置可执行文件的环境信息、工作目录等。下面的例子描述了如何使用 ProcessBuilder 设置这些信息。

```Java
ProcessBuilder pb = new ProcessBuilder("Command", "arg2", "arg2", ''');
// 设置环境变量
Map<String, String> env = pb.environment();
env.put("key1", "value1");
env.remove("key2");
env.put("key2", env.get("key1") + "_test"); 
pb.directory("..\abcd"); // 设置工作目录
Process p = pb.start(); // 建立子进程
```

## 参考文献
[详解 Java 中多进程编程的实现](http://www.ctolib.com/topics-62414.html)