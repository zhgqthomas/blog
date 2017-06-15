---
title: HTTP协议原理
date: 2016-08-18 18:36:07
categories:
  - 网络编程
tags: 
	- HTTP
---

> 原文出处： [刘望舒的专栏](http://blog.csdn.net/itachi85/article/details/50982995)

#####前言
这篇文章是这个系列的开篇，作为移动开发者，开发的应用不免会对网络进行访问，虽然现在已经有很多的开源库帮助我们可以轻而易举的访问网络，但是我们仍要去了解网络访问的原理，这也是一个优秀开发人员所必备的知识点。这篇文章我们就先来了解一下 HTTP 协议原理。
##### HTTP 简介
HTTP 是一个属于应用层的面向对象的协议，由于其简捷、快速的方式，适用于分布式超媒体信息系统。它于 1990 年提出，经过几年的使用与发展，得到不断地完善和扩展。
######HTTP协议的主要特点
- 支持 C/S（客户/服务器）模式。
- 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有 GET、HEAD、POST，每种方法规定了客户与服务器联系的类型不同。由于 HTTP 协议简单，使得 HTTP 服务器的程序规模小，因而通信速度很快。
- 灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由 Content-Type 加以标记。
- 无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
- 无状态：HTTP 协议是无状态协议，无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

###### HTTP URL 的格式如下
```
http://host[":"port][abs_path]
```
http 表示要通过 HTTP 协议来定位网络资源；host 表示合法的 Internet 主机域名或者 IP 地址；port 指定一个端口号，为空则使用默认端口 80；abs_path 指定请求资源的 URI（Web 上任意的可用资源）。HTTP 有两种报文分别是请求报文和响应报文，让我们先来看看请求报文。
##### HTTP的请求报文
先来看看请求报文的一般格式：
![](http://upload-images.jianshu.io/upload_images/854027-7f37e7860f26b417.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通常来说一个HTTP请求报文由请求行、请求报头、空行、和请求数据 4 个部分组成。
###### 请求行
请求行由请求方法，URL字段和HTTP协议的版本组成，格式如下：
```
Method Request-URI HTTP-Version CRLF
```

<!-- more -->

其中 Method 表示请求方法；Request-URI 是一个统一资源标识符；HTTP-Version 表示请求的 HTTP 协议版本；CRLF 表示回车和换行（除了作为结尾的 CRLF 外，不允许出现单独的 CR 或 LF 字符）。
HTTP 请求方法有 8 种，分别是 GET、POST、DELETE、PUT、HEAD、TRACE、CONNECT 、OPTIONS。其中 PUT、DELETE、POST、GET 分别对应着增删改查，对于移动开发最常用的就是 POST 和 GET 了。
1. GET：请求获取Request-URI所标识的资源
2. POST：在Request-URI所标识的资源后附加新的数据
3. HEAD 请求获取由Request-URI所标识的资源的响应消息报头
4. PUT 请求服务器存储一个资源，并用Request-URI作为其标识
5. DELETE 请求服务器删除Request-URI所标识的资源
6. TRACE 请求服务器回送收到的请求信息，主要用于测试或诊断
7. CONNECT 保留将来使用
8. OPTIONS 请求查询服务器的性能，或者查询与资源相关的选项和需求

例如我去访问我的CSDN博客地址请求行是：
```
GET http://blog.csdn.net/itachi85 HTTP/1.1
```
###### 请求报头
在请求行之后会有 0 个或者多个请求报头，每个请求报头都包含一个名字和一个值，它们之间用“：”分割。请求头部会以一个空行，发送回车符和换行符，通知服务器以下不会有请求头。关于请求报头，会在后面的消息报头一节做统一的解释。
###### 请求数据
请求数据不在 GET 方法中使用，而是在 POST 方法中使用。POST 方法适用于需要客户填写表单的场合，与请求数据相关的最常用的请求头是 Content-Type 和 Content-Length。
##### HTTP 的响应报文
先来看看响应报文的一般格式：
![](http://upload-images.jianshu.io/upload_images/854027-fe54983f12f0e2d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
HTTP 的响应报文由状态行、消息报头、空行、响应正文组成。响应报头后面会讲到，响应正文是服务器返回的资源的内容，先来看看状态行。
###### 状态行
状态行格式如下：
```
HTTP-Version Status-Code Reason-Phrase CRLF
```
其中，HTTP-Version 表示服务器 HTTP 协议的版本；Status-Code 表示服务器发回的响应状态代码；Reason-Phrase 表示状态代码的文本描述。
状态代码有三位数字组成，第一个数字定义了响应的类别，且有五种可能取值：
- 100~199：指示信息，表示请求已接收，继续处理
- 200~299：请求成功，表示请求已被成功接收、理解、接受
- 300~399：重定向，要完成请求必须进行更进一步的操作
- 400~499：客户端错误，请求有语法错误或请求无法实现
- 500~599：服务器端错误，服务器未能实现合法的请求

常见的状态码如下：
- 200 OK：客户端请求成功
- 400 Bad Request：客户端请求有语法错误，不能被服务器所理解
- 401 Unauthorized：请求未经授权，这个状态代码必须和 WWW-Authenticate 报头域一起使用
- 403 Forbidden：服务器收到请求，但是拒绝提供服务
- 500 Internal Server Error：服务器发生不可预期的错误
- 503 Server Unavailable：服务器当前不能处理客户端的请求，一段时间后可能恢复正常

例如访问我的CSDN博客地址响应的状态行是：
```
HTTP/1.1 200 OK
```
##### HTTP 的消息报头
消息报头分为通用报头、请求报头、响应报头、实体报头等。消息头由键值对组成，每行一对，关键字和值用英文冒号“:”分隔。
###### 通用报头
既可以出现在请求报头，也可以出现在响应报头中
- Date：表示消息产生的日期和时间
- Connection：允许发送指定连接的选项，例如指定连接是连续的，或者指定 “close” 选项，通知服务器，在响应完成后，关闭连接
- Cache-Control：用于指定缓存指令，缓存指令是单向的（响应中出现的缓存指令在请求中未必会出现），且是独立的（一个消息的缓存指令不会影响另一个消息处理的缓存机制）

###### 请求报头
请求报头通知服务器关于客户端求求的信息，典型的请求头有：
- Host：请求的主机名，允许多个域名同处一个IP地址，即虚拟主机
- User-Agent：发送请求的浏览器类型、操作系统等信息
- Accept：客户端可识别的内容类型列表，用于指定客户端接收那些类型的信息
- Accept-Encoding：客户端可识别的数据编码
- Accept-Language：表示浏览器所支持的语言类型
- Connection：允许客户端和服务器指定与请求/响应连接有关的选项，例如这是为 Keep-Alive 则表示保持连接。
- Transfer-Encoding：告知接收端为了保证报文的可靠传输，对报文采用了什么编码方式。

###### 响应报头
用于服务器传递自身信息的响应，常见的响应报头：
- Location：用于重定向接受者到一个新的位置，常用在更换域名的时候
- Server：包含可服务器用来处理请求的系统信息，与 User-Agent 请求报头是相对应的

###### 实体报头
实体报头用来定于被传送资源的信息，既可以用于请求也可用于响应。请求和响应消息都可以传送一个实体，常见的实体报头为：
- Content-Type：发送给接收者的实体正文的媒体类型
- Content-Lenght：实体正文的长度
- Content-Language：描述资源所用的自然语言，没有设置则该选项则认为实体内容将提供给所有的语言阅读
- Content-Encoding：实体报头被用作媒体类型的修饰符，它的值指示了已经被应用到实体正文的附加内容的编码，因而要获得 Content-Type 报头域中所引用的媒体类型，必须采用相应的解码机制。
- Last-Modified：实体报头用于指示资源的最后修改日期和时间
- Expires：实体报头给出响应过期的日期和时间

##### 应用举例
要想查看网页或者手机请求网络的请求报文和响应报文有很多种方法，这里推荐采用Fiddler，在 [Android 利用 Fiddler 进行网络数据抓包](http://www.trinea.cn/android/android-network-sniffer/)这篇文章中详尽介绍了如何使用 Fiddler，在这里就不赘述了。打开 Fiddler，然后用浏览器访问我的 CSDN 博客网站：
![](http://upload-images.jianshu.io/upload_images/854027-3867cc029093944b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击红色画笔的区域就可以看到请求报文和响应报文了
请求报文：
```
GET http://blog.csdn.net/itachi85 HTTP/1.1                                //请求行
Host: blog.csdn.net                                                       //请求报头
Connection: keep-alive
Cache-Control: max-age=0       
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36 QQBrowser/9.3.6872.400
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
Cookie: bdshare_firstime=1443768140949; uuid_tt_dd=5028529250430960147_20151002;
...省略
```
很容易看出访问的是我的博客地址 <http://blog.csdn.net/itachi85>,请求的方法是 GET，因为是 GET 方法所以并没有请求数据。
响应报文：
```
HTTP/1.1 200 OK                                                         //状态行
Server: openresty                                                       //响应报头
Date: Sun, 27 Mar 2016 08:26:54 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Keep-Alive: timeout=20
Vary: Accept-Encoding
Cache-Control: private
X-Powered-By: PHP 5.4.28
Content-Encoding: gzip
                                                                        //不能省略的空格
28b5                                    
        }ysI   1ߡFsgl n- ]{^_ { 'z!     C ,  m# 0 !l   `  4x  ly .ݪ*  
  ڴzAt_Xl *  9'O  ɬ  '  ק   3  ^1a
...省略
```
响应报文没什么可说的，接下来我们配置好手机网络代理，访问一个应用的界面
请求报文：
```
POST http://patientapi.shoujikanbing.com/api/common/getVersion HTTP/1.1       //请求行
Content-Length: 226                                                          //请求报头
Content-Type: application/x-www-form-urlencoded
Host: patientapi.shoujikanbing.com
Connection: Keep-Alive
User-Agent: Mozilla/5.0 (Linux; U; Android 4.4.4; zh-cn; MI NOTE LTE Build/KTU84P) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1
Accept-Encoding: gzip
                                                             //不能省略的空格，下面是请求数据
clientversion=2_2.0.0&time=1459069342&appId=android&channel=hjwang&sessionId=0d1cee1f31926ffa8894c64804efa855101d56eb21caf5db5dcb9a4955b7fbc9&token=b191944d680145b5ed97f2f4ccf03058&deviceId=869436020220717&type=2&version=2.0.0
```
从请求报文的请求行来看，请求的方法是 POST，请求地址为 <http://patientapi.shoujikanbing.com/api/common/getVersion>，很显然是获取版本信息的接口。
响应报文：
```
HTTP/1.1 200 OK                                                              //状态行
Server: nginx                                                               //响应报头
Date: Sun, 27 Mar 2016 09:02:20 GMT
Content-Type: text/html;charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Vary: Accept-Encoding
Set-Cookie: sessionId=0d1cee1f31926ffa8894c64804efa855101d56eb21caf5db5dcb9a4955b7fbc9; expires=Mon, 28-Mar-2016 09:02:20 GMT; Max-Age=86400; path=/; domain=.shoujikanbing.com
Set-Cookie: PHPSESSID=0d1cee1f31926ffa8894c64804efa855101d56eb21caf5db5dcb9a4955b7fbc9; path=/; domain=.shoujikanbing.com
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Content-Encoding: gzip
                                                   //不能省略的空格
17f                                                //实体报文编码格式为gzip所以显示在这里的响应数据是乱码
       mP N @     "E ?    n m   1  
w ( HL (1^ P nK  E ѷ93'3gNLH  7P  $c \  T 4a6   L:+ 1dY%$g   h H   +
 
...省略
```
响应报文的实体采用的编码格式为为gzip，所以在Fiddler软件中显示的是乱码。