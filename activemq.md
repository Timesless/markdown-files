## 谋定而后动，三思而后行

+ API发送和接收
+ 高可用与容错
+ 持久化
+ ACK
+ 延迟与定时
+ spring整合

**应用解耦，流量削峰，异步任务**

查看进程的方法：

+ ps -ef|grep tomcat | grep -v tomcat

+ netstat -anp | grep 61616

+ lsof -i:61616

带日志启动： ./activemq start > /activemq.log

---

**JMS架构**

``` mermaid
graph LR
A[Connection Factory] --> B[Connection]
B[Connection] --> C[Session]
C[Session] --> |1| MessageProducer
C[Session] --> |2| MessageConsumer
```

**工厂创建连接，连接创建会话**

---

<kbd>DESTINATION <kbd>队列</kbd><kbd>主题</kbd></kbd>

目的地两大模式：

+ 队列Queue： 点对点，无时间相关性，负载均衡模式消息只能被一个消费者消费
+ 主题Topic： 发布订阅，没有消费者消息将被丢弃

说明： Queue数据默认在MQ服务器以文件形式保存，Topic无状态

> 常用API
>
> new ActiveMQConnectionFactory(brokeUrl);
>
> factory.createConnection();
>
> connection.start();
>
> connection.createSession(**tranacted, acknowledge**)；
>
> session.createQueue() | createTopic();
>
> session.createProducer(queue);
>
> session.createConsumer(queue);
>
> producer.send();
>
> **recieve方式** consumer.recieve();
>
> **监听方式**
>
> 创建接口的匿名内部类：
>
> consumer.setMessageListener(new MessageListerner(...))
>
> System.in.read();
>
> producer.close(); session.close(); factory.close();

---

**JAVAEE 13个核心规范**

**JDBC, JNDI, EJB, RMI , Java IDL, JSP, Servlet, XML, JMS, JTA(Java Transaction API), JTS, Java Mail, JAF**

Java命名和目录接口， Java事务服务，可扩展标记语言

> **JMS规范与落地**
>
> + 消息头：Destination，DeliveryMode，Expiration，Priority，**MessageID**
> + 属性： setStringProperty("k", "v")
> + 消息体：**TextMessage，MapMessage**，BytesMessage，StreamMessage，ObjectMessage
>
> **JMS可靠性**
>
> - **持久化**
>
> 队列方式：producer.setDeliveryMode(DeliveryMode.NON_PRESISTENT | PRESISTENT)
>
> Topic方式： 需先设置**持久化订阅**
>
> ​	connection.setClientID("z3");
>
> ​	session.createDurableSubscriber(top, "remark ..");
>
> ​	connection start();
>
> + **事务**
>     + 生产者事务
>     + 消费者事务，commit()
> + **ACK **acknowledge
>     + AUTO_ACK
>     + CLIENT_ACK
>     + DUPS_OK_ACK，允许重复
>
> 事务与ACK同时开启：按事务提交，会自动ACK，ACK但不提交事务，会产生重复消费。**以事务为准**

---

**Broker**

嵌入式activeMQ实例

``` java
BrokerServier broker = new BrokerService();
broker.setuseJmx(true);
broker.addConnector("tcp://localhost:61616");
broker.start();
// 引入jackson—bind依赖
```

**集群搭建**

``` xml
<presistenceAdapter>
	<repilcatedLevelDB
          directory="${activemq.data}/leveldb"
          replicas = "3"
          bind="tcp://0.0.0.0:61616"
          zkAddress="localhost:2181,localhost:2182,localhost:2183"
          hostname="localhost"
          sync="local_disk"
          zkPath="/activemq/leveldb-stores "/>
</presistenceAdapter>
```

