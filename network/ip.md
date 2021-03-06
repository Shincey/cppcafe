# IP协议



IP协议是最为核心的协议。所有的TCP、UDP、ICMP及IGMP数据都以IP数据报格式传输。IP提供不可靠、无连接的数据传输服务。

* **不可靠**指不能保证IP数据报能成功到达目的地。如果发生某种错误，丢弃该数据报，然后发送ICMP消息给发送端。
* **无连接**指IP并不维护任何关于后续数据报的状态信息。每个数据报的处理时独立的。也说明，IP数据报可以不按发送顺序接受。

IP协议除了寻址和路由以外，还可以防止数据包环路、实现流量控制、验证数据包完整性、实现数据包分片和组装。

***

**IP首部**

IP数据报的格式如图所示，普通的IP首部长为20个字节，除非含有选项字段。

![81587113320_.pic.jpg](https://i.loli.net/2020/04/17/M4tz1aTH8RfeQ6I.png)

4字节的32bit值按下面次序传输：首先0\~7bit，其次8\~15bit，然后16\~23bit，最后是24\~31bit。这种输出次序称为big endian字节序（大端）。由于TCP/IP首部所有二进制在网络中传输时都要求这种次序，因此它又称作网络字节序。**以little endian（小端）方式存储的二进制数，必须在传输数据之前把首部转换为网络字节序**。

* 版本号：IPv4、IPv6
* 首部长度：首部占32bit字的数目，包括任何选项。由于它是4bit字段，因此首部最长为60个字节。
* 服务类型：3bit的优先权子字段（已被忽略），4bit的TOS子字段（最小时延、最大吞吐量、最高可靠性、最小费用，只能置其中1位，若都为0则为一般服务）和1bit的未用位但必须置0。
* 总长度：链路层可能需要填充一些数据以达到最小长度，所以IP层需要知道IP数据报长度。
* 标识：标识字段唯一地标识主机发送的每一份数据报。通常每发送一份报文就加1。可用于分片重组，确认分片属于哪个进程。
* 标志：标志是否还有分片或能否执行分片。
* 偏移：标识分片的位置，用于重组。
* TTL：time-to-live，设置了数据报可以经过的最多路由器数，一旦经过一个处理它的路由器就减1，为0时直接丢弃，并发送ICMP报文通知发送方。
* 协议：标记上层应用，如TCP：6、UDP：17、ICMP：1。
* 首部校验和：**根据IP首部计算的校验和码，不对后面数据进行计算。ICMP、IGMP、UDP、TCP首部中的检验码会同时对首部和数据进行校验。**求一份数据报的IP校验和，首先把首部校验和字段设置为0，然后对首部每16bit进行二进制反码求和，结果存放在校验和字段中。接收方在验证时，同样对首部每16bit进行反码求和，如果首部传输过程中没有发生任何错误，计算结果应该全为1。若不是则丢弃，不用发送差错报文，由上层发现并重传。

小结：

* 标识、标志和便宜可以实现分片及重组
* 头部长度和总长度可以划分头部和上层数据界线
* TTL，不能避免环路的形成，但可以防止数据包在网络中无限循环
* TOS(type of service)，现DSCP(Differentiated Services Field)，为不同的IP数据包定义不同的服务质量。
* 首部校验和校验完整性
* 协议字段标记上层应用，指明该数据报该交由哪个协议模块处理
* IP地址

***

**IP路由**

IP可以从本地（TCP、UDP、ICMP、IGMP）接收数据报，或从一个网络接口接收数据报，进行转发。当数据报来自某个网络接口时，IP首先检查目的IP地址是否为本机IP地址之一或者IP广播地址。如果是这样，数据报就被送到由IP首部协议字段指定的协议模块处理；如果不是这样，那么若本机IP层具备路由功能，则转发，否则丢弃。

IP层在内存中有一个路由表，当收到一份数据报并进行发送时，对该表进行搜索。路由表每一项大概包含这些信息：

* **目的地址**：可以是完整的主机地址，也可以是网络地址，由该表目中标志字段指定。
* **网络掩码**：与目的地址一起标识目的主机或者路由器所在的网段的地址
* **下一跳地址**，或有直接相连的网络IP地址。
* **接口**：本地用于发送数据包的网络接口。

IP路由选择是**逐跳进行**的，并不知道到达目的地址的完整路径，只为数据报传输提供下一跳路由器的IP地址。**数据报在各站的传输过程中，目的IP地址始终不变，但是封装的目的链路层地址在每一站都可以改变。（参考ARP协议理解）**

IP路由选择主要完成以下功能：

1. 搜索路由表，如果找到能够完全匹配（**网络号和主机号都要匹配**）目的IP地址的条目，则把报文转发给该条目指定的下一跳。
2. 搜索路由表，寻找**网络号匹配**的条目，如果找到则把报文发送到该条目指定的下一跳。
3. 搜索路由表，寻找默认的条目，如果找到则把报文发送给该条目指定的下一站路由器。

如果上面的步骤都没有成功，那么该数据报就不能被传送。如果该报文来自本地，则一般会向应用程序返回一个“主机不可达”或“网络不可达”的错误（ICMP报文）。

***

**动态路由**

**路由器之间通过采用选路协议来告知对方当前所连接的网络**。路由器上有一个进程称为路由守护程序（routing daemon），它运行选路协议，并与相邻的路由器进行通信来告知和获得彼此连接的网络情况，更新内核中的路由表。路由是由路由守护程序动态地增加或删除，而不是来自于系统引导时加载。这种路由器之间适时的交换信息，通过信息扩展使所有路由器得知网络的变化。

如果路由守护程序发现前往同一目的地址存在多条路由，那么它以某种方式将最佳路由加入到内核路由表。如果发现一条链路已经断开，它可以删除受影响的路由或增加另一条路由以绕过该问题。

Internet是以一组自制系统(AS, Autonomous System)的方式组织的，比如一个校园网是一个AS。每个AS可以选择该AS中各个路由器之间的选路协议（选路协议有多个）。这种协议称为内部网关协议IGP(Interiro Gateway Protocol)。IGP有选路信息协议RIP(Routing Information Protocol)、开放最短路径优先OSPF(Open Shortest Path First)等。

BGP是不同AS的路由器之间进行通信的外部网关协议。BGP系统与其它BGP系统之间交换网络可到达信息，这些信息包括数据到达这些网络所必须经过的AS中的所有路径。这些信息足以构造一副自制系统连接图。然后根据连接图删除选路环，制定选路策略。

***

**子网寻址**

现在所有主机都要求支持子网编址，不是把IP地址单纯看成一个网络号和一个主机号组成，而是把主机号再分成一个子网号和主机号。

一个主要原因是因为A类、B类地址的主机号分配了太多的空间，可容纳的主机数为$2^{24}-2,2^{16}-2$。（全0和全1的主机号都是无效有特殊用途的）。

由管理员决定是否建立子网，以及分配多少比特子网号和主机号。例如下图的B类网络地址（140.252）的一种子网编址，在剩下的16bit中，8bit用于子网号，8bit用于主机号，就允许有254个子网，每个子网可以有254台主机。（子网号的bit数和主机号的bit数由管理员划定）。

![51587109089_.pic.jpg](https://i.loli.net/2020/04/17/ZqfymRPGcj9LOKD.png)

子网划分对外部路由器来说隐藏了内部网络组织的细节，内部的所有子网里的主机可通过一台路由器接入Internet。

在主机引导时，除了需要获取IP地址，还需要知道多少bit用于子网号以及多少bit用于主机号，这个过程通过**子网掩码**实现。其中值为1的bit留给网络号和子网号，0的bit留给主机号。

![71587110926_.pic.jpg](https://i.loli.net/2020/04/17/se2ASi1L5ymZXdN.png)

给定IP地址和子网掩码以后，主机就可以确定IP数据报的目的是：

* 本子网上的主机
* 本网络中其它子网的主机
* 其他网络上主机

***

**网络命令**

* ifconfig：对网络接口进行查询和配置。

  ```
  ifconfig -a
  ifconfig en0 //指定接口名字参数
  ```

* netstat：也提供系统上的接口信息，`-i`参数将打印接口信息，`-n`参数则打印出IP地址，而不是主机名字。

***

**参考**

[1] TCP/IP详解卷1
