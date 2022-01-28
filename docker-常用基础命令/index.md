# Docker-常用基础命令


- **docker 服务重启，关闭，启动及版本查看**

 ```shell
  [root@localhost /] systemctl restart docker.service  #重启服务
  [root@localhost /] systemctl stop docker.service     #关闭服务
  [root@localhost /] systemctl start docker.service    #启动服务
  [root@iZm5e3hwzuo58e05kxjiifZ /] docker -v #docker版本查看
  Docker version 18.06.1-ce, build e68fc7a
 ```

- **docker 搜索/下载/查看镜像**
<!--more-->
 ```shell
  [root@localhost /] docker search centos|head -3 #搜索镜像
  NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
  centos                             The official build of CentOS.                   4754                [OK]                
  [root@localhost /] docker pull centos #下载镜像
  Using default tag: latest.............
  Digest: sha256:6f6d986d425aeabdc3a02cb61c02abb2e78e57357e92417d6d58332856024faf
  Status: Downloaded newer image for centos:latest
  [root@localhost /] docker images #查看镜像
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              latest              5182e96772bf        7 weeks ago         200MB
 ```

- **docker 创建一个容器（前台/后台并指定映射目录和端口）**

    ```shell
    -i：允许我们对容器内的 (STDIN) 进行交互
  -t：在新容器内指定一个伪终端或终端
  -v：是挂在宿机目录， /docker_test是宿机目录，/yufei是当前docker容器的目录，宿机目录必须是绝对的。
  --name：是给容器起一个名字，可省略，省略的话docker会随机产生一个名字
  -P 指定映射的端口
  --net #指定网络
  --link 链接到另一个容器
  --------------------------------------------------------------------------------------------------
  docker run -it -v /test:/test  --name centos /bin/bash #创建容器并进入（交互模式退出会后容器会自动关闭）
  docker run -d -v /test:/test centos tail -f /dev/null #创建容器并放入后台运行（退出容器不会关闭）
  docker run -dit -v /test:/test centos /bin/bash #创建容器并放入后台运行（进入后台和tty模式，退出容器不会关闭）
  docker run -d -v /test:/test -P 80:80 nginx:latest
  #后台启动并运名为nginx的容器，然后将容器的80端口映射到物理机的80端口.
    ```

-  **查看docker创建的所有容器**

    ```shell
  [root@iZm5e3hwzuo58e05kxjiifZ rc.d] docker ps -a #查看所有创建的容器包括已经停止的容器
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
  70e151cd2766        centos              "/bin/bash"         7 seconds ago       Exited (0) 5 seconds ago                       zealous_mclean
  dfdf33852d47        centos              "/bin/bash"         20 seconds ago      Up 19 seconds                                  frosty_saha
  [root@iZm5e3hwzuo58e05kxjiifZ rc.d] docker ps #查看所有运行的容器
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  dfdf33852d47        centos              "/bin/bash"         22 seconds ago      Up 21 seconds                           frosty_saha
  [root@iZm5e3hwzuo58e05kxjiifZ rc.d] docker  ps -l #查看最新创建的容器
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
  70e151cd2766        centos              "/bin/bash"         29 minutes ago      Exited (0) 29 minutes ago                       zealous_mclean
    ```

- **docker 利用已存在的容器创建一个镜像（Dockerfile构建镜像略）**

    ```shell
  -a #提交的镜像作者
  -c #使用Dockerfile指令来创建镜像
  -m #提交时附上说明文字
  -p #在commit时，将容器暂停
  -------------------------------------------------------------------------------------------
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker commit -a "王云龙" -m "创建的新镜像" redis wyl5588redis-test
  sha256:9c2d2fc6e09cb35543fbb2467db90e741dc6b7daabab83924534bcfe6641bbe2
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  wyl5588redis-test   latest              9c2d2fc6e09c        3 seconds ago       83.4MB
    ```

- **docker 修改镜像标签，并推送**

    ```shell
  [root@iZm5e3hwzuo58e05kxjiifZ Dockerfile] docker tag centos 192.168.8.88:5000/centos:v1.0 #给centos镜像打一个行的tag
  [root@iZm5e3hwzuo58e05kxjiifZ Dockerfile] docker images
  REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
  192.168.8.88:5000/centos   v1.0                5182e96772bf        7 weeks ago         200MB
  centos                     latest              5182e96772bf        7 weeks ago         200MB
  [root@iZm5e3hwzuo58e05kxjiifZ Dockerfile]docker push 192.168.8.88:5000/centos:v1.0 #将本地docker中的镜像推送到镜像仓库中
    ```

   

- **docker镜像的导入，导出，删除**

    ```shell
  docker save 5588/mongo3.2 5588/redis 5588/nginx 5588/qiantai >Qiantai_images.tar  #镜像导出
  docker load </Docker_Images/Qiantai.images.tar #镜像导入
  docker rmi centos #删除centos镜像
  docker rmi -f centos #强制删除
  docker images -q #获取进行的ID
  docker rmi -f $(docker images -q)#删除全部镜像 
    ```

   

- **docker 容器与宿主机文件拷贝**

    ```shell
  [root@iZm5e3hwzuo58e05kxjiifZ ~] touch admin
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker cp ./admin nginx:/tmp/
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker exec -it nginx /bin/bash
  root@aaefa2aebc8b:/ ls /tmp
  admin
  root@aaefa2aebc8b:/ touch /tmp/wyltest
  exit
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker cp -a nginx:/tmp/wyltest ./
  Error: No such container:path: nginx:/tmp/*
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker cp -a nginx:/tmp/admin ./
    ```

   

-  **查看docker 容器详情如：ip等**

    ```shell
  [root@iZm5e3hwzuo58e05kxjiifZ rc.d] docker inspect frosty_saha #查看容器详情如ip等frosty_saha为容器别名
  [
      {
          "Id": "dfdf33852d470d0cd8e70a4b9aad36a00585579952834471159100aacea885d9",
          "Created": "2018-09-28T04:44:11.394993867Z",
          "Path": "/bin/bash",
          "Args": [],
          "State": {
              "Status": "running",....
  )]
    ```

   

- **docker 关闭，启动，重启，删除容器**

    ```shell
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker start frosty_saha #启动容器
  frosty_saha
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker stop frosty_saha #关闭容器
  frosty_saha
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker restart frosty_saha #重启容器
  frosty_saha
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  dfdf33852d47        centos              "/bin/bash"         41 minutes ago      Up 13 seconds                           frosty_saha
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker rm -f frosty_saha #强制删除容器
  frosty_saha
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker rm -f $(docker ps -qa) #强制删除所有容器
  70e151cd2766
    ```

   

- **docker  进入某一容器**

    ```shell
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker ps 
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
  aaefa2aebc8b        5588/nginx          "nginx -g 'daemon of…"   51 seconds ago      Up 51 seconds       0.0.0.0:80->80/tcp        nginx
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker exec -it nginx /bin/bash #进入某一容器
  root@aaefa2aebc8b:/#
    ```
    
   

- **docker 容器外创建一个后台任务**

    ```shell
  [root@izm5e3hwzuo58e05kxjiifz ~] docker exec -d qiantai  python /mnt/log/tbxScripts.py
    ```

-  **docker 查看某一容器的 进程，日志，端口**

    ```shell
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker top nginx #查看nginx容器进程
  UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
  root                2018                1984                0                   13:35               ?                   00:00:00            nginx: master process nginx -g daemon off;
  101                 2094                2018                0                   13:35               ?                   00:00:00            nginx: worker process
  101                 2095                2018                0                   13:35               ?                   00:00:00            nginx: worker process
  101                 2096                2018                0                   13:35               ?                   00:00:00            nginx: worker process
  101                 2097                2018                0                   13:35               ?                   00:00:00            nginx: worker process
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker logs -tf nginx #查看容器日志
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker port nginx #查看容器映射的端口
  80/tcp -> 0.0.0.0:80
    ```

- **docker 容器监控**

    ```shell
  [root@iZm5e3hwzuo58e05kxjiifZ ~] docker stats #容器监控
  CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
  aaefa2aebc8b        nginx               0.00%               3.145MiB / 15.51GiB   0.02%               17.7kB / 233kB      0B / 0B             5
  da5fc7ab052c        qiantai             0.78%               1.909GiB / 15.51GiB   12.31%              327MB / 367kB       0B / 8.19kB         11
  f4ce4818308c        mongodb3.2          0.03%               616MiB / 15.51GiB     3.88%               364kB / 327MB       0B / 1.05MB         24
  eb841f913814        redis               0.00%               6.312MiB / 15.51GiB   0.04%               2.78kB / 1.
    ```

