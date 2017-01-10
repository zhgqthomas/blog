---
title: Docker 快速搭建 Shadowsocks 服务
date: 2017-01-10 14:28:32
categories:
  - 翻墙
tags:
  - Docker
  - Shadowsocks
---
最近在搬瓦工买的 VPS 无奈被 GFW 给封了，所以只能另外购买一个 VPS，因为 [Linode](https://www.linode.com/?r=6ae95b8d05cd79e2be0eff9c89f18865e8acec52) 在搞 13 周年内存免费大升级，2GB RAM 单核的 VPS 只要每月 10 美刀。性价比，我感觉是所有 VPS 提供商中最高的，而且现在 Linode 还有东京的机房，所以新的 VPS 就选择了 Linode。 

开通了 Linode 在东京的 VPS 后，速度确实比在搬瓦工有明显的提升。买了国外的 VPS，第一件事情就是要搭建自己的翻墙服务，根据我之前写过得[《如何在 VPS 上搭建 VPN 来翻墙》](http://blog.shadark.com/2017/01/08/build-vpn-on-vps/),可以很容易的搭建起来，但是每次更换 VPS 或者给 VPS 重装系统都需要搭建一次，太过麻烦。所以这次想使用 Docker 来制作一个 [Shadowsocks docker 镜像](https://hub.docker.com/r/zhgqthomas/shadowsocks/)。这样以后每次更换 VPS 或重装系统后，只是下载镜像就可以了。虽然说，shadowsocks 只是个 socks5 的服务而已，使用 docker 大材小用了，但是其实也是为了实践 docker 玩玩而已，不喜误喷，谢谢~

<!-- more -->

#### Docker 的安装
关于 Docker 的具体介绍，这里就不多写了。有兴趣的，请查看[这里](https://yeasy.gitbooks.io/docker_practice/content/introduction/what.html)。
##### 系统要求
Docker 需要安装在 64 位的 x86 平台或 ARM 平台上（如[树莓派](https://www.raspberrypi.org/)），并且要求内核版本不低于 3.10。但实际上内核越新越好，过低的内核版本可能会出现部分功能无法使用，或者不稳定。

搬瓦工是一个基于 OpenVZ 的虚拟容器，暂且把它理解为 Docker 上的一个容器吧！直接搭建 Shadowsocks 服务也还行，但是想要在这么一个容器上再跑 Docker，就会因为无法升级Linux内核而无法启动 docker。因为OpenVZ 和 Docker 一样，容器与宿主机共用内核，因此容器作为寄生者肯定是没有升级内核的权限的，这也是我放弃搬瓦工的原因。

可以通过如下命令检查自己的内核版本详细信息
 ``` bash
$ uname -a
Linux device 4.4.0-45-generic #66~14.04.1-Ubuntu SMP Wed Oct 19 15:05:38 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```
##### 安装 (Debian 7 Wheezy)
因为我的系统使用的是 Debian 7 Wheezy，所以以下的命令都是基于 Debian  7 的。 其他系统请参考 [Ubuntu](https://docs.docker.com/engine/installation/linux/ubuntulinux/)，[Red Hat Enterprise Linux](https://docs.docker.com/engine/installation/linux/rhel/)，[CentOS](https://docs.docker.com/engine/installation/linux/centos/)，[Fedora](https://docs.docker.com/engine/installation/linux/fedora/)，[Debian](https://docs.docker.com/engine/installation/linux/debian/)，[Arch Linux](https://docs.docker.com/engine/installation/linux/archlinux/)，[CRUX Linux](https://docs.docker.com/engine/installation/linux/cruxlinux/)，[Gentoo](https://docs.docker.com/engine/installation/linux/gentoolinux/)，[Oracle Linux](https://docs.docker.com/engine/installation/linux/oracle/)

###### 升级内核
Debian 7 的内核默认为 3.2，为了满足 Docker 的需求，应该安装 `backports` 的内核。
执行下面的命令添加 `backports` 源
```bash
$ echo "deb http://http.debian.net/debian wheezy-backports main" | sudo tee /etc/apt/sources.list.d/backports.list
```
升级到 `backports` 内核
```bash
$ sudo apt-get update
$ sudo apt-get -t wheezy-backports install linux-image-amd64
```
##### 添加 apt 镜像源
我们需要使用 Docker 官方提供的软件源，因此，我们需要添加 APT 软件源。由于官方源使用 HTTPS 以确保软件下载过程中不被篡改。因此，我们首先需要添加使用 HTTPS 传输的软件包以及 CA 证书。
```bash
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates
```
为了确认所下载软件包的合法性，需要添加 Docker 官方软件源的 GPG 密钥
```bash
$ sudo apt-key adv \ --keyserver hkp://ha.pool.sks-keyservers.net:80 \ --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```
然后，我们需要向 source.list
 中添加 Docker 软件源，下表列出了不同的 Ubuntu 和 Debian 版本对应的 APT 源。

|     操作系统版本      |                            REPO                              |
|---------------------|--------------------------------------------------------------|
| Precise 12.04 (LTS) | `deb https://apt.dockerproject.org/repo ubuntu-precise main` |
| Trusty 14.04 (LTS)  | `deb https://apt.dockerproject.org/repo ubuntu-trusty main`  |
| Xenial 16.04 (LTS)  | `deb https://apt.dockerproject.org/repo ubuntu-xenial main`  |
|   Debian 7 Wheezy   | `deb https://apt.dockerproject.org/repo debian-wheezy main`  |
|   Debian 8 Jessie   | `deb https://apt.dockerproject.org/repo debian-jessie main`  |
| Debian Stretch/Sid  | `deb https://apt.dockerproject.org/repo debian-stretch main` |
用下面的命令将 APT 源添加到 `source.list`（将其中的 `<REPO>` 替换为上表的值）
```bash
$ echo "<REPO>" | sudo tee /etc/apt/sources.list.d/docker.list
```
添加成功后，更新 apt 软件包缓存
```bash
$ sudo apt-get update
```
##### 安装
在一切准备就绪后，就可以安装最新版本的 Docker 了，软件包名称为 `docker-engine`。
```bash
$ sudo apt-get install docker-engine
```
如果系统中存在旧版本的 Docker （`lxc-docker`, `docker.io`），会提示是否先删除，选择是即可。
##### 启动 Docker 引擎
```bash
$ sudo service docker start
```
##### 建立 docker 用户组
默认情况下，`docker` 命令会使用 [Unix socket](https://en.wikipedia.org/wiki/Unix_domain_socket) 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。
建立 `docker` 组：
```bash
$ sudo groupadd docker
```
将当前用户加入 `docker` 组：
```bash
$ sudo usermod -aG docker $USER
```
##### 镜像加速器
国内访问 Docker Hub 有时会遇到困难，此时可以配置镜像加速器。国内很多云服务商都提供了加速器服务，例如：
- [阿里云加速器](https://cr.console.aliyun.com/#/accelerator)
- [DaoCloud 加速器](https://www.daocloud.io/mirror#accelerator-doc)
- [灵雀云加速器](http://docs.alauda.cn/feature/accelerator.html)

注册用户并且申请加速器，会获得如 `https://jxus37ad.mirror.aliyuncs.com` 这样的地址。我们需要将其配置给 Docker 引擎。

对于使用 [upstart](http://upstart.ubuntu.com/) 的系统而言，编辑 `/etc/default/docker` 文件，在其中的 `DOCKER_OPTS` 中添加获得的加速器配置 `--registry-mirror=<加速器地址>`，如：
```bash
DOCKER_OPTS="--registry-mirror=https://jxus37ad.mirror.aliyuncs.com"
```
重新启动服务。
```bash
$ sudo service docker restart
```
Linux系统下配置完加速器需要检查是否生效，在命令行执行 `ps -ef | grep dockerd`，如果从结果中看到了配置的 `--registry-mirror` 参数说明配置成功。
```bash
$ sudo ps -ef | grep dockerd
root      5346     1  0 19:03 ?        00:00:00 /usr/bin/dockerd --registry-mirror=https://jxus37ad.mirror.aliyuncs.com
$
```
#### 使用 [docker-shadowoscks](https://hub.docker.com/r/zhgqthomas/shadowsocks/) 镜像
##### Docker 获取镜像命令
[Docker Hub](https://hub.docker.com/explore/) 上有大量的高质量的镜像可以用，这里我们就说一下怎么获取这些镜像并运行。
从 Docker Registry 获取镜像的命令是 `docker pull`。其命令格式为：
```bash
docker pull [选项] [Docker Registry地址]<仓库名>:<标签>
```
具体的选项可以通过 `docker pull --help` 命令看到，这里我们说一下镜像名称的格式。
* Docker Registry地址：地址的格式一般是 `<域名/IP>[:端口号]`。默认地址是 Docker Hub。
* 仓库名：如之前所说，这里的仓库名是两段式名称，既 `<用户名>/<软件名>`。对于 Docker Hub，如果不给出用户名，则默认为 `library`，也就是官方镜像。

比如：
```bash
$ docker pull ubuntu:14.04
14.04: Pulling from library/ubuntu
bf5d46315322: Pull complete
9f13e0ac480c: Pull complete
e8988b5b3097: Pull complete
40af181810e7: Pull complete
e6f7c7e5c03e: Pull complete
Digest: sha256:147913621d9cdea08853f6ba9116c2e27a3ceffecf3b492983ae97c3d643fbbe
Status: Downloaded newer image for ubuntu:14.04
```
上面的命令中没有给出 Docker Registry 地址，因此将会从 Docker Hub 获取镜像。而镜像名称是 `ubuntu:14.04`，因此将会获取官方镜像 `library/ubuntu` 仓库中标签为 `14.04` 的镜像。
##### 获取 shadowsocks 镜像
熟悉 `docker pull` 命令之后，获取 shadowsocks 的镜像就很容易了
```bash
$ docker pull zhgqthomas/shadowsocks
```
然后就可以把我制作的 docker shadowsocks 镜像下载到本地
##### 启动镜像
镜像下载完成后，剩下的步骤就容易很多，只需要使用 Docker 启动命令`docker run` ，将 shadowsocks 服务启动起来就 okay 了
```bash
$ sudo docker run --name shadowsocks -d -p 2998:2998 zhgqthomas/shadowsocks -s 0.0.0.0 -p 2998  -k $password -m aes-256-cfb
```
将其中的`$password` 替换为自己想使用的密码即可，然后 shadowsocks 的服务就启动起来了，使用多平台的 [shadowsocks 客户端](https://shadowsocks.org/en/download/clients.html)即可访问 Google 等被墙的网站了。