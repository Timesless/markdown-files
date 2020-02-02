#### Flow 和 Stream

stream专注于数据处理，FlowAPI则专注于数据的流通，比如对数据的请求，减速，丢弃，阻塞等。

你同时也可以使用Streams作为数据源，在必要时丢弃其中的数据， 你也可以在Subscriber中使用Streams以进行数据的归并。

从技术上讲，Flows可以替换Streams，考虑以下做法： **我们创建一个Publisher作为int数组数据源，然后在Processor中转换Integer为String，最后创建一个Subscriber来归并到一个String中**。此时，使用Streams更加合理。因为这不是控制两个模块或两个线程间的数据通信。

#### stream

是由生产者（producer）生产并由一个或多个消费者消费的元素（item）的序列，这种生产者--消费者也被称之为source/sink模型或发布者--订阅者（publisher-subscriber）模型

流处理机制

+ pull 
+ push

publisher和subscriber都已相同的速率工作，这是理想的情况。

+ backpressure

当发布者快于订阅者，订阅者要么以一个无界缓冲区保存元素，要么丢弃元素，一个解决方案是**背压策略**：subscriber通知publisher减慢速率并保持元素，直到subscriber准备好处理更多元素。当publisher元素不可用时，在同步请求中，subscriber必须无限等待。解决方案是**在两端进行异步处理**

#### reactive-streams（响应式流 | 反应流）

响应式流在pull和push流处理机制之间动态切换

``` java
public interface Publisher<T> {
    void subscribe(Subscriber<? super T> s);
}

public interface Subscriber<T> {
    void onSubscribe(Subscription s);
    void onNext(T t);
    void onError(Throwable t);
    void onComplete();
}

public interface Subscription {
    void request(long n);
    void cancel();
}

public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
    
}
```



#### JDK 9的响应式流

``` java
java.util.concurrent.Flow
    Flow.Publisher<T>
    Flow.Subscriber<T>
    	// 这里 * ? 为regex中关键字，onNext可被调用多次，一旦onError | onComplete那么订阅终止
    	onSubscribe onNext* (onError | onComplete) ?
    Flow.Subscription
    Flow.Processor<T, R>
    
// Publisher接口实现类
SubmissionPublisher<T>
    /**
     * 构造函数如下
     * 空构造函数使用默认Executor，会调用ForkJoinPool的commonPool()获取
     */
    SubmissionPublisher();
	SubmissionPublisher(Executor e, int maxBufferCapacity);
	SubmissionPublisher(Executor e, int maxBufferCapacity, Biconsumer<? super Flow.Subscriber<? super T>, ? super Throwable> handler);

	/**
	 * 发布元素
	 * submit会阻塞，直到订阅者的资源可用于发布元素
	 * onDrop，在丢弃订阅者元素前调用test(), 返回true则重试该项
	 */
	int offer(T item, long timeout, TimeUnit unit, BiPredicate<Flow.Subscriber<? super T>, ? super T> onDrop>);
	int offer(T item, BiPredicate<Flow.subscriber<? super T>, ? super T> onDrop);
	int submit(T item);
```

> tips
>
> 需要订阅者才能使用发布者发布的元素， 但SubmissionPublisher包含一个consume()，它允许添加一个希望处理所有已发布元素的订阅者，并且对（错误，完成）通知不感兴趣，返回一个CompleteableFuture<Void>
>
> 没有提供subscribe的实现类，需要选择第三方库，或者自己实现。

+ publisher

``` java
main(String[] args) {
    // 
    CompleteableFuture<Void> future = null;
    try (SubmissionPublisher<Long> pub = new SubmissionPublisher<>()) {
        // 256 defaultBufferCapacity();
        sout(pub.getMaxBufferCapacity());
        // 自己实现消费
        future = pub.consume(System.out::Println)
        // 阻塞，直到订阅者可接收
        LongStream.rangeClosed().forEach(pub::submit)
    }
    if (null != future) {
        try {
            future.get();
        } catch (InterruputedException e) {
            e.printStackTrace();
        }
    }
}
```

+ SimpleSubscriber

``` java
public class SimpleSubscriber implements Flow.Subscriber<Long> {    
    private Flow.Subscription subscription;
    // Subscriber name
    private String name = "Unknown";
    // Maximum number of items to be processed by this subscriber
    private final long maxCount;
    // keep track of number of items processed
    private long counter;
    public SimpleSubscriber(String name, long maxCount) {
        this.name = name;
        this.maxCount = maxCount <= 0 ? 1 : maxCount;
    }
    public String getName() {
        return name;
    }
    @Override
    public void onSubscribe(Flow.Subscription subscription) {
        this.subscription = subscription;
        System.out.printf("%s subscribed with max count %d.%n", name, maxCount);        
        // Request all items in one go
        subscription.request(maxCount);
    }
    @Override
    public void onNext(Long item) {
        counter++;
        System.out.printf("%s received %d.%n", name, item);
        if (counter >= maxCount) {
            System.out.printf("Cancelling %s. Processed item count: %d.%n", name, counter);           
            subscription.cancel();
        }
    }
    @Override
    public void onError(Throwable t) {
        System.out.printf("An error occurred in %s: %s.%n", name, t.getMessage());
    }
    @Override
    public void onComplete() {
        System.out.printf("%s is complete.%n", name);
    }
}
```

