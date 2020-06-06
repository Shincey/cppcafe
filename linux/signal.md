# 信号

信号是由用户、系统或者进程发送给目标进程的信息，以通知目标进程某个状态的改变或系统异常。

## 1 信号概述

### 1.1 发送信号

Linux下，一个进程给其它进程发送信号的API是`kill`函数，定义如下：

```c++
#include <sts/types.h>
#include <signal.h>
int kill(pid_t pid, int sig);
```

该函数把信号`sig`发给目标进程，目标进程由`pid`参数指定，其可能的取值及含义如下所示：

| `pid`参数  | 含义 |
|---|---|
| `pid>0`  |  信号发给PID为`pid`的进程 |
|`pid=0`|信号发给本进程组内的其它进程|
|`pid=-1`|信号发送给出init进程外所有进程，但发送者需要拥有对目标进程发送信号的权限|
|`pid<-1`|信号发送给组ID为`-pid`的进程组中的所有成员|

`kill`函数成功时返回0,失败时返回-1并设置`errno`，几个可能的`errno`：

|`errno`|含义|
|---|---|
|`EINVAL`|无效的信号|
|`EPERM`|该进程没有权限发送信号给任何一个目标进程|
|`ESRCH`|目标进程或进程组不存在|

Linux定义的信号都大于0,如果`sig`取值为0,则`kill`函数不发送任何信号。

### 1.2 信号处理

目标进程在接收到信号时，需要定一个接受函数处理。信号处理函数原型如下：

```c++
#include <bits/signum.h>
#define SIG_DFL ((__sighandler_t) 0)
#define SIG_IGN ((__sighandler_t) 1)
```

`SIG_IGN`表示忽略目标信号，`SIG_DFL`表示使用信号的默认处理方式。信号的默认处理方式有如下几种：结束进程(Term)、忽略信号(Ign)、结束继承并生成核心转储文件(Core)、暂停进程(Stop)、以及继续进程(Cont)。

### 1.3 Linux信号

Linux的可用信号都定义在`bit/signum.h`头文件中，其中包括标准信号和POSIX实时信号。

[点击这查询信号](https://man7.org/linux/man-pages/man7/signal.7.html)

和网络编成密切相关的几个信号：`SIGHUP`、`SIGPIPE`、`SUGURG`。

### 1.4 中断系统调用

如果程序在执行处于阻塞状态的系统调用时接收到信号，并且我们为该信号设置了信号处理函数，则默认情况下系统调用将被中断，并且`errno`被设置为`EINTR`。可以使用`sigaction`函数为信号设置`SA_RESTART`标志并自动重启被该信号中断的系统调用。

对于默认行为是暂停进程的信号，比如`SIGSTOP`、`SIGTTIN`，如果没有为它们设置信号处理函数，则它们也可以中断某些系统调用(比如，`connect`、`epoll_wait`)。POSIX没有规定这种行为，这是Linux特有的。

## 2 信号函数

### 2.1 signal系统调用

要为一个信号设置处理函数，可以使用下面的`signal`系统调用：

```c++
#include <signal.h>
_sighandler_t signal (int sig, _sighandler_t _handler)
```

`sig` - 要捕获的信号类型
`_handler` - 用于指定信号`sig`的处理函数，时`_sighandler_t`类型的函数指针。
函数成功时返回一个函数指针，该函数指针的类型也是`_sighandler_t`，这个返回值时前一次调用`signal`函数时传入的函数指针，或者是信号`sig`对应的默认处理函数指针`SIG_DEF`（如果第一次调用`signal`的话）。
出错时返回`SIG_ERR`，并设置`errno`。

### 2.2 sigaction系统调用

设置信号处理函数的更健壮的接口是如下的系统调用：

```c++
#include <signal.h>
int sigaction(int sig, const struct sigaction* act, struct sigaction* oact);
```

成功时返回0,失败时返回-1。
`sig` - 指出要捕获的信号类型
`act` - 指定新的信号处理方式
`oact` - 输出信号先前的处理方式（如果不为NULL的话）
`act`和`oact`都是`sigaction`结构体类型的指针，该结构体描述了信号处理的细节：

```c++
struct sigaction {
#ifdef __USE_POSIX199309
    union {
        _sighandler_t sa_handler;
        void (*sa_sigaction) (int, siginfo_t*, void*);
    } __sigaction_handler;

#define sa_handler __sigaction_handler.sa_handler
#define sa_sigaction __sigaction_handler.sa_sigaction

#else
    _sighandler_t sa_handler;
#endif

    _sigset_t sa_mask;
    int sa_flags;
    void (*sa_restorer) (void);
};
```

该结构体中`sa_handler`指定信号处理函数，`sa_mask`成员设置进程的信号掩码（确切的说是在进程原有信号掩码的基础上增加信号掩码），以指定哪些信号不能发送给本进程。`_sigset_t`类型指定一组信号。

`sa_flags`可选有：`SA_NOCLDSTOP`、`SA_NOCLDWAIT`、`SA_SIGINFO`、`SA_ONSTACK`、`SA_RESTART`等等... [点击查看更多](https://nxmnpg.lemoda.net/2/sigaction)

## 3 信号集

### 3.1 信号集函数

Linux使用`sigset_t`表示一组信号，其定义如下：

```c++
#include <bits/sigset.h>
# define _SIGSET_NWORDS (1024 / (8 * sizeof(unsigned long int)))
typedef struct {
    unsigned long int __val[SIGSET_NWORDS]；
} __sigset_t
```

可以看出`sigset`是一个长整型数组，每个元素的每个位表示一个信号。Linux提供如下一组函数来设置、修改、删除和查询信号集：

```c++
#include <signal.h>
int sigemptyset (sigset_t* _set) //清空信号集
int sigfillset (sigset_t* _set) //在信号集中设置所有信号
int sigaddset (sigset_t* _set, int _signo) //将信号_signo添加到信号集中
int sigdelset (sigset_t* _set _signo) //将信号_signo从信号集中删除
int sigismember (_const sigset_t* _set, int _signo) //测试_signo是否在信号集中
```

### 3.2 进程信号掩码

上面提到可以利用`sigaction`结构体的`sa_mask`成员来设置进程的信号掩码。此外，如下函数也可以用于设置或查看进程的信号掩码：

```c++
#include <signal.h>
int sigprocmask (int _how, _const sigset_t* _set, sigset_t* _oset);
```

`_set` - 指定新的信号掩码
`_oset` - 输出原来的信号掩码（如果不为NULL的话）
如果`_set`不为NULL，则`_how`指定设置进程信号掩码的方式：

|_how参数|含义|
|---|---|
|`SIG_BLOCK`|新的进程信号掩码是当前值和`_set`指定信号集的并集|
|`SIG_UNBLOCK`|新的进程信号掩码是当前值和~`_set`信号集的交集，因此`_set`指定的信号集将不被屏蔽|
|`SIG_SETMASK`|直接将进程信号掩码设置为`_set`|

如果`_set`为NULL，则进程信号掩码不变，此时我们仍然可以利用`_oset`参数来获得进程当前的信号掩码。
`sigprocmask`成功时返回0，失败时返回-1并设置`errno`。

### 3.3 被挂起的信号

设置进程信号掩码后，被屏蔽的信号将不能被进程接收。如果给进程发送一个被屏蔽的信号，则操作系统将该信号设置为进程的一个被挂起的信号。如果我们取消对被挂起信号的屏蔽，则它能立即被进程接收到。如下函数可以获得进程当前被挂起的信号集：

```c++
#include <signal.h>
int sigpending (sigset_t* set);
```

`set` - 用于保存被挂起的信号集。显然，进程即使多次接收到同一个被挂起的信号，`sigpending`函数也只能反映一次。并且再次使用`sigprocmask`使能挂起的信号时，该信号的处理函数也只能被触发一次。
`sigpending`成功时返回0，失败时返回-1并设置`errno`。
在多进程多线程环境中，要以进程、线程为单位来处理信号和信号掩码。我们不能设想新创建的进程、线程具有和父进程、主线程完全相同的信号特征。比如`fork`调用产生的子进程将继承父进程的信号掩码，但具有一个空的挂起信号集。

## 4 统一事件源

信号是一种异步事件：信号处理函数和程序的主循环是两条不同的执行路线。很显然，信号处理函数需要尽可能快地执行完毕，以确保该信号不被屏蔽太久（为了避免一些竞态条件，信号在处理期间，系统不会再次触发它）。

一种典型的解决方案是：把信号的主要处理逻辑放到程序的主循环中，当信号处理函数被触发，它只是简单地通知主循环程序接收到信号，并把信号值传递给主循环，主循环再根据接收到的信号值执行目标信号对应的逻辑代码。信号处理函数通常使用管道来将信号传递给主循环：信号处理函数往管道的写端写入信号值，主循环则从管道的读端读出该信号值。

那么主循环怎么知道管道上何时有数据可读呢？这很简单，我们只需要使用I/O复用系统调用来监听管道的读端文件描述符上的可读事件。如此一来，信号事件就和其它I/O事件一样被处理，即统一事件源。

## 5 网络编程相关信号

### 5.1 SIGHUP

当挂起进程的控制终端时，`SIGHUP`信号将被触发。对于没有控制终端的网络后台程序而言，它们通常利用`SIGHUP`信号来强制服务器重读配置文件。如`xinetd`超级服务程序。

`xinetd`程序在接收到`SIGHUP`信号之后，循环读取`/etc/xinetd.d`目录下的每个子配置文件，并检测其变化。如果某个正在运行的子服务的配置文件被修改以停止服务，则`xinetd`主进程将给该子服务进程发送`SIGTERM`信号以结束它。如果某个子服务的配置文件被修改以开启服务，则`xinetd`将创建新的`socket`并将其绑定到该服务对应的端口上。

### 5.2 SIGPIPE

默认情况下，往一个读端关闭的管道或`socket`连接中写数据将引发`SIGPIPE`信号，我们需要在代码中捕获到并处理该信号，或者至少忽略它，因为程序接收到该信号的默认行为时结束进程，而我们绝对不希望因为错误的写操作而导致程序退出。引起`SIGPIPE`信号的写操作将设置`errno`为`EPIPE`。

可以使用`send`函数的`MSG_NOSIGNAL`标志来禁止写操作触发`SIGPIPE`信号。在这种情况下，我们应该使用`send`函数反馈的`errno`值来判断管道或者`socket`连接的读端是否已经关闭。

此外，也可以利用I/O复用系统调用来检测管道和`socket`连接的读端是否已经关闭。以`poll`为例，当管道的读端关闭时，写端文件描述符上的`POLLHUP`事件将被触发；当`socket`连接被对方关闭时，`socket`上的`POLLRDHUP`事件将被触发。

### 5.3 SIGURG

在Linux环境下，内核通知应用程序带外数据到达主要有两种方法，一种是I/O复用技术，`select`系统调用在接收到带外数据时将返回，并向应用程序报告`socket`上的异常事件，另外一种方法就是使用`SIGURG`信号。


## 参考

摘录自 《Linux高性能服务器编程》游双 著 机械工业出版