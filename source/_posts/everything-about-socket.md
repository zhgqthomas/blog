---
title: 关于 Socket 编程的一切
date: 2017-01-10 17:25:08
categories:
  	- 网络编程
tags:
	- Socket
---
### Socket 简介
#### 协议简介
- 协议相当于相互通信的程序间达成的一种约定，它规定了分组报文的结构、交换方式、包含的意义以及怎样对报文所包含的信息进行解析。
- TCP/IP 协议族有 IP 协议、TCP 协议和 UDP 协议。
- TCP 协议和 UDP 协议使用的地址叫做端口号，用来区分同一主机上的不同应用程序。TCP 协议和 UDP 协议也叫端到端传输协议，因为他们将数据从一个应用程序传输到另一个应用程序，而 IP 协议只是将数据从一个主机传输到另一个主机。
- 在 TCP/IP 协议中，有两部分信息用来确定一个指定的程序：互联网地址和端口号：其中互联网地址由 IP 协议使用，而附加的端口地址信息则由传输协议（TCP 或 UDP 协议）对其进行解析。
- 现在 TCP/IP 协议族中的主要 socket 类型为流套接字（使用 TCP 协议）和数据报套接字（使用 UDP 协议），其中通过数据报套接字，应用程序一次只能发送最长 65507 个字节长度的信息。
- 一个 TCP/IP 套接字由一个互联网地址，一个端对端协议（TCP 协议或 UDP 协议）以及一个端口号唯一确定。
- 每个端口都标识了一台主机上的一个应用程序，实际上，一个端口确定了一个主机上的一个套接字。主机中的多个程序可以同时访问同一个套接字，在实际应用中，访问相同套接字的不同程序通常都属于一个应用（如 web 服务程序的多个副本），但从理论上讲，它们可以属于不同的应用。

<!-- more -->

#### 基本套接字
- 编写 TCP 客户端程序，在实例化 Socket 类时，要注意，底层的 TCP 协议只能处理 IP 协议，如果传递的第一个参数是主机名字而不是你 IP 地址，Socket 类具体实现的时候会将其解析成相应的地址，若因为某些原因连接失败，构造函数会抛出一个 IOException 异常。
- TCP 协议读写数据时，read()方法在没有可读数据时会阻塞等待，直到有新的数据可读。另外，TCP 协议并不能确定在 read()和 write()方法中所发送信息的界限，接收或发送的数据可能被 TCP 协议分割成了多个部分。
- 编写 TCP 服务器端的程序将在 accept()方法处阻塞，以等待客户端的连接请求，一旦取得连接，便要为每个客户端的连接建立一个 Socket 实例来进行数据通信。
- 在 UDP 程序中，创建 DatagramPacket 实例时，如果没有指定远程主机地址和端口，则该实例用来接收数据（尽管可以调用 setXXX()等方法指定），如果指定了远程主机地址和端口，则该实例用来发送数据。
- UDP 程序在 receive()方法处阻塞，直到收到一个数据报文或等待超时。由于 UDP 协议是不可靠协议，如果数据报在传输过程中发生丢失，那么程序将会一直阻塞在 receive()方法处，这对客户端来说是肯定不行的，为了避免这个问题，我们在客户端使用 DatagramSocket 类的 setSoTimeout()方法来制定 receive()方法的最长阻塞时间，并指定重发数据报的次数，如果每次阻塞都超时，并且重发次数达到了设置的上限，则关闭客户端。
- UDP 服务器为所有通信使用同一套接字，这点与 TCP 服务器不同，TCP 服务器则为每个成功返回的 accept()方法创建一个新的套接字。
- 在 UDP 程序中，DatagramSocket 的每一次 receive()调用最多只能接收调用一次 send()方法所发送的数据，而且，不同的 receive()方法调用绝对不会返回同一个 send()方法所发送的额数据。
- 在 UDP 套接字编程中，如果 receive()方法在一个缓冲区大小为 n 的 DatagramPscket 实例中调用，而接受队列中的第一个消息长度大于 n，则 receive()方法只返回这条消息的前 n 个字节，超出的其他字节部分将自动被丢弃，而且也没有任何消息丢失的提示。因此，接受者应该提供一个足够大的缓存空间的 DatagramPacket 实例，以完整地存放调用 receive() 方法时应用程序协议所允许的最大长度的消息。一个 DatagramPacket 实例中所运行传输的最大数据量为 65507 个字节，即 UDP 数据报文所能负载的最多数据，因此，使用一个有 65600 字节左右缓存数组的数据总是安全的。
- 在 UDP 套接字编程中，每一个 DatagramPacket 实例都包含一个内部消息长度值，而该实例一接收到新消息，这个长度值便可能改变（以反映实际接收的消息的字节数）。如果一个应用程序使用同一个 DatagramPacket 实例多次调用 receive()方法，每次调用前就必须显式地将消息的内部长度重置为缓冲区的实际长度。
- 另一个潜在问题的根源是 DatagramPacket 类的 getData()方法,该方法总是返回缓冲区的原始大小，忽略了实际数据的内部偏移量和长度信息。

### Java TCP Socket 编程
#### Java 对 TCP 的支持
TCP 协议提供面向连接的服务，通过它建立的是可靠地连接。Java 为 TCP 协议提供了两个类：Socke 类和 ServerSocket 类。
- Socket 实例代表了 TCP 连接的一个客户端
- ServerSocket 实例代表了 TCP 连接的一个服务器端

服务端在调用 accept（）等待客户端的连接请求时会阻塞，直到收到客户端发送的连接请求才会继续往下执行代码，因此**要为每个 Socket 连接开启一个线程。**

服务器端要同时处理 ServerSocket 实例和 Socket 实例，而客户端只需要使用 Socket 实例。

每个 Socket 实例会关联一个 InputStream 和 OutputStream 对象，我们**通过将字节写入套接字的 OutputStream 来发送数据，并通过从 InputStream 来接收数据。**

#### TCP 连接的建立步骤
客户端向服务器端发送连接请求后，就被动地等待服务器的响应。典型的 TCP 客户端要经过下面三步操作：
1. 创建一个 Socket 实例：构造函数向指定的远程主机和端口建立一个 TCP 连接
2. 通过套接字的 I/O 流与服务端通信
3. 使用 Socket 类的 close 方法关闭连接

服务端的工作是建立一个通信终端，并被动地等待客户端的连接。典型的 TCP 服务端执行如下两步操作：
1. 创建一个 ServerSocket 实例并指定本地端口，用来监听客户端在该端口发送的 TCP 连接请求
2. 重复执行:
    1. 调用 ServerSocket 的 accept（）方法以获取客户端连接，并通过其返回值创建一个 Socket 实例
    2. 为返回的 Socket 实例开启新的线程，并使用返回的 Socket 实例的 I/O 流与客户端通信； 通信完成后，使用 Socket 类的 close（）方法关闭该客户端的套接字连接

#### TCP Socket Demo
下面给出一个客户端服务端 TCP 通信的 Demo，该客户端在 20006 端口请求与服务端建立 TCP 连接，客户端不断接收键盘输入，并将其发送到服务端，服务端在接收到的数据前面加上“echo”字符串，并将组合后的字符串发回给客户端，如此循环，直到客户端接收到键盘输入“bye”为止。

Client: 
```Java
package zyb.org.client;  

import java.io.BufferedReader;  
import java.io.IOException;  
import java.io.InputStreamReader;  
import java.io.PrintStream;  
import java.net.Socket;  
import java.net.SocketTimeoutException;  

public class Client1 {  
    public static void main(String[] args) throws IOException {  
        //客户端请求与本机在20006端口建立TCP连接   
        Socket client = new Socket("127.0.0.1", 20006);  
        client.setSoTimeout(10000);  
        //获取键盘输入   
        BufferedReader input = new BufferedReader(new InputStreamReader(System.in));  
        //获取Socket的输出流，用来发送数据到服务端    
        PrintStream out = new PrintStream(client.getOutputStream());  
        //获取Socket的输入流，用来接收从服务端发送过来的数据    
        BufferedReader buf =  new BufferedReader(new InputStreamReader(client.getInputStream()));  
        boolean flag = true;  
        while(flag){  
            System.out.print("输入信息：");  
            String str = input.readLine();  
            //发送数据到服务端    
            out.println(str);  
            if("bye".equals(str)){  
                flag = false;  
            }else{  
                try{  
                    //从服务器端接收数据有个时间限制（系统自设，也可以自己设置），超过了这个时间，便会抛出该异常  
                    String echo = buf.readLine();  
                    System.out.println(echo);  
                }catch(SocketTimeoutException e){  
                    System.out.println("Time out, No response");  
                }  
            }  
        }  
        input.close();  
        if(client != null){  
            //如果构造函数建立起了连接，则关闭套接字，如果没有建立起连接，自然不用关闭  
            client.close(); //只关闭socket，其关联的输入输出流也会被关闭  
        }  
    }  
}  
```
Server 处理 TCP 连接请求的代码如下:
```Java
package zyb.org.server;  

import java.net.ServerSocket;  
import java.net.Socket;  

public class Server1 {  
    public static void main(String[] args) throws Exception{  
        //服务端在20006端口监听客户端请求的TCP连接  
        ServerSocket server = new ServerSocket(20006);  
        Socket client = null;  
        boolean f = true;  
        while(f){  
            //等待客户端的连接，如果没有获取连接  
            client = server.accept();  
            System.out.println("与客户端连接成功！");  
            //为每个客户端连接开启一个线程  
            new Thread(new ServerThread(client)).start();  
        }  
        server.close();  
    }  
}  
```
Server 需要用到多线程，以下是单独的多线程类:
```Java
package zyb.org.server;  

import java.io.BufferedReader;  
import java.io.InputStreamReader;  
import java.io.PrintStream;  
import java.net.Socket;  

/** 
 * 该类为多线程类，用于服务端 
 */  
public class ServerThread implements Runnable {  

    private Socket client = null;  
    public ServerThread(Socket client){  
        this.client = client;  
    }  

    @Override  
    public void run() {  
        try{  
            //获取Socket的输出流，用来向客户端发送数据  
            PrintStream out = new PrintStream(client.getOutputStream());  
            //获取Socket的输入流，用来接收从客户端发送过来的数据  
            BufferedReader buf = new BufferedReader(new InputStreamReader(client.getInputStream()));  
            boolean flag =true;  
            while(flag){  
                //接收从客户端发送过来的数据  
                String str =  buf.readLine();  
                if(str == null || "".equals(str)){  
                    flag = false;  
                }else{  
                    if("bye".equals(str)){  
                        flag = false;  
                    }else{  
                        //将接收到的字符串前面加上echo，发送到对应的客户端  
                        out.println("echo:" + str);  
                    }  
                }  
            }  
            out.close();  
            client.close();  
        }catch(Exception e){  
            e.printStackTrace();  
        }  
    }  

}  
```
### UDP Socket 编程
#### Java 对 UDP 的支持
UDP 协议提供的服务不同于 TCP 协议的端到端服务，它是面向非连接的，属不可靠协议，UDP 套接字在使用前不需要进行连接。实际上，UDP 协议只实现了两个功能：
- 在 IP 协议的基础上添加了端口
- 对传输过程中可能产生的数据错误进行了检测，并抛弃已经损坏的数据

Java 通过 DatagramPacket 类和 DatagramSocket 类来使用 UDP 套接字，客户端和服务器端都通过DatagramSocket 的 send()方法和 receive()方法来发送和接收数据，用 DatagramPacket 来包装需要发送或者接收到的数据。

#### UDP 通信建立的步骤
UDP 客户端首先向被动等待联系的服务器发送一个数据报文。一个典型的 UDP 客户端要经过下面三步操作：
1. 创建一个 DatagramSocket 实例，可以有选择对本地地址和端口号进行设置，如果设置了端口号，则客户端会在该端口号上监听从服务器端发送来的数据
2. 使用 DatagramSocket 实例的 send()和 receive()方法来发送和接收 DatagramPacket 实例，进行通信
3. 通信完成后，调用 DatagramSocket 实例的 close()方法来关闭该套接字

由于 UDP 是无连接的，因此UDP服务端不需要等待客户端的请求以建立连接。另外，**UDP服务器为所有通信使用同一套接字，这点与TCP服务器不同，TCP服务器则为每个成功返回的accept()方法创建一个新的套接字**。一个典型的UDP服务端要经过下面三步操作
1. 创建一个 DatagramSocket 实例，指定本地端口号，并可以有选择地指定本地地址，此时，服务器已经准备好从任何客户端接收数据报文
2. 使用 DatagramSocket 实例的 receive()方法接收一个 DatagramPacket 实例，当 receive()方法返回时，数据报文就包含了客户端的地址，这样就知道了回复信息应该发送到什么地方
3. 使用 DatagramSocket 实例的 send()方法向服务器端返回 DatagramPacket 实例

#### UDP Socket Demo
下面给出一个客户端服务端 UDP 通信的 Demo（没有用多线程），该客户端在本地 9000 端口监听接收到的数据，并将字符串"Hello UDPserver"发送到本地服务器的 3000 端口，服务端在本地 3000 端口监听接收到的数据，如果接收到数据，则返回字符串"Hello UDPclient"到该客户端的 9000 端口。在客户端，由于程序可能会一直阻塞在 receive()方法处，因此这里我们在客户端用 DatagramSocket 实例的 setSoTimeout()方法来指定 receive（）的最长阻塞时间，并设置重发数据的次数，如果最终依然没有接收到从服务端发送回来的数据，我们就关闭客户端。

Client:
```Java
package zyb.org.UDP;  

import java.io.IOException;  
import java.io.InterruptedIOException;  
import java.net.DatagramPacket;  
import java.net.DatagramSocket;  
import java.net.InetAddress;  

public class UDPClient {  
    private static final int TIMEOUT = 5000;  //设置接收数据的超时时间  
    private static final int MAXNUM = 5;      //设置重发数据的最多次数  
    public static void main(String args[])throws IOException{  
        String str_send = "Hello UDPserver";  
        byte[] buf = new byte[1024];  
        //客户端在9000端口监听接收到的数据  
        DatagramSocket ds = new DatagramSocket(9000);  
        InetAddress loc = InetAddress.getLocalHost();  
        //定义用来发送数据的DatagramPacket实例  
        DatagramPacket dp_send= new DatagramPacket(str_send.getBytes(),str_send.length(),loc,3000);  
        //定义用来接收数据的DatagramPacket实例  
        DatagramPacket dp_receive = new DatagramPacket(buf, 1024);  
        //数据发向本地3000端口  
        ds.setSoTimeout(TIMEOUT);              //设置接收数据时阻塞的最长时间  
        int tries = 0;                         //重发数据的次数  
        boolean receivedResponse = false;     //是否接收到数据的标志位  
        //直到接收到数据，或者重发次数达到预定值，则退出循环  
        while(!receivedResponse && tries<MAXNUM){  
            //发送数据  
            ds.send(dp_send);  
            try{  
                //接收从服务端发送回来的数据  
                ds.receive(dp_receive);  
                //如果接收到的数据不是来自目标地址，则抛出异常  
                if(!dp_receive.getAddress().equals(loc)){  
                    throw new IOException("Received packet from an umknown source");  
                }  
                //如果接收到数据。则将receivedResponse标志位改为true，从而退出循环  
                receivedResponse = true;  
            }catch(InterruptedIOException e){  
                //如果接收数据时阻塞超时，重发并减少一次重发的次数  
                tries += 1;  
                System.out.println("Time out," + (MAXNUM - tries) + " more tries..." );  
            }  
        }  
        if(receivedResponse){  
            //如果收到数据，则打印出来  
            System.out.println("client received data from server：");  
            String str_receive = new String(dp_receive.getData(),0,dp_receive.getLength()) +   
                    " from " + dp_receive.getAddress().getHostAddress() + ":" + dp_receive.getPort();  
            System.out.println(str_receive);  
            //由于dp_receive在接收了数据之后，其内部消息长度值会变为实际接收的消息的字节数，  
            //所以这里要将dp_receive的内部消息长度重新置为1024  
            dp_receive.setLength(1024);     
        }else{  
            //如果重发MAXNUM次数据后，仍未获得服务器发送回来的数据，则打印如下信息  
            System.out.println("No response -- give up.");  
        }  
        ds.close();  
    }    
} 
```

#### 注意的地方
##### UDP 套接字和 TCP 套接字的一个微小但重要的差别：UDP 协议保留了消息的边界信息。
DatagramSocket 的每一次 receive()调用最多只能接收调用一次 send()方法所发送的数据，而且，不同的 receive()方法调用绝对不会返回同一个 send()方法所发送的额数据。

当在 TCP 套接字的输出流上调用 write()方法返回后，所有调用者都知道数据已经被复制到一个传输缓存区中，实际上此时数据可能已经被发送，也有可能还没有被传送，而 UDP 协议没有提供从网络错误中恢复的机制，因此，并不对可能需要重传的数据进行缓存。这就意味着，当send（）方法调用返回时，消息已经被发送到了底层的传输信道中。

##### UDP 数据报文所能负载的最多数据，亦及一次传送的最大数据为 65507 个字节
当消息从网络中到达后，其所包含的数据被 TCP 的 read()方法或 UDP 的 receive()方法返回前，数据存储在一个先进先出的接收数据队列中。对于已经建立连接的 TCP 套接字来说，所有已接受但还未传送的字节都看作是一个连续的字节序列。然而，对于 UDP 套接字来说，接收到的数据可能来自不同的发送者，一个 UDP 套接字所接受的数据存放在一个消息队列中，每个消息都关联了其源地址信息，每次 receive()调用只返回一条消息。如果 receive()方法在一个缓存区大小为 n 的 DatagramPacket 实例中调用，而接受队里中的第一条消息的长度大于 n，则 receive()方法只返回这条消息的前 n 个字节，超出部分会被自动放弃，而且对接收程序没有任何消息丢失的提示！

出于这个原因，接受者应该提供一个有足够大的缓存空间的 DatagramPacket 实例，以完整地存放调用 receive()方法时应用程序协议所允许的最大长度的消息。一个 DatagramPacket 实例中所允许传输的最大数据量为 65507 个字节，也即是 UDP 数据报文所能负载的最多数据。因此，可以用一个 65600 字节左右的缓存数组来接受数据。

##### DatagramPacket 的内部消息长度值在接收数据后会发生改变，变为实际接收到的数据的长度值
每一个 DatagramPacket 实例都包含一个内部消息长度值，其初始值为 byte 缓存数组的长度值，而该实例一旦接受到消息，这个长度值便会变为接收到的消息的实际长度值，这一点可以用 DatagramPacket 类的 getLength()方法来测试。如果一个应用程序使用同一个 DatagramPacket 实例多次调用 receive()方法，每次调用前就必须显式地将其内部消息长度重置为缓存区的实际长度，以免接受的数据发生丢失

##### DatagramPacket 的 getData()方法总是返回缓冲区的原始大小，忽略了实际数据的内部偏移量和长度信息
由于 DatagramPacket 的 getData()方法总是返回缓冲数组的原始大小，即刚开始创建缓冲数组时指定的大小，在上面程序中，该长度为 1024，因此如果我们要获取接收到的数据，就必须截取 getData()方法返回的数组中只含接收到的数据的那一部分。 在 Java1.6 之后，我们可以使用 Arrays.copyOfRange()方法来实现，只需一步便可实现以上功能
```Java
byte[] destbuf = Arrays.copyOfRange(dp_receive.getData(),dp_receive.getOffset(),
dp_receive.getOffset() + dp_receive.getLength());
```
当然，如果要将接收到的字节数组转换为字符串的话，也可以采用本程序中直接 new 一个 String 对象的方法
```Java
new String(dp_receive.getData(),dp_receive.getOffset(),
dp_receive.getOffset() + dp_receive.getLength());
```