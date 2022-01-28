# Linux常用命令

## 系统

- **uname -a****

  > 查看内核/操作系统/CPU信息

- **head -n 1 /etc/issue**

  > 查看操作系统版本

- ***cat /proc/cpuinfo**

  > 查看`CPU`信息

- **hostname****

  > 查看计算机名
<!--more-->
- **lspci -tv**

  > 列出所有`PCI`设备

- **lsusb -tv**

  > 出所有USB设备

- **lsmod**

  > 列出加载的内核模块

- **env**

  > 查看环境变量

## 资源

- **free -m**

  > 查看内存使用量和交换区使用量

- **df -h**

  > 查看各分区使用情况

- **du -sh <目录名>**

  > 查看指定目录的大小

- **grep MemTotal /proc/meminfo**

  > 查看内存总量

- **grep MemFree /proc/meminfo**

  > 查看空闲内存量

- **uptime**

  > 查看系统运行时间、用户数、负载

- **cat /proc/loadavg**

  > 查看系统负载

## 磁盘和分区

- **mount | column -t**

  > 查看挂接的分区状态

- **fdisk -l**

  > 查看所有分区

- **swapon -s**

  > 查看所有交换分区

- **hdparm -i /dev/hda**

  > 查看磁盘参数(仅适用于IDE设备)

- **dmesg | grep IDE**

  > 查看启动时IDE设备检测状况

## 网络

- **ifconfig**

  > 查看所有网络接口的属性

- **iptables -L**

  > 查看防火墙设置

- **route -n**

  > 查看路由表

- **netstat -lntp**

  > 查看所有监听端口

- **netstat -antp**

  > 查看所有已经建立的连接

- **netstat -s**

  > 查看网络统计信息

## 进程

- **ps -ef**

  > 查看所有进程

- **top**

  > 实时显示进程状态

## 用户

- **w**

  > 查看活动用户

- **id <用户名>**

  > 查看指定用户信息

- **last**

  > 查看用户登录日志

- **cut -d: -f1 /etc/passwd**

  > 查看系统所有用户

- **cut -d: -f1 /etc/group**

  > 查看系统所有组

- **crontab -l**

  > 查看当前用户的计划任务

## 服务

- **chkconfig --list**

  > 列出所有系统服务

- **chkconfig --list | grep on**

  > 列出所有启动的系统服务

## 程序

- **apt-get update**

  > 在修改`/etc/apt/sources.list`或者`/etc/apt/preferences`之后运行该命令。此外您需要定期运行这一命令以确保您的软件包列表是最新的。

- **apt-get install packagename**

    > 安装一个新软件包（参见下文的aptitude）

- **apt-get remove packagename**

    > 卸载一个已安装的软件包（保留配置文件）

- **apt-get --purge remove packagename**

    > 卸载一个已安装的软件包（删除配置文件）

- **dpkg --force-all --purge packagename**

    > 有些软件很难卸载，而且还阻止了别的软件的应用，就可以用这个，不过有点冒险。

- **apt-get autoclean apt**

    > 会把已装或已卸的软件都备份在硬盘上，所以如果需要空间的话，可以让这个命令来删除你已经删掉的软件

- **apt-get clean**

    > 这个命令会把安装的软件的备份也删除，不过这样不会影响软件的使用的。

- **apt-get upgrade**

    > 更新所有已安装的软件包

- **apt-get dist-upgrade**

    > 将系统升级到新版本

- **apt-cache search string**

    >  在软件包列表中搜索字符串

- dpkg -l package-name-pattern

    > 列出所有与模式相匹配的软件包。如果您不知道软件包的全名，您可以使用“**`package-name-pattern`**”。

- **aptitude**

    > 详细查看已安装或可用的软件包。与`apt-get`类似，`aptitude`可以通过命令行方式调用，但仅限于某些命令——最常见的有安装和卸载命令。由于`aptitude比apt-get`了解更多信息，可以说它更适合用来进行安装和卸载。

- **apt-cache showpkg pkgs**

    > 显示软件包信息。

- **apt-cache dumpavail**

    > 打印可用软件包列表。

- **apt-cache show pkgs**

    > 显示软件包记录，类似于`dpkg –print-avail`。

- **apt-cache pkgnames**

    > 打印软件包列表中所有软件包的名称。

- **dpkg -S file**

    > 这个文件属于哪个已安装软件包。

- **dpkg -L package**

    > 列出软件包中的所有文件。

- **apt-file search filename**

  > 查找包含特定文件的软件包（不一定是已安装的），这些文件的文件名中含有指定的字符串。

- **apt-file**

  > 是一个独立的软件包。您必须 先使用apt-get install来安装它，然后运行`apt-file update`。如果`apt-file search`

- **filename**

  > 输出的内容太多，您可以尝试使用`apt-file search`

- **sfilename | grep -w filename**

  > `s`（只显示指定字符串作为完整的单词出现在其中的那些文件名）或者类似方法，例如：`apt-file search filename | grep /bin/`（只显示位于诸如/bin或/usr/bin这些文件夹中的文件，如果您要查找的是某个特定的执行文件的话，这样做是有帮助的）。


