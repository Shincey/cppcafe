# Socket API

## 1 socket 地址API

socket最开始的含义是一个IP地址和端口对（ip, port）。它唯一地表示了使用TCP通信的一方。

### 1.1 主机字节序和网络字节序

字节序分为大端序和小端序，大端字节序指一个整数的高位字节存储在内存的低地址位，低位字节存储在内存的高地址位；小端字节序相反。通过运行下面程序查看本机字节序。

<div style="text-align:center"><img src="https://i.loli.net/2020/05/12/u9YdXyvEcSjhwb8.png" alt="image.png" style="zoom: 45%;" /></div>

如今多数主机都是小端字节序，又称主机字节序。而网络传输中使用大端字节序，又称网络字节序。所以在传输时首先需要将主机字节序转换为网络字节序，接收时需要将网络字节序转换为主机字节序。

Linux提供4个函数来完成主机字节序和网络字节序的转换：

```c++
#include <netinet/in.h>
unsigned long int htonl(unsigned long int hostlong);
unsigned short int htons(unsigned short int hostshort);
unsigned long int ntohl(unsigned long int netlong);
unsigned short int ntohs(unsigned short int netshort);
```

顾名思义，`hton`指host to network，后面的`l`和`s`指数据类型。

### 1.2 通用socket地址

socket网络编程接口中表示socket地址的结构体是`sockaddr`：

```c++
#include <bits/socket.h>
struct sockaddr {
	sa_family_t sa_family;
  char sa_data[14];
}
```

`sa_family_t`是地址族类型，通常与协议族类型对应。常见的协议族类型和对应的地址族类型如下表：

| 协议族   | 地址族   | 描述             |
| -------- | -------- | ---------------- |
| PF_UNIX  | AF_UNIX  | UNIX本地域协议族 |
| PF_INET  | AF_INET  | TCP/IPv4协议族   |
| PF_INET6 | AF_INET6 | TCP/IPv6协议族   |

`PF_*`和`AF_*`都定义在`<bits/socket.h>`中，且二者对应的值相等。

`sa_data`成员变量用于存放socket地址值，但不同的协议族的地址值具有不同的含义和长度。如`PF_UNIX`有长度可达108字节的文件路径名；`PF_INET`有16bit端口号，32bit的IPv4地址，共6字节；`PF_INET6`有16bit端口号，32bit流标识，128bit的IPv6地址，32bit范围ID，共26字节。所以14字节的`sa_data`无法容纳多数协议族的地址值，所以Linux定义以下新的通用socket地址结构体：

```c++
#include <bits/socket.h>
struct sockaddr_storage {
	sa_family_t sa_family;
  unsigned long int __ss_align;
  char __ss_padding[128-sizeof(__ss_align)];
}
```

这个结构体提供了足够大的空间存放地址值。`__ss_align`用于内存对齐。

### 1.3 专用socket地址

为了更方便使用，Linux为各个协议族提供了专门的socket地址结构体。

UNIX本地域协议族使用如下专用socket地址结构体：

```c++
#include <sys/un.h>
struct sockaddr_un {
  sa_family_t sin_family; //地址族：AF_UNIX
  char sun_path[108];  //文件路径名
}
```

TCP/IP协议族有`socket_in`和`socket_in6`两个专用的socket地址结构体，分别用于IPv4和IPv6：

```c++
struct sockaddr_in {
  sa_family_t sin_family; //地址族：AF_UNIX
  u_int16_t sin_port; //端口号，网络字节序表示
  struct in_addr sin_addr; //IPv4地址结构体
}
struct in_addr {
  u_int32_t s_addr; //IPv4地址，网络字节序表示
}
struct socket_in6 {
  sa_family_t sin6_family; //地址族：AF_UNIX
  u_int16_t sin6_port; //端口号，网络字节序表示
  u_int32_t sin6_flowinfo; //流信息，应设置为0
  struct in6_addr sin6_addr; //IPv6地址结构体
  u_int32_t sin6_scope_id; // scope ID，尚处于试验阶段
}
struct in6_addr {
  unsigned char sa_addr[16];
}
```

所有专用的socket地址结构体在实际使用时都需要转化为socket地址类型`sockaddr`（强制转换即可），因为所有的socket编程接口使用的地址参数类型都是`sockaddr`。

### 1.4 IP地址转化函数

人们更习惯用可读性较好的字符串来表示IP地址，如点分十进制表示IPv4地址，以及用十六进制字符串表示IPv6地址。编程中需要转换为二进制数才能使用，记录日志时又需要将二进制数表示的地址转化为可读性更好的字符串表示的地址。

下面3个函数可用于点分十进制字符串表示的IPv4地址和网络字节序整数表示的IPv4地址之间的转换：

```c++
#include <arpa/inet.h>
in_addr_t inet_addr(const char * strptr);
int inet_aton(const char * cp, struct in_addr* inp);
char* inet_ntoa(struct in_addr in);
```

* `inet_addr`函数将点分十进制字符串表示的IPv4地址转化为网络字节序整数表示的IPv4地址，失败时返回`INADDR_NONE`。
* `inet_aton`和`inet_addr`完成一样的功能，但将结果存储到`inp`指向的内存中，成功时返回1，失败时返回0。
* `inet_ntoa`函数将网络字节序整数表示的IPv4地址转化为点分十进制字符串表示的IPv4地址。需要注意的是，该函数内部使用一个静态变量存储转化结果，函数返回值指向该静态变量，因此该函数是不可重入的。

以下更新的2个函数同样可以完成上述函数的功能，同时适用于IPv4和IPv6：

```c++
#include <arpa/inet.h>
int inet_pton(int af, const char* src, void* dst);
const char* inet_ntop(int af, const void* src, char* dst, socklen_t cnt);
```

* `inet_pton`函数将用字符串表示的IP地址`src`转化为网络字节序整数表示的IP地址，并将结果存放在`dst`中指向的内存中。`af`指定地址族(`AF_INET、AF_INET6`)。成功时返回1，失败返回0。

* `inet_ntop`函数进行相反的转换，`af`，`src`，`dst`含义相同，`cnt`用于指定目标存储单元的大小。下面两个宏可以指定大小，分别用于IPv4、IPv6：

  ```C++
  #include <netinet/in.h>
  #define INET_ADDRSTRLEN 16
  #define INET6_ADDRSTRLEN 46
  ```

  `inet_ntop`成功时返回目标存储单元地址，失败返回`NULL`，并设置`errno`。



## 2 socket 基础API

对于UNIX/LINUX系的系统来说，所有东西都是文件，包括socket，它就是一个可读可写可控制可关闭的文件描述符。

### 2.1 创建socket

通过下面来创建一个socket：

```c++
#include <sys/types.h>
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```

`domain`参数告诉系统使用哪个底层协议，对于TCP/IP协议族而言，该参数应该设置为`PF_INET/PF_INET6`，对于UNIX本地域协议族而言，该参数应该设置为`PF_UNIX`。

`type`参数指定服务类型，服务类型主要有`SOCK_STREAM`（流服务）和`SOCK_UGRAM`（数据报服务）。对于TCP/IP协议族来说，分别表示传输层使用TCP/UDP协议。自Linux内核2.6.17后，还可以接受`SOCK_NONBLOCK`和`SOCK_CLOEXEC`，分别表示将新创建的socket设为非阻塞的，以及用fork调用创建子进程时在子进程中关闭该socket。

`protocol`参数是在前两个参数构成的协议集合下，再选择一个具体的协议。不过一般前两个都确定了某个协议，所以常设置为0，表示使用默认协议。

`socket`函数成功时返回一个socket文件描述符，失败时返回-1，并设置`errno`。

### 2.2 命名socket

创建socket时指定了地址族，但并未指定使用该地址族中哪个具体的socket地址。将一个socket与socket地址绑定称为给socket命名。在服务器程序中，通常要命名socket，这样客户端才知道如何连接它。客户端通常不需要命名，而是采用匿名方式，即使操作系统自动分配socket地址。

```c++
#include <sys/types.h>
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr* my_addr, socklen_t addrlen);
```

`bind`将`my_addr`指向的socket地址分配给未命名的`sockfd`文件描述符，`addrlen`指出该socket地址的长度。

`bind`成功时返回0，失败时返回-1并设置`errno`。常见的两个`errno`是：

* `EACCES`：被绑定的地址是受保护地址，仅超级用户能够访问。比如普通用户将socket绑定到知名服务端口(0~1023)上。
* `EADDRINUSE`：被绑定的地址正在使用中，比如将一个地址绑定到一个处于`TIME_WAIT`状态的socket地址。

### 2.3 监听socket

`socket`被命名后，还不能马上接受客户连接，需要使用如下系统调用创建一个监听队列以存放待处理的客户连接：

```c++
#include <sys/socket.h>
int listen(int sockfd, int backlog);
```

`sockfd`指定别监听的socket，`backlog`提示内核监听队列的最大长度。监听队列的长度如果超过`backlog`，服务器将不受理新的连接，客户端也将收到`ECONNREFUSED`错误信息。在内核2.2版本之前，`backlog`参数指所有半连接(`SYN_RCVD`)和全连接(`ESTABLISHED`)的socket上限，但之后只表示全连接状态的socket的上限。处于半连接状态的socket的上限由`/proc/sys/net/ipv4/tcp_max_syn_backlog`内核参数指定。

`listen`成功时返回0，失败时返回-1并设置`errno`。

### 2.4 接受连接

下面的系统调用从`listen`监听队列中接受一个连接：

```c++
#include <sys/types.h>
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr* addr, socklen_t* addrlen);
```

`sockfd`是执行过`listen`系统调用的监听socket（执行过`listen`，处于LISTEN状态的socket）。`addr`参数用来获取被连接的远程socket地址，该socket地址的长度由`addrlen`参数指出。

`accept`成功时返回一个新的连接socket，该socket唯一地标识了被接受的这个连接，服务器可通过读写这个socket来与被连接对应的客户端通信。失败时返回-1并设置`errno`。

### 2.5 发起连接

如果说服务器通过`listen`被动接受连接，那客户端需要通过以下系统调用来主动发起连接：

```c++
#include <sys/types.h>
#include <sys/socket.h>
int connect(int sockfd, const struct sockaddr* serv_addr, socklen_t addrlen);
```

`sockfd`参数由socket系统调用返回一个socket。`serv_addr`是服务器监听的socket地址，`addrlen`则指这个地址的长度。

`connect`成功时返回0，失败时返回-1并设置`errno`。一旦成功建立连接，`sockfd`就唯一标识了这个连接，客户端可以通过读写`sockfd`来与服务器通信。失败时两个常见的`errno`：

* `ECONNREFUSED`：目标端口不存在，连接被拒绝。
* `ETIMEDOUT`：连接超时。

### 2.6 关闭连接

关闭一个连接实际上就是关闭该连接对应的socket，和关闭普通文件描述符一样：

```c++
#include <unistd.h>
int close(int fd);
```

`fd`是待关闭的socket，不过`close`系统调用并非总是立即关闭一个连接，而是将`fd`的引用计数减1。只有当`fd`的引用计数为0时，才真正关闭连接。多进程程序中，一次`fork`系统调用默认使父进程中打开的socket的引用计数加1，因此我们必须在父进程和子进程中都对该socket执行`close`调用才能将连接关闭。

如果要立即终止连接，可以使用`shutdown`系统调用：

```c++
#include <sys/socket.h>
int shutdown(int sockfd, int howto);
```

`sockfd`是待关闭的socket，`howto`参数决定了`shutdown`的行为，如下表：

| 可选值    | 含义                                                         |
| --------- | ------------------------------------------------------------ |
| SHUT_RD   | 关闭`sockfd`上的读，并且该socket接收缓冲区中的数据都被丢弃   |
| SHUT_WR   | 关闭`sockfd`上的写，发送缓冲区的数据会在真正关闭连接前全部发送出去，这种情况连接处于半关闭状态 |
| SHUT_RDWR | 同时关闭`sockfd`上的读和写                                   |

`shutdown`可以分别关闭读和写，`close`只能同时关闭。`shutdown`成功时返回0，失败时返回-1并设置`errno`。

### 2.7 数据读写

对文件的读写操作`read`和`write`同样适用于socket。但是socket编程接口提供了几个专用于socket读写的系统调用，它们增加了对数据读写的控制。

**其中用于TCP流数据读写的系统调用是：**

```c++
#include <sys/types.h>
#include <sys/socket.h>
ssize_t recv(int sockfd, void* buf, size_t len, int flags);
ssize_t send(int sockfd, const void* buf, size_t len, int flags);
```

`recv`读取`sockfd`上的数据，`buf`和`len`分别指定读缓冲区的位置和大小，`flags`通常设置为0。`recv`成功时返回实际读到的数据长度，它可能小于我们期望的长度`len`。因此我们可能要多次调用`recv`才能读到完整的数据。`recv`返回0意味着双方已经关闭连接。出错时返回-1并设置`errno`。

`send`往`sockfd`上写数据，`buf`和`len`分别指定写缓冲区的位置和大小，`send`成功时返回实际写入数据的长度，失败返回-1并设置`errno`。

`flags`参数为数据收发提供额外的控制，取决以下一个或多个值的逻辑或：

| 选项名        | 含义                                                         | send | recv |
| ------------- | ------------------------------------------------------------ | ---- | ---- |
| MSG_CONFIRM   | 指示数据链路层协议持续监听对方的回应，直到得到答复。它仅用于SOCK_DGRAM和SOCK_RAW类型的socket | Y    | N    |
| MSG_DONTROUTE | 不查看路由表，直接将数据发给本地局域网内的主机。这表示发送者确切的知道目的主机就在本地网络上 | Y    | N    |
| MSG_DONTWAIT  | 对socket的此次操作将是非阻塞的                               | Y    | Y    |
| MSG_MORE      | 告诉内核应用程序还有更多的数据要发送，内核将超时等待新数据写入TCP发送缓冲区后一并发送。这样可以防止TCP发送多个过小的报文段 | Y    | N    |
| MSG_WAITALL   | 读操作仅在读取到指定数量的字节后才返回                       | N    | Y    |
| MSG_PEEK      | 窥探读缓存中的数据，此次读操作不会导致这些数据被清除         | N    | Y    |
| MSG_OOB       | 发送或接收紧急数据                                           | Y    | Y    |
| MSG_NOSIGNAL  | 往读端关闭的管道或socket连接中写数据时不引发SIGPIPE信号      | Y    | N    |

**用于UDP数据报读写的系统调用：**

```c++
#include <sys/types.h>
#include <sys/socket.h>
ssize_t recvfrom(int sockfd, void* buf, size_t len, int flags, struct sockaddr* src_addr, socklen_t* addrlen);
ssize_t sendto(int sockfd, void* buf, size_t len, int flags, struct sockaddr* dest_addr, socklen_t addrlen);
```

`recvfrom`读取`sockfd`上面的数据，`buf`和`len`分别指定读缓冲区的位置和大小，因为UDP通信没有连接的概念，所以我们每次读取数据都要获取发送端的socket地址。

`sendto`往`sockfd`上写数据，`buf`和`len`分别指定写缓冲区的位置和大小，`dest_addr`指定接收端的socket地址。

`flags`参数和之前的一样。

值得一提的是，`recvfrom`和`sendto`也可以用于面向连接的socket数据读写，只要把socket地址参数设置为NULL，因为已经建立连接的话，就已经知道其socket地址了。

**通用的数据读写**

socket编程接口还提供一对通用的数据读写系统调用，不仅能用于TCP，也可以用于UDP：

```c++
#include <sys/socket.h>
ssize_t recvmsg(int sockfd, struct msghdr* msg, int flags);
ssize_t sendmsg(int sockfd, struct msghdr* msg, int flags);

struct msghdr {
  void* msg_name; //socket 地址
  socklen_t msg_namelen; // socket地址长度
  struct iovec* msg_iov; //分散的内存块
  int msg_iovlen; //分散的内存块的数量
  void* msg_control; //指向辅助数据的起始位置
  socklen_t msg_controllen; //辅助数据的大小
  int msg_flags; //复制函数中的flags参数，并在调用过程中更新
};

struct iovec {
  void* iov_base; //内存起始地址
  size_t iov_len; //内存长度
};
```

`msg_name`指向一个socket地址结构变量，它指定通信对方的socket地址。对于面向连接的TCP协议，该成员没有意义，必须设置为NULL。

`iovec`封装了一块内存的起始位置和长度，`msg_iovlen`指定这样的数据有多少个。对于`recvmsg`而言，数据将被读取被存放在`msg_iovlen`块分散的内存中，这些内存的起始位置和长度由`msg_iov`数组指定，这称为分散读。对于`sendmsg`而言，`msg_iovlen`块分散的内存中的数据将被一并发送，这称为集中写。

`msg_control`和`msg_controllen`用于辅助数据的传送，例如使用它们实现进程间传递文件描述符。

`msg_flags`无须设定，它会复制`recvmsg/sendmsg`函数中的`flags`。

### 2.8 地址信息函数

在某些情况下，我们想知道一个连接socket的本端socket地址，以及远端的socket地址：

```c++
#include <sys/socket.h>
int getsockname(int sockfd, struct sockaddr* address, socklen_t* address_len);
int getpeername(int sockfd, struct sockaddr* address, socklen_t* address_len);
```

`getsockname`获取`sockfd`对应的本端socket地址，并存储于`address`中，长度存储于`address_len`中。如果实际socket地址长度大于address所指内存大小，那么该socket地址将被截断。成功时返回0，失败时返回-1并设置`errno`。

`getpeername`获取`sockfd`对应的远端socket地址，参数信息相同。

### 2.9 socket选项

如果说`fcntl`系统调用是控制文件描述符属性的通用POSIX方法，那么下面两个系统调用则是专门用来读取和设置socket文件描述符属性的方法：

```c++
#include <sys/socket.h>
int getsockopt(int sockfd, int level, int option_name, void* option_value, socklen_t* restric option_len);
int setsockopt(int sockfd, int level, int option_name, const void* option_value, socklen_t option_len);
```

`sockfd`指定被操作socket；`level`参数指定要操作哪个协议的选项（属性），如IPv4、IPv6、TCP等；`option_name`指定选项的名子；`option_value`和`option_len`分别是被操作选项的值和长度。不同选项具有不同类型的值。

`getsockopt`和`setsockopt`都是成功时返回0，失败时返回-1并设置`errno`。

**SO_REUSEADDR 选项**

可以通过设置socket选项`SO_REUSEADDR`来强制使用被处于`TIME_WAIT`状态的连接占用的socket地址，例如：

```c++
int sock = socket(PF_INET, SOCK_STREAM, 0);
assert(sock > 0);
int reuse = 1;
setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &reuse, sizeof(reuse));

struct sockaddr_in address;
bzero(&address, sizeof(address));
address.sin_family = AF_INET;
inet_pton(AF_INET, ip, &address.sin_addr);
address.sin_port = htons(port);
int ret = bind(sock, (struct sockaddr*)&address, sizeof(address));
```

经过`setsockopt`设置后，即使`sock`处于`TIME_WAIT`状态，与之绑定的socket地址也可以立即被重用。此外，我们也可以通过修改内核参数`/proc/sys/net/ipv4/tcp_tw_recycle`来快速回收被关闭的socket，从而使得TCP连接根本不进入`TIME_WAIT`状态，进而允许应用程序立即重用本地的socket地址。

**SO_RCVBUF和SO_SNDBUF选项**

`SO_RCVBUF`和`SO_SNDBUF`选项分别表示TCP接受缓冲区和发送缓冲区的大小。不过，当我们用`setsockopt`来设置TCP的接收缓冲区和发送缓冲区的大小时，系统都会将其值加倍，并且不得小于某个最小值。TCP接受缓冲区的最小值是256字节，而发送缓冲区最小值是2048字节（不同系统可能不同）。此外，可以直接修改内核参数`/proc/sys/net/ipv4/tcp_rmem`和`/proc/sys/net/ipv4/tcp_wmem`来强制TCP接收缓冲区和发送缓冲区的大小没有最小值限制。

**SO_RCVLOWAT和SO_SNDLOWAT选项**

这两个选项分别表示TCP接收缓冲区和发送缓冲区的低水位标记。它们一般被I/O复用系统调用用来判断socket是否可读可写。当TCP接收缓冲区可读数据的总数大于其低水位标记时，I/O复用系统调用将通知应用程序可以从对应的socket上读取数据；当TCP发送缓冲区中的空闲空间大于其低水位标记时，I/O复用系统调用将通知应用程序可以往对应的socket上写入数据。默认情况下，这两个标记都是1字节。

**SO_LINGER选项**

该选项用于控制close系统调用在关闭TCP连接时的行为。默认情况下，当我们使用close系统调用来关闭一个socket时，close将立即返回，TCP模块负责把该socket对应的TCP发送缓冲区中残余数据发送给对方。可以通过设置该选项，实现立即返回不发送残余数据，或者其它行为。



## 3 网络信息API

socket地址两个要素，ip地址和端口号，都是用数值表示的，不便于记忆，也不便于扩展。我们可以通过主机名来访问一台机器，从而避免使用IP地址。同样，我们也可以通过服务名称来代替端口号，比如下面两个命令完全相同：

```shell
telnet 127.0.0.1 80
telnet localhost www
```

**gethostbyname和gethostbyaddr**

`gethostbyname`函数通过主机名称获取主机完整信息，`gethostbyaddr`函数根据IP地址获取主机完整信息。`gethostbyname`通常现在`etc/hosts`配置文件中查找主机，如果没有找到，再去访问DNS服务器。

```c++
#include <netdb.h>
struct hostent* gethostbyname(const char* name);
struct hostent* gethostbyaddr(const void* addr, size_t len, int type);

struct hostent {
  char* h_name; //主机名
  char** h_aliases; //主机别名列表，可能有多个
  int h_addrtype; //地址类型
  char** h_addr_list; //按网络字节序列出的主机IP地址列表
}
```

`type`指定IP地址的类型，包括`AF_INET / AF_INET6`。

**getservbyname和getservbyport**

前者是根据名称获取某个服务的完整信息，后者是根据端口号获取某个服务的完整信息。它们实际上都是通过读取`/etc/services`文件来获取服务的信息的。定义如下：

```c++
#include <netdb.h>
struct servent* getservbyname(const char* name, const char* proto);
struct servent* getservbyport(int port, const char* proto);

struct servent {
  char* s_name; //服务名称
  char** s_aliases; //服务的别名列表，可能有多个
  int s_port; //端口号
  char* s_proto; //服务类型 tcp/udp
}
```

`name`参数指定目标服务的名字，`port`参数指定目标服务对应的端口号。`proto`参数指定服务类型，给它传递`tcp`表示获取流服务，传递`udp`表示获取数据报服务，传递`NULL`表示获取所有类型服务。

**getaddrinfo**

既可以通过主机名获得IP地址（内部使用的是`gethostbyname`），也能通过服务名称获得端口号（内部使用的是`getservbyname`）。它是否可重入取决内部调用的`gethostbyname`和`getservbyname`是否是重入版本。函数定义如下：

```c++
#include <netdb.h>
int getaddrinfo(const char* hostname, const char* service, const struct addrinfo* hints, struct addrinfo** result);

struct addrinfo {
  int ai_flags; //
  int ai_family; //地址族
  int ai_socktype; //服务类型
  int ai_protocol; //
  socklen_t ai_addrlen; //socket地址...的长度
  char* ai_canonname; // 主机名别称
  struct sockaddr* ai_addr; //指向socket地址
  struct addrinfo* ai_next; //指向下一个addrinfo对象
}
```

`hostname`可以接收主机名，也可以接收字符串表示的IP地址。同样`service`可以接收服务名，也可以接收字符串表示的十进制端口号。`hints`参数是应用程序给`getaddrinfo`的一个提示，以对`getaddrinfo`的输出进行更精确的控制。`hints`参数可以被设置为`NULL`，表示允许`getaddrinfo`反馈任何可用结果。`result`指向一个存储反馈结果的链表。

**getnameinfo**

能够通过socket地址同时获得以字符串表示的主机名（内部使用`gethostbyaddr`）和服务名（内部使用`getservbyport`）。是否可重入取决内部使用的函数是否可重入。函数定义：

```c++
#include <netdb.h>
int getnameinfo(const struct sockaddr* sockaddr, socklen_t addrlen, char* host, socklen_t hostlen, char* serv, socklen_t servlen, int flags);
```

`getnameinfo`将返回的主机名存储在`host`指向的缓存中，将服务名存储在`serv`指向的缓存中，`hostlen`和`servlen`分别指定这两个缓冲的长度。`flags`控制`getnameinfo`的行为。

`getaddrinfo`和`getnameinfo`都是成功返回0，失败返回错误码。
