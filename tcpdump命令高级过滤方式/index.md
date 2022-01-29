# tcpdump命令高级过滤方式


首先了解如何从包头过滤信息

```shell
proto[x:y]          : 过滤从x字节开始的y字节数。比如ip[2:2]过滤出3、4字节（第一字节从0开始排）
proto[x:y] & z = 0  : proto[x:y]和z的与操作为0
proto[x:y] & z !=0  : proto[x:y]和z的与操作不为0
proto[x:y] & z = z  : proto[x:y]和z的与操作为z
proto[x:y] = z      : proto[x:y]等于z
```

操作符 : **>, <, >=, <=, =, !=**

## IP头（IPV4）

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    | <-- optional
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            DATA ...                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

> **中文：**
>
> ![268981-20180530231811755-859452716.jpg-24.1kB][1]

```c
/*IP头定义，共20个字节*/
typedef struct _IP_HEADER
{
	char m_cVersionAndHeaderLen;     　　//版本信息(前4位)，头长度(后4位)
	char m_cTypeOfService;      　　　　　 // 服务类型8位
	short m_sTotalLenOfPacket;    　　　　//数据包长度
	short m_sPacketID;      　　　　　　　 //数据包标识
	short m_sSliceinfo;      　　　　　　　  //分片使用
	char m_cTTL;        　　　　　　　　　　//存活时间
	char m_cTypeOfProtocol;    　　　　　 //协议类型
	short m_sCheckSum;      　　　　　　 //校验和
	unsigned int m_uiSourIp;     　　　　　//源ip
	unsigned int m_uiDestIp;     　　　　　//目的ip
} __attribute__((packed))IP_HEADER, *PIP_HEADER ;
```

- **版本**：

  ​	指IP协议的版本，通信双方使用的IP协议版本必须一致。一般的值为0100（IPv4），0110（IPv6）。

-  **首部长度**：

  ​	长度4比特。这个字段的作用是为了描述IP包头的长度，因为在IP包头中有变长的可选部分。该部分占4个bit位，单位为32bit（4个字节），即本区域值= IP头部长度（单位为bit）/(8*4)，因此，一个IP包头的长度最长为“1111”，即15*4＝60个字节。IP包头最小长度为20字节。

- **优先级与服务类型**：

  ​	长度8比特，定义了数据包传输的紧急程度以及时延、可靠性、传输成本等。

-    **总长度**：

  ​	16比特，以字节为单位描述IP包的总长度（包括头部和数据两部分），最大值为65535。第二行中标识符、标志和段偏移量通常联合使用，用于数据拆分时的分组和重组。

-    **标识符**：

  ​	对于上层发来的较大的数据包，往往需要拆分。路由器将一个大包进行拆分后，拆出来的所有部分被标上相同的值，该值即为标识符，用于告诉目的端哪些包属于同一个大包。

-    **标志**：

  ​	长度3比特。该字段第一位不使用。第二位是DF（Don't Fragment）位，DF位设为1时表明路由器不能对该上层数据包分段。如果一个上层数据包无法在不分段的情况下进行转发，则路由器会丢弃该上层数据包并返回一个错误信息。第三位是MF（More Fragments）位，当路由器对一个上层数据包分段，则路由器会在除了最后一个分段的IP包的包头中将MF位设为1。

-    **段偏移量**：

  ​	长度13比特，表示一个数据包在原先被拆分前的大包中的位置。接收端据此来还原和组装IP包。

-    **TTL**：

  ​	表示IP包的生存时间，长度8比特。长度8比特。当IP包进行传送时，先会对该字段赋予某个特定的值。当IP包经过每一个沿途的路由器的时候，每个沿途的路由器会将IP包的TTL值减少1。如果TTL减少为0，则该IP包会被丢弃。这个字段可以防止由于路由环路而导致IP包在网络中不停被转发。

-    **协议号**：

  ​	长度8比特，标识上一层即传输层在本次数据传输中所使用的协议。比如6代表TCP，17代表UDP等

-    **首部校验和**：

  ​	长度16位。用来做IP头部的正确性检测，但不包含数据部分。 因为每个路由器要改变TTL的值,所以路由器会为每个通过的数据包重新计算这个值。

-    **源地址**：

  ​	长度32比特，标识IP包的起源地址。

-    **目标地址**：

  ​	长度32比特，表示IP包的目的地址。

-    **可选项**：

  ​	可变长字段，主要用于测试，由起源设备跟据需要改写。

-    **填充**：

  ​	因为IP包头长度（Header Length）部分的单位为32bit，所以IP包头的长度必须为32bit的整数倍。因此，在可选项后面，IP协议会填充若干个0，以达到32bit的整数倍

## IP选项

`一般`的IP头是20字节，但IP头有选项设置，不能直接从偏移21字节处读取数据。IP头有个长度字段可以知道头长度是否大于20字节。

通常第一个字节的二进制值是：01000101，

分成两个部分：

> **0100 = 4** 表示IP版本 
>
> **0101 = 5** 表示IP头32 bit的块数，
>
> 5 x 32 bits = 160 bits or 20 bytes

如果第一字节第二部分的值大于5，那么表示头有IP选项。

下面介绍有过滤方法

**0100 0101 **: 第一字节的二进制
**0000 1111** : 与操作
<=========
**0000 0101** : 结果

正确的过滤方法:

```shell
tcpdump -i eth1 'ip[0] & 15 > 5'
```

或者

```shell
tcpdump -i eth1 'ip[0] & 0x0f > 5'　
```

## 分片标记

当发送端的MTU大于到目的路径链路上的MTU时就会被分片，分片信息在IP头的第七和第八字节：

```
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |Flags|      Fragment Offset    |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Bit 0**: 保留，必须是0
**Bit 1**: (DF) 0 = 可能分片, 1 = 不分片
**Bit 2**: (MF) 0 = 最后的分片, 1 = 还有分片

**Fragment Offset**字段只有在分片的时候才使用。

要抓带DF位标记的不分片的包，第七字节的值应该是：

01000000 = 64

```shell
tcpdump -i eth1 'ip[6] = 64'
```



## 抓分片包

- 匹配MF，分片包

```shell
tcpdump -i eth1 'ip[6] = 32'
```

最后分片包的开始3位是0，但是有Fragment Offset字段。

- 匹配分片和最后分片

```shell
tcpdump -i eth1 '((ip[6:2] > 0) and (not ip[6] = 64))'
```

测试分片可以用下面的命令：

```shell
ping -M want -s 3000 192.168.1.1
```

## 匹配小TTL

TTL字段在第九字节，并且正好是完整的一个字节，TTL最大值是255，二进制为11111111。

可以用下面的命令验证一下：

```shell
$ ping -M want -s 3000 -t 256 192.168.1.200
ping: ttl 256 out of range
```
```
 +-+-+-+-+-+-+-+-+
 |  Time to Live |
 +-+-+-+-+-+-+-+-+
```

- 在网关可以用下面的命令看看网络中谁在使用`traceroute`

```shell
tcpdump -i eth1 'ip[8] < 5'
```

## 抓大于X字节的包

- 大于600字节

```shell
tcpdump -i eth1 'ip[2:2] > 600'
```

## 更多的过滤方式

首先还是需要知道TCP基本结构

- TCP头

```shell
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |       |C|E|U|A|P|R|S|F|                               |
| Offset|  Res. |W|C|R|C|S|S|Y|I|            Window             |
|       |       |R|E|G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

　　![268981-20180530234728337-1534993130.jpg-60.1kB][2]

```c
 
/*TCP头定义，共20个字节*/
typedef struct _TCP_HEADER
{ 
	short m_sSourPort;        　　　　　　//源端口号16bit
	short m_sDestPort;       　　　　　　 //目的端口号16bit 
	unsigned int m_uiSequNum;       　　//序列号32bit 
	unsigned int m_uiAcknowledgeNum;  //确认号32bit 
	short m_sHeaderLenAndFlag;      　　//前4位：TCP头长度；中6位：保留；后6位：标志位
	short m_sWindowSize;       　　　　　//窗口大小16bit
	short m_sCheckSum;        　　　　　 //检验和16bit
	short m_surgentPointer;      　　　　 //紧急数据偏移量16bit
}__attribute__((packed))TCP_HEADER, *PTCP_HEADER;
/*TCP头中的选项定义 

kind(8bit)+Length(8bit，整个选项的长度，包含前两部分)+内容(如果有的话)

KIND =
	1表示 无操作NOP，无后面的部分
	2表示 maximum segment   后面的LENGTH就是maximum segment选项的长度（以byte为单位，1+1+内容部分长度）
	3表示 windows scale     后面的LENGTH就是 windows scale选项的长度（以byte为单位，1+1+内容部分长度）
	4表示 SACK permitted    LENGTH为2，没有内容部分
	5表示这是一个SACK包     LENGTH为2，没有内容部分
	8表示时间戳，LENGTH为10，含8个字节的时间戳
*/
```

16位源端口号和16位目的端口号。

**32位序号**：

​	一次TCP通信过程中某一个传输方向上的字节流的每个字节的编号，通过这个来确认发送的数据有序，比如现在序列号为1000，发送了1000，下一个序列号就是2000。

**32位确认号**：

​	用来响应TCP报文段，给收到的TCP报文段的序号加1，三握时还要携带自己的序号。

**4位头部长度**：

​	标识该TCP头部有多少个4字节，共表示最长15*4=60字节。同IP头部。

**6位保留**：

​	6位标志。URG（紧急指针是否有效）ACK（表示确认号是否有效）PSH（提示接收端应用程序应该立即从TCP接收缓冲区读走数据）RST（表示要求对方重新建立连接）SYN（表示请求建立一个连接）FIN（表示通知对方本端要关闭连接）

**16位窗口大小**：

​	TCP流量控制的一个手段，用来告诉对端TCP缓冲区还能容纳多少字节。

**16位校验和**：

​	由发送端填充，接收端对报文段执行CRC算法以检验TCP报文段在传输中是否损坏。

**16位紧急指针**：

​	一个正的偏移量，它和序号段的值相加表示最后一个紧急数据的下一字节的序号。

**标志位字段（U、A、P、R、S、F）**：

占6比特。各比特的含义如下：

- **URG**：紧急指针（urgent pointer）有效。
- **ACK**：确认序号有效。
- **PSH**：接收方应该尽快将这个报文段交给应用层。
- **RST**：重建连接。
- **SYN**：发起一个连接。
- **FIN**：释放一个连接。
- **窗口大小字段**：占16比特。此字段用来进行流量控制。单位为字节数，这个值是本机期望一次接收的字节数。
- **TCP校验和字段**：占16比特。对整个TCP报文段，即TCP头部和TCP数据进行校验和计算，并由目标端进行验证。
- **紧急指针字段**：占16比特。它是一个偏移量，和序号字段中的值相加表示紧急数据最后一个字节的序号。
- **选项字段**：占32比特。可能包括"窗口扩大因子"、"时间戳"等选项。

 

- 抓取源端口大于1024的TCP数据包

```shell
tcpdump -i eth1 'tcp[0:2] > 1024'
```

- 匹配TCP数据包的特殊标记

TCP标记定义在TCP头的第十四个字节

```shell
 +-+-+-+-+-+-+-+-+
 |C|E|U|A|P|R|S|F|
 |W|C|R|C|S|S|Y|I|
 |R|E|G|K|H|T|N|N|
 +-+-+-+-+-+-+-+-+
```

- 只抓SYN包，第十四字节是二进制的00000010，也就是十进制的2

```shell
tcpdump -i eth1 'tcp[13] = 2'
```

- 抓SYN, ACK （00010010 or 18）

```shell
tcpdump -i eth1 'tcp[13] = 18'
```

- 抓SYN或者SYN-ACK

```shell
tcpdump -i eth1 'tcp[13] & 2 = 2'
```

- 抓PSH-ACK

```shell
tcpdump -i eth1 'tcp[13] = 24'
```

- 抓所有包含FIN标记的包（FIN通常和ACK一起，表示幽会完了，回见）

```shell
tcpdump -i eth1 'tcp[13] & 1 = 1'
```

- 抓RST

```shell
tcpdump -i eth1 'tcp[13] & 4 = 4'
```

## 常用的字段偏移名字

tcpdump考虑了一些数字恐惧症者的需求，提供了部分常用的字段偏移名字：

- `icmptype` (ICMP类型字段)
- `icmpcode` (ICMP符号字段)
- `tcpflags` (TCP标记字段)

**ICMP类型值有**：

`icmp-echoreply`, `icmp-unreach`, `icmp-sourcequench`, `icmp-redirect`,` icmp-echo`,` icmp-routeradvert`,` icmp-routersolicit`,` icmp-timxceed`,` icmp-paramprob`,` icmp-tstamp`,` icmp-tstampreply`,` icmp-ireq`,` icmp-ireqreply`,` icmp-maskreq`,` icmp-maskreply`

**TCP标记值**：

`tcp-fin`, `tcp-syn`, `tcp-rst`, `tcp-push`, `tcp-push`, `tcp-ack`, `tcp-urg`

这样上面按照TCP标记位抓包的就可以写直观的表达式了：

- 只抓SYN包

```shell
tcpdump -i eth1 'tcp[tcpflags] = tcp-syn'
```

- 抓SYN, ACK

```shell
tcpdump -i eth1 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack != 0'
```

## 抓SMTP数据

```shell
tcpdump -i eth1 '((port 25) and (tcp[(tcp[12]>>2):4] = 0x4d41494c))'
```

抓取数据区开始为"MAIL"的包，"MAIL"的十六进制为0x4d41494c。

## 抓HTTP GET数据

```shell
tcpdump -i eth1 'tcp[(tcp[12]>>2):4] = 0x47455420'
```

"GET "的十六进制是47455420

## 抓SSH返回

```shell
tcpdump -i eth1 'tcp[(tcp[12]>>2):4] = 0x5353482D'
```

　"SSH-"的十六进制是0x5353482D

```shell
tcpdump -i eth1 '(tcp[(tcp[12]>>2):4] = 0x5353482D) and (tcp[((tcp[12]>>2)+4):2] = 0x312E)'
```


  [1]: https://upyun.oneone.life/upyun-img/20210303171904.png
  [2]: https://upyun.oneone.life/upyun-img/20210303171915.png

