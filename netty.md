#### 1 用户空间与内核空间

> OS采用虚拟存储器，对32位OS寻址空间（虚拟存储空间）2<sup>32</sup> = 4G。
>
> OS核心是内核（kernel），独立于普通的应用程序，可以访问受保护的内存空间，及底层硬件设备的所有权限。
>
> 为保护内核安全，用户进程不能直接操作内核。OS将虚拟空间划为两部分
>
> + 内核空间
> + 用户空间
>
> 对32位Linux，最高的1G字节（虚拟地址0xC0000000 ~ 0xFFFFFFFF）供内核使用，称内核空间
>
> 低3G字节（虚拟地址0x0000000 ~ 0xBFFFFFFF）供各进程使用，称用户空间



#### 2 进程切换

分时CPU进程切换都在OS内核支持下完成

+ 1 保存处理机上下文，包括PC与各寄存器的值
+ 更新PCB
+ 将当前进程PCB移入相应队列，如就绪、在某事件等待队列
+ 载入另一个进程，更新其PCB
+ 更新内存管理的数据结构
+ 恢复处理机上下文



#### 3 进程阻塞

正在执行的进程，如请求系统资源失败，数据尚未到达等，由系统自动执行阻塞原语，使自己由运行态变为阻塞状态，是一种主动行为所以只有处于运行态的进程（获取CPU）才能转为阻塞状态

阻塞状态的进程，不占用CPU资源



#### 4 文件描述符fd

File Descriptor：指向文件的引用

在表现形式上是一个非负整数，实际是一个索引值，指向内核为每一个进程锁维护的该==进程打开文件的记录表。==当进程打开一个现有文件或创建新文件，内核向进程返回（系统调用）一个fd



#### 5 缓存I/O

又被称作标准I/O，大多数文件系统默认I/O操作都是缓存I/O。在Linux的缓存I/O机制中，OS将I/O的数据缓存在==文件系统的页缓存（page cache）==，也就是说，数据会先被拷贝到OS内核的缓冲区中，然后才会操作内核缓冲区拷贝到用户空间



**缺点**

数据传输过程，需要在用户空间与内核空间进行多次数据拷贝。所以有mmap直接映射



#### 6 IO模式

- 阻塞I/O（Blocking IO）
- 非阻塞I/O（Non-Blocking IO）
- ==**IO Multiplexing（Reactor / Event Driven I/O）**==
    1. 单Reactor单线程
    2. 单Reactor多线程
    3. 主从Reactor（Netty基于主从Reactor改进）

+ 异步I/O（Asynchronous IO）



**

I/O多路复用，一个进程可以监听多个描述符，一旦某个描述符就绪（一般是读写就绪）能通知用户程序进行相应读写操作。但select，poll，epoll本质是同步I/O，它们都需要自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O，只有请求是客户端发起，数据拷贝操作由OS完成，再通知用户程序，整个过程不阻塞

==I/O多路复用原理是select，poll，epoll这个function不断轮询所负责的所有socket，当某个socket就绪，select()函数返回，就通知用户进程调用read将数据从kernel缓冲区拷贝到用户进程==



##### select

``` c
int select(int n, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
```

select函数监视的fd有三类：

+ writefds
+ readfds
+ exceptfds

调用select()阻塞，当描述符就绪（可读，可写，except）或超时（timeout）时==**select返回，然后通过遍历fdset来找到就绪的描述符**==

**缺点**

单进程能打开的文件描述符在Linux上为1024



##### poll

``` c
int poll(struct pollfd *fds, unsigned int nfds, int timeout);

struct pollfd {
    int fd;	// file descriptor
    short events;	// requested events to watch
    short revents;	// returned events witnessed
}
```

pollfd结构包含要监听的event和发生的event，==pollfd没有最大数量描述符限制，和select一样，poll返回后，需要轮询pollfd来获取就绪的描述符==



**从上面来看，select和poll都需要在返回后，遍历文件描述符获取已经就绪的socket**



##### epoll

在Linux内核2.6提出，是select和poll的增强版本，==没有描述符限制，使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中（内核采用类似callback回调机制，通知用户进程，而不需要遍历fd）==，这样在用户空间和内核空间的copy只需一次。



**工作模式**

epoll对文件描述符的操作有两种模式：LT（level trigger）和ET（edge trigger）。

LT模式是默认模式，LT模式与ET模式的区别如下： 

LT模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。

 ET模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。



ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模 式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件 描述符的任务饿死。



**优点**

+ 1GB的内存，可打开10万fd，通过cat /proc/sys/fs/file-max查看

+ I/O的效率不会随监听fd的数量增加而下降，epoll不同于select和poll的轮询，而是通过每个fd定义的回调函数实现的，只有就绪的fd才会执行回调



##### 总结

在select / poll中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而epoll事先通过epoll_ctl()来注册一个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait()+时便得到通知。(此处去掉了遍历文件描述符，而是通过监听回调的的机制。这正是epoll的魅力所在。)



#### Netty线程模型

阻塞I/O：

​	client请求 --> 处理线程handler（read，业务，send）

Reactor模式：

​	 基于IO多路复用，多个连接公用一个阻塞对象

 1. client --> <kbd>Reactor(select | dispatch)</kbd> --> event handler（read，业务，send）

 2. <kbd>Reactor</kbd>在主线程运行，主线程处理连接请求，读取发送数据。正的业务处理分发给worker线程池

    client --> Reactor --> event handler（read，send） <--> worker线程池（耗时业务）

3. Reactor在主线程运行，只处理连接请求。read，业务，send分发给subReactor线程池处理，线程池再分发给worker线程池处理耗时业务

    client --> Reactor --> subReactor线程池（read，send） <--> worker线程池（耗时业务）

    

Netty基于主从Reactor多线程模型。

+ BossGroup（处理accept，生成<kbd>NioSocketChannel</kbd>，注册到WorkerGroup中某一个selector）
+ WorkerGroup（网络读写）

==NioEventLoopGroup：是一个事件循环组，每一个事件循环是NioEventLoop。==

==NioEventLoop：一个不断循环处理任务的线程==

1. 轮询事件（Boss轮询accept，Worker轮询读写）
2. 处理（Boss生成Nio..，注册到Worker上某一个selector，Worker执行业务...）
3. runAllTasks，即处理任务队列的任务

每个WorkerGroup（NioEventLoopGroup）处理业务会使用<kbd>ChannePipeline</kbd>的一系列handler

- 管道（ChannelPipeline）： 关联多个handler，数据依次经过所有handler（业务处理）
- 通道（NioSocketChannel）： 数据读写
- ChannelHandlerContext



##### TaskQueue

一个channel对应一个taskQueue，耗时操作可提交到队列异步执行

+ 自定义普通任务提交到TaskQueue

ctx.channel().eventLoop().execute(() -> { Thread.sleep(5000);  });

+ 自定义定时任务提交到scheduleTaskQueue

ctx.channel().eventLoop.schedule(new Runnable() {...}, 5, TimeUnit.SECONDS);

+ 非当前Reactor线程调用channel各种方法（推送系统，Map管理所有channel）



##### 异步模型

 ==Netty中的I/O操作是异步的， 包括bind，write，connect会简单返回一个ChannelFuture==

==通过Future-Listener对象主动获取 | 通知机制获取IO操作结果==

