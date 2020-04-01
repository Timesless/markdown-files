> **数据CAP**
>
> Consistency（强一致性）
>
> Availability（高可用性）
>
> Partition Tolerance（分区容错性）

``` yaml
# web actuator暴露监控端点
management: 
	endpoints:
		web:
			exposure:
				include: "*"
```



### 注册中心

#### eureka

> eureka server
>
> Eureka保护模式（默认开启）：EurekaServer在一定时间内（默认90s）没有收到某个微服务的心跳包将会注销该微服务。在保护模式下EurekaServer不会注销任何微服务 **AP**
>
> eureka client

``` properties
# 依赖
org.springframework.cloud
spring-clould-starter-netflix-eureka-server
spring-clould-starter-netflix-eureka-client

# 是否注册
register-with-eureka: false
# 订阅服务
fetch-registry: false
# 保护模式 AP
enable-self-preservation: false

# 相关注解
@EnableEurekaServer
@EnbaleEurekaClient
@EnableDiscoveryClient
```

#### zookeeper

> 服务创建为临时节点：**CP**

``` properties
spring-cloud-starter-zookeeper-discovery
```

#### Consul

> **CP**

``` properties
# maven
spring-cloud-starter-consul-discovery
```



### 服务调用

#### Ribbon

> **客户端负载均衡**
>
> Ribbon = 负载均衡 + RestTemplate

``` java
// RestTemplate
@Configuration
@Bean
@LoadBalanced
public RestTemplate getRestTemplate() {
    return new RestTemplate();
}
// 获取更详细信息使用xxxforEntity返回ResponseEntity对象

// Ribbon策略 可替换，定义规则不能放在@ComponentScan扫包文件夹下
@RibbonClient

interface IRule {}
// RoundRobinRule 请求次数取模
// RandomRule
// RetryRule
// WeightedResponseTimeRule
// BestAvailableRule
// AvailabilityFilteringRule
// ZoneAvoidanceRule

```

#### OpenFeign

> Feign集成Ribbon
>
> 我们只需要创建微服务接口并使用注解的方式来配置它
>
> **Ribbon + RestTemplate调用，Feign通过接口 + 注解@FeignClient调用**

``` properties
# maven
spring-cloud-starter-openfeign

# 相关注解
@EnableFeignClient
@FeignClient

# 超时控制（默认等待1s），日志打印
ribbon.ReadTimeout = 5000
	  .ConnectTimeout
logging.level
```

### 服务降级

#### Hystrix

> **扇出：**服务A调用B,C服务，B,C调用其它服务
>
> 处理**分布式系统延迟和容错**，当服务超时或失败，避免级联故障
>
> **服务降级(fallback) -> 服务熔断(break) -> 服务限流(flow limit)**

``` properties
# maven 已停更
spring-cloud-starter-netflix-hystrix

# 相关注解
```

#### Sentinel

### 网关

#### Zuul / Zuul2

#### Gateway

> 基于Spring 5.0 + Spring Boot2.x + WebFlux(Reactor模式框架Netty)
>
> 路由 + 断言 + 过滤链

``` properties
# maven
org.spring.framework
spirng-cloud-starter-gateway

# 默认过滤规则
Path Route Predicate
Before Route Predicate
Between Route Predicate
After Route Predicate
Cookie Route Predicate
Header Route Predicate
Host Route Predicate
Method Route Predicate
Query Route Predicate

# 过滤器链
# pre | post
# global
```

``` java
// curl
curl http:localhost:8080
    
// 自定义过滤器记录日志
GateayFilter implements GlobalFilter, Orderd {
    MoNo<Void> filter(ServerWebExchange change, GatewayFilterChain chain) {
        log.info(...);
        if (logic) {
            // 放行
            chain.filter(exchange);
        } else {
            log.info("错误");
            HttpResponse resp = exchange.getResponse();
            resp.setStatusCode(HttpStatus.NOT_FOUND);
            return resp.setComplete();
        }
    }
    
    int orderd() {
        return 0;
    }
}
```

### 服务配置

> 1. config + bus
>
> 2. nacos

#### SpringCloud Config

> 分布式服务配置中心
>
> 服务端（连接github） + 客户端（连接服务端）
>

``` properties
# 客户端 服务端maven
spring-cloud-config
spring-cloud-config-server

# 注解
@EnableConfigServer

# 分支，服务名，环境
lable, name, profiles

# 客户端刷新配置
@RefreshScope
```

#### SpringCloud Bus

> 传递分布式系统中系统间的消息
>
> 构建一个 **共用的消息主题**，让所有微服务实例连接，产生的**消息会被所有实例监听和消费**
>
> 1. 通知某一个客户端
> 2. 通知服务端

``` properties
# 安装 Rabbit MQ
# exchange == topic
# 可视化插件
rabbitmq-plugins enable rabbitmq_management
# web端口15672，服务端口5672 guest guest
rabbitmq:
	host: localhost
	port: 5672
	username: guest
	password: guest

# maven
spring-cloud-stater-bus-amqp
```

### SpringCloud Stream

> 消息驱动
>
> 屏蔽消息中间件差异，统一消息编程模型

``` properties
# 注解
# @Input @Output @StreamListener @EnableBinding
@EnableBinding(Source.class)
@EnbaleBinding(Sink.class)

# producer
Soucre -> MessageChannel -> Binder
#consumer
Binder -> MessageChannel -> Sink

# cloud stream配置
bindings:
	input:
		destination: testExchange
		content-type: application/json
		binder: defaultRabbit
		# 分组解决重复消费
		group: groupA
```

#### SpringCloud sleuth

> 服务跟踪，服务链路调用图
>
> zipkin展示web项目 9411

``` properties
# maven
spring-cloud-starter-zipkin

# 运行zipkin jar包
```

## SpringCloud Alibaba

> 

#### Nacos

> AP | CP
>
> Nacos = Eureka + SpringCloud Config + Bus

``` properties
# maven
spring-cloud-starter-alibaba-nacos-config
spring-cloud-starter-alibaba-nacos-discovery
```

