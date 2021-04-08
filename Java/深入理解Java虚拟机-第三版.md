

## 二. 自动内存管理



### 2.  JVM 内存模型



#### 2.1 概述





#### 2.2 运行时数据区





#### 2.3 HotSpot 对象探秘





#### 2.4 OOM 异常





#### 2.5 小结





### 3. 垃圾收集器与内存分配



#### 3.1 概述





#### 3.2 对象已死？





#### 3.3 垃圾收集算法





#### 3.4 HotSpot 算法实现细节





#### 3.5 经典垃圾收集器



#### 3.6 低延迟垃圾收集器





#### 3.7 选择合适的 GC



#### 3.8 内存分配与回收



#### 3.9 小结



### 4. 虚拟机监控，故障处理工具



#### 4.1 概述



#### 4.2 基础故障处理工具



#### 4.3 可视化故障处理工具



#### 4.4 HotSpot插件及工具



#### 4.5 小结



### 5. 调优分析与实战



#### 5.1 概述



#### 5.2 案例



#### 5.3 Eclipse 调优



#### 5.4 小结





## 三. 虚拟机执行子系统



### 6. 类文件结构



#### 6.1 概述



#### 6.2 无关性的基石





#### 6.3 Class 类文件结构





#### 6.4 字节码指令



#### 6.5 共有设计，私有实现





#### 6.6 Class 文件结构的发展



#### 6.7 小结





### 7. 虚拟机类加载机制



#### 7.1 概述



#### 7.2 类加载时机



#### 7.3 类加载过程



#### 7.4 类加载器



#### 7.5 Java 模块化系统



#### 7.6 小结





### 8. 虚拟机字节码执行引擎



#### 8.1 概述



#### 8.2 栈帧



#### 8.3 方法调用



#### 8.4 动态类型语言支持



#### 8.5 基于栈的字节码解释引擎



#### 8.6 小结



### 9. 类加载及执行子系统实战



#### 9.1 概述



#### 9.2 案例分析



#### 9.3 实现远程执行功能



#### 9.4 小结



## 四. 程序编译与代码优化







### 10. 前端编译与优化



#### 10.1 概述



#### 10.2 Javac 编译器



#### 10.3 语法糖



#### 10.4 插入式注解处理器



#### 10.5 小结





### 11. 后端编译与优化



#### 11.1 概述





#### 11.2 即时编译器





#### 11.3 提前编译器





#### 11.4 编译器优化技术





#### 11.5 深入 Graal 编译器





#### 11.6 小结





## 五. 高效并发

多任务处理是现代计算机必备的，主要原因是 CPU 速度与存储子系统和通信子系统的速度差距太大

**大量时间花费在磁盘 I/O，网络通讯，数据库访问**

**衡量服务性能的好坏：TPS（Transactions Per Second），QPS（Queries Per Second）**

 如果线程之间频繁争用数据，互相阻塞甚至死锁，将会大大降低程序的并发能力 

> 高效并发是最后一部分，将会向读者介绍虚拟机如何实现多线程，多线程之间共享和数据竞争导致的一系列问题的解决方案。



### 12. Java内存模型与线程



#### 12.2 硬件的效率与一致性









#### 12.3 Java内存模型



#### 12.5 Java与协程



#### 12.6 本章小结





### 13. 线程安全与锁优化



#### 13.1 概述



#### 13.2 线程安全



#### 13.3 锁优化



#### 13.4 本章小结





## Appendix



### A. 在 Windows 下编译 OpenJDK6



### B. 展望 Java 技术的未来



#### B.1 Sumatra 项目

> OpenJDK 子项目 Sumatra
>
> 目前 GPU 的运算能力，并行能力远超过 CPU，在图形领域发掘显卡的潜力是近几年计算机发展方向之一，例如 C 语言的 CUDA。
>
> **Sumatra 项目是为 Java 提供使用 GPU「Graphics Processing Unit」和 APU「 Accelerated  Processing Unit」运算能力的工具，以后会成为 Java 语言层面的 API，或者为 Lambda 和其它 JVM Lang 提供并行运算支持**



#### B.2 并行

在 JDK 外围，也出现专为实现并行计算需求的框架，如 Apache 的 Hadoop Map/Reduce

能够运行在成千个机器组成的集群上，并以一种可靠的容错方式并行处理 TB 级数据集

另外出现诸如 Scala、Clojure、Erlang 等天然并行计算能力的语言



#### B.3 Coin 项目

Java5 曾对 Java 语言进行了一次扩充，添加了：

**自动装箱、泛型、动态注解、枚举、可变参数、Enhance For**

>  Sun（Oracle）专门为改进 Java 语法在 OpenJDK 中建立了 Coin 子项目来统一处理Java语法的细节修改，如对二进制数的原生 支持、在switch语句中支持字符串、“<>”操作符、异常处理的改进、简化变长参数方法调用、面向资 源的try-catch-finally语句等都是在Coin项目之中提交的内容。  



#### B.4 64位虚拟机

64bit VM 首先是内存问题：

+ 指针膨胀「可以开启指针压缩，-XX:+UseCompressedOops（默认值）」
+ 对齐补白

消耗更多内存，通常比 32bit 增加 10% ~ 30%（应该是未开启指针压缩的测试数据），性能降低 15% 左右



### C. 虚拟机字节码指令集



### D. 对象查询语言「OQL」



#### D.1 SELECT

> SELECT 用于从堆转储快照中选择什么内容，用法与传统 SQL 语句类似

``` sql
SELECT * FROM java.lang.String

// 查询特定的列「字段」
SELECT toString(s), s.count, s.value from java.lang.String s

// 通过 @ 使用 Java 对象的内存属性访问器。MAT提供一系列内置函数来获取分析相关的信息
SELECT toString(s), s.@usedHeapSize, s.@retainedHeapSize FROM java.lang.String s

// 使用列别名
SELECT toString(s) AS Val,
	s.@usedHeapSize AS "Shallow Size",
	s.@retainedHeapSize AS "Retained Size"
FROM java.lang.String s

// 使用 OBJECTS 把 SELECT 数据转换为对象
SELECT OBJECTS dominators(s) FROM java.lang.String s

// DISTINCT
SELECT DISTINCT classof(s) FROM java.lang.String s
```

> 上面的栗子：
>
> `dominators()` 返回一个对象数组
>
> `classof()` 返回对象所属的类



#### D.2 FROM

FROM 子句可以接受以下几种描述方式：

1. 通过类名进行查询（`SELECT * FROM java.lang.String`）
2. 通过正则匹配一组类（`SELECT * FROM "java\.lang\..*"`）
3. 通过类对象在堆转储快照中的地址进行查询（`SELECT * FROM 0Xe14a100`）
4. 通过对象在堆转储快照中的 ID 进行查询（`SELECT * FROM 3022`）
5. 在子查询中进行查询

`SELECT * FROM (SELECT * FROM java.lang.Class c where c implements java.util.List)`

返回堆转储快照中所有实现 java.util.List 接口的类的对象



##### D.2.1 包含子类

``` sql
// 指定类的子类列入查询结果集中
SELECT * FROM INSTANCEOF java.lang.ref.Reference

SELECT * FROM $snapshot.getClassesByName("java.lang.ref.Reference", true)
```

> 这个查询的结果集中会包含 WeakReference、SoftReference、PhantomReference类型的对象，它们都继承自 java.lang.Reference



+ 查询 java.lang.String 的类对象

`SELECT * FROM OBJECTS java.lang.String`



#### D.3 WHERE

`>=, <=, >, <, [NOT] LIKE, [NOT] IN`

``` sql
SELECT * FROM java.lang.String s where s.count > 100
SELECT * FROM java.lang.String s WHERE toString(s) LIKE ".*day"
SELECT * FROM java.lang.String s WHERE s.value NOT IN dominators(s)

# =, !=
SELECT * FROM java.lang.String s WHERE toString(s) = "monday"

# AND
SELECT * FROM java.lang.String s
	WHERE s.count > 100 AND s.@retainedHeapSize > s.usedHeapSize
	
# OR
SELECT * FROM java.lang.String s WHERE s.count > 1000 OR s.value.@length > 1000
```



+ 文字表达式

文字表达式包括：布尔值、字符串、整形、长整型、null

``` sql
SELECT * FROM java.lang.String s
WHERE (s.count > 1000) = true
WHERE toString(s) = "monday"
WHERE s.@retainedHeapSize > 1024L
WHERE s.@GCRootInfo != null
```



#### D.4 属性访问器



##### D.4.1 常用的 Java 内存属性

<table>
  <thead>
  	<th>目标</th>
    <th>接口</th>
    <th>属性</th>
    <th>含义</th>
  </thead>
  <tr>
    <td rowspan="6">任意堆中对象</td> 
    <td rowspan="6">Iobject</td>
    <td>objectId</td>
    <td>快照中对象的ID</td>
  </tr>
  <tr>
    <td>objectAddress</td>
    <td>快照中对象的地址</td>
  </tr>
  <tr>
    <td>Class</td>
    <td>对象所属的类</td>
  </tr>
  <tr>
    <td>usedHeapSize</td>
    <td>对象的 ShallowSize</td>
  </tr>
  <tr>
    <td>retainedHeapSize</td>
    <td>对象的 RetainedSize</td>
  </tr>
  <tr>
    <td>displayName</td>
    <td>对象显示的名称</td>
  </tr>
  <tr>
    <td>类对象</td>
    <td>Iclass</td>
    <td>classLoaderId</td>
    <td>类加载器的 ID</td>
  </tr>
  <tr>
    <td>任意数组</td>
    <td>Iarray</td>
    <td>length</td>
    <td>数组的长度</td>
  </tr>
</table>



##### D.4.2 调用 OQL Java 方法

**加 “()” 会被 MAT 解释为一个 OQL Java 方法调用，这个方法是通过反射执行的**



##### D.4.3 OQL 内建函数

格式为：`<function>(<parameter>)`

| 函数名              | 作用                                           |
| ------------------- | ---------------------------------------------- |
| toHext(number)      | 以16进制打印数字                               |
| toString(object)    | 用一个字符串标识对象的内容                     |
| dominators(object)  | 返回直接持有当前对象的对象列表                 |
| outbounds(object)   | 获取对象的外部引用                             |
| inbounds(object)    | 获取对象的内部引用                             |
| classof(object)     | 获取对象所属的类型对象                         |
| dominatorof(object) | 返回直接持有当前对象的对象列表，如果没有返回-1 |
|                     |                                                |



