# 在Ubuntu16.04和Centos7上启用TCP-BBR


> 1. 新增atrandys大佬的一键安装脚本，支持`centos7+`/`debian9+`/`ubuntu16+`:
> ```shell
> wget --no-check-certificate https://raw.githubusercontent.com/cx9208/Linux-NetSpeed/master/tcp.sh
> chmod +x tcp.sh
> ./tcp.sh
> ```
> 2. 新增Google原版BBR一键安装脚本：
> ```shell
> wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
> chmod +x bbr.sh
> ./bbr.sh
> ```

## BBR简介
`BBR` 是 Google 推出的一个「TCP 拥塞控制算法」，它是以 Linux 内核模块的形式加载，可以最大化 Linux Server 的网络吞吐量。

简单地说，开启 `BBR` 的 Linux Server 和不开启 `BBR` 的 Linux Server，在持续传输数据方面可以有非常大的不同。

`BBR` 尽管还没有在主流发行版中默认开启，但 Google 已经在 YouTube 网站上实践了很久，可以说是很成熟的一样技术了。

## 检测 BBR 是否开启

在开始之前，先看看 `BBR` 是否已经启用了，执行这条指令可以返回当前 Linux 内核可以使用的 TCP 拥堵控制算法：

<!--more-->
```shell
sysctl net.ipv4.tcp_available_congestion_control
```

例如，在我的Server上返回了如下内容：

```shell
net.ipv4.tcp_available_congestion_control = cubic reno
```

可以看到是没有BBR的，因为默认的 Ubuntu 16.04 用的是 `Linux 4.4.0` 内核，所以自然是看不到 `BBR` 的。

我们再次确认下系统当前启用的拥塞算法：

```shell
sysctl net.ipv4.tcp_congestion_control
```

返回的内容是：

```shell
net.ipv4.tcp_congestion_control = cubic
```

可以看到系统使用的是 `cubic` 这个默认的算法。接下去我们通过最标准的模式来为这台 Ubuntu 16.04 启用 `BBR`

## 为Ubuntu 16.04 安装/启用 BBR
### 安装 4.10+ 新内核

`BBR` 只能配合 Linux Kernel 4.10 以上内核才能使用。但是在 Ubuntu 16.04 上怎么使用 4.10 呢？难道要手动下载和安装吗？

不能！这会有一个安全隐患，手动下载安装的新内核，无法保证后续能得到及时的安全更新。那么怎么办？这里推荐使用 `HWE` 版本的内核，它就在官方源里。

HWE，即：`HareWare Enablement`，是专门为在老的系统上支持新的硬件而推出的内核。你可以像安装其他软件包一样在 Ubuntu 16.04 里非常容易的安装它，只需要执行下面的命令：

```shell
sudo apt-get install linux-generic-hwe-16.04
```

对！只需要这样就OK了！

安装好以后**重启系统**，然后输入：

```shell
uname -a
```

我的Server输出如下：

```shell
Linux oneone 4.15.0-74-generic #83~16.04.1-Ubuntu SMP Wed Dec 18 04:56:23 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

可以看到系统内核已经升级到`4.15.0`了。

### 启用 BBR

接下来就可以为新内核装载 BBR 模块了，分别执行：

```shell
sudo modprobe tcp_bbr

echo "tcp_bbr" | sudo tee -a /etc/modules-load.d/modules.conf
```

接下来我们再来查看系统支持的拥塞算法，可以看到`BBR`已经在里面了：

```shell
root@oneone:~# sysctl net.ipv4.tcp_available_congestion_control

net.ipv4.tcp_available_congestion_control = reno cubic bbr
```

接下来就正式启用BBR，把它设为系统的默认拥塞算法，分别执行：

```shell
echo "net.core.default_qdisc=fq" | sudo tee -a /etc/sysctl.conf

echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee -a /etc/sysctl.conf

sudo sysctl -p
```

最后，再来验证一下是否设置成功，可以看到`BBR`已经是默认算法：

```shell
root@oneone:~# sysctl net.ipv4.tcp_congestion_control

net.ipv4.tcp_congestion_control = bbr
```

## 为Centos7 安装/启用 BBR

### 安装 4.10+ 新内核

先查看系统版本：
```shell
cat /etc/redhat-release
#例如我的系统版本是7.6，输出如下:
CentOS Linux release 7.6.1810 (Core)
```

添加elrepo源，然后升级内核，操作命令如下：

```shell
#添加、更新源
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

#安装内核
yum --enablerepo=elrepo-kernel install kernel-ml -y
```

安装完成后，可以使用下方的命令查看系统已经安装了内核

```shell
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg

#例如我的系统查询结果如下，其中‘5.5.8’版本的内核是刚刚安装的：
0 : CentOS Linux (5.5.8-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (4.14.129-bbrplus) 7 (Core)
2 : CentOS Linux (0-rescue-05cb8c7b39fe0f70e3ce97e5beab809d) 7 (Core)
```

接着我们就把刚安装的`CentOS Linux (5.5.8-1.el7.elrepo.x86_64) 7 (Core)`内核设置为默认：

```shell
grub2-set-default 0
```

设置完默认内核之后，使用`uname -a`查看时发现当前使用的内核还是之前的版本，这是因为切换内核后需要重启系统才能生效：

```shell
#查看还是之前的版本：
[root~]# uname -a
Linux ip-172-31-21-55.ap-southeast-1.compute.internal 4.14.129-bbrplus #1 SMP Tue Jun 25 12:23:41 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

#重启系统即可生效
[root@~]# reboot
```

### 启用 BBR

编辑`sysctl.conf`文件，并添加如下内容：

```shell
# 使用vim编辑文件
vim /etc/sysctl.conf

#在文件中添加如下内容并保存：
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

加载系统参数，并确认配置是否生效，如果生效会打印上方刚刚新增的内容：

```shell
sysctl -p 
```

最后，再次验证一下是否设置成功：

```shell
[root@~]# sysctl net.ipv4.tcp_congestion_control

#输出如下，则表示默认加速算法已经是bbr
net.ipv4.tcp_congestion_control = bbr
```

