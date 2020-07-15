## Redis数据结构



### SDS

simple dynamic string：简单动态字符串



### 链表





### 字典



### 整数集合

==intset：是集合键底层实现之一。当集合只包含整数值元素，且元素数量不多时，Redis采用整数集合作为集合键的底层实现。==

``` shell
SADD numbers 1 3 5 7 9
TYPE numbers
> list
OBJECT ENCODING numbers
> "intset"
```





### ZipList





### SkipList



### 对象

Redis对象系统是基于引用计数的内存回收机制

``` c
typedef struct redisObject {
    // 类型
    unsigned type: 4;
    // 编码
    unsigned enconding: 4;
    // 指向数据的指针
    void *ptr;
    // 
} robj;
```



+ REDIS_STRING 字符串对象
+ REDIS_LIST 列表对象
+ REDIS_HASH 哈希对象
+ REDIS_SET 集合对象
+ REDIS_ZSET 有序集合对象

当我们称一个数据库键为“字符串键”时，我们指“这个数据键对应的值为字符串对象”

