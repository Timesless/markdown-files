### 第一部分 Netty





### 第三部分 Zookeeper



#### 10 ZK分布式协调

ZooKeeper是Hadoop的正式子项目，它是一个针对大型分布式系统的可靠协调系统，**==提供的功能包括：配置维护、名字服务、分布式同步、组服务等。==**



##### 10.1 集群安装

ZK集群中，需要一个主节点（Leader），ZK集群规则如下：

+ 节点个数为奇数个（可用节点数量 > 总节点数 / 2，如果节点为偶数个可能出现不满足这个规则）

+ 至少3个节点



步骤

+ 为3个节点创建日志、数据目录（d:/log/zoo1，log/zoo2...，d:/data/zoo1，...）

+ 创建myid文件
    + myid文件的唯一作用是记录（伪）节点的编号
    + myid文件是一个文本文件，文件名称为myid
    + myid文件内容为一个数字，表示节点的编号
    + 在myid文件中只能有一个数字，不能有其他的内容
    + myid文件的存放位置，默认处于数据目录下。
+ 创建对应配置文件（zoo1.cfg，...）

``` properties
dataDir = d:/data/zoo1/
dataLogDir = d:/log/zoo1/

#客户端连接端口（其余为2182，2183）
clientPort = 2181

# 集群配置
# server.myid = host:port:port
# 第一个port：zk集群节点通信端口，第二个port：zk集群leader选举端口
server.1 = 127.0.0.1:2887:3887
server.2 = 127.0.0.1:2888:3888
server.3 = 127.0.0.1:2889:3889

tickTime = 3000
# follower与leader数据同步时间，允许在tickTime * initLimit时间内完成
initLimit = 10
# 心跳最大延迟，tickTime * syncLimit，否则脱离集群
syncLimit = 5
```



##### 10.2 使用ZK进行分布式存储



**Zookeeper存储模型**

是一颗以“/”为根节点的树，ZK每个节点叫做ZNode，按照层次关系形成ZNode树

整个树性目录结构常驻内存，为保证高吞吐低延迟，ZK每个节点的负载（payload）上限1MB



**ZK命令列表**

``` shell
create
ls
get
set
delete

> get /

# 当前节点创建时的事务id
cZxid = 0x0
ctime = Thu Jan...
# 当前节点修改时的事务id，与子节点无关
mZxid = 0x0
mtile = ...
# 子节点最后一次修改（创建 | 删除）的事务id，与孙子节点无关
pZxid = 0x40000193
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 3
```

所返回的节点信息主要有事务id、时间戳、版本号、数据长度、子节点的数量等。

事务id记录着节点的状态，ZooKeeper状态的每一次改变都对应着一个递增的事务id（Transaction id），该id称为Zxid，它是全局有序的，每次ZooKeeper的更新操作都会产生一个递增的Zxid



#####  10.3 Zookeeper客户端

ZooKeeper官方API功能比较简单，不推荐使用，第三方开源客户端API，主要有ZkClient和Curator。

+ ZkClient

``` java
ZkClient client = new ZkClient("localhost:2181");
String path = "/test";
boolean exists = client.exists(path);
if (!exists) {
    client.createPersistent(path);
}
```

+ Curator

Curator还为ZooKeeper客户端框架提供了一些比较普遍的、开箱即用的、分布式开发用的解决方案，例如Recipe、共享锁服务、Master选举机制和分布式计算器

==“Guava is to Java that Curator to ZooKeeper”==

curator包含三个包：

+ curator-framework（对ZK API的封装）
+ curator-client（封装客户端操作）
+ curator-recipes（封装一些高级特性：分布式计数器，分布式Barrier，分布式锁，选举）

``` java
CuratorFramework client = CuratorFrameworkFactor.newClient(connectionString);
String path = "/test/node-1";
byte[] payload = "hello".getBytes("utf-8");
client.create()
    .creatingParentsIfNeeded()
    .withMode(CreateMode.PERSISTENT)
    .forPath(path, payload);

byte[] data = client().getData().forPath(path);
new String(data, "UTF-8");

client.getChildren().forPath(path);
client.setData().forPath(path, newPayload);

// 异步操作
AsyncCallback.StringCallback callback = new AsyncCallback.StringCallback() {
    @Override
    public void processResult(int i, String s, Oject o, String s1) {
        sout(i);
        sout(s);
        sout(o);
        sout(s1);
    }
}
client.setData().inBackground(callback).forPath(path, newPayload);
```

CRUD（Create、Retrieve、Update、Delete）



**Zookeeper节点类型**

+ PERSISTENT
+ PERSISTENT_SEQUENTIAL
+ EPHEMERAL（会话有效期内）
+ EPHEMERAL_SEQUENTIAL



##### 10.4 *ZK实践



###### ZK分布式命名服务

dubbo大致思路：

+ 服务提供者启动时，在ZK注册节点`/dubbo/${serviceName}/providers`写入自己的API地址
+ 服务消费者启动时，订阅对应节点下URL地址，获得服务提供者提供的API

``` java
// 按以上思路实现
```





###### 分布式ID生成

+ UUID
+ Zookeeper生成ID
+ 分布式缓存Redis生成ID（利用INCR,INCRBY原子性）
+ SnowFlake
+ MongoDb的ObjectId



**ZK分布式ID生成**

``` java
class IDMaker {
    
    /**
     * 在某一固定路径上创建zk节点
     * 分布式ID增长快，创建临时节点
     */
    String createSequenceNode(String zkPathPrefix) {
        return client.create()
            .creatingParentsIfNeeded()
            .withMode(CreateMode.EPHEMERAL_SEQUENTIAL)
            .forPath(zkPathPrefix);
    }
    
    String makeId(String zkNodeName) {
        String str = createSequenceNode(zkNodeName);
        // zk会在节点名称后面添加递增序列值
        int idx = str.lastIndexOf(nodeName);
        if (idx >= 0) {
            return str.subString(idx);
        }
        return str;
    }
}
```



###### ZK 实现SnowFlake ID案列

snowflake-64bit

+ 1（始终0）
+ 41（时间戳，epoch之后69年）
+ 10（机器id，最多容纳1024个节点）
+ 12（0-4095序列值，这个值在同一毫秒同一节点从0开始不断累加）

**==总体来说，在工作节点1024顶配场景，同一毫秒生成ID数量为：1024 * 4096 = 4194304==**



自定义zk snowflake ID

在SnowFlake算法中，第三个部分是工作机器ID，可以结合上一节的命名方法，并通过ZooKeeper管理workId，免去了手动频繁修改集群节点去配置机器ID的麻烦。

+ 1（始终0）

+ 40（时间戳，持续32年）
+ 13（机器ID，8192节点数）
+ 10（同一毫秒同一节点的序列值）

``` java
class ZKSnowflakeIdGenerator {
    // 单例
    public static ZKSnowflakeIdGenerator instance = new ZKSnowflakeIdGenerator();
    // 私有化构造器
    private ZKSnowflakeIdGenerator() {}
    
    public synchronized void init(long workerId) {
        instance.workerId = workerId;
    }
    
    // 当前时间戳
    private static final long START_TIME = Instant.now().toEpochMilli();
    // workerid 位数，最多支持8192个节点
    private static final int WORKER_ID_BITS = 13;
    // 序列号，同一毫秒同一节点递增序列值
    private static final int SEQUENCE_BITS = 10;
    
    /**
     * 减法效率不如移位
     */
    // 最大workerId掩码，8091
    private static final long MAX_WORKER_ID = ~(-1L << WORKER_ID_BITS);
    // 最大序列号掩码 1023
    private static final long MAX_SEQUENCE = ~(-1L << SEQUENCE_BITS);
    
    // worker 编号移位
    private final static long APP_HOST_ID_SHIFT = SEQUENCE_BITS;
    // 时间戳移位
    private final static long TIMESTAMP_LEFT_SHIFT = WORKER_ID_BITS + APP_HOST_ID_SHIFT;
    
    // 当前项目，werkerid
    private long workerId;
    // 上次生成ID的时间戳
    private long lastTimetamp = -1L;
    // 当前毫秒生成的序列值
    private long sequence = 0L;
    
    public long nextId() {
        return generateId();
    }
    
    private synchronized long generateId() {
        long current = System.currentTimeMillis();
        if (current < lastTimeStamp) return -1;
        // 属于同一毫秒内，递增序列值
        if (current == lastTimeStamp) {
            sequence = (sequence + 1) & MAX_SEQUENCE;
            
            // 序列已最大，那么不用生成。阻塞到下一毫秒
            if (sequence == MAX_SEQUENCE) {
                current = this.nextMs(lastTimestamp);
            }
        } else {
            sequence = 0L;
        }
        lastTimestamp = current;
        // 时间戳左移23位
        long time = (current - START_TIME) << TIMESTAMP_LEFT_SHIFT;
        // workerId左移10位
        long workerId = workerId << APP_HOST_ID_SHIFT;
        
        return sequence | time | workerId;
    }
    
    /**
     * lastTimestamp 上次生成id的毫秒值
     */
    private long nextMs(long lastTimeStamp) {
        long current = System.currentTimeMills();
        while (current <= time)
            current = System.currentTimeMills();
        return current();
    }
}

/**
 * 对zk 实现的snowflake id生成测试
 */
class client() {
    main(String[] args) {
        long workerId = zk当前节点的id（参见分布式同一命名服务）;
        
        var instance = ZKSnowflakeIdGenerator.instance();
        instance.init(workerId);
        final HashSet set = Collections.synchronizedCollection(new HashSet<>(5000_000_0));
        ExecutorService executor = Executors.newFixedThreadPool(10);
        long start = System.currentTimeMills();
        IntStream.rangeClose(0,10).forEach(i -> {
            executor.execute(() -> {
                for(long i = 0; i < 5000_000; ++i) {
                    long id = instance.nextId();
                    // ? synchronized
                    set.add(id);
                }
            })
        });
        executor.shutdown();   
        try {TimeUnit.sleep(TimeUnit.SECONDS, 10);} catch(InterruptedException e){}
        sout(System.currentTimeMills() - start);
    }
}
```



关键算法解释：

+ 在单节点上获得下一个ID，使用Synchronized控制并发，没有使用CAS（Compare And Swap，比较并交换）的方式，是因为CAS不适合并发量非常高的场景；
+ 如果在一台机器上当前毫秒的序列号已经增长到最大值1023，则使用while循环等待直到下一毫秒；
+ 如果当前时间小于记录的上一个毫秒值，则说明这台机器的时间回拨了，于是阻塞，一直等到下一毫秒。



#### 10.5 分布式事件监听

curator客户端API提供两种类型的事件监听：

+ 标准的观察者模式（只能监听一次，重复监听需要每次监听事件再注册）
+ 缓存监听模式（本地缓存视图与ZK服务器视图对比，且在数据同步时触发相应事件）
    + Node Cache节点缓存（监听ZNode节点）
    + Path Cache子节点缓存（监听ZNode的子节点）
    + Tree Cache树缓存（监听ZNode和ZNode的子节点）

``` java
// 1 观察者模式
Watcher watcher = new Watcher() {
    @Override
    // watchedEvent时curator封装过的API，ZK集群传输的是watcherEvent
    public void process(WatchedEvent we) {
        log.info("事件发生[0]", we);
    }
}
byte[] content = client.getData().usingWatcher().forPath(path);
log.info("监听节点的内容：[0]", new String(content, "utf-8"));
// 改变数据
client().setData().forPath(path, "new content".getBytes());
client.setData().forPath(path, "second new content".getBytes());
// 可以看到只会触发一次监听事件的发生


/**
 * 2
 * NodeCache, NodeCacheListener
 * PathCache, PathCacheListener
 * TreeCache, TreeCacheListener
 */
nodeCache.getListener().addListener(listener).start();

// 事件对应的一些枚举
NODE_ADDED;
NODE_UPDATE;
NODE_REMOVED;
CHILD_ADDED;
...;
```





#### 10.6 ZK分布式锁

分布式锁：跨JVM进程数据同步



###### 分布式锁方式一

**思路：**

一个ZooKeeper分布式锁，首先需要创建一个父节点，尽量是持久节点（PERSISTENT类型），然后每个要获得锁的线程都在这个节点下创建个临时顺序节点。由于Zk节点是按照创建的顺序依次递增的，为了确保公平，可以简单地规定，编号最小的那个节点表示获得了锁

**步骤：**

每个线程抢占锁之前，先抢号创建自己的ZNode（EPHMERAL_SEQUENTIAL）。同样，释放锁的时候，就需要删除抢号的ZNode。在抢号成功之后，如果不是排号最小的节点，就处于等待通知的状态。等谁的通知呢？不需要其他人，只需要等前一个ZNode的通知即可。当前一个ZNode被删除的时候，就是轮到了自己占有锁的时候。第一个通知第二个、第二个通知第三个...。ZK节点监听机制能完美的实现这种传递，只需监听自己前一个节点的删除事件

**ZK内部机制，即使网络异常或其他原因导致占用锁的客户端失联，锁依然能释放，临时节点也能被删除**

``` java
// 可重入锁
final AtomicInteger lockCount = new AtomicInteger(0);
boolean lock() {
    synchronized(this) {
        if (lockCount.get() == 0) {
            lockCount.incrementAndGet();
        } else {
            if (! thread.equals(Thread.currentThread())) {
                return false;
            }
            lockCount.incrementAndGet();
            return true;
        }
    }
}
```



###### 分布式锁方式二

**参见awesome-java**

所有client抢占式创建同一个临时节点，誰创建成功代表誰获取锁



###### Curator提供的分布式锁

`InterProcessMutex`可重入锁

``` java
class InterProcessMutexTest {
    
    int count = 0;
    
    main(String[] args) {
        CuratorFramework client = ...;
        final InterProcessMutex mutex = new InterProcessMutex(client, "/mutex");

        for(int i = 0; i < 10; ++i) {
            FutureTaskScheduler.add(() -> {
               try {
                   mutex.accquire();
                   for (int j = 0; j < 10; ++j)
                       ++ count;
                   mutex.release();
               } catch(InterruptedException e) {
                   e.printStackTrace();
               }
            });
        }
    }
}
```



###### 总结

（1）优点：ZooKeeper分布式锁（如InterProcessMutex），能有效地解决分布式问题，不可重入问题，使用起来也较为简单。

（2）缺点：ZooKeeper实现的分布式锁，性能并不太高。为什么呢？因为每次在创建锁和释放锁的过程中，都要动态创建、销毁暂时节点来实现锁功能。大家知道，Zk中创建和删除节点只能通过Leader（主）服务器来执行，然后Leader服务器还需要将数据同步到所有的Follower（从）服务器上，这样频繁的网络通信，性能的短板是非常突出的。



目前分布式锁主流的方案有两种：

（1）基于Redis的分布式锁。适用于并发量很大、性能要求很高而可靠性问题可以通过其他方案去弥补的场景。（2）基于ZooKeeper的分布式锁。适用于高可靠（高可用），而并发量不是太高的场景。

``` c
/**
 * 1
 * setnx + lua
 * set k v [ex 200 px 20000] [nx | xx]
 */

// 获取锁（unique_value可以是UUID等）
SET resource_name unique_value NX PX 30000

// 释放锁（lua脚本中，一定要比较value，防止误解锁）
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
    
/**
 * 2 redission
 * 用法与ReentranLock类似。
 */
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.3.2</version>
</dependency>
```





#### 11 Redis分布式缓存



##### 11.1 Redis字符串

String类型是Redis中最简单的数据结构。它既可以存储文字（例如"helloworld"），又可以存储数字（例如整数10086和浮点数3.14），还可以存储二进制数据（例如10010100）。

（1）设值：SET Key Value [EX seconds]

（2）批量设值：MSET Key Value[Key Value ...]

（3）批量添加：MSETNX Key Value [Key Value...]

（4）获取：GET Key

（5）批量获取：MGET Key[Key...]

（6）获取长度：STRLEN Key

（7）为Key键对应的整数Value值增加1:INCR Key

（8）为Key键对应的整数Value值减少1:DECR Key

（9）为Key键对应的整数Value值增加increment:INCRBY Key increment

（10）为Key键对应的整数Value值减少decrement:DECRBY Keydecrement

（11）为Key键对应的浮点数Value值增加increment:INCRBYFLOAT Keyincrement



##### 11.2 List列表

Redis的List类型是基于双向链表实现的，可以支持正向、反向查找和遍历

List列表的典型应用场景：

+ 网络社区中最新的发帖列表
+ 简单的消息队列
+ 最新新闻的分页列表
+ 博客的评论列表
+ 排队系统等等。



（1）右推入：RPUSH Key Value [Value ...]

（2）左推入：LPUSH Key Value [Value ...]

（3）左弹出：LPOP Key

（4）右弹出：RPOP key

（5）获取列表的长度：LLEN Key

（6）获取列表指定位置上的元素：LINDEX Key index

（7）获取指定索引范围之内的所有元素：LRANGE Key start stop

（8）设置指定索引上的元素：LSET Key index Value



##### 11.3 哈希表

Redis中的Hash哈希表是一个String类型的Field字段和Value值之间的映射表

（1）设置字段-值：HSET Key Field Value；

（2）获取字段-值：HGET Key Field；

（3）检查字段是否存在：HEXISTS Key Field；

（4）删除给指定的字段：HDEL Key Field [Field ...]；

（5）查看指定的Field字段是否存在：HEXISTS Key Field；

（6）获取所有的Field字段：HKEYS Key；

（7）获取所有的Value值：HVALS Key。



##### 11.4 集合Set

（1）添加元素：SADD Key member1 [member2 ……]

（2）移除元素：SREM Key member1[member2……]

（3）判断某个元素：SISMEMBER Key member

（4）获取集合的成员数：SCARD Key

（5）获取集合中的所有成员：SMEMBERS Key



##### 11.5 有序集合

（1）添加成员：ZADD Key Score1 member1 [ScoreN memberN…]

（2）移除元素：ZREM Key member1[memberN……]

（3）取得分数：ZSCORE Key member

（4）取得成员排序：ZRANK Key member

（5）成员加分：ZINCRBY Key Score member

（6）区间获取：ZRANGEBYSCORE Key min max [WITHSCORES] [LIMIT]

（7）获取成员数：ZCARD Key

（8）区间计数：ZCOUNT Key min max

（9） 按索引遍历元素：zrange Key start stop



#### 12 Jedis编程

``` xml
<dependency>
	<groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```



``` java
/**
 * jedis对redis命令封装，用法基本一致。
 *
 */
new Jedis("localhost", 6379);

@Test
public void operateForString() {
    jedis.set("k", "v");
    jedis.setnx();
    jedis.rename("k", "newK");
    jedis.exists("k");
    jedis.strlen("newK");
    jedis.getrange("newK", 0, -1);
    jedis.append("newK", "vExt");
}

@Test
public void operateForList() {
    jedis.ping();
    jedis.rpush("list", "1", "2", "3");
    jedis.llen("list");
    jedis.lindex("list", "1");
    lpop();
    rpop();
    lset("list", 0, "0");
    lrange("list", 0, -1);
}

@Test
public void operateForHash() {
    jedis.hset("config", "ip", "127.0.0.1");
    Map<String, String> map = new HashMap<>(4);
    map.put();
    jedis.hmset("config", map);
    jedis.hgetAll("config");
    jedis.hincrByFloat("config", "port", 1);
    jedis.hkeys("config");
    jedis.hvals("config");
    jedis.hlen("config");
    hexists("config", "port");
    hdel("config", "weight");
}

@Test
public void operateForSet() {
    jedis.sadd("set", "1", "2", "3");
    jedis.scard("set");
    jedis.smembers("set");
    jedis.sismembers("set", "2");
    jedis.spop("set");
    jedis.srem("set", "2");
}

@Test
public void operateForZset() {
    jedis.zadd("rank", 100.0, "tony");
    jedis.zrem("rank", "tony");
    // 索引
    zremrangeByRank("rank", start, stop);
    zremrangeByScore("rank", min, max);
    jedis.zrangeByScore("rank", min, max);
    zrevrangeByScore("rank", min, max);
    zcount("rank", min, max);
    zscore("rank", "tony");
    zrevrank;
}

jedis.close();
```



###### 12.1 JedisPool

数据库连接的底层是一条Socket通道，创建和销毁很耗时。在数据库连接过程中，为了防止数据库连接的频繁创建、销毁带来的性能损耗，常常会用到连接池（Connection Pool），例如淘宝的Druid连接池、Tomcat的DBCP连接池，C3p0

JedisPool参数：

+ maxTotal：最大连接数，默认8
+ maxIdle：最大空闲连接数，默认8
+ minIdle：最少空闲连接数，默认0（如果JedisPool开启了空闲连接的有效性检测，如果连接空闲且无效则销毁）
+ blockWhenExhausted：当资源池耗尽调用者是否等待，默认true（true时，maxWaitMillis才会生效）
+ maxWaitMillis：资源池耗尽后，调用者最大等待事件
+ testOnBorrow：向资源池借用连接时是否做有效性检测（ping，若连接无效则移除），默认false
+ testOnReture：向资源池归还连接时是否做有效性检测（连接无效则移除），默认false

+ testWhileIdle：创建一个线程对空闲连接做有效性检测

+ timeBetweenEvictionRunsMills
+ minEvictableIdleTimeMills
+ numTestsPerEvictionRun
+ jmxEnabled：是否开启jmx监控，默认true

``` java

JedisConfig config = new JedisConfig();
int max = Runtime.getRuntime().availableProcessors() << 1;
config.setMaxTotal(max);
config.setMaxIdle(max >> 1);
config.setMaxWaitTimeMills(1000 * 10);
config.setTestOnBorrow(true);

new JedisPool(config, "127.0.0.1", 6379);
```



###### 12.2 spring-data-redis

``` xml
<dependency>
	<groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>${springboot-version}</version>
</dependency>

<!-- spring-redis.xml -->
<context:property-placeholder location="classpath:redis.propertites" />

<!-- redis数据源 -->
<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <!--
	最大空闲数
	最大连接数
	...
	-->
</bean>

<!-- spring提供的连接池管理工厂，工厂创建连接，连接创建会话 -->
<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
	<property name="hostName" value="${redis.host}" />
    ...
</bean>

<!-- redisTemplate -->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
	<property name="connectionFactory" ref="jedisConnectionFacotry" />
    <!-- keySerializer -->
    <!-- valueSerializer -->
    <!-- hashKeySerializer -->
    <!-- hashValueSerializer -->
    
    <!-- 开启事务 -->
    <property name="enableTransactionSupport" value="true" />
</bean>

<!-- 可以选择将redisTemplate封装为spring一个普通的service，根据提供更为简便的API -->

```



**封装5大对象的命令API**

+ ValueOperations（字符串类型操作API集合）

`redisTemplate.opsForValue().set...`

+ ListOperations

`redisTemplate.opsForList().`

+ SetOperations

`redisTemplate.opsForSet().`

+ ZSetOperations

`redisTemplate.opsForZSet().`

+ HashOperations

`redisTemplate.opsForHash().`

将setnx —> setIfAbsent，其余操作类似jedis



##### 12.3 Spring缓存抽象

参见spring-srping5doc-缓存



##### 12.4 SpEL

Spring Expression Language

支持如下的表达式：

（1）基本表达式：字面量表达式、关系，逻辑与算术运算表达式、字符串连接及截取表达式、三目运算及Elivis表达式、正则表达式、括号优先级表达式；

（2）类型表达式：类型访问、静态方法/属性访问、实例访问、实例属性值存取、实例属性导航、instanceof、变量定义及引用、赋值表达式、自定义函数等等。

（3）集合相关表达式：内联列表、内联数组、集合，字典访问、列表，字典，数组修改、集合投影、集合选择；不支持多维内联数组初始化；不支持内联字典定义；

（4）其他表达式：模板表达式。



运算符：

（1）算术运算符：SpEL提供了以下算术运算符：加（+）、减（-）、乘（*）、除（/）、求余（%）、幂（^）、求余（MOD）和除（DIV）等算术运算符。MOD与“%”等价，DIV与“/”等价，并且不区分大小写

（2）关系运算符：SpEL提供了以下关系运算符：等于（==）、不等于（!=）、大于（>）、大于等于（>=）、小于（<）、小于等于（<=），区间（between）运算等等

（3）逻辑运算符：SpEL提供了以下逻辑运算符：与（and）、或（or）、非（！或NOT

（4）字符串运算符：SpEL提供了以下字符串运算符：连接（+）和截取（[]）。例如：#{'Hello ' + 'World! '}的结果为“Hello World! ”。#{'HelloWorld! '[0]} 截取第一个字符“H”，目前只支持获取一个字符。

（5）三目运算符

（6）正则表达式匹配符：SpEL提供了字符串的正则表达式匹配符：matches。例如： #{'123' matches '\\d{3}' }

（7）类型访问运算符：SpEL提供了一个类型访问运算符：“T(Type)”,“Type”表示某个Java类型，实际上对应于Java类java.lang.Class实例。“Type”是类的全限定名（包括包名），“java.lang”中的类除外#{T(String).valueOf(1)}，表示将整数1转换成字符串。

（8）变量引用符：SpEL提供了一个上下文变量的引用符“#”，在表达式中使用“#variableName”引用上下文变量。



**==SpEL提供了一个变量定义的上下文接口——EvaluationContext，并且提供了标准的上下文实现——StandardEvaluationContext。通过EvaluationContext接口的setVariable(variableName, value)方法，可以定义“上下文变量”，这些变量在表达式中采用“#variableName”的方式予以引用。在创建变量上下文Context实例时，还可以在构造器参数中设置一个rootObject作为根，可以使用“#root”引用根对象，也可以使用“#this”引用根对象。==**

``` java
main(String[] args) {
    /**
     * SpEL上下文
     */
    public void testSpelContext() {
        ExpressionParser parser = new SpelExpressionParser();
        EvaluationContext context = new StandardEvaluationContext();
        // 缓存注解上下文：CacheEvaluationContext();
        context.setVariable("foo", "bar");
        parser.parseExpression("#foo").getValue(context, String.class);
    }
}
```



##### 12.5 SpEL缓存注解上下文

对应于加在方法上的缓存注解（如@CachePut和 @Cacheable）, spring提供了专门的上下文类CacheEvaluationContext，这个类继承于基础的方法注解上下文MethodBasedEvaluationContext，而这个方法则继承于StandardEvaluationContext（大家熟悉的标准注解上下文）。

``` java

class CacheEvaluationContext extends MethodBasedEvaluationContext {
    // 构造器
    CacheEvaluationContext(Object rootObject,
                           Method method,
                           Object[] arguments,
                           ParameterNameDiscoverer parameterNameDiscoverer) {
        super(rootObject, method, arguments, parameterNameDiscoverer);
    }
}
```

基于构造器所以我们可以使用的属性，root可省略，对于方法参数可以使用p名称空间指定

|   属性名    | 说明                          | 示例                 |
| :---------: | ----------------------------- | -------------------- |
| methodName  | 方法名                        | #root.methodName     |
|   method    | 方法                          | #root.method.name    |
|   target    | 目标对象                      | #root.target         |
| targetClass | 目标对象类                    | #root.targetClass    |
|    Args     | 参数列表                      | #root.args[0]        |
|   Caches    | @Cachable(value={"c1", "c2"}) | #root.caches[0].name |



#### 13 亿级流量高并发架构

+ Netty集群

主要用来负责维持和客户端的TCP连接，完成消息的发送和转发。（**那不就是Spring WebFlux吗**）

+ Zookeeper集群

负责Netty Server集群的管理，包括注册、路由、负载均衡。集群IP注册和节点ID分配。主要在基于ZooKeeper集群提供底层服务。

+ Redis集群

负责用户、用户绑定关系、用户群组关系、用户远程会话等等数据的缓存。缓存其他的配置数据或者临时数据，加快读取速度。

+ Mysql集群

+ SpringCloud WEB服务集群
+ RocketMQ消息队列集群

主要是将优先级不高的操作，从高并发模式转成低并发的模式。例如，可以将离线消息发向消息队列，然后通过低并发的异步任务保存到数据库。



##### 13.1 设计

（1）核心Netty4.x + spring4.x + ZooKeeper 3.x + redis 3.x + rocketMQ 3.x+mysql 5.x+ monggo 3.x

（2）短连接服务：spring cloud基于RESTful短连接的分布式微服务架构，完成用户在线管理、单点登录系统。

（3）长连接服务：Netty主要用来负责维持和客户端的TCP连接，完成消息的发送和转发。

（4）消息队列：rocketMQ高速消队列。

（5）数据库：mysql+mongodb

（6）序列化协议：Protobuf + JSON

Protobuf是最高效的二进制序列化协议，用于长连接。JSON是最紧凑的文本协议，用于短连接。



在高并发IM系统中，存在两类的服务器。一类短连接服务器和一个长连接服务器。

短连接服务器也叫Web服务器，主要功能是实现用户的登录鉴权和拉取好友、群组、数据档案等相对低频的请求操作。

长连接服务器也叫IM即时通信服务器，主要作用就是用来和客户端建立并维持长连接，实现消息的传递和即时的转发。并且，分布式网络非常复杂，长连接管理是重中之重，需要考虑到连接保活、连接检测、自动重连等方方面面的工作。

如果用户规模庞大，无论是短连接Web服务器，还是长连接IM服务器，都需要进行横向的扩展。因此需要引入一个新的角色，短连接Web网关（WebGate）。

WebGate短连接网关的职责，首先是代理大量的Web服务器，从而无感知地实现短连接的高并发。在客户端登录时和进行其他短连接时，不直接连接Web服务器，而是连接Web网关。围绕Web网关和Web高并发的相关技术，目前非常成熟，可以使用SpringCloud或者Dubbo等分布式Web技术，也很容易扩展。



除此之外，大量的IM服务器，又如何协同和管理呢？基于ZooKeeper或者其他的分布式协调中间件，可以非常方便、轻松地实现一个IM服务器集群的管理，包括而且不限于命名服务、服务注册、服务发现、负载均衡等管理。



一般来说，短连接的服务接口都是基于应用层HTTP协议的HTTP API或者RESTful API实现的，通过JSON文本格式返回数据。

**如何在Java服务器端调用其他节点的HTTP API或者RESTful API呢？**

+ JDK原生的URLConnection

+ Apache的HttpClient / HttpComponents

+ Netty的异步HttpClient

+ Spring的RestTemplate

目前用的最多的，基本上是第二种

**首先作一个解释，什么是RESTful API。REST的全称是RepresentationalState Transfer（表征状态转移，也有译成表述性状态转移），它是一种API接口的风格**



但HttpClient不能根据接口发负载去判断调用哪个接口，所以咋们上Open Feign + Ribbon

可以使用Feign来调用多个服务器的同一个接口。Feign不仅可以进行同接口多服务器的负载均衡，一旦使用了Feign作为HTTP API的客户端，调用远程的HTTP接口就会变得像调用本地方法一样简单。