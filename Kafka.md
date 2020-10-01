# 深入理解Kafka：核心设计与实践原理





#### 2.1 客户端开发

步骤：

+ 配置生产者客户端参数及创建生产者实例
+ 构建消息
+ 发送消息
+ 关闭实例

构建的消息对象 ProducerRecord，包含了多个属性，原本需要发送的与业务相关的消息体只是其中的一个 value 属性，比如“Hello，Kafka！”只是ProducerRecord对象中的一个属性。ProducerRecord类的定义如下（只截取成员变量）

``` java
public class ProducerRecord<K, V> {
    private final String topic;	// 主题
    private final Integer parition;	// 分区号
    private final Headers headers;	// 消息头
    private final K key;
    private final V value;
    // CreateTime / LogAppendTime（追加到日志文件的时间）
    private final Long timestamp;	// 消息时间戳
}
```



**参数`ProducerConfig`**

``` java
static Propertites initConfig() {
    new Propertites();
    props.put(ProducerConfig.BOOTSTRAP_SERVER_CONFIG, brokerList);
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
              StringSerializer.class.getName());
    props.put(VALUE_, );
    return props;
}

// 其它方式
new KafkaProducer<>(props, new StringSerializer(), 
                          new StringSerializer());
```