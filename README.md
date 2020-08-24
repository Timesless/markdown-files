``` properties
.org 非盈利组织

git release版本会打一个tag

jpg最小，png颜色丰富，gif支持简单透明的动态图

rc文件： Run-Control

静态嵌套类static nested classes，内部类inner classes

URL： 协议://host:端口/路径[?query][#fragment]

CNN：卷积神经网络
计算机三大难题：CPU，编译器，操作系统
X86: 8086
AMD64

Node(C++)和Deno(Rust)是JavaScript非浏览器运行时

IEEE: 电气电子工程师学会
	IEEE 754 - 浮点运算规范
	IEEE 802 - 局域网及城域网

美国国家标准学会（ANSI）
国际标准化组织（ISO）

GCC是GNU（GNU's Not Unix）开发的编译器套装（GNU Compliler Collection）

贝尔实验室：Unix, C

> man ascii

final ReentrantLock lock = this.lock;
访问局部变量（load）比getField快，同时也比getStatic快

加锁后共享变量的值是从主存（DRAM）获取，而不是寄存器或高速缓存（SRAM）
释放锁时将共享变量的值刷新回主存

Java引用和C++指针：
	作为参数时： Java是值传递，C++是传递的指针
	c++指针初始值是int（不初始化比较危险）可以运算 ++，--，Java引用初始值是null无法运算

对于一个基本的web服务器来说，事件通常有三种类型，网络事件、信号、定时器

servlet服务器：Jetty，Undertow，Reactor Netty，Tomcat

** don't call us, we will call u

IO Multiplexing = event driven IO

多路复用库：
evport > epoll > kqueue  > (poll / select)

select, poll, epoll本质上是同步I/O，当事件就绪后自己负责将数据从内核缓冲区拷贝到用户缓冲区，而异步I/O是内核负责数据的拷贝

IETF（Internet Engineering Task Force，互联网工程任务组）
RFC（Request For Comments，意见征求稿）由IETF发布的一系列备忘录

JMH：（Java microbenchmark harness，Java微基准套件）

函数式更新：不会直接修改已有对象，而是创建已有对象副本并更新

# linux后台运行进程
command &
nohup command

# 标准错误重定向到标准输出，标准输出重定向到test.log，&挂在到root进程
nohup java -Xms1024m -Xmx1024m -Xmn384m -jar test.jar > test.log 2>&1 &

Java中void只能修饰方法的返回值，并且返回值不含任何值。而对象类型Void实际包含了一个值，它有且仅有一个null值

smp 对称多处理结构
```



``` js
curl -X POST "http://www.baidu.com"
curl "http://www.baidu.com"
```



