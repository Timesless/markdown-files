### JDK 9

``` properties
# 模块化系统
jshell
# 语法
diamond, 接口私有方法（既然JDK8提供了默认方法，那为什么不提供私有方法呢）, try
# String结构
String存储由UTF-16的字符序列变为byte[]
```



### 基础知识

> 带标签break
>
> 带标签continue





### 数据类型



#### 大整数与BigDecimal

> 浮点数在内存中用科学计数法表示，实现用x * 2<sup>y</sup>表示实数的近似值，参考==CSAPP==
>
> BigInteger实现任意精度的整数运算
>
> BigDecimal实现任意精度的浮点运算
>

``` java
BigInteger add(BigInteger other);
subtract(other);
mutiply(other);
divide(other);
mod(other);
int compareTo(other);

// BigDecimal
// mode 舍入方式：银行家舍入，四舍五入...
BigDecimal divide(other, RoundingMode mode)
```



#### String

> ==Java为字符串的连接重载了 “+”运算符，且不提供给程序员重载运算符==
>
> 码点与代码单元：
>
> ​	Unicode对每个字符编码，一个字符对应一个码点
>
> ​	代码单元：是对于编码方式，编码方式中的最小单位，例如UTF-8编码可以用1B, 2B, 3B, 4B表示一个字符，最小是1B所以UTF-8的代码单元为1字节，同理UTF-16代码单元为2字节
>
> 
>
> String的length() 和 charAt()是返回指定的代码单元，所以在复杂汉字时可能不准确
>

``` java
// 返回int类型的“流”
IntStream codePoints();
// elements用delimiter连接
String.join(CharSequence delimiter, CharSequence ...elements) 
```



#### 数组

``` java
// 拷贝
Arrays.copyOf(arr, newLength);

// 排序，使用优化的快排
Arrays.sort(arr);
```





### 输入输出

``` java
// 新增API Console
Console console = System.console();
// 处理密码之后应及时用其它值覆盖数组元素
char[] passwd = console.readPassword("password:");

// 格式化
// %d 十进制数
// %s 字符串
// %c 字符
// %f 指数浮点数， %.2f
// %n 换行
// 使用s格式任意对象；对任意实现Formattable接口的对象将调用formatTo()，否则调用toString()
String.format()
```


