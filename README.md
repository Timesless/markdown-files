### noun

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

final ReentrantLock lock = this.lock;
访问局部变量（load）比getField快，同时也比getStatic快

加锁后共享变量的值是从主存（DRAM）获取，而不是寄存器或高速缓存（SRAM）
释放锁时将共享变量的值刷新回主存

Java引用和C++指针：
	作为参数时： Java是值传递，C++是传递的指针
	c++指针初始值是int（不初始化比较危险）可以运算 ++，--，Java引用初始值是null无法运算

对于一个基本的web服务器来说，事件通常有三种类型，网络事件、信号、定时器

servlet服务器：Jetty，Undertow，Reactor Netty，Tomcat

** don't call us, we will call u，好莱坞原则

IO Multiplexing = event driven IO

多路复用库：
evport > epoll = kqueue  > (poll / select)

select, poll, epoll本质上是同步I/O，当事件就绪后自己负责将数据从内核缓冲区拷贝到用户缓冲区，而异步I/O是内核负责数据的拷贝

IETF（Internet Engineering Task Force，互联网工程任务组）
RFC（Request For Comments，意见征求稿）由IETF发布的一系列备忘录

JMH：（Java microbenchmark harness，Java微基准套件）

函数式更新：不会直接修改已有对象，而是创建已有对象副本并更新

Java中void只能修饰方法的返回值，并且返回值不含任何值。
对象类型Void实际包含了一个值，它有且仅有一个null值

smp 对称多处理结构

etcd: 可信赖的分布式键值存储服务

BM25：Best Matching，文档相似度得分算法

字长，在X86和MIPS指令体系不相同，在80X86确定的字长为16b，在MIPS中字长为32b
机器字长：计算机一次整数运算所能处理的二进制数据的位数（32位字长、64位字长）
目前x86-64机实际支持48位宽的寻址，256TB
```



### common

``` shell
# 发送post命令
curl -X POST "http://www.baidu.com"
curl "http://www.baidu.com"

# ssh连接Linux shell
ssh @192.168.17.105

```



### yaml

> 语法：
>
> 1. 必须使用空格缩进
> 2. ~ 表示null
> 3. 使用 ' 来转义，双引号内的字符串不转义

``` shell
# yam支持的数据结构
1. 对象 hash： { name: n, age: 18 }
2. 数组
	animal: [cat, dog]
	animal:
	- cat
	- dog
3. 纯量
```



### 网络

``` shell
# 自签证书
cfssl：json格式
openssl

```





### 正则

> 零宽断言 / 环视
>
> + 正向零宽断言（**表示所在的位置右侧能够匹配**）
>
> `.+(?=\.txt)`
>
> 文本：
>
> txtfile.txt
>
> exefile.exe
>
> **匹配结果：**txtfile
>
> 
>
> + 正向否定零宽断言（**表示所在的位置右侧不能匹配**）
>
> `(.+)(?!\.txt)\.[^.]+$`
>
> 文本：
>
> txtfile.txt
>
> exefile.exe
>
> 匹配结果：exefile.exe
>
> 
>
> + 逆向零宽断言（**表示所在的位置左侧能匹配**）
>
> `(?<=name=)+`
>
> 文本：
>
> name=commoueve
>
> 匹配结果：commoueve
>
> 
>
> + 逆向否定零宽断言（**表示所在的位置左侧不能匹配**）
>
> `(?<!name=)`



