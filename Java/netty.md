> I/O线程模型
>
> - 传统阻塞IO模型
> - Reactor模式（IO Multiplexing）
>     1. 单Reactor单线程
>     2. 单Reactor多线程
>     3. 主从Reactor（Netty基于主从Reactor改进）
>
> 传统IO： client请求 --> 处理线程handler（read，业务，send）
>
> Reactor模式： 基于IO多路复用，多个连接公用一个阻塞对象
>
>  1. client --> <kbd>Reactor(select, dispatch)</kbd> --> event handler（read，业务，send）
>
>  2. <kbd>Reactor</kbd>在主线程运行，主线程处理连接请求，读取发送数据。正的业务处理分发给worker线程池
>
>     client --> Reactor --> event handler（read，send） <--> worker线程池（耗时业务）
>
> 3. Reactor在主线程运行，只处理连接请求。read，业务，send分发给subReactor线程池处理，线程池再分发给worker线程池处理耗时业务
>
>     client --> Reactor --> subReactor线程池（read，send） <--> worker线程池（耗时业务）

---

Netty基于主从Reactor多线程模型。

+ BossGroup（处理accept，生成<kbd>NioSocketChannel</kbd>，注册到WorkerGroup中某一个selector）
+ WorkerGroup（网络读写）

<kbd>NioEventLoopGroup</kbd>：是一个事件循环组，每一个事件循环是NioEventLoop。

NioEventLoop：是一个不断循环处理任务的线程

1. 轮询事件（Boss轮询accept，Worker轮询读写）
2. 处理（Boss生成Nio..，注册到Worker上某一个selector，Worker..）
3. runAllTasks，即处理任务队列的任务

每个WorkerGroup（NioEventLoopGroup）处理业务会使用<kbd>ChannePipeline</kbd>的一系列handler

- 管道（ChannelPipeline）： 关联多个handler，数据依次经过所有handler（业务处理）
- 通道（NioSocketChannel）： 注重数据读写
- <b>ChannelHandlerContext</b> 

---

**TaskQueue**： 一个channel对应一个taskQueue，耗时操作可提交到队列异步执行

+ 自定义普通任务提交到TaskQueue

ctx.channel().eventLoop().execute(() -> { Thread.sleep(5000);  });

+ 自定义定时任务提交到scheduleTaskQueue

ctx.channel().eventLoop.schedule(new Runnable() {...}, 5, TimeUnit.SECONDS);

+ 非当前Reactor线程调用channel各种方法（推送系统，Map管理所有channel）

----

**异步模型**： Netty中的I/O操作是异步的， 包括**bind，write，connect**会简单返回一个**ChannelFuture**

通过**Future-Listener**对象**主动获取 | 通知机制**获取IO操作结果

