### Nginx基础

==nginx默认以daemon在后台运行，后台包含一个master进程，和多个worker进程。nginx也支持多线程模式==

master监控管理worker，接收外界信号，向各woker发送信号。worker数量一般为cpu核数

master进程先建立需要listen的socket（listenfd文件描述符），然后fork多个worker进程

为保证只有一个进程处理该连接，worker在注册listenfd读事件前抢 accept_mutex，抢到互斥量（锁）的进程注册listenfd读事件， 调用accept接受连接，然后执行business，返回，断开连接。

==每个worker只有一个主线程，采用异步非阻塞方式（具体系统调用是select/poll/epoll/kqueue）来处理连接请求，主线程循环执行所有准备好的事件==

``` shell
// 重启nginx
kill -HUP pid  --0.8 later--> ./nginx -s reload
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





### Nginx.conf



> 全局块：boss_process & worker_process数量
>
> events块： 配置nginx & 用户网络连接
>
> http块： 反向代理，动静分离，缓存，日志...
>
> + http全局
> + server块：具体配置虚拟主机的代理（每个主机对应一个server块）
>     + server
>     + location 匹配

``` properties
# 命令
./nginx
# 向nginx进程发送信号
./nginx -s stop | reload | quit | reopen
# 指定配置文件启动
./nginx -c configLocation

############ 反向代理配置
# server1
sever {
	listen	80;
	server_name	localhost;	
	
	location / {
        # 反向代理路径
        proxy_pass	http://192.168.1.104:8080
	}
}

# server2
server {
	...
	# 正则匹配 ~
	location ~ /dev/ {
		proxy_pass	...
	}
	location ~ /prod/ {
		proxy_pass	...
	}
}

############ http块 负载均衡配置
# 轮询 权重 hash fair
# 集群列表
upstream balanceserver {
	# 1 权重
	# server localhost:8081	weight=5;
	# server localhost:8082	weight=10;
	
	# 2 ip_hash
	ip_hash;
	server localhost:8081	weight=5;
	server localhost:8082	weight=10;
	
	# 3 fair 响应时间
}
server {
	location / {
		# banlanceserver
		proxy_pass	http://balanceserver;
	}
}

############ 动静请求分离
# static -> html css img
# dynamic -> tomcat

# 方式一 -> 静态资源与动态资源独立部署到不同服务器
location /img/ {
	root /img/;
	# 列出当前文件夹内容
	autoindex	on;
}
location /css/ {
	root /css/;
}
# 方式二 -> 动静资源发布到一台服务器，通过配置区分
```