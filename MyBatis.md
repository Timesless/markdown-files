

## MyBatis Doc





## 1. XML配置

> 配置的顶层结构如下：

+ Configuration（配置）
  + properties（属性）
  + settings（设置）
  + typeAliases（类型别名）
  + typeHandlers（类型处理器）
  + objectFactory（对象工厂）
  + plugins（插件）
  + environments（环境配置）
    + environment（环境变量，dev | prod | test）
      + transactionManager（事务管理器）
      + dataSource（数据源）
  + databaseIdProvider（数据库厂商标识）
  + mappers（映射器）





### 1.1 类型处理器（typeHandlers）

MyBatis 在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。下表描述了一些默认的类型处理器。

**提示** 从 3.4.5 开始，MyBatis 默认支持 JSR-310（日期和时间 API） 。

| 类型处理器                | Java 类型                      | JDBC 类型                            |
| :------------------------ | :----------------------------- | :----------------------------------- |
| `BooleanTypeHandler`      | `java.lang.Boolean`, `boolean` | 数据库兼容的 `BOOLEAN`               |
| `ByteTypeHandler`         | `java.lang.Byte`, `byte`       | 数据库兼容的 `NUMERIC` 或 `BYTE`     |
| `ShortTypeHandler`        | `java.lang.Short`, `short`     | 数据库兼容的 `NUMERIC` 或 `SMALLINT` |
| `IntegerTypeHandler`      | `java.lang.Integer`, `int`     | 数据库兼容的 `NUMERIC` 或 `INTEGER`  |
| `LongTypeHandler`         | `java.lang.Long`, `long`       | 数据库兼容的 `NUMERIC` 或 `BIGINT`   |
| `FloatTypeHandler`        | `java.lang.Float`, `float`     | 数据库兼容的 `NUMERIC` 或 `FLOAT`    |
| `DoubleTypeHandler`       | `java.lang.Double`, `double`   | 数据库兼容的 `NUMERIC` 或 `DOUBLE`   |
| `BigDecimalTypeHandler`   | `java.math.BigDecimal`         | 数据库兼容的 `NUMERIC` 或 `DECIMAL`  |
| `StringTypeHandler`       | `java.lang.String`             | `CHAR`, `VARCHAR`                    |
| `ClobTypeHandler`         | `java.lang.String`             | `CLOB`, `LONGVARCHAR`                |
| `NStringTypeHandler`      | `java.lang.String`             | `NVARCHAR`, `NCHAR`                  |
| `NClobTypeHandler`        | `java.lang.String`             | `NCLOB`                              |
| `ByteArrayTypeHandler`    | `byte[]`                       | 数据库兼容的字节流类型               |
| `BlobTypeHandler`         | `byte[]`                       | `BLOB`, `LONGVARBINARY`              |
| `DateTypeHandler`         | `java.util.Date`               | `TIMESTAMP`                          |
| `DateOnlyTypeHandler`     | `java.util.Date`               | `DATE`                               |
| `TimeOnlyTypeHandler`     | `java.util.Date`               | `TIME`                               |
| `SqlTimestampTypeHandler` | `java.sql.Timestamp`           | `TIMESTAMP`                          |
| `SqlDateTypeHandler`      | `java.sql.Date`                | `DATE`                               |
| `SqlTimeTypeHandler`      | `java.sql.Time`                | `TIME`                               |



#### 自定义类型处理器

> 1. 实现TypeHandler接口
> 2. 继承BaseTypeHandler类
>
> 将它们映射到一个JDBC类型，以下类型处理器会覆盖已有的 Java String 类型以及 VARCHAR 类型的参数和结果的类型处理器

``` java
@MappedJdbcTypes(JdbcType.VARCHAR)
class TestTypeHandler extends BaseTypeHandler<String> {
  ...
}

```

```xml
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
```



### 1.2 对象工厂（objectFactory）

> ==每次 MyBatis 创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成实例化工作。 默认的对象工厂需要做的仅仅是实例化目标类==，要么通过默认无参构造方法，要么通过存在的参数映射来调用带有参数的构造方法。 如果想覆盖对象工厂的默认行为，可以通过创建自己的对象工厂来实现

```
// ExampleObjectFactory.java
public class ExampleObjectFactory extends DefaultObjectFactory {
  public Object create(Class type) {
    return super.create(type);
  }
  public Object create(Class type, List<Class> constructorArgTypes, List<Object> constructorArgs) {
    return super.create(type, constructorArgTypes, constructorArgs);
  }
  public void setProperties(Properties properties) {
    super.setProperties(properties);
  }
  public <T> boolean isCollection(Class<T> type) {
    return Collection.class.isAssignableFrom(type);
  }
}
```



### 1.3 环境变量（envrionments）

>  MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 Schema 的多个生产数据库中使用相同的 SQL 映射。还有许多类似的使用场景。 
>
>  **尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。** 
>
>  如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个 

为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。

```
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, properties);
```

environments 定义如何配置环境

```
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

注意一些关键点:

- 默认使用的环境 ID（比如：default="development"）。
- 每个 environment 元素定义的环境 ID（比如：id="development"）。
- 事务管理器的配置（比如：type="JDBC"）。
- 数据源的配置（比如：type="POOLED"）。



#### 事务管理器（transactionManager）

MyBatis中有两种类型的事务管理器

1. type = "JDBC"
2. type = "MANAGED"

> JDBC - 这个配置直接使用 JDBC 提交和回滚，它依赖从数据源获取的连接来管理事务作用域
>
> MANAGED - 这个配置几乎不做什么，它从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文），它会关闭连接，然而一些容器并不希望连接被关闭，所以需单独设置如:

```
<transactionManager type="MANAGED">
  <property name="closeConnection" value="false"/>
</transactionManager>
```

 **如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。** 



##### Spring统一事务管理（PlatformTransactionManager）

**针对不同的厂商提供不同`PlatfromTransactionManager` 接口的实现**

对于 Mybatis 提供 `DatasourceTxManager`（MyBatis / JDBC）

对于 Hibernate 提供`HibernateTxManager`



- DataSourceTxManager 具体策略，适用于JDBC/MyBatis
- HibernateTxManager 具体策略，适用于Hibernate
- TransactionTemplate 事务模板
- PlatformTransactionManager 事务操作策略接口
- AbstractPlatformTransactionManager 事务操作策略抽象类
- TxSynManager 事务同步管理器，在线程中同步数据库连接等信息
- DataSourceUtils 数据库操作Utils

> 如何在方法间共享Connection：ThreadLocal「5.3已经修改为`TransactionContext`」
>
> 请查看：[Spring统一事务模型](https://cloud.tencent.com/developer/article/1550114)



#### 数据源（dataSource）

有三种内建的数据源类型

+ UNPOOLED
+ POOLED
+ JNDI

> **UNPOOLE** - 每次请求打开和关闭连接
>
> **POOLED** -   利用“池”的概念将 JDBC 连接对象组织起,避免了创建新的连接实例时所必需的初始化和认证时间 
>
>  **JNDI** – 这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用 



### 1.4 映射器（mappers）

```
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
</mappers>
```

```
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
</mappers>
```

```
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
</mappers>
```

```
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```



## 2. XML映射器

SQL映射文件顶级元素

+ cache - 该命名空间的缓存配置
+ cache-ref - 引用其它命名空间的缓存
+ resultMap - 描述如何从数据库结果集中加载对象
+ ~~paramterMap~~ - 参数映射「已被废弃」，请使用行内参数映射
+ sql - 可被其它语句引用的语句块
+ insert - 映射insert语句
+ update - 映射update语句
+ delete - 映射delete语句
+ select - 映射select语句



### Select

``` xml
<select
  id="selectPerson"
  parameterType="int"
  parameterMap="deprecated"
  resultType="hashmap"
  resultMap="personResultMap"
  flushCache="false"
  useCache="true"
  timeout="10"
  fetchSize="256"
  statementType="PREPARED"
  resultSetType="FORWARD_ONLY">
```



| 属性               | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| `id`               | 在命名空间中唯一的标识符，可以被用来引用这条语句。           |
| `parameterType`    | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。 |
| `parameterMap`     | 用于引用外部 parameterMap 的属性，目前已被废弃。请使用行内参数映射和 parameterType 属性。 |
| `flushCache`       | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：（对 insert、update 和 delete 语句）true。 |
| `timeout`          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
| `statementType`    | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| `useGeneratedKeys` | （仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。 |
| `keyProperty`      | （仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（`unset`）。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| `keyColumn`        | （仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| `databaseId`       | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。 |

下面是 insert，update 和 delete 语句的示例：





### Insert

> 返回自动生成的主键：如果你的数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），那么你可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置为目标属性 

```xml
<insert id="insertAuthor" useGeneratedKeys="true" keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
  </foreach>
</insert>

# 随机id
<insert id="insertAuthor">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
  </selectKey>
  insert into Author
    (id, username, password, email,bio, favourite_section)
  values
    (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
</insert>
```

> 上述示例中： 首先会运行 selectKey 元素中的语句，并设置 Author 的 id，然后才会调用插入语句 



### Sql

> 定义可重用的SQL片段

``` xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>

<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from table t1
    cross join table t2
</select>
```



### 结果映射

>  `resultMap` 元素是 MyBatis 中最重要最强大的元素。它可以让你从 90% 的 JDBC `ResultSets` 数据提取代码中解放出来，并在一些情形下允许你进行一些 JDBC 不支持的操作 



 你已经见过简单映射语句的示例，它们没有显式指定 `resultMap`。比如： 

``` xml
<select id="selectUsers" resultType="User">
  select id, username, hashedPassword from t where id = #{id}
</select>
```

> 上述栗子：MyBatis会在幕后自动创建一个`ResultMap`，再根据属性名来映射到`JavaBean`，如果列与字段不对应，有两种解决方式
>
> 1. select user_name as "userName"
> 2. 显示配置\<resultMap>

``` xml
<resultMap id="beanResultMap" type="User">
	<id property="id" column="user_id"/>
  <result property="username" column="user_name"/>
  ...
</resultMap>
```



### 复杂结果映射

>  **我们希望每个数据库都具备良好的第三范式或 BCNF 范式，可惜它们并不都是那样**。 如果能有一种数据库映射模式，完美适配所有的应用程序，那就太好了，但可惜也没有。 而 ResultMap 就是 MyBatis 对这个问题的答案。 

``` xml
<!-- 非常复杂的语句 -->
<select id="selectBlogDetails" resultMap="detailedBlogResultMap">
  select
       B.id as blog_id,
       B.title as blog_title,
       B.author_id as blog_author_id,
       A.id as author_id,
       A.username as author_username,
       A.password as author_password,
       A.favourite_section as author_favourite_section,
       P.id as post_id,
       P.blog_id as post_blog_id,
       P.author_id as post_author_id,
       P.body as post_body,
       C.id as comment_id,
       C.post_id as comment_post_id,
       C.name as comment_name,
       C.comment as comment_text,
       T.id as tag_id,
       T.name as tag_name
  from Blog B
       left outer join Author A on B.author_id = A.id
       left outer join Post P on B.id = P.blog_id
       left outer join Comment C on P.id = C.post_id
       left outer join Post_Tag PT on PT.post_id = P.id
       left outer join Tag T on PT.tag_id = T.id
  where B.id = #{id}
</select>
```

>  **这个对象表示了一篇博客，它由某位作者所写，有很多的博文，每篇博文有零或多条的评论和标签。** 我们先来看看下面这个完整的例子，它是一个非常复杂的结果映射（假设作者，博客，博文，评论和标签都是类型别名）。 

``` xml
<resultMap id="resultMap" type="Blog">
	<constructor>
  	<idArg column="blog_id" javaType="int" />
  </constructor>
  <result property="title" column = "blog_title" />
  # 一对一
  <association property="author" javaType="Author">
  	<id property="id" column="author_id" />
    <result property="username" column="author_username" />
    <result property="password" column="author_password" />
    ...
  </association>
  # 一对多
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id" />
    <result property="subject" column="post_subject" />
    <association property="author" javaType="Author" />
    <collection property="comments" ofType="Comment">
    	<id property="id" column="comment_id" />
    </collection>
    <collection property="tags" ofType="Tag">
    	<id property="id" column="tag_id" />
    </collection>
    
    <discriminator javaType="int" column="draft">
    	<case value="1" resultType="DraftPost" />
    </discriminator>
  </collection>
</resultMap>
```



- constructor - 用于在实例化类时，注入结果到构造方法中
  - `idArg` - ID 参数；标记出作为 ID 的结果可以帮助提高整体性能
  - `arg` - 将被注入到构造方法的一个普通结果
- `id` – 一个 ID 结果；标记出作为 ID 的结果可以帮助提高整体性能
- `result` – 注入到字段或 JavaBean 属性的普通结果
- association– 一个复杂类型的关联；许多结果将包装成这种类型
  - 嵌套结果映射 – 关联可以是 `resultMap` 元素，或是对其它结果映射的引用
- collection– 一个复杂类型的集合
  - 嵌套结果映射 – 集合可以是 `resultMap` 元素，或是对其它结果映射的引用
- discriminator– 使用结果值来决定使用哪个resultMap
  - case– 基于某些值的结果映射
    - 嵌套结果映射 – `case` 也是一个结果映射，因此具有相同的结构和元素；或者引用其它的结果映射



#### id & result

```
<id property="id" column="post_id"/>
<result property="subject" column="post_subject"/>
```

>  *id* 和 *result* 元素都将一个列的值映射到一个简单数据类型（String, int, double, Date 等）的属性或字段。 
>
>  *id* 元素对应的属性会被标记为对象的标识符，在比较对象实例时使用。 这样可以提高整体的性能，尤其是进行缓存和嵌套结果映射（也就是连接映射）的时候。 



#### 关联（association）

> 处理”一对一“关联关系
>
> MyBatis有两种不同的方式加载关联
>
> + 嵌套 Select 查询（select中嵌套另一个select来实现关联关系）
> + 嵌套结果映射

```
<association property="author" column="blog_author_id" javaType="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
</association>
```

嵌套 Select 查询示例：

```xml
# 嵌套select
<resultMap id="blogResult" type="Blog">
  <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

<select id="selectAuthor" resultType="Author">
  SELECT * FROM AUTHOR WHERE ID = #{id}
</select>
```

嵌套结果映射示例：

```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author" column="blog_author_id" javaType="Author" resultMap="authorResult"/>
</resultMap>

# 也可直接嵌套在blogResult内
<resultMap id="authorResult" type="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
  <result property="password" column="author_password"/>
</resultMap>
```















































## 3.  动态SQL