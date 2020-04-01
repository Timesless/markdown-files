``` sql
-- 加载文本数据
LOAD DATA LOCAL INFILE 'data.txt' INTO TABLE table_test;

-- 查询表结构
DESCRIBE table_test
SHOW INDEX FROM table_test

select @min := MIN(price), @max := MAX(price) FROM shop;
select * from shop where price = @min or price = @max;

-- ddl
show create table shop
describe shop

select mysql_insert_id();

-- mysqlcheck使用SQL语句 CHECK | REPAIR | ANALYZE, OPTIMIZE TABLE table_test为用户提供方便的操作
OPTIMIZE TABLE shop;

-- mysqldump备份到文件
shell> mysqldump > dump.sql
-- mysqlpump备份到其它数据库

-- mysqlimport提供SQL语句的方式
LOAD DATA INFILE
shell> mysqlimport

-- mysqlshow提供SQL语句的方式
show columns from shop

-- mysqlbinlog 二进制日志文件
shell> mysqlbinlog master-bin.000001
-- mysqlbinglog从服务器备份
shell> mysqlbinlog --read-from-remote-server --host=host_name --raw
```

### 优化

#### where优化

``` sql

```

