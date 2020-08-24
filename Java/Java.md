

### 4. 泛型

``` java
/**
 * 						Food
 *					  /      \
 * 				   Fruit	  Meat
 * 				   /   \      /   \
 *				Apple Banana Pork Beef
 *             /   \
 * 			Green  Red
 * 上界通配符 => 协变	
 * 此时ArrayList<? extends Fruit>能获取Fruit在内的左下角所有类型（都以Fruit接收）
 * 		是ArrayList<Fruit, Apple, Banana, GreenApple, RedApple>的父类
 * 		即可以new这些子类，new ArrayList<GreenApple>
 * 此通配符限制add|set，因为编译器不知道是哪种子类被添加（避免运行时CCE）null可以
 * get是有效的，我们都以Fruit接收（LSP引用基类的地方可透明引用子类）
 */
ArrayList<? extends Fruit> lowerBound = new ArrayList<GreenApple>();
lowerBound.add(null);
Fruit fruit = lowerBound.get(0);
/**
 * 下界通配符 => 逆变
 * <ArrayList ? super Fruit>能添加Fruit在内的左下角所有
 * 		是ArrayList<Fruit, Food, Object>的父类
 * 		即可以new这些，new ArrayList<Food>
 * 此通配符限制get，编译器不知道哪种父类被get，所以只能以Object获取
 * add|set是有效的，因为我们都当作Fruit去add|set（LSP引用基类的地方可透明引用子类）
 */
ArrayList<? super Fruit> upperBound = new ArrayList<Food>();
upperBound.add(new Fruit());
upperBound.add(new Apple());
upperBound.add(new Banana());
upperBound.add(new GreenApple());
// 只能以Object获取
Object o = upperBound.get(0);

/**
 * PECS
 * Collections.copy()例程
		 * public static <T> void copy(List<? super T> dest, List<? extends T> src) {
				int srcSize = src.size();
				if (srcSize < COPY_THRESHOLD ||
					(src instanceof RandomAccess && dest instanceof RandomAccess)) {
					for (int i=0; i<srcSize; i++)
						dest.set(i, src.get(i));
				} else {
					ListIterator<? super T> di=dest.listIterator();
					ListIterator<? extends T> si=src.listIterator();
					for (int i=0; i<srcSize; i++) {
						di.next();
						di.set(si.next());
					}
				}
			}
		 */

/**
 * <?>无界通配符，用于读取，持有一种未知类型
 * List<Object>，可以持有任何类型
 */
ArrayList<Object> objs = new ArrayList<>();
ArrayList<?> general = new ArrayList<>();
Object o2 = general.get(1);

/**
 * 不能new一个确切的泛型数组（泛型无法使用运行时相关操作，比如new，instanceOf），考虑以下例程
 *  List<String>[] lsa = new List<String>[10];
			Object o = lsa;
			Object[] oa = (Object[]) o;
			List<Integer> li = new ArrayList<Integer>();
			li.add(new Integer(3));
			// Unsound, but passes run time store check
			oa[1] = li;
			String s = lsa[1].get(0); //ClassCastException
 * 所以编译器不允许这样创建
 * 可以通过通配符创建，get时都以Object获取
 */
ArrayList<String>[] genericStringArray = new ArrayList[10];
ArrayList<?>[] genericArray = new ArrayList<?>[10];
```



### 5. JUC

> 高内聚下线程操作资源类（实例变量 + 实例方法）
>
> 判断（是否该自己执行）执行（执行逻辑）通知（执行完毕唤醒其它线程）
>
> 防止虚假唤醒（always be uesd in a loop）



#### 5.1 JUC核心

+ `java.util.concurrent`

各种并发队列（ConcurrentHashMap，CopyOnWriteArraySet），执行器（ExecutorService），调度器（SecheduledExecutorService）

+ `java.util.concurrent.atomic`

各种原子类型，如AtomicInteger，LongAdder

+ `java.util.concurrent.locks`

AQS及其子类实现等（AbstractQueuedSynchronizer，ReentrantLock，LockSupport工具类）



####  5.2 Synchronized & Lock

Synchronized三种使用方式

``` java
// 资源类，高内聚提供sale()
private int number = 30;
private Object obj = new Object();
// 对象锁，this的所有同步方法
1.1 private synchronized void sale() { }
// 对象锁，obj的所有同步方法
1.2 private void sale() {
 synchronized(obj) { }
}
// 全局锁,Test.class字节码对应在JRE运行时中的Class<Test>
3. synchronized static void test() { }
```

Lock基本使用

``` java
lock.lock();
try {
    ...;
} finally {
    lock.unlock();
}
```



其它对比

| synchronized块内 | Condition c = lock.newCondition() |
| :--------------: | :-------------------------------: |
|      wait()      |              await()              |
|     notify()     |             signal()              |
|   notifyAll()    |            signalAll()            |



#### 5.3 Callable

`new Thread(Runnable target, String name)`

既然能传Runnable接口，那么就能传入Runnable接口的子接口 RunnableFuture<V>

RunnableFuture接口的实现类： FutureTask，那么也就是Runnable接口的实现类

再看FutureTask的构造方法

+ `FutureTask(Callable<V> callable)`
+ `FutureTask(Runnable)`

==FutureTask同时适配Runnable，Callable==

`new Thread(futureTask, "T1").start()`



+ 对于同一对象的同一方法提交给Callable，只会执行一次
+ get()；阻塞，请放在方法最后调用



#### 5.4 BlockingQueue

| 方法 |   异常    |  特殊值  |    阻塞    |         超时阻塞         |
| :--: | :-------: | :------: | :--------: | :----------------------: |
| 插入 |  add(E)   | offer(E) | ==put(E)== | offer(E, long, TimeUnit) |
| 删除 | remove()  |  poll()  | ==take()== |   poll(long, TimeUnit)   |
| 查看 | element() |  peek()  |    null    |           null           |



阻塞队列实现类

+ `ArrayBlockingQueue`数组实现的有界阻塞队列
+ `LinkedBlockingQueue`链表实现的有界（默认为Integer.MAX_VALUE）阻塞队列
+ `PriorityBlockingQueue`支持优先级排序的无界阻塞队列
+ `DelayQueue`支持延时获取元素的无界阻塞队列，内部以PriorityQueue实现
+ `SynchronousQueue`单个元素的有界阻塞队列（==容量为0，每个插入必须等待另一个线程删除，反之亦然==）
+ `LinkedBlockingDeque`链表实现的双端阻塞队列（容量默认Integer.MAX_VALUE）



`TransferQueue extends BlockingQueue`

==生产者会一直阻塞直到所添加到队列的元素被某一个消费者所消费（不仅仅是添加到队列里就完事）==

+ `LinkedTranferQueue`是LinkedBlockingQueue & SynchronousQueue的组合

put()线程，首先查看head是否是take()，如果是直接交出数据，否则追加到队列，立刻返回

take()线程，首先查看head是否是put()，如果是直接拿走数据，如果不是追加到tail，并阻塞



**LinkedTranferQueue**

是SynchronousQueue，ConcurrentLinkedQueue，LinkedBlockingQueue的超集，且提供了无锁CAS实现

``` java
/**
 * E： 如果put类型就是实际的值，反之null
 * boolean：是否包含数据，put() 为true， 反之false
 * int：执行类型，立即返回NOW，异步ASYNC，阻塞SYNC，超时TIMED
 * long：超时数值，只在TIMED有作用
 */
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



#### 5.5 Executor

Executor提供对Runnable支持

ExecutroService同时提供Runnable，Callable支持



继承结构：

Interface Executor

+ Interface ExecutorService
    + class AbstractExecutorService
        + **class ThreadPoolExecutor**
    + Interface ScheduledExecutorService
        + SecheduledThreadPoolExecutor



#### 5.6 ThreadPoolExecutor

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



##### 拒绝策略

当任务数达到maximum + workQueue.size()时触发拒绝策略

+ 默认抛出RejectedExecutionException -> new ThreadExecutor.AbortPolicy();
+ 将某些任务回退给调用者（main） -> **CallerRunsPolicy**
+ 丢弃 -> **discardPolicy**
+ 丢弃等待最长的任务 -> **discardOldestPolicy**



##### Executors工具类

``` java
// 比较恐怖，任务队列为无界
static ExexutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
                                  // 无参构造，容量为Integer.MAX_VALUE
                                 new LinkedBlockingQueue<Runnable>());
}
```



#### 5.7 Fork/Join

+ ForkJoinPool extends AbstractExecutorService
+ ForkJoinTask implements Future
    + `RecursiveTask<V> extends ForkJoinTask`
    + `RecursiveAction extends ForkJoinTask`



#### 5.8 CompletableFuture

可组合式异步回调

``` java
// 无返回值异步
CompletableFuture<Void> voidFuture = CompletableFuture.runAsync(() -> sout("异步执行"));
// 有返回值
CompletableFutre<Integer> future = CompletableFuture.supplyAsync(
    sout("异步执行");
    return 200;
).whenComplete((result, throwable) -> {
    ...;
}).exceptionally(throwable -> {
    throwable.printStackTrace();
    return 404;
});
future.get() / future.join()
```



### 6. JMM

> JMM需要保证可见性，原子性，有序性
>
> ==Java内存模型规定所有实例变量存储在，主内存（堆），线程需将变量从主内存复制到工作内存，执行操作后复制回主内存，线程的通信需通过主内存==

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



