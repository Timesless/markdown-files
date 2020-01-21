# JUC

## java.util.concurrent

+ java.util.concurrent.atomic
+ java.util.concurrent.locks

---

***____高内聚低耦合前提下***

+ **线程 操作  资源类  = 实例变量 + 实例方法**
+ **判断（volatile标识 always be uesd in a loop） 执行 通知**
+ **防止虚假唤醒**

****

### 内置锁 & Lock

> **Synchronized**
>
> ``` java
> // 资源类，高内聚提供sale()
> private int number = 30;
> private Object obj = new Object();
> // 对象锁，this的所有同步方法
> 1.1 private synchronized void sale() { }
> // 对象锁，obj的所有同步方法
> 1.2 private void sale() {
>     synchronized(obj) { }
> }
> // 全局锁,Test.class字节码对应在JRE运行时中的Class<Test>
> 3. synchronized static void test() { }
> ```
> 
>**Interface Lock**
> 
>+ <kbd>ReentrantLock</kbd>
> + <kbd>ReentrantReadWriteLock.ReadLock</kbd>
> + <kbd>ReentrantReadWriteLock.WriteLock</kbd>
> 
>### 线程状态  Thread.State
> 
>+ NEW
> + RUNNABLE
> + BLOCKED
> + WAITING
> + TIMED_WAITING
> + TERMINATED
> 

---

###  函数式编程

``` java
int age = 23;	// 值是静态的不变化
f(x) = kx + 1;	// 值随着参数而变化

// JAVA8 函数式接口
@FunctionalInterface
interface test {
    void sayHello();
    default int add(int x, int y) { return x+y; }
    static int mul(int x, int y) { return x*y; }
}
```

### 四大函数式接口

+ **Consumer**
    + void accept(T t)
    + default Consumer addThen(Consumer)
+ **Function**
    + R apply(T t);
    + default Function addThen(Function);
    + static Function identity();  **always return its input argument**

``` java
Stream<String> stream = Stream.of("i", "love", "this");
stream.collect(Collectors.toMap(Function.identity(), String::length));
```

+ **Predicate**
    + boolean test(T t);
    + default Predicate and(Predicate);
    + default Predicate or(Predicate);
    + default Predicate negate();
    + static Predicate isEqual(Object target);

``` java
String str = "123";
Predicate<String> predicate = Predicate.isEqual(str);
predicate.test("123");
```

+ **Supplier**
    + T get();

+++

## Synchronized & Lock

| synchronized块内 | Condition c = lock.newCondition() |
| :--------------: | :-------------------------------: |
|      wait()      |              await()              |
|     notify()     |             signal()              |
|   notifyAll()    |            signalAll()            |

### Interface Callable

> new Thread(Runnable target, String name)
>
> **既然能传Runnable接口，那么就能传入Runnable接口的子接口 RunnableFuture<V>**
>
> RunnableFuture接口的实现类： FutureTask，那么也就是Runnable接口的实现类
>
> 再看FutureTask的构造方法： FutureTask(Callable<V> callable)； FutureTask(Runnable)；
>
> **结论： FutureTask同时适配了Runnable，Callable**
>
> *new Thread(futureTask, "T1").start()*
>
> + 对于同一对象的同一方法提交给Callable，只会执行一次
> + get()；阻塞，请放在方法最后

## Interface BlockingQueue

| 方法 |   异常    |  特殊值  |     阻塞      |         超时阻塞         |
| :--: | :-------: | :------: | :-----------: | :----------------------: |
| 插入 |  add(E)   | offer(E) | <b>put(E)</b> | offer(E, long, TimeUnit) |
| 删除 | remove()  |  poll()  | <b>take()</b> |   poll(long, TimeUnit)   |
| 查看 | element() |  peek()  |     null      |           null           |

所有阻塞API都调用**xfer(E, boolean, int, long)**

+ E： 如果put类型就是实际的值，反之null
+ boolean：是否包含数据，put() 为true， 反之false
+ int：执行类型，立即返回NOW，异步ASYNC，阻塞SYNC，超时TIMED
+ long：超时数值，只在TIMED有作用

``` java
private E xfer(E e, boolean haveData, int how, long nanos) {
    if (haveData && (e == null))
        throw new NullPointerException();
    Node s = null;                        // the node to append, if needed
    retry:
    for (;;) {                            // restart on append race
        // 从  head 开始
        for (Node h = head, p = h; p != null;) { // find & match first node
            // head 的类型。
            boolean isData = p.isData;
            // head 的数据
            Object item = p.item;
            // item != null 有 2 种情况,一是 put 操作, 二是 take 的 item 被修改了(匹配成功)
            // (itme != null) == isData 要么表示 p 是一个 put 操作, 要么表示 p 是一个还没匹配成功的 take
            if (item != p && (item != null) == isData) { 
                // 如果当前操作和 head 操作相同，就没有匹配上，结束循环，进入下面的 if 块。
                if (isData == haveData)   // can't match
                    break;
                // 如果操作不同,匹配成功, 尝试替换 item 成功,
                if (p.casItem(item, e)) { // match
                    // 更新 head
                    for (Node q = p; q != h;) {
                        Node n = q.next;  // update by 2 unless singleton
                        if (head == h && casHead(h, n == null ? q : n)) {
                            h.forgetNext();
                            break;
                        }                 // advance and retry
                        if ((h = head)   == null ||
                            (q = h.next) == null || !q.isMatched())
                            break;        // unless slack < 2
                    }
                    // 唤醒原 head 线程.
                    LockSupport.unpark(p.waiter);
                    return LinkedTransferQueue.<E>cast(item);
                }
            }
            // 找下一个
            Node n = p.next;
            p = (p != n) ? n : (h = head); // Use head if p offlist
        }
        // 如果这个操作不是立刻就返回的类型    
        if (how != NOW) {                 // No matches available
            // 且是第一次进入这里
            if (s == null)
                // 创建一个 node
                s = new Node(e, haveData);
            // 尝试将 node 追加对队列尾部，并返回他的上一个节点。
            Node pred = tryAppend(s, haveData);
            // 如果返回的是 null, 表示不能追加到 tail 节点,因为 tail 节点的模式和当前模式相反.
            if (pred == null)
                // 重来
                continue retry;           // lost race vs opposite mode
            // 如果不是异步操作(即立刻返回结果)
            if (how != ASYNC)
                // 阻塞等待匹配值
                return awaitMatch(s, pred, e, (how == TIMED), nanos);
        }
        return e; // not waiting
    }
}
```

实现类：

+ ArrayBlockingQueue -> 数组实现的有界阻塞队列
+ LinkedBlockingQueue  -> 链表实现的有界（**默认为Integer.MAX_VALUE**）阻塞队列
+ PriorityBlockingQueue -> 支持优先级排序的无界阻塞队列
+ DelayQueue -> 支持延时获取元素的无界阻塞队列，内部以PriorityQueue实现
+ SynchronousQueue -> 单个元素的有界阻塞队列
+ LinkedTranferQueue -> 是LinkedBlockingQueue & SynchronousQueue的组合

**put()线程，首先查看head是否是take()，如果是直接交出数据，否则追加到队列，立刻返回**

**take()线程，首先查看head是否是put()，如果是直接拿走数据，如果不是追加到tail，并阻塞**

+ LinkedBlockingDeque -> 链表实现的双端阻塞队列

---

## Interface Executor

+ Interface ExecutorService
    + AbstractExecutorService
        + **ThreadPoolExecutor**
    + Interface ScheduledExecutorService

### ThreadPoolExecutor

``` java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          // 工作队列，被提交但未被执行的任务
                          BlockingQueue<Runnable>() workQueue,
                          // 线程工厂，默认即可
                          ThreadFactory threadFactory,
                          // 拒绝策略，工作队列满且工作线程 >= maximumPoolSize
                          RejectedExecutionHandler handler);
```

### 4大拒绝策略（maximum + workQueue.size()）

+ 默认抛出RejectedExecutionException -> new ThreadExecutor.AbortPolicy();
+ 将某些任务回退给调用者（main） -> **CallerRunsPolicy**
+ 丢弃 -> **discardPolicy**
+ 丢弃等待最长的任务 -> **discardOldestPolicy**

### Executors

``` java
// 比较恐怖，任务队列为无界
static ExexutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
                                  // 无参构造，容量为Integer.MAX_VALUE
                                 new LinkedBlockingQueue<Runnable>());
}
```



## 分支合并框架

+ **ForkJoinPool**
+ **ForkJoinTask**
+ **RecursiveTask extends ForkJoinTask**

### CompletableFuture 异步回调

``` java
// 无返回值异步
CompletableFuture<Void> voidFuture = CompletableFuture.runAsync(() -> sout("异步执行"));
// 有返回值
CompletableFutre<Integer> future = CompletableFuture.supplyAsync(
    sout("异步执行");
    return 200;
).whenComplete((result, throwable) -> {
    
}).exceptionally(throwable -> {
    throwable.printStackTrace();
    return 404;
});
future.get();
```

# JVM

``` mermaid
graph LR
JVM --> OS --> C[硬件 SPAC]
```

<kbd>Class files</kbd> --> <kbd>Class Loader</kbd> --> <kbd>Runtime Data Area </kbd> --> <kbd>Execution Engine & Native Interface</kbd>

+ 运行时数据区 Runtime Data Area

> <kbd>方法区</kbd>
>
> <kbd>堆</kbd>
>
> <kbd>虚拟机栈 & 本地方法栈</kbd>
>
> <kbd>程序计数器</kbd>

+ 类加载器 ClassLoader
    + 双亲委派

> <kbd>将class字节码文件加载并转换为方法区运行时数据结构</kbd>
>
> 即： **Test.class被加载并初始化之后在方法区为 Class<Test>**
>
> ``` mermaid
> graph LR
> A[Bootstrap 根加载器] --> B[Extension 扩展加载器] --> AppClassLoader
> 
> ```
>
> Bootstrap加载**${JAVA_HOME}/jre/lib/rt.jar**
>
> Extension加载扩展的补丁，**${JAVA_HOME}/jre/lib/ext/*.jar**
>
> AppClassLoader加载用户自定义类
>
> **Test.class.getClassLoader()**

+ 本地方法接口<kbd>Native Interface</kbd>

+ **方法区**

> 类结构信息等

+ **栈**

每个方法执行会创建栈帧，存储**局部变量表，操作数栈，动态链接，方法出口等**

**如果不写final，那么有可能修改引用为null，那么原引用的对象可能被GC**

栈帧：

|                    方法索引                    |
| :--------------------------------------------: |
| **局部变量表: 输入输出参数,方法内定义的变量 ** |
|                  **操作数栈**                  |
|                **Class<Test>**                 |
|                    **父帧**                    |
|                    **子帧**                    |

+ 堆
    + 新生代	**Eden, From Survivor, To Survivor**
    + 老年代
    + **元空间， 物理上实现为堆外内存**

| 名称                    | 名称          |
| ----------------------- | ------------- |
| Young generation Space  | New \| Young  |
| Tenure generation Space | Old \| Tenure |

+ Young GC | Minor GC
+ Full GC | Major GC

<kbd>堆<kbd>Young 3/8<kbd>Eden</kbd><kbd>S0</kbd><kbd>S1</kbd></kbd><kbd>Old 5/8</kbd></kbd>

[PSYoungGen: 2048k->488k(2560k)] 2048k -> 773k(9728k), 0.0015secs]

[Times: user=0.08 sys = 0.02, real=0.00 secs]

### 垃圾回收算法

+ 复制
+ 标记清除
+ 标记清除压缩

**内存连续可采用指针碰撞**

# JMM

+ 可见性
+ 原子性
+  有序性

**内存模型规定所有实例变量存储在，主内存（堆），线程需将变量从主内存复制到工作内存，执行操作后复制回主内存，线程的通信需通过主内存**

``` java
// 资源类
class Resource {
    int num = 10;
    void changeNum() { this.num = 20; }
    // A线程在2秒之后修改num，但主线程是不知道的，需要添加volatile关键字
    main() {
        Resource r = new Resource();
        new Thread(() -> {
            TimeUnit.SECONDS.sleep(2);
            r.changeNum();
        }, "A").start();
        while (r.num == 10) {  }
        sout("mission is over");
    }
}
```

