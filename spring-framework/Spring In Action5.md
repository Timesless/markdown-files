

### 1 Spring 基础

#### 1.2 Spring 项目结构

+ mvnw和mvnw.cmd：这是Maven包装器（wrapper）脚本。借助这些脚本，即便你的机器上没有安装Maven，也可以构建项目。
+ pom.xml：这是Maven构建规范，随后我们将会深入介绍该文件。
+ TacoCloudApplication.java：这是Spring Boot主类，它会启动该项目。随后，我们会详细介绍这个类。
+ application.properties：这个文件起初是空的，但是它为我们提供了指定配置属性的地方。在本章中，我们会稍微修改一下这个文件，但是我会将配置属性的详细阐述放到第5章。
+ static：在这个文件夹下，你可以存放任意为浏览器提供服务的静态内容（图片、样式表、JavaScript等），该文件夹初始为空。
+ templates：这个文件夹中存放用来渲染内容到浏览器的模板文件。这个文件夹初始是空的，不过我们很快就会往里面添加Thymeleaf模板。
+ TacoCloudApplicationTests.java：这是一个简单的测试类，它能确保Spring应用上下文可以成功加载。在开发应用的过程中，我们会将更多的测试添加进来。



``` xml
<modelVersion>4.0.0</modelVersion>
<groupId>com.yangzl</group>
<artifactId>taco-cloud</artifactId>
<version>1.0-SNAPSHOT</version>
<packaging>jar</packaging>
<name>taco-cloud</name>

<parent>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.2.RELEASE</version>
    <!-- lookup parent from repository -->
    <relativePath />
</parent>

<propertities></propertities>

<dependencies>
	<dependency>
    	<gorupId>org.springframework.boot</gorupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

+ 请留意<parent>元素，更具体来说是它的<version>子元素。这表明我们的项目要==以spring-boot-starter-parent作为其父POM。除了其他的一些功能之外，这个父POM为Spring项目常用的一些库提供了依赖管理==，现在你不需要指定它们的版本，因为这是通过父POM来管理的。使用Spring Boot 2.3.2定义来继承依赖管理。

+ SpringBoot starter依赖的特别之处在于它们本身并不包含库代码，而是传递性地拉取其他的库。这种starter依赖主要有3个好处。
    + 构建文件会显著减小并且更易于管理
    + 我们能够根据它们所提供的功能来思考依赖，而不是根据库的名称。如果是开发Web应用，那么你只需要添加web starter就可以
    + 我们不必再担心库版本的问题，**你可以永远相信Spring**



#### 1.3 初始化Spring应用

``` java
@SpringBootApplication
main(String[] args) {
    SpringApplication.run(AppName.class, args);
}
```

@SpringBootApplication是一个组合注解

+ @SpringBootConfiguration：将该类声明为配置类。我们可以按需添加基于Java的Spring框架配置。这个注解实际上是@Configuration注解的特殊形式。
+ @EnableAutoConfiguration：启用Spring Boot的自动配置。这个注解会告诉Spring Boot自动配置它认为我们会用到的组件。
+ @ComponentScan：启用组件扫描。这样我们能够通过像@Component、@Controller、@Service这样的注解声明其他类，Spring会自动发现它们并将它们注册为Spring应用上下文中的组件。



##### 1.3.2 静态文件

图片是使用相对于上下文的“/images/TacoCloud.png”路径来进行引用的。回忆一下我们的项目结构，像图片这样的静态资源是放到“/src/main/resources/static”文件夹中的。这意味着，在项目中，TacoCloud Logo图片必须要位于“/src/main/resources/static/images/TacoCloud.png”。



##### 1.3.3 测试Mvc

``` java
@Runwith(SpringRunner.class)
@WebMvcTest(xxxController.class)
class xxxControllerTest {
    @Autowried
    private MockMvc mock;
    
    @Test
    public void testPage throws Exception {
        mock.perform(get('/path'))
            .addExpect(status().isOk())
    }
}
```



##### 1.3.5 Spring Boot DevTools

+ 变更代码自动重启
+ 面向浏览器资源（模板，JavaScript，样式表）发生变化自动刷新浏览器
+ 自动禁用模板缓存



### 2 开发Web应用



### 3 Spring Data

Spring Data是一个非常大的伞形项目，由多个子项目组成，较流行的Spring Data项目包括：

+ Spring Data JPA： 基于关系型数据库进行JPA持久化
+ Spring Data MongDB：持久化到Mongo文档数据库
+ Spring Data Neo4j：持久化到Neo4j图数据库
+ Spring Data Redis：持久化到Redis key-value存储
+ Spring Data Cassandra：持久化到Cassandra数据库



启用Spring Data JPA

``` xml
<dependency>
	<gorupId>org.springframework.boot</gorupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```



### 4 Spring Security

启用Spirng Security

``` xml
<dependency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```



security starter有如下安全特性

+ 所有HTTP请求路径都需要认证
+ 不需要特定的角色和权限
+ 无登录页面
+ 认证过程是通过HTTP basic认证对话框实现的
+ 系统只有一个用户，user
+ 密码随机，记录在日志文件中



但我们想做的至少应该有如下功能

+ 通过登录页面提示用户认证，而不是HTTP basic对话框
+ 提供多个用户
+ 对不同路径执行不同的安全策略（如登录注册不需认证）



#### 4.2 配置Spring Security

+ XML
+ Java注解



**Spring Security为用户存储提供多个方案**

+ 基于内存的用户存储
+ 基于JDBC
+ 以LDAP作为后端
+ 自定义用户详情服务

不管使用哪种用户存储，都可以通过覆盖`WebSecurityConfigureAdapter`中configure()方法来进行配置

``` java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigureAdapter {
    
    @Override
    protected void configure(AuthenticationManagerbUilder auth) throws Exception {
        /**
         * 1 基于内存，使用builder风格构建认证细节
         */
        auth.inMemoryAuthentication()
            .withUser("buzz")
            .password("buzzz")
            .authorities("ROLE_USER")
            .and()
            .withUser("fizz")
            .password("fizz")
            .authorities("ROLE_USER");
        
        /**
         * 2.1 基于JDBC的用户存储，默认方式
         */
        @Autowired
        private DataSource dataSource;
        auth.jdbcAuthentication().dataSource(dataSource);
        
        // 这是Security内部执行的SQL查询，在有必要时时可重写
        psfs DEF_USERS_BY_USERNAME_QUERY =
            "select username, password, enabled from users where username = ?";
        psfs DEF_AUTHORITIES_BY_USERNAME_QUERY = 
            "select username, authority from authorities where username = ?“;
        psfs DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY = 
            “select g.id, g.group_name, ga.authority from group g, group_members gm, group_authorities ga where gm.username = ? and g.id = ga.group_id and g.id = gm.group_id”;
            
        /**
         * 2.2 重写查询SQL
         * 这里我们只重写了认证和基本权限的查询语句, groupAuthoritiesByUsername()
         */
        auth.jdbcAuthentication().dataSource(dataSource)
            .usersByusernameQuery(userSql)
            .authoritiesByUsernameQuery(authSql)
            .passwordEncoder(new StandardPassswordEncoder("123456"));
        
        /**
         * 3.1 以LDAP作为后端的用户存储，默认配置
         */
        auth.ldapAuthentication()
            .userSearchFilter("(uid={0})")
            .gorupSearchFilter("member={0}");
        /**
         * 3.2 覆盖默认配置
         */
        
    }
}

/**
 * 验证的加密方式
 * BCryptPasswordEncoder：使用bcrypt强哈希加密
 * NoOpPasswordEncoder：不进行任何加密
 * Pbkdf2PasswordEncoder：PBKDF2加密
 * ScyrptPasswordEncoder： scrypt哈希加密
 * StandardPasswordEncoder: SHA-256哈希加密
 * 自定义加密如下
 * 不管使用何种加密方式，都不会执行解密。只会对比
 */
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);
    boolean matches(CharSequence rawPassword, String encodePassword);
}
```



#### 4.3 CSRF

Srping Security提供内置的CSRF保护，默认启用。只需要在每个表单添加"_csrf"的隐藏字段

``` html
<input type = "hidden" name="_csrf" value="${_csrf.token }" />
```



#### 总结

+ Spring Security的自动配置是实现基本安全性功能的好办法，但是大多数的应用都需要显式的安全配置，这样才能满足特定的安全需求。

+ 用户详情可以通过用户存储进行管理，它的后端可以是关系型数据库、LDAP或完全自定义实现。

+ Spring Security会自动防范CSRF攻击。

+ 已认证用户的信息可以通过SecurityContext对象（该对象可由SecurityContextHolder.getContext()返回）来获取，也可以借助@AuthenticationPrincipal注解将其注入到控制器中。



### 5 *配置属性

==配置属性只是Spring应用上下文中bean的属性而已，它们可以通过多个源进行设置，包括JVM系统属性、命令行参数以及环境变量。==

+ 自动配置bean
+ 将配置属性应用到组件上
+ 使用Spring profile



#### 5.1 自动配置

Spring两种配置：

+ bean装配
+ 属性注入



@Bean注解一般在初始化bean的同时立即为它的属性设置值，如：

``` java
@Bean
public DataSource dataSource() {
    return new EmbeddedDataSourceBuilder()
        .setType(H2)
        .addScript("taco_schema.sql")
        .addScripts("user_data.sql", "ingredient_data.sql")
        .build();
}
```

Spring Boot自动配置解决方法：

==如果在运行时类路径上找到H2依赖，那么Spring Boot会自动在Spring应用上下文创建对应的DataSource bean，且这个bean默认会运行schema.sql、data.sql脚本==



##### 5.1.1  环境抽象

Spring环境会拉去多个属性源，包括：

+ JVM系统属性
+ OS环境变量
+ 命令行参数（-D参数）
+ 应用属性配置文件（yml / properties）

==Spring将这些属性聚合到一个源中，通过该源注入到Spring的bean中（“Spring环境”）。==

举个栗子：

``` properties
# 一下几种方式等价
server.port = 9090

java -jar tacloud.jar --server.prot=9090

$ export SERVER_PORT=9090
```



##### 5.1.2 配置数据源

``` yaml
spring:
  datasource: 
  	url: jdbc:mysql://localhost/tacocloud
  	username: root
  	password: 123456
  	# 可以无需指定jdbc驱动类
  	driver-class-name: com.mysql.cj.jdbc.Driver
  	
  	# 指定sql脚本
  	schema:
  	  - init.sql
  	data:
  	  - data.sql
  	  
   # 如果无法使用显示数据源配置，而是用JNDI
   jndi-name: java:/comp/env/jdbc/tacoCloudDS
```

Spring Boot在自动化配置DataSource bean时，如果存在Tomcat的JDBC连接池，那么会自动使用该连接池

``` xml
# tomcat数据源配置，name表示JNDI名称
<Resource name="jdbc/tacoCloudDS"
          auth="Container"
          type="javax.sql.DataSource"
          maxActive="100" maxIdle="30" maxWait="60" wait_timeout="18800"
          timeBetweenEvictionRunsMillis="300000"
          minEvictableIdleTimeMillis="600000"
          username="root" password="jdzxdb"
          driverClassName="com.mysql.jdbc.Driver"
          url="jdbc:mysql://localhost:3306/test?					     comautoReconnect=true&failOverReadOnly=false" 
          removeAbandoned="true"
          removeAbandonedTimeout="60"
          logAbandoned="true" />
```

否则在类路径下尝试其他连接池

+ HikariCP
+ Commons DBCP 2



##### 5.1.3 HTTPS嵌入式服务器

让服务器处理HTTPS请求，我们首先使用keytool命令行工具生成keystore：

`$ keytool -keystore mykeys.jks -alias tomcat -keyalg RSA`

``` yaml
server:
  port: 8443
  ssl:
  	# 如果要包含在jar包中，需使用classpath: URL来引用
  	key-store: file:// URL
  	key-store-password: 123456
  	ke-password: 123456
```



##### 5.1.4 日志配置

==默认情况下，Spring Boot通过Logback，以INFO级别写入控制台。==

如需修改配置我们只需要在src/main/resource下新建logback.xml，如下：

``` xml
<configuration>
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    	<encoder>
        	<pattern>
            	%d {HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
            </pattern>
        </encoder>
    </appender>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    
    </appender>
    <logger name="root" level="INFO" />
    <root level="INFO">
    	<appender-ref ref="STDOUT" />
    </root>
</configuration>
```



如果仅仅是修改日志级别和输出文件等配置，我们只需要配置yaml即可：

``` yaml
logging:
  path: /var/logs/
  # 默认文件达到10MB会轮换
  file: tacocloud.log
  level:
  	root: WARN
  	org.springframework.security: DEBUG
  	com.yangzl.core: INFO
```



##### 5.1.5 使用属性值

``` yaml
gretting:
  welcome: hello ${spring.application.name}
```



#### 5.2 自定义配置属性

Spring Boot提供@ConfigurationProperties，bean会根据Spring环境注入属性值

@Value：只接受字符串类型的属性值

@ConfigurationProperties：支持复杂结构的属性值（字段与属性一一对应）

``` java
@ConfigurationProperties(prefix="taco.orders")
public class OrderController {
    private int pageSize = 20;
    private void setPageSize(int pageSize) {
        this.pageSize = pageSize;
    }
    ...;
}

// yaml
taco:
  orders:
	pageSize: 10
```



==**如果要在生产环境中快速更改，我们可以将属性设置为系统环境变量，这样无需重新构建和部署应用**==

`$ export TACO_ORDERS_PAGESIZE=10`

对以上代码，我们可以将@ConfigurationProperties注解的类单独提取为配置类，如：

``` java
@Setter
@Component
@Validated
@ConfigurationProperties(prefix="taco.orders")
public class TacoProps {
    // Hibernate Validator默认被包含在Spring Boot依赖中
    @Min(value=5, message="between 5 and 25")
    @Max(value=25, message="between 5 and 25")
    private int pageSize;
}

// 在需要配置的类中注入即可
class OrderController {
    private OrderProps props;
    public void setOrderProps(OrderProps props) {
        this.props = props;
    }
    ...;
}
```



##### 5.2.2 声明属性元信息

在IDE中配置属性元信息，防止IDE黄色警告信息，并且IDE能自动补全

在META-INF下创建`additional-spring-configuration-metadata.json`



#### 5.3 profile

``` yaml
spring:
  # 取值dev,test,prod
  # 对应配置application-dev.yaml, *-test.yaml, *-prod.yaml
  profiles: prod
  
  # 可以通过3个中划线区分配置文件为多部份
--- 
spring:
  profile:
  # 1 激活profile
  	active: prod

# 2 shell 方式
$ export SPIRNG_PROFILES_ACTIVE=prod

# 3 命令行参数
java -jar tacocloud.jar --spring.profiles.active=prod
java -jar tacocloud.jar -Dspring.profiles.active=prod
```



##### 5.3.3 profile条件化bean

``` java
@Bean
// dev和qa环境都创建该bean
@Profile({"dev", "qa"})
@Profile("!prod")
public CommandLineRunner commandLineRunner() {}

// 我们还可以在@Configuration注解的类上使用@Profile
```



## 第2部分 Spring集成

涵盖Spring应用与其它应用集成的话题

+ 异步通信技术Spring发送和接收JMS（Java Message Service）、RabbitMQ、Kafka的消息

+ REST API

+ Spring Integration集成



### 6 REST服务

+ Spring MVC定义REST
+ 启用超链接REST资源

+ 自动化基于repository的REST端点



#### 6.1 RESTful控制器

> 是否要采用SPA？
>
> MPA（MultiPage Applcation），SPA（Single-Page Application）
>
> SPA提供前后端解耦，使后端功能适用多个用户界面（如移动应用），而MPA更是和简单的Web



+ GetMapping
+ PostMapping，创建资源
+ DeleteMapping
+ PutMapping，更新资源
+ PatchMapping，更新资源
+ RequestMapping



##### 6.1.1 获取数据

``` java

@GetMapping("/{id}")
public Taco tacoById(@PathVariable("id") String id) {
    ...;
    
    /**
     * 使用ResponseEntity
     * ResponseEntity，带有状态码信息，和数据
     */
    Optional<Taco> opTaco = tacoService.findById(id);
    if (opTaco.isPresent()) {
        return new ResponseEntity<>(opTaco.get(), HttpStatus.OK);
    }
    return new ResponseEntity<>(null, HttpStatus.NOT_FOUND);
}

// curl测试
curl localhost:8080/design/recent
```



##### 6.1.2 发送数据

``` java
/**
 * 没有指定path则使用类上路径
 * 正常运行情况下：
 * @ResponseStatus：响应201状态码（不仅请求成功，还创建了一个资源），200信息不足
 */
@PostMapping(consumes=MediaType.APPLICATION_JSON_VALUE)
@ResponseStatus(HttpStatus.CREATED)
public Taco postTaco(@RequestBody Taco taco) {
    return tacoService.save(taco);
}
```



##### 6.1.3 更新数据

为什么会有两种不同的HTTP方法来更新资源？

尽管PUT经常被用来更新资源，但它在语义上其实是GET的对立面。GET请求用来从服务端往客户端传输数据，而PUT请求则是从客户端往服务端发送数据。从这个意义上讲，PUT真正的目的是执行大规模的替换（replacement）操作，而不是更新操作。HTTP PATCH的目的是对资源数据打补丁或局部更新。

举个栗子：

`@PutMapping tacoService.save(order)`时，**==它可能需要客户端将完整的点歌单数据从PUT请求提交到服务器，从语义上讲，如果该订单省略了某个属性，那么该值会被null所覆盖==**



而HTTP PATCH可以做局部更新，其实是自己编写代码控制逻辑

``` java
@PatchMapping(path="/{orderId}", consumes="application/json")
public Order patchOrder(@PathVariable("orderId")String orderId, @RequestBody Order patch) {
    Order order = orderService.findById(orderId).get();
    if (patch.getName() != null) {
        order.setName(patch.getName());
    }
    if (patch.getStreet() != null) {
        order.setStreet(patch.getStreet());
    }
    return orderService.save(order);
}
```



##### 6.1.4 删除数据

``` java
@DeleteMapping("/{orderId}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deleteOrder(@PathVariable("orderId")String orderId) {
    orderService.deleteById(orderId);
    // 如果资源不存在则也可以通过ResponseEntity返回更多信息
}
```



#### 6.2 启用超媒体

目前为止，我们创建的API消费端只需知道URL，就可以正常运行。但URL经常变化的时候会怎样呢？

硬编码的客户端掌握的旧API的信息，因此客户端无法正常运行。

==**超媒体作为应用状态引擎（Hypermedia as the Engine of Application State， HATEOAS）是一种创建自描述API的方式（API将描述自己的URL）**==。API返回的资源中会包含相关资源的连接，客户端只需连接最少的API URL就可以导航整个API

``` json
{
    "_embedded": {
        "tacoResourcelist": [
            {
                "name": 'Egg',
                "ingredients": [
                    {
                        "name": "water",
                    // 导航API的超链接
                        "_links": {
                            "self": {"href": "http://localhost:8080/ingre"}
                        }
                	}, {
                        // 其它配料
                    }
                ]
            }
        ]
    },
    // 引用自身API
    "_links": {
        "recents": {
            "href": "http://localhost:8080/design/recent"
        }
    }
}
```

==这种特殊风格HATEOAS被称为HAL（超文本应用语言，Hypertext Application Language），是一种在JSON响应中嵌入URL的简单通用格式==



Spring HATEOAS项目为Spring提供超链接支持，它提供一些类和资源装配器（assembler），在Spring MVC返回之前为其添加URL



**启用超媒体功能**

``` xml
<dependency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```



##### 6.2.1 添加超链接

``` java
@GetMapping("/recent")
public Resource<Resource<Taco>> recentTacos() {
    Resource<Resource<Taco> resources = Resources.wrap(tacos)
    resources.add(new Link("http://localhost:8080/design/recents", "recents"));
    // 当然我们需要单独处理Taco为Resource<Taco>
    return resources;
    
    /**
     * 构建者模式 ControllerLinkBuilder
     * 1. 指定控制器即可
     * 2. 指定控制器的方法，会使用方法的映射路径
     */
    resources.add(ControllerLinkBuilder.linkTo(TacoController.class)
                 .slash("recent")
                 .withRel("recents"));
    
    resources.add(
        ControllerLinkBuilder.linkTo(
            ControllerLinkBuilder.methodOn(TacoController.class)
                            .recentTacos()).withRel("recents"));
}

```



**为每个Taco包装为Resource**

``` java
@Getter
public class TacoResource extends ResourceSupport {
    private final String name;
    private final List<Ingerdient> ingredients;
    
    public TacoResource(Taco taco) {
        this.name = taco.getName();
        this.ingredients = taco.getIngredients();
    }
}
```

> **注意：**
>
> “领域” 和 “资源” 应该各自独立还是同一个类呢？
>
> 有些人将领域和资源对象合二为一，有些人单独创建
>
> 在某些场景下资源链接用不到，且需要暴露ID属性，所以这里选择单独创建资源类



#### 6.3 Spring Data REST

Spring Data有特殊魔法，它能够基于我们定义的接口自动创建repository实现，且它还有另一项技巧，能帮助我们定义应用的API



Spring Data REST会为Spring Data创建的repository自动生成REST API，只需将Spring Data REST添加到构建文件中

``` xml
<dependency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```



**定义基础路径**

``` yaml
spring:
  data:
    rest:
      base-path: /api
      
# tacos调用，自动计算复数形式为tacoes
$ curl http://localhost:8080/api/tacoes
```



**修改映射路径**

``` java
@RestResource(rel="tacos", path="tacos")
public class Taco {}

// 调用
$ curl http://localhost:8080/api/tacos
```



##### 6.3.2 分页和排序

Spring Data REST，提供可选的page，size，sort参数。默认size20

``` shell
$ curl localhost:8080/api/tacos?size=5
$ curl localhost:8080/api/tacos?size=5&page=1
$ curl localhost:8080/api/tacos?sort=id,desc&page=0&size=10
```



##### 自定义REST端点

``` java
@RepositoryRestController
public class RecentTacoController {
    // 返回JSON + HAL格式
    @GetMappling(path="/tacos/recent", produces="application/hal+json")
    ...;
}
```



#### 总结

+ REST端点可以通过Spring MVC来创建，这里的控制器与面向浏览器的控制器遵循相同的编程模型。

+ 为了绕过视图和模型的逻辑并将数据直接写入响应体中，控制器处理方法既可以添加@ResponseBody注解也可以返回ResponseEntity对象。

+ @RestController注解简化了REST控制器，使用它的话，处理器方法中就不需要添加@ResponseBody注解了。

+ Spring HATEOAS为Spring MVC控制器返回的资源启用了超链接功能。

+ 借助Spring Data REST，Spring Data repository可以自动导出为RESTAPI。





### 7 消费REST服务

+ RestTemplate消费REST API
+ Traverson导航超媒体API



Spring应用可以采用多种方式来消费REST API，包括以下几种方式

+ RestTemplate：Spring核心框架提供的简单、同步REST客户端
+ Traverson：Spring HATEOAS提供的支持超链接、同步的REST客户端，其灵感来源于同名的JavaScript库。
+ WebClient：Spring 5所引入的反应式、异步REST客户端。



#### 7.1 RestTemplate

使用RestTemplate消费REST API（端点）

从客户端的角度来看，与REST资源进行交互涉及很多工作，而且大多数都是很单调乏味的样板式代码。如果使用较低层级HTTP库，客户端就需要创建一个客户端实例和请求对象、执行请求、解析响应、将响应映射为领域对象，并且还要处理这个过程中可能会抛出的所有异常



RestTempalte提供41个与REST资源交互的方法，我们考虑12个独立的操作，重载构成41个方法

| 方法            | 描述                                                      |
| --------------- | --------------------------------------------------------- |
| delete          | 对URL上的资源执行HTTP DELETE操作                          |
| exchange        | 在URL上执行特定HTTP方法，返回包含对象的ResponseEntity     |
| execute         | 在URL上执行特定HTTP方法，，返回一个从响应体映射得到的对象 |
| getForEntity    | 发送HTTP GET请求，返回包含响应对象的ResponseEntity        |
| getForObject    | 发送HTTP GET请求，返回响应对象                            |
| headForHeaders  | 发送HTTP HEAD请求，返回包含特定资源URL的HTTP头信息        |
| optionsForAllow | 发送HTTP OPTIONS请求，返回特定URL的Allow头信息            |
| patchForObject  | 发送HTTP PATCH请求，返回响应对象                          |
| postForEntity   | POST数据到一个URL，返回包含响应对象的ResponseEntity       |
| postForLocation | POST数据到一个URL，返回创建资源的URL                      |
| postForObject   | POST数据到一个URL，返回响应的对象                         |
| put             | PUT资源到特定的URL                                        |

除了TRACE之外，RestTemplate对每种标准HTTP方法提供了至少一个方法，除此之外，**execute()和exchange()提供了层次较低的通用方法，以便进行任意HTTP操作**



表中大多数操作以3种方法进行重载：

+ 使用String作为URL格式，并提供可变参数列表指定URL参数
+ 使用String作为URL格式，并提供Map<String, String>止明URL参数
+ 使用java.net.URI作为URL格式，不支持参数化URL



**创建实例**

``` java
// 1
RestTemplate rest = new RestTemplate();

// 2
@Bean
public RestTempalte restTemplate() {
    return new RestTemplate();
}
```



##### 7.1.1 GET

``` java
public Ingredient getIngredient(String id) {
    
    // 可变参数方式
    return rest.getForObject("http://localhost:8080/ingredients/{id}",
                            Ingredient.class, id);
    
    // map参数方式
    Map<String, String> param = new HashMap<>(2);
    pram.put("id", id);
    rest.getForObject("localhost:8080/ingredients/{id}",
                     Ingredient.class, param);
    
    // URI方式
    Map<String, String> urlParam = new HashMap<>(2);
    urlParam.put("id", id);
    URI uri = UriComponentsBuilder
        .fromHttpUrl("http://localhost:8080/ingredients/{id}")
        .build(urlParam);
    rest.getForObject(url, Ingredients.class);
    
    /**
     * getForEntity
     * 不是响应载荷的领域对象，而是包裹领域对象的ResponseEntity对象，可以访问响应细节信息
     * 请求： 请求行（POST /index.html HTTP/1.1 ） + 请求头(各种请求头部信息) + 请求实体(name=123&pwd=123)
     * 响应： 响应行(HTTP/1.1 200 OK) + 响应头 + 响应实体
     */
    rest.getForEntity("localhost:8080/ingredients/{id}", id, Ingredients.class);
    entity.getHeader().getDate();
    return entity.getBody();
}
```



##### 7.1.2 PUT

``` java
public void updateIngredient(Ingredient ingre) {
    rest.put("localhost:8080/ingredients/{id}", ingre, ingre.getId())
}
```



##### 7.1.3 DELETE

``` java
public void deleteIngredient(Ingredient ingre) {
    rest.delete("localhost:8080/ingredients/{id}", ingre.getId());
    
    // 也适用其它重载
}
```



##### 7.1.4 POST

``` java
Public Ingredient createIngredient(Ingredient ingre) {
    rest.postForObject("localhost:8080/ingredients", ingre, Ingredient.class)
}

// postForLocation
public URI createIngredient(Ingredient ingre) {
    rest.postForLocation("localhost:8080/ingredients", ingre);
}

// 如果同时需要响应信息和载荷实体，那么使用postForEntity
public Ingredient createIngredient(Ingredient ingre) {
    entity = rest.postForEntity("localhost:8080/ingredients", ingredient, Ingredient.class);
    entity.getHeaders().getLocation();
    return entity.getBody();
}
```



如果你所消费的API在响应中包含了超链接，那么RestTemplate处理起来比较复杂，我们可以考虑Traverson



#### 7.2 Traverson 导航 REST API

Traverson来源于Spring Data HATEOAS项目

``` java
Traverson travers = new Traverson(
    URL.create("http://localhost:8080/api"), MediaTypes.HAL_JSON);
```

这里我们将Traverson指向Taco Cloud的基础URL



### 8 异步消息

本章内容：

+ 异步化消息
+ JMS、RabbitMQ、Kafka
+ 从代理拉取消息
+ 监听消息

**==Spring提供的3种异步消息方案：==**

+ Java消息服务（Java MessageService，JMS）
+ RabbitMQ和高级消息队列协议（Advanced MessageQueueing Protocol，AMQP）
+ Apache Kafka。



#### 8.1 JMS

JMS是一个Java标准，定义了使用消息代理（message broker）的通用API，同JDBC为不同消息队列提供通用接口

Spring还提供了消息驱动POJO的理念：这是一个简单Java对象，能够以异步的方式相应队列/主题上到达的消息



##### 8.1.1 JMS环境

``` xml
<dependency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-strater-artemis</artifactId>
</dependency>
```

Artemis是重新实现的下一代ActiveMQ，使ActiveMQ变成了遗留方案，因此我们选择Artemis，默认端口61616，在生产环境我们一般需要配置如下：

| artmis属性              | 描述                           | activemq                   |
| ----------------------- | ------------------------------ | -------------------------- |
| spring.artemis.host     | 代理主机 / 代理URL             | spring.activemq.broker-url |
| spring.artemis.port     | 代理端口                       |                            |
| spring.artemis.user     | 用来访问代理的用户（可选）     | spring.activemq.user       |
| spring.artemis.password |                                | spring.activemq.password   |
|                         | 是否启用在内存中运行代理(true) | spring.activemq.in-memory  |

``` yaml
spring:
  artemis:
  	host: 127.0.0.1
  	port: 61617
  	user: root
  	password: 123456
  	
  activemq:
    broker-url: tcp://127.0.0.1:61616
    # 如果为true则运行在内存中，限制了只有同一个应用发布和消费消息才能使用
    in-memory: false
```



##### 8.1.2 JmsTemplate

将JMS starter依赖（Artemis / ActiveMQ）添加到构建文件后，Spring Boot会自动配置一个`JmsTemplate`，我们可以将它注入到其它bean中使用



我们来看看JmsTemplate的方法列表

``` java
// 发送原始消息
void send(MessageCreator creator) throws JmsException;
void send(Destination destination, MessageCreator creator) throws;
void send(String destination, MessageCreator creator) throws ;

// 发送根据对象转换而成的消息
void convertAndSend(Object message) throws ;
void convertAndSend(Destination, Object) throws;
void convertAndSend(String, Object) throws ;

// 发送根据对象转换而成的消息，且带有后处理功能
void convertAndSend(Object message, MessagePostProcesser postProcesser);
void concertAndSet(Destination, Object, MessagePostProcesser);
void convertAndSet(String, Object, MessagePostProcesser);
```



```java
@Service
public class MessgeProducer {
    @Autowired
    private JmsTemplate jms;
    public void sendOrderMessage(Order order) {
        jms.send(new MessageCreator() {
            @Override
            public Message createMessage(Session s) throws JmsException {
                return s.createObjectMessage(order);
            }
        });
        
        // lambda方式
        jms.send(session -> session.createObjectMessage(order));
    }
}
```

此时我们没有指定目的地，所以我们配置一个默认destination

``` yaml
spring:
  jms:
  	template:
  		default-destination: tacocloud.order.queue
```



如果需指定发送的destination，那么我们可以声明Destination bean

``` java
import org.apache.activemq.artemis.jms.client;
@Bean
public Destination orderQueue() {
    return new ActiveMQQueue("tacocloud.order.queue");
}

@Bean
public ActiveMQQueue logQueue() {
    return new ActivceMQQueue("tacocloud.log.queue");
}

// 发送消息
class Producer {
    @Autowired
    private Destination orderQueue;
    @Autowired
    private Destination logQueue;
    
    void sendOrder(Order order) {
        jms.send(orderQueue, session -> session.createObjectMessage(order))
    }
    
    void sendLog(Log log) {
        jms.send(logQueue, session -> session.createObjectMessage(log))
    }
    
    /**
 	 * 消息发送之前进行转换,convertAndSend()
 	 * 尽管send()方法不是很困难，但要求一个MessageCreator，而我们只是发送一个对象而已
 	 */
    void sendOrder(Order order) {
        jms.convertAndSend("tacocloud.order.queue", order);
    }
}


```

