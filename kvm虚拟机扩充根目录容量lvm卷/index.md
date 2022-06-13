# kvm虚拟机扩充根目录容量(lvm卷)


## 给虚拟机添加硬盘(或扩容)

1. 停止虚拟机
2. 在管理页面给虚拟机新增一块硬盘
3. 重新启动虚拟机

## 登入虚拟机，对新增硬盘进行分区

下方所示为硬盘扩容非新增，但操作方式都一致

```bash
root@zentao:/home/ubuntu# fdisk /dev/sda

Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

The size of this disk is 4 TiB (4398046511104 bytes). DOS partition table format can not be used on drives for volumes larger than 2199023255040 bytes for 512-byte sectors. Use GUID partition table format (GPT).


Command (m for help): p  ###查看已有分区

Disk /dev/sda: 4 TiB, 4398046511104 bytes, 8589934592 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x039343bf

Device     Boot   Start       End   Sectors   Size Id Type
/dev/sda1  *       2048   1499135   1497088   731M 83 Linux
/dev/sda2       1501182 419428351 417927170 199.3G  5 Extended
/dev/sda5       1501184 419428351 417927168 199.3G 8e Linux LVM

Command (m for help): F   ###查看可分配空间
Unpartitioned space /dev/sda: 3.8 TiB, 4183299194880 bytes, 8170506240 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes

    Start        End    Sectors  Size
419428352 8589934591 8170506240  3.8T

Command (m for help): n   ###创建分区
Partition type
   p   primary (1 primary, 1 extended, 2 free)
   l   logical (numbered from 5)
Select (default p): p        
Partition number (3,4, default 3): 
First sector (1499136-4294967295, default 1499136): 419428352
Last sector, +sectors or +size{K,M,G,T,P} (419428352-4294967294, default 4294967294):            

Created a new partition 3 of type 'Linux' and of size 1.8 TiB.

Command (m for help): p
Disk /dev/sda: 4 TiB, 4398046511104 bytes, 8589934592 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x039343bf

Device     Boot     Start        End    Sectors   Size Id Type
/dev/sda1  *         2048    1499135    1497088   731M 83 Linux
/dev/sda2         1501182  419428351  417927170 199.3G  5 Extended
/dev/sda3       419428352 4294967294 3875538943   1.8T 83 Linux
/dev/sda5         1501184  419428351  417927168 199.3G 8e Linux LVM

Partition table entries are not in disk order.

Command (m for help): t   ###修改分区type
Partition number (1-3,5, default 5): 3    ###指定要修改的分区
Partition type (type L to list all types): 8e   ###8e对应的type为Linux LVM

Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): w   ###写入分区表
The partition table has been altered
```

## 扩容根目录

1. 查看VG Name
```bash
root@zentao:/home/ubuntu# vgdisplay 
  --- Volume group ---
  VG Name               zentao-vg
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  7
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               4.00 TiB
  PE Size               4.00 MiB
  Total PE              1048390
  Alloc PE / Size       1048390 / 4.00 TiB
  Free  PE / Size       0 / 0   
  VG UUID               8l2jBN-VnmO-TphJ-RqBA-rnda-ROoW-QE4pQ5

```

2. 查看目录挂载信息

```bash
root@zentao:/home/ubuntu# df -h
Filesystem                   Size  Used Avail Use% Mounted on
udev                         3.9G     0  3.9G   0% /dev
tmpfs                        799M  9.1M  790M   2% /run
/dev/mapper/zentao--vg-root  2.2T   26G  2.1T   2% /
tmpfs                        3.9G     0  3.9G   0% /dev/shm
tmpfs                        5.0M     0  5.0M   0% /run/lock
tmpfs                        3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1                    720M   58M  625M   9% /boot
overlay                      2.2T   26G  2.1T   2% /var/lib/docker/overlay2/6bc943202e08c87727eda74a4f454f62a10103e59f53be5e049ce1ee0d1298ce/merged
overlay                      2.2T   26G  2.1T   2% /var/lib/docker/overlay2/ed8c8d79e65da008e4fad92b40e9eba5632216832f941df1d1a933ca9753e36f/merged
overlay                      2.2T   26G  2.1T   2% /var/lib/docker/overlay2/5a36352524032e25eca0a973fbab10ca20b333f06ee871a34648fca264b97ea9/merged
tmpfs                        799M     0  799M   0% /run/user/1000
```

2. 分别执行以下命令对根目录进行扩容

```bash
mkfs.ext4 /dev/sda3

pvcreate /dev/sda3

vgextend zentao-bg /dev/sda3

###直接将所有空余空间扩容，或者也可以指定扩容大小
lvextend -l +100%FREE /dev/mapper/zentao--vg-root  

resize2fs /dev/mapper/zentao--vg-root

```

3. 查看扩容后的根目录大小

```bash
root@zentao:/home/ubuntu# df -h
Filesystem                   Size  Used Avail Use% Mounted on
udev                         3.9G     0  3.9G   0% /dev
tmpfs                        799M  9.1M  790M   2% /run
/dev/mapper/zentao--vg-root  4.0T   26G  3.8T   1% /
tmpfs                        3.9G     0  3.9G   0% /dev/shm
tmpfs                        5.0M     0  5.0M   0% /run/lock
tmpfs                        3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1                    720M   58M  625M   9% /boot
overlay                      4.0T   26G  3.8T   1% /var/lib/docker/overlay2/ed8c8d79e65da008e4fad92b40e9eba5632216832f941df1d1a933ca9753e36f/merged
overlay                      4.0T   26G  3.8T   1% /var/lib/docker/overlay2/6bc943202e08c87727eda74a4f454f62a10103e59f53be5e049ce1ee0d1298ce/merged
overlay                      4.0T   26G  3.8T   1% /var/lib/docker/overlay2/5a36352524032e25eca0a973fbab10ca20b333f06ee871a34648fca264b97ea9/merged
tmpfs                        799M     0  799M   0% /run/user/1000
root@zentao:/home/ubuntu#
root@zentao:/home/ubuntu# lsblk
NAME                  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                     8:0    0     4T  0 disk 
├─sda1                  8:1    0   731M  0 part /boot
├─sda3                  8:3    0   1.8T  0 part 
│ └─zentao--vg-root   252:0    0     4T  0 lvm  /
└─sda5                  8:5    0 199.3G  0 part 
  ├─zentao--vg-root   252:0    0     4T  0 lvm  /
  └─zentao--vg-swap_1 252:1    0   980M  0 lvm  [SWAP]
sdb                     8:16   0     2T  0 disk 
└─sdb1                  8:17   0     2T  0 part 
  └─zentao--vg-root   252:0    0     4T  0 lvm  /
sr0                    11:0    1   873M  0 rom  
```
