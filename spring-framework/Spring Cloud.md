

### 1 Spring Cloud技术栈

+ 服务注册与发现（Eureka）
+ 服务调用与负载（Ribbon、OpenFeign）
+ 服务熔断降级（Hystrix）
+ 服务网关（Zuul、GateWay）
+ 服务分布式配置（Spring Cloud Config）
+ 服务开发（Spring Boot）



各框架版本：

+ cloud：Hoxton.SR7

+ boot：2.3.2.RELEASE

+ cloud alibaba：2.2.1.RELEASE

+ maven：3.5
+ gradle：6.1



#### cloud 技术升级

服务注册中心：

​	Eureka（停更）

​	Zookeeper + Dubbo（老项目）

​	Consul（go）

​	✔ Nacos（Spring Cloud Alibaba）



服务调用

​	✔ Ribbon

​	✔ LoadBalancer

​	✔ OpenFeign



服务降级：

​	Hsytrix（停更）

​	✔ Sentinel（Spring Cloud Alibaba）



服务网关：

​	Zull / Zuul2

​	✔ Gateway



服务配置

​	Config

​	✔ Nacos



服务总线

​	Bus

​	✔ Nacos



### 2 构建项目

+ 父工程

声明依赖，dependencyManagement

``` groovy
// build.gradle

buildScript {
    allprojects {
        jar {
            enabled = true
        }
    }
}

allprojects {
   group 'com.yangzl'
   version '1.0-SNAPSHOT'
}

subprojects {
    
}

```

