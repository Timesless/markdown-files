## 一 Nginx实战



### 1. Nginx解析

> nginx是一个轻量级、基于HTTP协议、高性能的反向代理服务器和静态Web服务器



#### 1.1 正反向代理与网关

+ 代理

    > 代理服务器的协议必定是HTTP
    >
    > 代理服务器只做URL重写或转发

|               正向代理               |               反向代理               |
| :----------------------------------: | :----------------------------------: |
|           是对客户端的代理           |           是对服务端的代理           |
|          架设在客户端的主机          |          架设在服务端的主机          |
| 正向代理是要知道访问目标服务器的地址 | 客户端访问时无需知道真正的服务器地址 |
|                                      |                                      |
|               **特点**               |               **特点**               |
|              隐藏客户端              |              隐藏服务端              |
|                 缓存                 |                 缓存                 |
|               提升速度               |        负载均衡、动静资源分离        |
|                 授权                 |              分布式路由              |



+ 网关

    > 











### 2. Nginx.conf















### 3. Nginx调优













































































































## 二 Nginx程序设计

==nginx默认以daemon在后台运行，后台包含一个master进程，和多个worker进程。==

> nginx也支持多线程模式

master监控管理worker，接收外界信号，向各woker发送信号。worker数量一般为cpu核数

master进程先建立需要listen的socket（listenfd文件描述符），然后fork多个worker进程

> 为保证只有一个进程处理该连接，worker在注册listenfd读事件前抢 accept_mutex
>
> 抢到互斥量（锁）的进程注册listenfd读事件，调用accept接受连接，然后执行business，返回，断开连接。

==每个worker只有一个主线程，采用异步非阻塞方式（具体系统调用是select/poll/epoll/kqueue）来处理连接请求，主线程循环执行所有准备好的事件==

``` shell
# 向nginx发送信号

# 重启nginx
./nginx -s reload
./nginx -s stop
```

对于一个基本的web服务器来说，事件通常有三种类型：网络事件、信号、定时器

+ 网络事件：异步非阻塞
+ 信号：信号会中断程序当前运行，在改变状态后继续执行。如果是系统调用可能导致系统调用失败
+ 定时器：nginx定时事件在红黑树维护，每次epoll_wait之前，拿出定时器事件的最小时间



#### Connection

Connection是对tcp连接的封装。包括连接socket，读，写事件。

``` c
struct ngx_connection_t {
}
```


