

#### durid连接池设置数据源编码集

``` java
@Primary
@Bean(name = "amrDataSource")
@ConfigurationProperties(prefix = "spring.datasource.druid.amr")
public DataSource getDataSource() {
  DruidDataSource durid = DruidDataSourceBuilder.create().build();
  durid.setConnectionInitSqls(Collections.singletonList("set names utf8mb4"));
  return durid;
}
```



####  Spring事务失效问题

``` xml
<context:component-scan base-package="com.yinhai" use-default-filters="false">
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Component"/>
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Service"/>
	</context:component-scan>
```

