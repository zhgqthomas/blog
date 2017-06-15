---
title: Ubuntu 14.04 编译 Android 5.1 源码
date: 2016-08-17 18:34:45
categories:
  - Android
tags: 
	- Android
---

最近因为工作的需要，对 Android 5.1 的源码进行了下载及编译，希望此文能给同是 Android 开发的同胞们一点帮助。
##### Ubuntu 版本
推荐使用 Ubuntu 14.04 这个版本，最开始是用 Ubuntu 16.04， 但是在编译源码的时候会因为很多系统库的问题出现 bug，故又降回到了 14.04 这个版本。
##### 更新 Ubuntu 的源
因为很多大家已知的原因，如果用原生的 Ubuntu 的源在用 apt-get 下载软件的时候速度会很慢，所以一般情况下会使用国内的一些源来进行替代，我是使用清华大学的 [Ubuntu 源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)，亲测速度很快。通过以下的命令来修改 Ubuntu 的源文件。
```bash
$ sudo vim /etc/apt/source.list
```
将清华大学的源替换到 source.list 文件里，然后运行以下命令即可
```bash
$ sudo apt-get update
```

<!-- more -->

##### 下载 repo
repo是一种代码版本管理工具，它是由一系列的 Python 脚本组成，封装了一系列的 git 命令，用来统一管理多个 git 仓库。因为 Android 源码引用了很多开源项目，每一个子项目都是一个 git 仓库，每个 git 仓库都有很多分支版本，为了方便统一管理各个子项目的 git 仓库，需要一个上层工具批量进行处理，因此 repo 诞生。
```bash
$ mkdir ~/bin
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```
通过以上的三条命令，就把 repo 下载到了 ~/bin 的文件夹下。(第二个命令在执行时可能会因为 GFW 的问题无法下载，可以通过使用 proxychains 来实现翻墙，具体就看该[文章](http://www.jianshu.com/p/2f51144c35c9)的更新部分)
##### 建立工作目录
```bash
$ mkdir android-source
$ cd android-source
```
初始化仓库
```bash
$ repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest
```
如果提示无法连接到 gerrit.googlesource.com，可以编辑 ~/bin/repo，把 REPO_URL 一行替换成下面的：
```xml
REPO_URL = 'https://gerrit-google.tuna.tsinghua.edu.cn/git-repo'
```
如果需要某个特定的 Android 版本([列表](https://source.android.com/source/build-numbers.html#source-code-tags-and-builds))：
```bash
$ repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1
```
同步源码树(以后只需执行这条命令来同步):
```bash
$ repo sync
```
P.S. 大家也可以直接下载该[脚本](https://gist.github.com/zhgqthomas/ed860abe3dc8c8384447aed95011a35e), 通过该脚本可以在下载中断的时候自动执行 repo sync 命令。
##### 环境搭建
与开发安卓 app 不同的是编译 android 源码需要用 openjdk ，推荐使用 openjdk7 这个版本
```bash
$ sudo apt-get install openjdk-7-jdk
```
输入密码后系统就去安装了，然后就是配置jdk的环境变量
执行
```bash
$ sudo vim /etc/profile
```
在文件的末尾部分添加
```bash
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/
export PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
```
保存之后，最后运行以下命令使其生效
```xml
$ source /etc/profile
```
安装完 openjdk 后，就需要下载编译所依赖的一些包
```bash
$ sudo apt-get install git-core gnupg flex bison gperf build-essential \
zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
libgl1-mesa-dev libxml2-utils xsltproc unzip
```
#####搭建 ccache (可选)
你可以自由选择是否使用 ccache 编译工具，该工具对 C 和 C++ 进行缓存来达到加快编译的效果，如果你会经常使用到 make clean 或会经常生成不同架构的 ROM，建议开启 ccache
开启 ccache 的方法很简单，在源码根目录下输入以下命令
```bash
$ export USE_CCACHE=1
$ export CCACHE_DIR=/<path_of_your_choice>/.ccache
$ prebuilts/misc/linux-x86/ccache/ccache -M 50G
```
##### 编译
环境搭建完成后，就可以进入编译环节了，但在编译之前还需要在 Android 源代码根目录中执行如下的命令设置一些 Shell 函数
```bash
$ source build/envsetup.sh
```
在编译 Android 源代码之前还需要做最后一件事情，就是设置编译的目标，也就是为哪些设备编译 Android 源代码，可以通过以下命令来查看当前 Android 支持的所有目标
```bash
$ lunch
```
在输出的列表中，直接输入前面的序号即可选择相应的目标
最后可以在 Android 源代码的根目录直接执行 make 命令编译整个 Android 源代码，如果读者的机器是多核 CPU，可以指定编译时利用的 CPU 核数。例如，执行下面代码会利用 4 个 CPU 核。
```bash
$ make -j4
```
编译完  Android 源代码后，会在<Android 源代码目录>生成一个 out 目录，所有编译生成的目标文件都在该目录的相应子目录中。其中最重要的有 3 个镜像文件， ramdisk.img、system.img 和 userdata.img.
Google 公司已经在官方网站上发布了最新的编译 Android 源代码的方式，读者可以[点击这里](http://source.android.com/source/initializing.html),但需要科学上网，科学上网方法请[点击这里](http://www.jianshu.com/p/2f51144c35c9).
请转载时,标明文章[出处](http://www.jianshu.com/p/a499d43fd3c0).谢谢