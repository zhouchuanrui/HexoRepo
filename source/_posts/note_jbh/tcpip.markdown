title: TCP/IP小笔记
Status: Public
Tags: TCP/IP
date: 2013-4-2 19:11:26
---

[TOC]

资料整理自《嵌入式Linux应用程序开发标准教程》.

# 参考模型

OSI(Open System interconnection)模型分为7层: 应用层, 表示层, 会话层, 传输层, 网络层, 数据链路层和物理层. 

>这个7层协议模型虽然规定的非常细致和完善, 但在实际中却得不到广泛的应用, 其重要的原因之一就在于它过于复杂. 

TCP/IP模型简化为4层: 应用层, 传输层, 网络层和网络接口层.

<!--more-->

# 协议族

列一下个层次上的重要协议:

层次|协议
---|---
应用层|telnet, ftp
传输层|TCP, UDP
网络层|ICMP, IGMP, IPv4, IPv6
网络接口层|ARP, RARP, MPLS

再解释下几个协议:

- ARP: 用于获得同一物理网络中的硬件主机地址.
- MPLS: 多协议标签协议, 是很有发展前景的下一代网络协议~~~汗...
- IP: 负责在主机和网络之间寻址和路由数据包.
- ICMP: 用于发送有关数据包的传送错误的协议, ping.exe用的就是这个协议.
- IGMP: 被IP主机用来向本地多路广播路由器报告主机组成员的协议.
- TCP: 为应用程序提供可靠的通信连接, 适合于一次传送大批量数据的情况. 并适用于要求得到响应的应用程序.
- UDP: 提供了无连接通信, 且不对传送包进行可靠性保证. 适合于一次传输少量数据, 可靠性则由应用层来负责.

QQ的登录用的是TCP, 发送消息用的是UDP.

# TCP和UDP

## TCP和三次握手

>通常应用程序通过打开一个socket来使用TCP服务, TCP管理到其他socket的数据传输. 可以说, 通过IP的源/目的可以唯一的区分网络中两个设备的连接, 通过socket的源/目的可以唯一的区分网络中两个应用程序的连接.

>TCP对话通过三次握手来进行初始化. 三次握手的目的是使数据段的发送和接收同步, 告诉其他主机其一次可接收的数据量, 并建立连接.

这个过程是:

1. 客户端向服务器发送一个包含SYN(J)同步标志位的数据段发送会话请求.
1. 服务器成功接收到发来的数据包后, 发送包含ACK(J+1)应答标志位, SYN(K)同步标志位还有即将发送的数据段的起始字节顺序号, 将收到的下一个数据段的字节顺序号的这样一个数据段.
1. 客户端成功接收到发来的数据包后, 发送包含ACK(K+1)应答标志位, 确认循序号和确认号的数据段.

## UDP

>UDP即用户数据报协议, 它是一种无连接协议, 因此不需要像TCP那样通过三次握手来建立一个连接. 同时, 一个UDP应用可以同时作为应用的客户端和服务器端. 由于UDP协议并不需要建立一个明确的连接, 因此建立在UDP上的应用要比建立在TCP上的应用简单很多. UDP比TCP协议更加高效, 也能更好的解决实时性的问题. 

# 端口

TCP/IP协议中的端口, 端口号为0~65535. 端口有多种分类标准, 这里介绍两种常见的分类.

1. 按端口号分布
	- 0~1023, 知名端口, 这些端口号一般固定的分配给一些服务. 比如21端口分配给FTP服务, 25端口分配给SMTP, 80端口分配给HTTP, 135端口分配给RPC.
	- 1024~65535, 动态端口, 这些端口号一般不固定分配给某个服务, 只要运行的程序向系统提出访问网络的申请, 系统就可以从这些端口号中分配一个供给该程序使用.
1. 按协议类型划分, 可以分为TCP端口, UDP端口, IP端口和ICMP端口等.
	- TCP端口, 即传输控制端口, 需要在客户端和服务器之间建立连接, 有FTP(21), Telnet(23), SMTP(25)和HTTP(80)等.
	- UDP端口, 即用户数据包协议端口, 无需在客户端和服务器之间建立连接. 常见的有DNS(53), SNMP(161)和QQ(8000, 4000)等.

常见的端口有0, 1, 7, 21, 22, 23, 53, 67, 68, 69, 79, 80, 99等, 这些都是什么呢?

# socket编程

>在Linux中的网络编程是通过socket接口来进行的. 人们常说的socket是一种特殊的I/O接口, 它也是一种文件描述符. socket是一种常用的进程之间通信机制, 通过它不仅可以实现本地机器上的进程之间的通信, 而且可以通过网络在不同的机器的进程之间通信.
每一个socket都用一个半相关描述{协议, 本地地址, 本地端口}; 一个完整的套接字则用一个相关描述{协议, 本地地址, 本地端口, 远程地址, 远程端口}来表示. socket也有一个类似于打开文件的函数调用, 该函数返回一个整型的socket描述符, 随后的连接建立, 数据传输等操作都是通过socket来实现的.

## socket类型

常见的socket有三种类型:

1. 流式(SOCK\_STREAM): 使用TCP协议, 提供可靠的, 面向连接的通信流.
1. 数据报(SOCK\_DGRAM): 使用UDP协议.
1. 原始socket: 允许对底层的协议(IP, ICMP等)进行直接访问, 主要用于协议的开发.

项目里使用的是SOCK\_STREAM类型的socket.

## 函数

来说一下一些基本函数:

- `int socket(int family, int type, int protocol)`
用于建立socket连接, 可指定socket和协议组等信息, 返回一个非负的socket描述符.在建立了socket连接之后, 可对sockaddr或sockaddr\_in结构进行初始化, 以保存建立的socket地址信息.
- `int bind(int sockfd, struct sockaddr *my_addr, int addrlen)`
用于将本地IP绑定到端口号, 主要用于TCP连接, 而在UDP中则不需要.
- `int listen(int sockfd, int backlog)`
在**服务器**端建立套接字并绑定地址后, 需要准备在该套接字上接收新的连接请求. 这时通过调用listen()来创建一个等待队列, 在其中存放未处理的客户端连接请求.
- `int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen)`
**服务器**端创建完等待队列之后, 调用这个函数来等待并接收客户端连接请求.
- `int connect(int sockfd, struct sockaddr *serv_addr, int addrlen)`
在TCP中用于bind()之后的**客户端**, 用于与服务器端建立连接. 在UDP中由于没有用bind()函数, 这个函数兼有bind()函数的作用.
- `int send(int sockfd, const void *msg, int len, int flags)`
`int recv(int sockfd, void *buf, int len, unsigned int flags)`
用于收发数据.
- `int sendto(int sockfd, const void *msg, int len, unsigned int flags, const struct sockaddr *to, int tolen)`
`int recvfrom(int sockfd, void *buf, int len, unsigned int flags, struct sockaddr *from, int *fromlen)`
在TCP中, 这俩函数作用与上面的俩函数作用相同. 在UDP中, 可以用在没有执行connect()函数的情况下, 自动寻找指定地址并进行连接.

上面讲的这些有收发行为的函数(即除了socket(), bind()外)都是阻塞性质的, 要处理多路复用的话就要用fcntl()函数和select()函数.

