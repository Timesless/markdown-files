### JVM

``` mermaid
graph LR
JVM --> OS --> C[硬件 SPAC]
```

Class files --> Class Loader --> Runtime Data Area --> Execution Engine & Native Interface



+ 类加载器 ClassLoader（双亲委派）

> **将class字节码文件加载并转换为方法区运行时数据结构**
>
> 即： Test.class被加载并初始化之后在方法区为 Class<Test>
>
> ``` mermaid
> graph LR
> A[Bootstrap 根加载器] --> B[Extension 扩展加载器] --> AppClassLoader
> 
> ```
>
> Bootstrap --> **${JAVA_HOME}/jre/lib/rt.jar**
>
> Extension --> 扩展的补丁：**${JAVA_HOME}/jre/lib/ext/*.jar**
>
> AppClassLoader --> 用户自定义类：Test.class.getClassLoader()



+ 运行时数据区（Runtime Data Area）

> 方法区，堆虚拟机栈 & 本地方法栈，堆，程序计数器



每个方法执行会创建栈帧，存储**局部变量表，操作数栈，动态链接，方法出口**

| 局部变量表（输入输出参数，方法内定义的变量。在执行前能确定大小） |
| :----------------------------------------------------------: |
|                           操作数栈                           |
|                           动态链接                           |
|                           方法出口                           |

+ 堆

> 新生代（Eden，From Survivor，To Survivor）
>
> 老年代
>
> **元空间，逻辑上在堆连续，物理上实现为堆外内存**



| 名称                    | 其它叫法      |
| ----------------------- | ------------- |
| Young generation Space  | New \| Young  |
| Tenure generation Space | Old \| Tenure |

#### 垃圾回收算法

> 复制（新生代），标记清除，标记清除压缩

GC

``` tex
Young GC（Minor GC），Full GC（Major GC）
[PSYoungGen: 2048k->488k(2560k)] 2048k -> 773k(9728k), 0.0015secs]
[Times: user=0.08 sys = 0.02, real=0.00 secs]
```





#### JVM部分指令集

``` properties
0xb6	invokevirtual	调用实例方法
0xb7	invokespecial	调用超类构造方法，实例初始化方法，私有方法
0xb8	invokestatic	调用静态方法
0xb9	invokeinterface	调用接口方法
0xba	invokedynamic	调用动态链接方法

0xbb	new	创建一个对象，并将其引用值压入栈顶
0xbc	newarray	创建一个指定原始类型（如int, float, char…）的数组，并将其引用值压入栈顶
0xbd	anewarray	创建一个引用型（如类，接口，数组）的数组，并将其引用值压入栈顶
0xbe	arraylength	获得数组的长度值并压入栈顶

0xbf	athrow	将栈顶的异常抛出
0xc0	checkcast	检验类型转换，检验未通过将抛出ClassCastException
0xc1	instanceof	检验对象是否是指定的类的实例，如果是将1压入栈顶，否则将0压入栈顶

0xc2	monitorenter	获得对象的锁，用于同步方法或同步块
0xc3	monitorexit	释放对象的锁，用于同步方法或同步块
```



+ 源码

``` java
private int sample2(int a, int b) {
    int rs = a + b;
    return rs;
}

private void slefindecre() {
    int i = 6;
    ++ i;
    i ++;
    -- i;
    i --;
}
```

+ 指令

``` java
private int sample2(int, int);
    descriptor: (II)I
    flags: ACC_PRIVATE
    Code:
      stack=2, locals=4, args_size=3
         0: iload_1
         1: iload_2
         2: iadd
         3: istore_3
         4: iload_3
         5: ireturn
      LineNumberTable:
        line 16: 0
        line 17: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lcom/yinhai/dxpms/diff/service/DayTest;
            0       6     1     a   I
            0       6     2     b   I
            4       2     3    rs   I

private void slefindecre();
    descriptor: ()V
    flags: ACC_PRIVATE
    Code:
      stack=1, locals=2, args_size=1
         0: bipush        6
         2: istore_1
         3: iinc          1, 1
         6: iinc          1, 1
         9: iinc          1, -1
        12: iinc          1, -1
        15: return
      LineNumberTable:
        line 28: 0
        line 29: 3
        line 30: 6
        line 31: 9
        line 32: 12
        line 33: 15
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      16     0  this   Lcom/yinhai/dxpms/diff/service/DayTest;
            3      13     1     i   I
```

**stack**

> 最大操作数栈，JVM运行时会根据这个值来分配栈帧(Frame)中的操作栈深度

**locals**

> 局部变量所需的存储空间，单位为Slot（4字节），Slot是虚拟机为局部变量分配内存时所使用的最小单位。
> 方法参数(包括实例方法中的隐藏参数this)，显示异常处理器的参数(try catch中的catch块所定义的异常)，方法体中定义的局部变量都需要使用局部变量表来存放。
>
> locals的大小并不一定等于所有局部变量所占的Slot之和，因为局部变量中的Slot是可以重用的。

**args_size**

> 方法参数的个数，每个实例方法都会有一个隐藏参数this

**LocalVariableTable**

> 该属性的作用是描述帧栈中局部变量与源码中定义的变量之间的关系。
> start 表示该局部变量在哪一行开始可见，length表示可见行数，Slot代表所在帧栈位置，Name是变量名称，然后是类型签名。

