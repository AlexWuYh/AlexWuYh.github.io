# 鲲鹏920服务器上安装arm版本的docker


因为鲲鹏920处理是ARM架构的，所以一般的x86架构的包是不能安装的，该文档主要基于arm版本的centos7系统来编写(其他arm系统也可参考本文档)。


> 另附上一个华为官方的arm版本centos软件源：
> `https://repo.huaweicloud.com/centos-altarch/7/os/aarch64/Packages/`


## 1、下载docker官方提供的aarch64的二进制软件包

可使用PC输入链接下载后再拷贝到服务器上(针对服务器不能连外网的情况)，也可以直接在服务器上使用`wget`命令进行下载


```bash
# 此处提供的19.03.6版本的docker，如果需要其他版本直接修改版本号即可

wget -c https://download.docker.com/linux/static/stable/aarch64/docker-19.03.6.tgz
```

## 2、解压包并拷贝文件到`/usr/bin`目录

先解压下载的docker二进制包：

```bash
tar -xvf docker-19.03.06.tgz
```

然后拷贝`./docker/`目录下的所有文件到`/usr/bin/`:

```bash
mv docker/* /usr/bin/
```

## 3、配置system服务以及开机自启

### i、新增systemd的配置文件

把docker服务加入到systemd中来维护，需要在`/lib/systemd/system/`目录下新增`docker.service`和`docker.socket`文件，该文件的内容可从docker的官方github对应分支的`contrib/init/systemd/`目录下获取，可以直接拷贝文件内容使用。


- 对应链接如下：

    https://github.com/docker/engine/tree/19.03/contrib/init/systemd


    >版本需与docker版本一致，上面的链接适配的是docker 19.03版本，如需其他版本修改链接中的版本号即可



- docker.service

```bash
# /lib/systemd/system/docker.service
# 也可直接复制以下内容
# 注意该内容只适配docker19.03版本，其他版本使用可能会导致docker不能正常启动

[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target docker.socket firewalld.service
Wants=network-online.target
Requires=docker.socket

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd://
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=1048576
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target


```

- docker.socket

```bash
# /lib/systemd/system/docker.socket
# 也可直接复制以下内容
# 注意该内容只适配docker19.03版本，其他版本使用可能会导致docker不能正常启动

[Unit]
Description=Docker Socket for the API
PartOf=docker.service

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target

```

### ii、新建docker用户及组

分别执行如下命令：

```bash
#创建docker组
groupadd docker

#创建docker用户
useradd docker -g docker
```

### iii、启动docker服务

分别执行如下命令：

```bash
#重新加载systemd配置
systemctl daemon-reload

#启动docker服务
systemctl start docker
```

启动完成后执行`docker version`命令，如果能正确输出版本号则安装成功

### iv、设置docker服务为开机自启

执行如下命令：

```bash
systemctl enable docker
```
