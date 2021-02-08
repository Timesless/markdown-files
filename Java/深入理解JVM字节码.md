### 1.1 初探class文件
“计算机科学领域的任何问题都可以通过增加一个间接的中间层来解决”



### 1.2 class文件结构剖析

> Java虚拟机规定用u1、u2、u4三种数据结构来表示1 、2、4字节无符号整数，相同类型的若干条数据集合用表（table）的形式来存储。表是一个变长的结构，由代表长度的表头n和紧随着的n个数据项组成。class文件采用类似C语言的结构体来存储数据

class文件由下面十个部分组成：

❏ 魔数（Magic Number） 

❏ 版本号（Minor&Major Version）

❏ 常量池（Constant Pool） 

❏ 类访问标记（Access Flag） 

❏ 类索引（This Class） 

❏ 超类索引（Super Class） 

❏ 接口表索引（Interface） 

❏ 字段表（Field） 

❏ 方法表（Method） 

❏ 属性表（Attribute）



#### 1.2.1 魔数

很多文件都以固定的几字节开头作为魔数，比如PDF文件的魔数是 %PDF-（十六进制0x255044462D）, png文件的魔数是 \x89PNG（十六进制0 x89504E47）。文件格式的制定者可以自由地选择魔数值，只要魔数值还没有被广泛采用过且不会引起混淆即可。
CAFEDEAD作为对象持久化文件的魔数，选择了CAFEBABE作为class文件的魔数。

#### 1.2.2 版本号

在魔数之后的四个字节分别表示副版本号（Minor Version）和主版本号（Major Version）



#### 1.2.3 常量池

紧随版本号之后的是常量池数据区域，常量池是类文件中最复杂的数据结构。对于JVM字节码来说，如果操作数是很常用的数字，比如0，这些操作数是内嵌到字节码中的。如果是字符串常量和较大的整数等，class文件则会把这些操作数存储在常量池（Constant Pool）中，当使用这些操作数时，会根据常量池的索引位置来查找。

常量池分为两部分。

1）常量池大小（cp_info_count）：常量池是class文件中第一个出现的变长结构。既然是池，就有大小，常量池大小由两个字节表示。假设常量池大小为 n，常量池真正有效的索引是1～n-1。也就是说，如果constant_pool_count等于10, constant_pool数组的有效索引值是1～9。0属于保留索引，可供特殊情况使用。 2）常量池项（cp_info）集合：最多包含 n-1个元素。为什么是最多呢？long和double类型的常量会占用两个索引位置，如果常量池包含了这两种类型的元素，实际的常量池项的元素个数比 n-1要小。

1. CONSTANT_Integer_info和CONSTANT_Float_info CONSTANT_Integer_info和CONSTANT_Float_info这两种结构分别用来表示int和float类型的常量，两者的结构很类似，都用4个字节来表示具体的数值常量，它们的结构定义如下所示。 ￼  CONSTANT_Integer_info {￼  u1 tag;￼  u4 bytes;￼  }￼ ￼  CONSTANT_Float_info {￼  u1 tag;￼  u4 bytes;￼  }

2. CONSTANT_Long_info和CONSTANT_Double_info CONSTANT_Long_info和CONSTANT_Double_info这两种结构分别用来表示long和double类型的常量，二者都用8个字节表示具体的常量数值

3. CONSTANT_Utf8_info CONSTANT_Utf8_info存储了字符串的内容，结构如下所示。

  ``` c
  CONSTANT_Utf8_info {￼
    u1 tag;
    u2 length;￼
    u1 bytes[length];￼
  }
  ```

  它由三部分构成：第一个字节是tag，值为固定值1; tag之后的两个字节length并不是表示字符串有多少个字符，而是表示第三部分byte数组的长度；第三部分是采用MUTF-8编码的长度为length的字节数组。
  为了能搞清楚MUTF-8，需要知道UTF-8编码是如何实现的。UTF-8是一种变长编码方式，使用1～4个字节表示一个字符
  那MUTF-8有什么不一样呢？它们之间的区别如下。 1）MUTF-8里用两个字节表示空字符（"\0"），把前面介绍的双字节表示格式110xxxxx 10xxxxxx中的x全部填0，也即0 xC080，而在标准UTF-8编码中只用一个字节0 x00表示。这样做的原因是在其他语言中（比如C语言）会把空字符当作字符串的结束，而MUTF-8这种处理空字符的方式保证字符串中不会出现空字符，在C语言处理时不会意外截断。 2）MUTF-8只用到了标准UTF-8编码中的单字节、双字节、三字节表示方式，没有用到4字节表示方式。编码在U+FFFF之上的字符，Java使用“代理对”（surrogate pair）通过2个字符表示，比如emoji表情
  可以看到x对应的空字符表示为010002 C080，其中第一个字节01表示CONSTANT_Utf8_info类型，紧随其后的两个字节0 x0002表示byte数组的长度，最后的两个字节0 xC080印证了之前的描述。
  前三个字节ED A0 BD对应的二进制为111011011010000010111101，根据UTF-8三字节表示方式，去掉第一个字节的1110、第二和第三个字节的10，剩下的位是1101100000111101，也即0 xD83D，同理可得剩下的3字节对应0 xDE02，得到这个emoji的编码为4字节“0xD83D DE02”，对应的MUTF-8解码过程如下所示。

4. CONSTANT_String_info CONSTANT_String_info用来表示java.lang.String类型的常量对象。它与CONSTANT_Utf8_info的区别是CONSTANT_Utf8_info存储了字符串真正的内容，而CONSTANT_String_info并不包含字符串的内容，仅仅包含一个指向常量池中CONSTANT_Utf8_info常量类型的索引。 CONSTANT_String_info的结构由两部分构成，第一个字节是tag，值为8 , tag后面的两个字节是一个名为string_index的索引值，指向常量池中的CONSTANT_Utf8_info

5. CONSTANT_Class_info CONSTANT_Class_info结构用来表示类或接口，它的结构与CONSTANT_String_info非常类似，可用下面的伪代码表示。 ￼  CONSTANT_Class_info {￼  u1 tag;￼  u2 name_index;￼  } 它由两部分组成，第一个字节是tag，值固定为7 , tag后面的两个字节name_index是一个常量池索引，指向CONSTANT_Utf8_info常量，这个字符串存储的是类或接口的全限定名

6. CONSTANT_NameAndType_info CONSTANT_NameAndType_info结构用来表示字段或者方法
    CONSTANT_NameAndType_info结构由三部分组成，第一部分tag值固定为12，后面的两个部分name_index和descriptor_index都指向常量池中的CONSTANT_Utf8_info的索引，name_index表示字段或方法的名字，descriptor_index是字段或方法的描述符，用来表示一个字段或方法的类型

7. CONSTANT_Fieldref_info、CONSTANT_Methodref_info和CONSTANT_InterfaceMethodref_info
    “testMethod”，方法类型为“（ILjava/lang/String;）V”

8. CONSTANT_MethodType_info、CONSTANT_MethodHandle_info和CONSTANT_InvokeDynamic_info 从JDK1.7开始，为了更好地支持动态语言调用，新增了3种常量池类型（CONSTANT_MethodType_info、CONSTANT_MethodHandle_info和CONSTANT_InvokeDynamic_info）

  #### 1.2.5

  this_class、super_name、interfaces 这三部分用来确定类的继承关系，this_class表示类索引，super_name表示直接父类的索引，interfaces表示类或者接口的直接父接口。 this_class是一个指向常量池的索引，表示类或者接口的名字

  #### 1.2.6 字段表

  紧随接口索引表之后的是字段表（fields），类中定义的字段会被存储到这个集合中，包括静态和非静态的字段，它的结构可以用下面的伪代码表示。
  字段结构分为4个部分：

  第一部分access_flags表示字段的访问标记，用来标识是public、private还是protected，是否是static，是否是final等；

  第二部分name_index用来表示字段名，指向常量池的字符串常量；

  第三部分descriptor_index是字段描述符的索引，指向常量池的字符串常量；

  最后的attributes_count、attribute_info表示属性的个数和属性集合

  字段描述符（field descriptor）用来表示某个field的类型，在JVM中定义一个int类型的字段时，类文件中存储的类型并不是字符串int，而是更精简的字母I。 根据类型的不同，字段描述符分为三大类。 

  1）原始类型，byte、int、char、float等这些简单类型使用一个字符来表示，比如J对应long类型，B对应byte类型。

   2）引用类型使用L；的方式来表示，为了防止多个连续的引用类型描述符出现混淆，引用类型描述符最后都加了一个“;”作为结束，比如字符串类型String的描述符为“Ljava/lang/String;”。 

  3）JVM使用一个前置的“[”来表示数组类型，如int[] 类型的描述符为“[I”，字符串数组String[] 的描述符为“[Ljava/lang/String;”

  #### 1.2.7 方法表

  方法表的作用与前面介绍的字段表非常类似，类中定义的方法会被存储在这里，方法表也是一个变长结构
  方法method_info结构分为四部分：第一部分access_flags表示方法的访问标记，用来标记是public、private还是protected，是否是static，是否是final等；接下来的name_index、descriptor_index分别表示方法名和方法描述符的索引值，指向常量池的字符串常量；attributes_count和attribute_info表示方法相关属性的个数和属性集合，
  方法Object foo（int i, double d, Thread t）的描述符为“（IDLjava/lang/Thread;）Ljava/lang/Object;”，其中“I”表示第一个参数i的参数类型int,“D”表示第二个参数d的类型double,“Ljava/lang/Thread;”表示第三个参数t的类型Thread,“Ljava/lang/Object;”表示返回值类型Object，
  4．方法属性表 方法属性表是method_info结构的最后一部分。前面介绍了方法的访问标记和方法签名，还有一些重要的信息没有出现，如方法声明抛出的异常，方法的字节码，方法是否被标记为deprecated等，属性表就是用来存储这些信息的。与方法相关的属性有很多，其中比较重要的是Code和Exceptions属性，其中Code属性存放方法体的字节码指令，Exceptions属性用于存储方法声明抛出的异常。属性的细节我们将在1.2.8节中进行介绍。

9. ConstantValue属性 ConstantValue属性出现在字段field_info中，用来表示静态变量的初始值，

10. Code属性 Code属性是类文件中最重要的组成部分，它包含方法的字节码，除native和abstract方法以外，每个method都有且仅有一个Code属性
    3）max_stack表示操作数栈的最大深度，方法执行的任意期间操作数栈的深度都不会超过这个值。它的计算规则是：有入栈的指令stack增加，有出栈的指令stack减少，在整个过程中stack的最大值就是max_stack的值，增加和减少的值一般都是1，但也有例外：LONG和DOUBLE相关的指令入栈stack会增加2 , VOID相关的指令则为0
    4）max_locals表示局部变量表的大小，它的值并不等于方法中所有局部变量的数量之和。当一个局部作用域结束，它内部的局部变量占用的位置就可以被接下来的局部变量复用了。

   5）code_length和code用来表示字节码相关的信息。其中，code_length表示字节码指令的长度，占用4个字节；code是一个长度为code_length的字节数组，存储真正的字节码指令。 

  6）exception_table_length和exception_table用来表示代码内部的异常表信息
  异常表在编译期确定，所以无法捕获泛型异常
  catch_type表示需要处理的catch的异常类型是什么，它用两个字节表示，指向常量池中类型为CONSTANT_Class_info的常量项。如果catch_type等于0，则表示可处理任意异常，可用来实现finally语义。 当JVM执行到这个方法 [start_pc, end_pc）范围内的字节码发生异常时，如果发生的异常是这个catch_type对应的异常类或者它的子类，则跳转到code字节数组handler_pc处继续处理。
  catch_type表示需要处理的catch的异常类型是什么，它用两个字节表示，指向常量池中类型为CONSTANT_Class_info的常量项。如果catch_type等于0，则表示可处理任意异常，可用来实现finally语义。 当JVM执行到这个方法 [start_pc, end_pc）范围内的字节码发生异常时，如果发生的异常是这个catch_type对应的异常类或者它的子类，则跳转到code字节数组handler_pc处继续处理。
  attributes_count和attributes[] 用来表示Code属性相关的附属属性，Java虚拟机规定Code属性只能包含这四种可选属性：LineNumberTable、LocalVariableTable、LocalVariableTypeTable、StackMapTable。以LineNumberTable为例，LineNumberTable用来存放源码行号和字节码偏移量之间的对应关系，属于调试信息，不是类文件运行的必需属性，默认情况下都会生成。如果没有这个属性，那么在调试时就没有办法在源码中设置断点，也没有办法在代码抛出异常时在错误堆栈中显示出错的行号信息。

  

  ## 第2章 字节码基础

  ❏ 基于寄存器和基于栈虚拟机实现的优缺点 

  ❏ 字节码的分类

  ❏ 类型转换指令

  ❏ for循环的字节码实现

  ❏ switch-case的tableswitch和lookupswitch两种实现

  ❏ String的switch实现原理 ❏ ++i与i++ 的字节码原理 

  ❏ Java异常处理原理

   ❏ finally语句块一定会执行的原因

  ❏ try-with-resources语法糖背后的原理

  ❏ 对象创建、类初始化相关的字节码指令

  ### 2.1 字节码概述

  字节码文件非常小巧紧凑，但也直接限制整个JVM操作码指令集的数量最多只能有256个，目前已经使用超过了200个。
  字节码使用大端序（Big-Endian）表示，即高位在前，低位在后的方式，比如字节码getfield 0002，表示的是getfiled 0x00＜＜8 | 0x02（getfield #2）。 

  字节码并不是某种虚拟CPU的机器码，而是一种介于源码和机器码中间的一种抽象表示方法，不过字节码通过JIT（Just In Time）技术可以被进一步编译成机器码。

  ❏ 加载和存储指令，比如iload将一个整型值从局部变量表加载到操作数栈；

  ❏ 控制转移指令，比如条件分支ifeq； 

  ❏ 对象操作，比如创建类实例的指令new； 

  ❏ 方法调用，比如invokevirtual指令用于调用对象的实例方法； 

  ❏ 运算指令和类型转换，比如加法指令iadd； 

  ❏ 线程同步，比如monitorenter和monitorexit这两条指令用于支持synchronized关键字的语义； 

  ❏ 异常处理，比如athrow显式抛出异常。

  ### 2.2 Java虚拟机栈和栈帧

  虚拟机常见的实现方式有两种：基于栈（Stack based）和基于寄存器（Register based）。

  典型的基于栈的虚拟机有Hotspot JVM、.net CLR，而典型的基于寄存器的虚拟机有Lua语言虚拟机LuaVM和Google开发的Android虚拟机DalvikVM。
  第1行调用ADD指令将R0寄存器和R1寄存器中的值相加存储到寄存器R2中。第2行返回R2寄存器的值。第3行是lua的一个特殊处理，为了防止有分支漏掉了return语句，lua始终在最后插入一行return语句
  ❏ 基于栈的指令集架构的优点是移植性更好、指令更短、实现简单，但是不能随机访问堆栈中的元素，完成相同功能所需的指令数一般比寄存器架构多，需要频繁地入栈出栈，不利于代码优化。 

  ❏ 基于寄存器的指令集架构的优点是速度快，可以充分利用寄存器，有利于程序做运行速度优化，但操作数需要显式指定，指令较长。
  Hotspot JVM是一个基于栈的虚拟机，每个线程都有一个虚拟机栈用来存储栈帧，每次方法调用都伴随着栈帧的创建、销毁。
  -Xss来指定线程栈的大小，比如 -Xss:256k用于将栈的大小设置为256KB。 每个线程都拥有自己的Java虚拟机栈，一个多线程的应用会拥有多个Java虚拟机栈，每个栈拥有自己的栈帧
  栈帧是用于支持虚拟机进行方法调用和方法执行的数据结构，随着方法调用而创建，随着方法结束而销毁。栈帧的存储空间分配在Java虚拟机栈中，每个栈帧拥有自己的局部变量表（Local Variable）、操作数栈（Operand Stack）和指向常量池的引用
  1．局部变量表 每个栈帧内部都包含一组称为局部变量表的变量列表，局部变量表的大小在编译期间就已经确定，对应class文件中方法Code属性的max_locals字段
  可以看到foo方法只有两个参数，args_size却等于3。当一个实例方法（非静态方法）被调用时，第0个局部变量是调用这个实例方法的对象的引用，也就是我们所说的this。调用方法foo（2019, "hello"）实际上是调用foo（this, 2019, "hello"）
  javap输出中的locals=4表示局部变量表的大小等于4。局部变量表的大小并不是方法中所有局部变量的数量之和，它与变量的类型和变量作用域有关。当一个局部作用域结束，它内部的局部变量占用的位置就可以被接下来的局部变量复用
  2．操作数栈 每个栈帧内部都包含一个称为操作数栈的后进先出（LIFO）栈，栈的大小同样也是在编译期间确定。Java虚拟机提供的很多字节码指令用于从局部变量表或者对象实例的字段中复制常量或者变量到操作数栈，也有一些指令用于从操作数栈取走数据、操作数据和把操作结果重新入栈。在方法调用时，操作数栈也用于准备调用方法的参数和接收方法返回的结果。
  整个JVM指令执行的过程就是局部变量表与操作数栈之间不断加载、存储的过程，
  调用一个成员方法会将this和所有参数入栈，调用完毕this和参数都会出栈。如果方法有返回值，会将返回值入栈
  计算stack的方式如下：遇到入栈的字节码指令，stack+=1或者stack+=2（根据不同的指令类型），遇到出栈的字节码指令，stack则相应减少，这个过程中stack的最大值就是max_stack，也就是javap输出的stack的值

  ### 2.3 字节码指令

  加载（load）和存储（store）相关的指令是使用得最频繁的指令，分为load类、store类、常量加载这三种。
  1）load类指令是将局部变量表中的变量加载到操作数栈，比如iload_0将局部变量表中下标为0的int型变量加载到操作数栈上，根据不同的数据变量类型还有lload、fload、dload、aload这些指令，分别表示加载局部变量表中long、float、double、引用类型的变量。

   2）store类指令是将栈顶的数据存储到局部变量表中，比如istore_0将操作数栈顶的元素存储到局部变量表中下标为0的位置，这个位置的元素类型为int，根据不同的数据变量类型还有lstore、fstore、dstore、astore这些指令。 

  3）常量加载相关的指令，常见的有const类、push类、ldc类。const、push类指令是将常量值直接加载到操作数栈顶，比如iconst_0是将整数0加载到操作数栈上，bipush 100是将int型常量100加载到操作数栈上。ldc指令是从常量池加载对应的常量到操作数栈顶，比如ldc #10是将常量池中下标为10的常量数据加载到操作数栈上。
  使字节码更加紧凑，int型常量值根据值 n 的范围，使用的指令按照如下的规则。

  ❏ 若n在[-1, 5] 范围内，使用iconst_n的方式，操作数和操作码加一起只占一个字节。比如iconst_2对应的十六进制为0 x05。-1比较特殊，对应的指令是iconst_m1（0x02）。

  ❏ 若n在[-128, 127] 范围内，使用bipush n的方式，操作数和操作码一起只占两个字节。比如 n 值为100（0x64）时，bipush 100对应十六进制为0 x1064。

  ❏ 若n在[-32768, 32767] 范围内，使用sipush n的方式，操作数和操作码一起只占三个字节，比如 n 值为1024（0x0400）时，对应的字节码为sipush 1024（0x110400）。

  ❏ 若n在其他范围内，则使用ldc的方式，这个范围的整数值被放在常量池中，比如 n值为40000时，40000被存储到常量池中，加载的指令为ldc #i, i为常量池的索引值。
  字节码指令的别名很多是使用简写的方式，比如ldc是load constant的简写，bipush对应byte immediate push, sipush对应short immediate push。
  dup指令用来复制栈顶的元素并压入栈顶，后面讲到创建对象的时候会用到dup指令。swap用于交换栈顶的两个元素
  图2-8 dup、pop、swap指令 还有几个稍微复杂一点的栈操作指令：dup_x1、dup2_x1和dup2_x2
  dup_x1是复制操作数栈栈顶的值，并插入栈顶以下2个值，看起来很绕，把它拆开来看其实分为了五步

  ``` java
  v1 = stack.pop(); // 弹出栈顶的元素，记为v1￼
  v2 = stack.pop(); // 再次弹出栈顶的元素，记为v2￼
  state.push(v1); // 将v1 入栈￼
  state.push(v2); // 将v2 入栈￼
  state.push(v1); // 再次将v1 入栈
  ```

  接下来看一个dup_x1指令的实际例子，代码如下。 ￼

  ``` java
  public class Hello {￼
    private int id;￼
      public int incAndGetId() {￼
      return ++id;￼
    }￼
  } 
  // incAndGetId方法对应的字节码如下。
    public int incAndGetId();￼
    0: aload_0￼
    1: dup￼
    2: getfield #2 // Field id:I￼
    5: iconst_1￼
    6: iadd￼
    7: dup_x1￼
    8: putfield #2 // Field id:I￼
    11: ireturn
  ```

  假如id的初始值为42，调用incAndGetId方法执行过程中操作数栈的变化如图2-10所示
  0行：aload_0将this加载到操作数栈上。

  第1行：dup指令将复制栈顶的this，现在操作数栈上有两个this，栈上的元素是 [this, this]。

  第2行：getfield #2指令将42加载到栈上，同时将一个this出栈，栈上的元素变为[this, 42]

  第5行：iconst_1将常量1加载到栈上，栈中元素变为[this, 42, 1]。

  第6行：iadd将栈顶的两个值出栈相加，并将结果43放回栈上，现在栈中的元素是[this, 43]。

  第7行：dup_x1将栈顶的元素43插入this之下，栈中元素变为 [43, this, 43]。

  第8行：putfield #2将栈顶的两个元素this和43出栈，现在栈中元素只剩下栈顶的[43]，最后的ireturn指令将栈顶的43出栈返回。

  虽然在Java语言层面，boolean、char、byte、short是不同的数据类型，但是在JVM层面它们都被当作int来处理，不需要显式转为int，字节码指令上也没有对应转换的指令。 

  有多种类型数据混合运算时，系统会自动将数据转为范围更大的数据类型，这种转换被称为宽化类型转换（widening）或自动类型转换，
  自动类型转换并不意味着不丢失精度，比如下面代码中将int值“123456789”转为float就出现了精度丢失的情况

  #### 2.3.4 控制转移指

   控制转移指令用于有条件和无条件的分支跳转，常见的if-then-else、三目表达式、for循环、异常处理等都属于这个范畴。对应的指令集包括： 

   ❏ 条件转移：ifeq、iflt、ifle、ifne、ifgt、ifge、ifnull、ifnonnull、if_icmpeq、if_icmpne、if_icmplt, if_icmpgt、if_icmple、if_icmpge、if_acmpeq和if_acmpne

  ❏ 复合条件转移：tableswitch、lookupswitch

  ❏ 无条件转移：goto、goto_w、jsr、jsr_w、ret

  ifle可以看作“if less or equal”的缩写，比较的值是0。如果想要比较的值不是0，需要用新的指令if_icmple表示“if int compare less or equal xx”。

  第18～22行的作用是把 $array[$i] 赋值给number。aload_3加载 $array到栈上，iload 5加载 $i到栈上，然后iaload指令把下标为 $i的数组元素加载到操作数栈上，随后istore 6将栈顶元素存储到局部变量表下标为6的位置上
  **iinc指令比较特殊，之前介绍的指令都是基于操作数栈来实现功能，它则是直接对局部变量进行自增，不用先入栈、执行加一操作，再将结果出栈存储到局部变量表，因此效率非常高，适合循环结构**

  编译器使用了tableswitch和lookupswitch两条指令来生成switch语句的编译代码。
  编译器会对case的值做分析，如果case的值比较“紧凑”，中间有少量断层或者没有断层，会采用tableswitch来实现switch-case
  lookupswitch指令，它的键值都是经过排序的，在查找上可以采用二分查找的方式，时间复杂度为O（log n）。
  switch-case语句在case比较“稀疏”的情况下，编译器会使用lookupswitch指令来实现，反之，编译器会使用tableswitch来实现。我们在第4章会介绍编译器是如何来判断case值的稀疏程度的。
  可以看到34行在hashCode冲突的情况下，编译器的处理不过是多一次调用字符串equals判断相等的比较
  **了解过字节码之后终于能明白为什么源码都用一个局部变量接受成员变量和静态变量了，成员变量get Field指令较多且慢**

  ``` java
  public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
      char val[] = value;
      for (int i = 0; i < value.length; i++) {
        h = 31 ＊ h + val[i];
      }
      hash = h;
    }
    return h;
  }
  ```

  

    `a ＊ 31 + b = c ＊ 31 + d￼  31＊(a-c)=d-b `其中一个特殊解是a-c=1, d-b=31，也就是只有两个字符的字符串“ab”与“cd”满足a-c=1, d-b=31，这两个字符串的hashCode就一定相等，比如“Aa”和“BB”,“Ba”和“CB”,“Ca”和“DB”，依次类推
  第11行：“iinc 0, 1”对局部变量表slot = 0的变量（i）直接加1，但是这时候栈顶的元素没有变化，还是0
  “j = i++;”在字节码层面是先把i的值加载到操作数栈上，随后才对局部变量i执行加一操作，留在操作数栈顶的还是i的旧值。如果把栈顶值赋值给j，则这个变量得到的是i加一之前的值。
  i = ++i对应的字节码还是第10～14行，可以看出“i = ++i;”先对局部变量表下标为0的变量加1，然后才把它加载到操作数栈上
  每个异常表项表示一个异常处理器，由from指针、to指针、target指针、所捕获的异常类型type四部分组成
  值得注意的是，当抛出异常时，Java虚拟机会自动将异常对象加载到操作数栈栈顶。 第7行：astore_1将栈顶的异常对象存储到局部变量表中下标为1的位置
  一个catch语句处理分支，异常表里面就会多一条记录，当程序出现异常时，Java虚拟机会从上至下遍历异常表中所有的条目。当触发异常的字节码索引值在某个异常条目的 [from, to）范围内，则会判断抛出的异常是否是想捕获的异常或其子类。 如果异常匹配，Java虚拟机会将控制流跳转到target指向的字节码继续执行；如果不匹配，则继续遍历异常表。如果遍历完所有的异常表还未找到匹配的异常处理器，那么该异常将继续抛到调用方（caller）中重复上述的操作。
  现在的Java编译器采用复制finally代码块的方式，并将其内容插入到try和catch代码块中所有正常退出和异常退出之前
  可以看到，受finally语句return的影响，虽然catch语句中有“return 1;”，在字节码层面只是将1暂存到临时变量中，没有机会执行返回，本例中foo方法的返回值为2。
  try中抛出的异常被finally抛出的异常淹没了，这也很好理解，从上一节介绍的内容可知finally中的代码块会在try抛出异常之前插入，即try抛出的异常被finally抛出的异常捷足先登先返回了。
  第一，try-with-resources语法并不是简单地在finally中加入closable.close（）方法，因为finally中的close方法如果抛出了异常会淹没真正的异常；第二，引入了Suppressed异常，既可以抛出真正的异常又可以调用addSuppressed附带上suppressed的异常。
  可以看到，虽然Java语法上允许我们把成员变量初始化和初始化语句块写在构造器方法之外，最终在编译以后都会统一编译进＜init> 方法
  new、dup、invokespecial对象创建三条指令
  本质上来理解导致必须要有dup指令的原因是＜init> 方法没有返回值，如果＜init>方法把新建的引用对象作为返回值，也不会存在这个问题
  javap输出字节码中的static {} 表示＜clinit> 方法。＜clinit> 不会直接被调用，它在四个指令触发时被调用（new、getstatic、putstatic和invokestatic）

  ## 第3章 字节码进阶

  ❏ 5条方法调用指令的联系和区别

  ❏ JVM方法分派机制与vtable、itable原理

  ❏ 通过HSDB来查看JVM运行时数据

  ❏ invokedynamic指令的介绍及在Lambda表达式中的作用

  ❏ 从字节码角度理解泛型擦除

  ❏ synchronized关键字的字节码原理

  ❏ 反射的底层实现原理

  ### 3.1 方法调用指令

  ❏ invokestatic：用于调用静态方法。

  ❏ invokespecial：用于调用私有实例方法、构造器方法以及使用super关键字调用父类的实例方法等。

  ❏ invokevirtual：用于调用非私有实例方法

  ❏ invokeinterface：用于调用接口方法。

  ❏ invokedynamic：用于调用动态方法。
  编译期无法知道，类似于C++ 中的虚方法。 在调用invokevirtual指令之前，需要将对象引用、方法参数入栈，调用结束对象引用、方法参数都会出栈，如果方法有返回值，返回值会入栈到栈顶
  invokevirtual根据对象的实际类型进行分派（虚方法分派），在运行时动态选择执行具体子类的实现方法。
  这些invoke special的方法是否在虚方法表里面？
  看到这里细心的读者可能会想为什么有了invokevirtual指令还需要invokespecial指令呢？这是出于效率的考虑，invokespecial调用的方法可以在编译期间确定，在JDK 1.0.2之前，invokespecial指令曾被命名为invokenonvirtual，以区别于invokevirtual。例如private方法不会因为继承被子类覆写，在编译期间就可以确定，所以private方法的调用使用invokespecial指令。
  看到这里细心的读者可能会想为什么有了invokevirtual指令还需要invokespecial指令呢？这是出于效率的考虑，invokespecial调用的方法可以在编译期间确定，在JDK 1.0.2之前，invokespecial指令曾被命名为invokenonvirtual，以区别于invokevirtual。例如private方法不会因为继承被子类覆写，在编译期间就可以确定，所以private方法的调用使用invokespecial指令。
  invokeinterface用于调用接口方法，同invokevirtual一样，也是需要在运行时根据对象的类型确定目标方法
  当C++ 类中包含虚方法时，编译器会为这个类生成一个虚方法表（vtable），每个类都有一个指向虚方法表的指针vptr。虚方法表是方法指针的数组，用来实现多态
  在命令行中使用g++ -std=c++11-fdump-class-hierarchy test.cpp会输出A和B的虚方法表，输出结果如下所示。 ￼  Vtable for A￼  A::_ZTV1A: 5u entries￼  0 (int (＊)(...))0￼  8 (int (＊)(...))(& _ZTI1A)￼  16 (int (＊)(...))A::method1￼  24 (int (＊)(...))A::method2￼  32 (int (＊)(...))A::method3￼ ￼  Vtable for B￼  B::_ZTV1B: 6u entries￼  0 (int (＊)(...))0￼  8 (int (＊)(...))(& _ZTI1B)￼  16 (int (＊)(...))A::method1￼  24 (int (＊)(...))B::method2￼  32 (int (＊)(...))A::method3￼  40 (int (＊)(...))B::method4 vtable除了包含虚方法以外，还包含了两个额外元素，这里暂时不用关心
  我们并不知道a指针所指向对象的真正类型，不确定它是A类还是B类抑或是其他A的子类，但可以确定每个method2方法都被放在虚函数表的offset为24的位置上，不会随类型的影响而不同
  在需要调用某个接口方法时，虚拟机会在itable的offset table中查找到对应的方法表位置和方法位置，随后在method table中查找具体的方法实现
  有了itable的知识，接下来看看invokevirtual和invokeinterface指令的区别。前面介绍过invokevirtual的实现依赖于Java的单继承特性，子类的虚方法表保留了父类虚方法表的顺序，但是因为Java的多接口实现，这一特性无法使用
  如果要用invokevirtual调用method3就不能直接从固定的索引位置取得对应的方法。只能搜索整个itable来找到对应方法，使用invokeinterface指令进行调用
  使用HSDB探究多态的原理 HSDB全称是Hotspot Debugger，它是一个内置的JVM工具，可以用来深入分析JVM运行时的内部状态。HSDB位于JDK安装目录下的lib/sa-jdi.jar中
  ❏ vtable、itable机制是实现Java多态的基础。

  ❏ 子类会继承父类的vtable。因为Java类都会继承Object类，Object中有5个方法可以被继承，所以一个空Java类的vtable的大小也等于5。

  ❏ 被final和static修饰的方法不会出现在vtable中，因为没有办法被继承重写，同理可以知道private修饰的方法也不会出现在vtable中。

  ❏ 接口方法的调用使用invokeinterface指令，Java使用itable来支持多接口实现，itable由offset table和method table两部分组成。在调用接口方法时，会先在offset table中查找method table的偏移量位置，随后在method table查找具体的接口实现。
  如果当前类找不到符合条件的函数，会在其父类中继续查找，如果obj所属的类与PrintStream没有继承关系，就算obj所属的类有符合条件的函数，上述调用也不会成功，因为类型检查不会通过。
  关注点在于对象的行为，能做什么，而不是关注对象所属的类型，不关注对象的继承关系。
  MethodHandle又称为方法句柄或方法指针，是java.lang.invoke包中的一个类，它的出现使得Java可以像其他语言一样把函数当作参数进行传递
  1）创建MethodType对象，MethodType用来表示方法签名，每个MethodHandle都有一个MethodType实例，用来指定方法的返回值类型和各个参数类型。 2）调用MethodHandles.lookup静态方法返回MethodHandles.Lookup对象，这个对象表示查找的上下文，根据方法的不同类型通过findStatic、findSpecial、findVirtual等方法查找方法签名为MethodType的方法句柄。 3）拿到方法句柄以后就可以调用具体的方法了，通过传入目标方法的参数，使用invoke或者invokeExact进行方法的调用。 前面介绍的4条invoke* 指令的方法分派规则固化在虚拟机中，invokedynamic则把如何查找目标方法的决定权从虚拟机下放到具体的用户代码中。JRuby作者就给invokedynamic下过一个定义：“Invokedynamic is a user-definable bytecode, You decide how the JVM implements it”，说的也是这个道理。
  ❏ JVM首次执行invokedynamic指令时会调用引导方法（Bootstrap Method）。 ❏ 引导方法返回一个CallSite对象，CallSite内部根据方法签名进行目标方法查找。它的getTarget方法返回方法句柄（MethodHandle）对象。 ❏ 在CallSite没有变化的情况下，MethodHandle可以一直被调用，如果CallSite有变化，重新查找即可。以def add（a, b）{ a + b } 为例，如果在代码中一开始使用两个int入参进行调用，那么极有可能后面很多次调用还会继续使用两个int，这样就不用每次都重新选择目标方法了。
  调用静态方法IndyInterface.bootstrap，返回值是一个CallSite对象，这个函数签名如下所示。 ￼  public static CallSite bootstrap(￼  Lookup caller, // the caller￼  String callType, // the type of the call￼  MethodType type, // the MethodType￼  String name, // the real method name￼  int flags // call flags￼  ) {￼  }
  Groovy采用invokedynamic指令有哪些好处呢？第一是标准化，使用Bootstrap Method、CallSite、MethodHandle机制使得动态调用的方式得到统一。第二是保持了字节码层的统一和向后兼容，把动态方法的分派逻辑下放到语言实现层，未来版本可以很方便地进行优化、修改。第三是高性能，拥有接近原生Java调用的性能，也可以享受到JIT优化等。
  代码清单3-8 MethodHandle示例 ￼  

  ``` java
  public static void main(String[] args) throws Throwable {￼
    MethodHandles.Lookup lookup = MethodHandles.lookup();￼
    MethodType mt = MethodType.methodType(Object.class,￼ Object.class, Object.class);
    CallSite callSite =￼IndyInterface.bootstrap(lookup, "invoke", mt, "add", 0);
    ￼MethodHandle mh = callSite.getTarget();￼
    mh.invokeExact(obj, "hello", "world");￼
  }
  ```

  

  ### 3.2 Lambda表达式的原理

  字节码中出现了一个名为lambda$main$0的静态方法，这段字节码比较简单，翻译为源码如下。 ￼  private static void lambda$main$0() {￼  System.out.println("hello, lambda");￼  }
  ❏ caller：表示JVM提供的查找上下文。 ❏ invokedName：表示调用函数名，在本例中invokedName为“run”。 ❏ samMethodType：表示函数式接口定义的方法签名（参数类型和返回值类型），本例中run方法的签名为“（）void”。 ❏ implMethod：表示编译时生成的Lambda表达式对应的静态方法invokestatic Test. lambda$main$0。 ❏ instantiatedMethodType：一般和samMethodType是一样或是它的一个特例，在本例中是“（）void”。 metafactory方法的内部细节是整个Lambda表达式最复杂的地方
  跟进InnerClassLambdaMetafactory类，看到它在默默生成新的内部类，类名的规则是ClassName$$Lambda$n。其中ClassName是Lambda所在的类名，后面的数字n按生成的顺序依次递增
  使用java -Djdk.internal.lambda.dumpProxyClasses=. Test运行Test类会发现在运行期间生成了一个新的内部类Test$$Lambda$1.class。这个类正是由InnerClassLambdaMetafactory使用ASM字节码技术动态生成的。它实现了Runnable接口，并在run方法里调用了Test类的静态方法lambda$main$0（）
  ❏ Lambda表达式声明的地方会生成一个invokedynamic指令，同时编译器生成一个对应的引导方法（BootstrapMethod）。❏ 第一次执行invokedynamic指令时，会调用对应的引导方法（Bootstrap Method），该引导方法会调用LambdaMetafactory.metafactory方法动态生成内部类。❏ 引导方法会返回一个动态调用CallSite对象，这个CallSite会最终调用实现了Runnable接口的内部类。❏ Lambda表达式中的内容会被编译成静态方法，前面动态生成的内部类会直接调用该静态方法。❏ 真正执行lambda调用的还是用invokeinterface指令。
  Java 8Lambda设计时的考虑以及实现方法。文中提到Lambda表达式可以通过内部类、method handle、dynamic proxies等方式实现，但是这些方法各有优劣。实现Lambda表达式需要达成两个目标，为未来的优化提供最大的灵活性，且能保持类文件字节码格式的稳定
  Lambda表达式采用的方式并不是在编译期间生成匿名内部类，而是提供一个稳定的字节码二进制表示规范
  这种方法把Lambda翻译的策略由编译期间推迟到运行时，未来的JDK会怎样实现Lambda表达式可能还会有变化。

  ### 3.3 泛型与字节码

  泛型可以在编译期帮我们发现一些明显的问题，不好的地方是泛型在设计上因为考虑兼容性等原因，留下了不少缺陷。Java泛型更像是一个语法糖
  ❏ 第0行：加载参数pair到操作数栈上。 ❏ 第1行：调用getfield指令把left值加载到操作栈上，可以看到left字段的字段类型为Object，并不是String。 ❏ 第4行：checkcast指令用来检查对象是否符合给定类型，这里是判断left值是否是java/lang/String类型。如果类型不匹配，checkcast指令会抛出java.lang. ClassCastException异常。
  Java的泛型是在javac编译器中实现的，在生成的字节码中，已经不包含泛型信息。这种在泛型使用时加上类型参数，在编译时再抹掉的过程被称为泛型擦除。
  由泛型附加的类型信息对JVM来说是不可见的，Java编译器会在编译时尽可能地发现可能出错的地方，但也不是万能的
  因为JVM的异常处理是通过异常表来实现的，如果要捕获的异常在编译期无法确定，就无法生成对应的异常表。
  在Java中，Object[] 数组可以是任何数组的父类，假设我们可以创建泛型数组，上面代码中的t的类型Pair＜Integer>[] 在编译以后被擦除为Pair[]，可以被转换赋值为类型为Object[] 的变量objArray，在语法上可以往objArray中存放任意类型的数据。这样原本定义只能存储元素类型为Pair＜Integer> 的数组，结果却可以存放任意类型的数据
  3.4 synchronized的实现原理
  3.4 synchronized的实现原理 synchronized是Java中的关键字，用于定义一个临界区（critical section）。临界区是指一次只能被一个线程执行的代码片段。synchronized保证方法和代码块在同一时刻只有一个线程可以进入临界区
  getfield #2指令获取字段count的值，iconst_1和iadd将取出的值加一，putfield #2指令将更新之后的值写回到字段count中，这是一个典型的read-modify-write的模式。 如果有两个线程同时调用了increase方法，各自通过getfield #2获得了count的值，随后执行了加1，最后各自将更新以后的值写回count中，count就只被加1，丢失了一次加1操作。
  ❏ 第0～2行：将this对象引用入栈，使用dup指令复制栈顶元素，并将它存入局部变量表位置为1的地方，现在栈上还剩下一个this对象引用。 ❏ 第3行：monitorenter指令尝试获取栈顶this对象的监视器锁，如果成功则继续往下执行，如果已经有其他线程的线程持有，则进入等待状态。 ❏ 第4～11行：执行 ++count。 ❏ 第14～15行：将this对象入栈，调用monitorexit释放锁。 ❏ 第19～23行：执行异常处理，我们代码中本来没有try-catch的代码，为什么字节码会加上这段逻辑呢？
  ❏ 用synchronized修饰的非静态方法，监视器锁是当前对象。 ❏ 用synchronized修饰的静态方法，监视器锁是当前类的类对象。 ❏ synchronized（lock）{} 同步代码块，监视器锁是lock对象。
  当线程执行到monitorenter指令时，会尝试获取栈顶对象对应监视器（monitor）的所有权，也就是尝试获取对象的锁。如果此时monitor没有其他线程占有，当前线程会成功获取锁，monitor计数器置为1。如果当前线程已经拥有了monitor的所有权，monitorenter指令也会顺利执行，monitor计数器加1。如果其他线程拥有了monitor的所有权，当前线程会阻塞，直到monitor计数器变为0。 当线程执行monitorexit时，会将监视器计数器减1，计时器值等于0时，锁被释放，其他等待这个锁的线程可以尝试去获取monitor的所有权。
  理解为下面这样的一段Java代码。 ￼  public void _foo() throws Throwable {￼  monitorenter(lock);￼  try {￼  bar();￼  } finally {￼  monitorexit(lock);￼  }￼  }

  ### 3.5 反射的实现原理

  15次调用以后会使用新的逻辑，利用GeneratedMethodAccessor1来调用反射的方法。MethodAccessorGenerator的作用是通过ASM生成新的类sun.reflect.GeneratedMethod-Accessor1。为了查看生成的类的内容，可以使用阿里的arthas工具。修改上面的代码，在main函数的最后加上“System.in.read（）;”让JVM进程不要退出。执行arthas工具中的./as.sh，会要求输入JVM进程
  为什么要设置为0～15次使用native方式来调用，15次以后使用ASM新生成的类来处理反射的调用呢？
  基于性能的考虑，JNI native调用的方式要比动态生成类调用的方式慢20倍左右，但是由于第一次字节码生成的过程比较慢，如果反射仅调用一次的话，采用生成字节码的方式反而比native调用的方式慢3～4倍。为了权衡这两种方式的利弊，Java引入了inflation机制
  3.5.2 反射的inflation机制 很多情况下，反射只会调用一两次，JVM于是想了一个办法，设置了一个sun.reflect.inflationThreshold阈值，默认等于15。当反射方法调用超过15次时（从0开始计算），会使用ASM生成新类，保证后面的调用比native要快。调用次数小于15次的情况下，直接用native的方式来调用，没有额外类的生成、校验、加载的开销。这种方式被称为inflation机制。
  sun.reflect.inflationThreshold，还有一个是是否禁用inflation的属性sun.reflect.noInflation
  第4章 javac编译原理简介
  编译原理是计算机科学皇冠上的明珠，也是程序员心中的一座高峰。
  编译原理与我们的日常工作息息相关，比如SQL的解析、自定义流程编排引擎、界面模板引擎、特定领域语言DSL、Hibernate HQL语句等，通过学习编译原理可以让我们更加深入地理解语言背后的底层机制，提高代码优化的能力
  javac这种将源文件转为字节码的过程在编译原理上属于前端编译，不涉及目标机器码相关的代码的生成和优化。JDK中的javac本身是用Java语言编写的，在某种意义上实现了javac语言的自举。javac没有使用类似YACC、Lex这样的生成器工具，所有的词法分析、语法分析等功能都是自己实现，代码比较精简和高效
  ❏ javac源码调试方法 ❏ javac编译过程的七个阶段和各阶段的作用 ❏ switch-case语句的tableswitch和lookupswitch指令选择的依据 ❏ Java语言规范背后的校验细节 ❏ 重载方法的选择过程
  4.2 javac的七个阶段
  1）parse：读取．java源文件，做词法分析（LEXER）和语法分析（PARSER） 2）enter：生成符号表 3）process：处理注解 4）attr：检查语义合法性、常量折叠 5）flow：数据流分析 6）desugar：去除语法糖 7）generate：生成字节码 接下来我们逐一介绍各项内容
  语法分析是在词法分析的基础上分析单词之间的关系，将其转换为计算机易于理解的形式，生成抽象语法树（Abstract Syntax Tree, AST）
  与其他大多数语言一样，javac也是使用递归下降法（recursive descent）来生成抽象语法树。主要功能由com.sun.tools.javac.parser.JavacParser类完成，语句“int k = i + j;”对应的AST如图4-8所示。
  从JDK6开始，javac支持在编译阶段允许用户自定义处理注解，大名鼎鼎的lombok框架就是利用了这个特性，通过注解处理的方式生成目标class文件，比在运行时反射调用性能明显提升
  以下面的代码为例： ￼  public static void method(Object obj) {￼  System.out.println("method # Object");￼  }￼ ￼  public static void method(String obj) {￼  System.out.println("method # String");￼  }￼ ￼  public static void main(String[] args) {￼  method(null);￼  } 在Java中允许方法重载（overload），但要求方法签名不能一样。调用method（null）实际是调用第二个方法输出“method # String”, javac在编译时会推断出最具体的方法，方法的选择在Resolve类的mostSpecific方法中完成。第二个方法的入参类型String是第一个方法的入参类型Object的子类，会被javac认为更具体
  com.sun.tools.javac.comp.ConstFold类的主要作用是在编译期将可以合并的常量合并，比如常量字符串相加，常量整数运算等，
  com.sun.tools.javac.comp.Infer类的主要作用是推导泛型方法的参数类型
  flow阶段主要用来处理数据流分析，主要由com.sun.tools.javac.comp.Flow类实现，很多编译期的校验在这个阶段完成，下面列举几个常见的场景
  下面这些某种意义上来说都算是语法糖：泛型、内部类、try-with-resources、foreach语句、原始类型和包装类型之间的隐式转换、字符串和枚举的switch-case实现、后缀和前缀运算符（i++ 和 ++i）、变长参数等。
  为什么不直接用ordinal值来作为case值呢？这是为了提供更好的性能，case值中的ordinal值不一定是连续的，通过SwitchMap数组可以把不连续的ordinal值转为连续的case值，编译成更高效的tableswitch指令。
  generate阶段的主要作用是遍历抽象语法树生成最终的Class文件，由com.sun.tools. javac.jvm.Gen类实现，下面列举了几个常见的场景。 1）初始化块代码并收集到＜init> 和＜clinit> 中
  编译器会自动帮忙生成一个构造器方法＜init>，没有写在构造器方法中的字段初始化、初始化代码块都被收集到了构造器方法中
  与static修饰的静态初始化的逻辑一样，javac会将静态初始化代码块和静态变量初始化收集到＜clinit> 方法中。
  2）把字符串拼接语句转换为StringBuilder.append的方式来实现，比如下面的字符串x和y的拼接代码
  3）为synchronized关键字生成异常表，保证monitorenter、monitorexit指令可以成对调用。 4）switch-case实现中tableswitch和lookupswitch指令的选择。 第2章介绍过，switch-case的实现会根据case值的稀疏程度选择tableswitch或者lookupswitch指令来实现，以下面的代码为例。
  我们来分析原因，这两个指令的选择逻辑在com/sun/tools/javac/jvm/Gen.java中，如下所示。 ￼  long table_space_cost = 4 + ((long) hi - lo + 1); // words￼  long table_time_cost = 3; // comparisons￼  long lookup_space_cost = 3 + 2 ＊ (long) nlabels;￼  long lookup_time_cost = nlabels;￼  int opcode =￼  nlabels > 0 &&￼  table_space_cost + 3 ＊ table_time_cost <=￼  lookup_space_cost + 3 ＊ lookup_time_cost￼  ?￼  tableswitch : lookupswitch; 在上面的例子中，nlables等于case值的个数，等于2 , hi表示case值的最大值1 , lo表示case值的最小值0，因此可以计算出使用tableswitch和lookupswitch的时间和空间代价，如下所示。 ￼  // table_space_cost表示tableswitch的空间代价￼  table_space_cost = 4 + (1-0 + 1) = 6￼  // table_time_cost表示tableswitch的时间代价，恒等于 3￼  table_time_cost = 3￼  // lookup_space_cost表示lookupswitch的空间代价￼  lookup_space_cost = 3 + 2 ＊ 2 = 7￼  // lookup_time_cost表示lookupswitch的时间代价￼  lookup_time_cost = 2 tableswitch和lookupswitch的总代价计算公式如下。 ￼  代价 = 空间代价 + 3 ＊时间代价 因此在case值为0 、1时，tableswitch的代价为6 + 3 * 3 = 15, lookupswitch的代价为7 + 3 * 2 = 13, lookupswitch的代价更小，javac选择了lookupswitch作为switch-case的实现指令。

  ### 5.2 顶层方法

  可以看到顶层方法的本质还是被编译为一个类的静态方法，这个类的类名是顶层文件名 +“Kt”

  ### 5.6 默认参数

  其中name是必填参数，sex和age是可选参数，不填写的情况下，sex默认值为0 , age默认值为18。当字段数量增加时，构造方法的数量也会相应增加，写起来非常烦琐。为了解决这种问题，程序员想出了Builder模式等方法，不过还是比较麻烦和臃肿。下面我们来看Kotlin是如何解决这个问题的。
  它的前三个参数与第一个构造器方法相同，第四个参数mask是一个二进制掩码，下面我们来分析这个掩码的作用。第二个构造器方法的字节码逐行分析如下。
  第0～8行是一段完整的逻辑，首先加载mask和常量2到栈顶，随后调用iand指令将mask与2进行二进制与运算，如果等于0则跳转到第9行继续执行。如果不等于0则把sex入参的值赋值为默认值0，这种情况对应没有给sex赋值的情况。 ❏ 第9～18行是一段完整的逻辑，再次加载mask和常量4到栈顶，随后调用iand指令将mask与4进行二进制与运算，如果等于0则跳转到第19行继续执行。如果不等于0则把age入参的值赋值为默认值18
  在上面的代码中，第一个User对象创建对应的mask值为6（b110），表示第二个入参sex和第三个入参age缺失，会给sex和age赋值为默认值。第二个User对象创建对应的mask为4（b100），表示第三个入参age缺失，会将age赋值为默认值。
  Kotlin的默认参数的实现方式是使用了一个整型掩码（mask），记录了此次调用有哪些位置的参数没有赋值，没有赋值的参数就会被设置为预设的默认值。

  #### 5.7 高级for循环

  注意while循环退出的条件是判断first是否与last相等，而不是first是否小于last，这是因为在IntProgression初始化的时候就已经计算好了last的值，可以用效率更高的等于运算在循环中进行比较判断

  ### 5.8 data class

  在Kotlin中，只需一行代码就可以声明一个只包含属性不包含方法的数据类（data class）

  ### 5.9 多返回值

  我们接触的大多数编程语言都是遵循一个方法最多只能有一个返回值，比如Java、C语言等。从汇编角度来理解，C语言的方法调用规约要求返回值通过EAX寄存器返回，如果要返回多个值只能将返回值包装到结构体struct等结构中
  从原理上来看，JVM的方法是不支持多返回值的，如果想返回多个值需要将对象包装到class中。Kotlin在语义上支持多个返回值，如下面的代码所示。
  可以看到Kotlin多返回值本质上是返回了一个对象，根据对象的字段进行赋值，方法返回值还是一个

  ### 5.10 协程的实现原理

  协程（coroutine）的概念在1958年就已经被提出，协程并不是Kotlin独有的特性，很多编程语言都有协程的概念，比如Go、Lua、Python、Javascript等，阿里的开源JDK也在虚拟机级别支持了协程。理解了Kotlin协程的实现原理对理解其他语言的协程也非常有帮助。
  协程可以理解为纯用户态的线程，创建和切换的消耗极低。一个协程可以被“挂起”，把执行权交给另外一个协程，当执行完一段时间以后又可以挂起将执行权交给其他的协程。Coroutine交替执行过程如图5-2所示。
  当一个协程把执行权转交给另外一个协程时，原协程需要保存上下文保存以便下一次可以恢复执行，比如局部变量需要被暂存。
  CPS是Continuation Passing Style的缩写，是指函数执行完以后，不再通过return语句返回给调用方返回值，而是将返回值当作参数，调用Continuation。CPS方法都会有一个额外的Continuation参数，表示该函数之后将要执行的代码。
  如果写过js回调地狱代码，一定对上面的代码不陌生，这种callback的方式写出来的代码很难看，开发起来很痛苦。Kotlin的协程就是“魔改”了编译器，使得开发人员可以用同步的方式写异步代码。
  suspend方法是Kotlin协程的基础，但是在Java虚拟机中并没有suspend这个关键字，也没有支持协程相关的字节码。Kotlin协程是在应用级别实现的，下面我们分几个部分来讲解suspend方法、Continuation和协程状态机。
  可以看到suspend关键字在编译为字节码以后消失了，取而代之的是给suspend方法增加了Continuation参数，这个Continuation参数表示协程接下来要处理什么，这就是前面介绍的CPS机制
  为了更好地理解协程状态机，对上面的代码稍作修改，增加一个局部变量a，代码如下所示。 ￼  suspend fun foo() {￼  val token = getTokenByLogin("zhang", "1234") // 挂起点 1￼  val a = 100;￼  val userInfo = getUserInfo(token); // 挂起点 2￼  processUserInfo(userInfo)￼  println(a)￼  } 上面的代码会生成三个Continuation。 ❏ 第一个Continuation对应foo函数本身，包含了全部五行代码。 ❏ 第二个Continuation对应getTokenByLogin（挂起点1），包含了它之后的四行代码逻辑。 ❏ 第三个Continuation对应getUserInfo（挂起点2），包含了之后的一行代码，值得注意的是它需要捕获局部变量a的值，以便协程恢复时依然能拿到a的值，协程的本质是在每个挂起点保存当前运行的状态。
  在Kotlin中，每个suspend的方法都需要一个Continuation实现，Continuation是通过状态机来实现，Kotlin编译器会把源码中的suspend方法替换为状态机的一部分，suspend方法通过不同状态间传递Continuation来实现协程的切换。以上面的代码为例，构造状态机的第一步是打标签
  其中sm变量（state machine）表示初始的Continuation，这个Continuation包含了foo整个函数内容，label等于初始值0。随后继续执行直到遇到第1个暂停点，状态机的label值被更新为1，执行getTokenByLogin。 当调度进入状态1时，暂停点1和暂停点2中间有一个后面会被访问到的局部变量a，会被状态机暂存下来，同时也能通过状态机sm的result变量拿到状态1的token值，然后使用这个token和sm变量调用getUserInfo，同时状态机label被更新为2。 当调度进入状态2时，通过状态机sm的result可以获得userInfo，通过sm的 $a变量可以恢复之前暂存的a的值。
  Kotlin协程的原理是每个挂起点和初始起点对应的Continuation都会转为状态机的一种状态，协程切换只是状态机切换到另外一种状态，使用CPS机制传递了协程上下文。

  #### 5.11 从字节码分析Kotlin编译器的bug

  可以看到1 .3版本中处理when选项类型不一致的情况时，是直接使用对象相等比较的方式，且kotlinc编译时会提示warning，如下所示
  第6章 ASM和Javassist字节码操作工具
  ❏ ASM包的组成结构 ❏ ASM Core API和Tree Api的使用和区别 ❏ Javassist API介绍 ❏ 利用ASM和Javassist工具修改class文件

  ### 6.1 ASM介绍

  如果你写过class文件的解析代码，就会发现这个过程极其烦琐，更别提增加方法、手动计算max_stack等操作了。
  ASM提供了两种生成和转换类的方法：基于事件触发的core API和基于对象的Tree API，这两种方式可以用XML解析的SAX和DOM方式来对照。 SAX解析XML文件采用的是事件驱动，它不需要一次解析完整个文档，而是按内容顺序解析文档，如果解析时符合特定的事件则回调一些函数来处理事件。SAX运行时是单向的、流式的，解析过的部分无法在不重新开始的情况下再次读取
  Core API中最重要的三个类是ClassReader、ClassVisitor、ClassWriter，字节码操作都是跟这个三个类打交道。 ClassReader是字节码读取和分析引擎，负责解析class文件。采用类似于SAX的事件读取机制，每当有事件发生时，触发相应的ClassVisitor、MethodVisitor等做相应的处理。
  ClassReader会把解析Class文件过程中的事件源源不断地通知给ClassVisitor对象调用不同的visit方法，ClassVisitor可以在这些visit方法中对字节码进行修改，ClassWriter可以生成最终修改过的字节码
  值得注意的是，ClassReader类accept方法的第二个参数flags是一个位掩码（bit-mask），可以选择组合的值有下面这些
  Tree API的方式相比Core API使用起来更简单，但是处理速度会慢30% 左右，同时会消耗更多的内存，在实际使用过程中可以根据场景做取舍
  在实际字节码转换中，经常需要给类新增一个字段以存储额外的信息，在ASM中给类新增一个字段非常简单，以下面的MyMain类为例，使用javac编译为class文件。
  在实际的使用中，为了避免添加的字段名与已有字段重名，一般会增加一个特殊的后缀或者前缀
  我们看看如何使用ASM给类新增一个方法。 3．新增方法 这里同样以MyMain类为例，给这个类新增一个xyz方法
  根据前面的知识可以知道xyz方法的签名为（ILjava/lang/String;）V
  如果仔细观察ClassVisitor类的visit方法，会发现visitField、visitMethod等方法是有返回值的，如果这些方法直接返回null，这些字段、方法将从类中被移除，代码如下所示。
  修改方法内容 前面有接触到MethodVisitor类，这个类用来处理访问一个方法触发的事件，与ClassVisitor一样，它也有很多visit方法，这些visit方法也有一定的调用时序，常用的如下所示。
  为了替换foo的方法体，一个可选的做法是在ClassVisitor的visitMethod方法返回null以删除原foo方法，然后在visitEnd方法中新增一个foo方法
  从源代码可以分析出，最大栈的大小为2（a, 100），局部变量表的大小为2（this, a）。一个可选的办法是在“mv.visitEnd（）;”代码之前新增“mv.visitMaxs（2, 2）;”以手动指定stack和locals的大小。 另一个方法是让ASM自动计算stack和locals，这与ClassWriter构造器方法参数有关
  在Java 6之后JVM在class文件的Code属性中引入了StackMapTable属性，作用是为了提高JVM在类型检查时验证过程的效率，里面记录的是一个方法中操作数栈与局部变量区的类型在一些特定位置的状态。
  因为stack和locals的计算复杂、容易出错，在正常使用中强烈建议使用ASM的COMPUTE_MAXS和COMPUTE_FRAMES，虽然有一点点性能损耗，但是代码更加清晰易懂，且更易于维护。
  ❏ onMethodEnter：方法开始或者构造器方法中父类的构造器调用以后被回调。 ❏ onMethodExit：正常退出和异常退出时被调用。正常退出指的是遇到RETURN、ARETURN、LRETURN等方法正常返回的情况。异常退出指的是遇到ATHROW指令，有异常抛出方法返回的情况。
  6.2 Javassist介绍
  ASM入门门槛还是挺高的，需要跟底层的字节码指令打交道，优点是小巧、性能好。Javassist是一个性能比ASM稍差但使用起来简单很多的字节码操作库，不需要使用者掌握字节码指令
  在Javassist中每个需要编辑的class都对应一个CtClass实例，CtClass的含义是编译时的类（compile time class），这些类会存储在ClassPool中。ClassPool是一个容器，存储了一系列CtClass对象
  比如上面的a和b对应局部变量表1和2的位置。在Javassist中访问方法参数使用$0 $1 ...，而不是直接使用变量名，将上面的代码修改为如下所示。

  ## 第7章 JavaInstrumentation原理

  Instrumentation看起来比较神秘，很少有书会详细介绍。
  ❏ APM产品：Pinpoint、SkyWalking、newrelic等。 ❏ 热部署工具：Intellij idea的HotSwap、Jrebel等。 ❏ Java诊断工具：Arthas等。
  ❏ Instrumentation基本概念 ❏ Instrumentation的核心API和它的两种使用方式介绍 ❏ Attach API的使用 ❏ Unix域套接字的原理

  ### 7.1 Java Instrumentation简介

  java.lang.instrument包，开发者可以更方便的实现字节码增强。其核心功能由java.lang.instrument.Instrumentation提供，这个接口的方法提供了注册类文件转换器、获取所有已加载的类等功能，允许我们在对已加载和未加载的类进行修改，实现AOP、性能监控等功能。
  调用addTransformer注册transformer以后，后续所有JVM加载类都会被它的transform方法拦截，这个方法接收原类文件的字节数组，在这个方法中可以做任意的类文件改写，最后返回转换过的字节数组，由JVM加载这个修改过的类文件。如果transform方法返回null，表示不对此类做处理
  Instrumentation接口的retransformClasses方法对JVM已经加载的类重新触发类加载。getAllLoadedClasses方法用于获取当前JVM加载的所有类对象。isRetransform-ClassesSupported方法返回一个boolean值表示当前JVM配置是否支持类重新转换的特性。
  Instrumentation有两种使用方式：第一种方式是在JVM启动的时候添加一个Agent的jar包；第二种方式是在JVM运行以后在任意时刻通过Attach API远程加载Agent的jar包。接下来分开进行介绍。
  7.2 Instrumentation与 -javaagent启动参数
  Instrumentation的第一种使用方式是通过JVM的启动参数 -javaagent来启动，一个典型的使用方式如下所示。 ￼  java -javaagent:myagent.jar MyMain 为了能让JVM识别到Agent的入口类，需要在jar包的MANIFEST.MF文件中指定Premain-Class等信息，一个典型的生成好的MANIFEST.MF内容如下所示。
  在premain方法中可以对class文件进行修改。这种机制可以认为是虚拟机级别的AOP，无须对原有应用做任何修改就可以实现类的动态修改和增强
  静态Instrumentation的处理过程如下，JVM启动后会执行Agent jar中的premain方法，在premain中可以调用Instrumentation对象的addTransformer方法注册ClassFile-Transformer。当JVM加载类时会将类文件的字节数组传递给transformer的transform方法，在transform方法中可以对类文件进行解析和修改，随后JVM就可以加载转换后的类文件
  下面使用Instrumentation的方式实现一个简单的方法调用栈跟踪，在每个方法进入和结束的地方都打印一行日志，实现调用过程的追踪效果
  通过上面的方式，我们在不修改MyTest类源码的情况下实现了调用链跟踪的效果，这个例子是APM实现的原型，更加完善的调用链跟踪实现会在后面的APM章节中介绍。

  ### 7.3 JVM Attach API介绍

  在JDK5中，开发者只能在JVM启动时指定一个javaagent，在premain中操作字节码，这种Instrumentation方式仅限于main方法执行前，存在很大的局限性。从JDK6开始引入了动态Attach Agent的方案，可以在JVM启动后任意时刻通过Attach API远程加载Agent的jar包，比如大名鼎鼎的arthas工具就是基于Attach API实现的。
  加载Agent的jar包只是Attach API的功能之一，我们常用jstack、jps、jmap工具都是利用Attach API来实现的
  因为是跨进程通信，Attach的发起端是一个独立的java程序，这个java程序会调用VirtualMachine.attach方法开始和目标JVM进行跨进程通信。
  JVM Attach API的实现主要基于信号和UNIX域套接字，接下来详细介绍这两部分的内容
  信号是某事件发生时对进程的通知机制，也被称为“软件中断”。信号可以看作一种非常轻量级的进程间通信，信号由一个进程发送给另外一个进程，只不过是经由内核作为一个中间人转发，信号最初的目的是用来指定杀死进程的不同方式。 每个信号都有一个名字，以“SIG”开头，最熟知的信号应该是SIGINT，我们在终端执行某个应用程序的过程中按下Ctrl+C一般会终止正在执行的进程，这是因为按下Ctrl+C会发送SIGINT信号给目标程序
  在Linux中，一个前台进程可以使用Ctrl+C进行终止，对于后台进程需要使用kill加进程号的方式来终止，kill命令是通过发送信号给目标进程来实现终止进程的功能。默认情况下，kill命令发送的是编号为15的SIGTERM信号，这个信号可以被进程捕获，选择忽略或正常退出。目标进程如果没有自定义处理这个信号就会被终止。对于那些忽略SIGTERM信号的进程，可以指定编号为9的SIGKILL信号强行杀死进程，SIGKILL信号不能被忽略也不能被捕获和自定义处理。
  这种情况下在终端中输入Ctrl+C, kill -3, kill -15都没有办法杀掉这个进程，只能用kill -9，如下所示。
  JVM对SIGQUIT的默认行为是打印所有运行线程的堆栈信息，在类UNIX系统中，可以通过使用命令kill -3pid来发送SIGQUIT信号
  接下来我们来看UNIX域套接字（UNIX Domain Socket）的概念。使用TCP和UDP进行socket通信是一种广为人知的socket使用方式，除了这种方式以外还有一种称为UNIX域套接字的方式，可以实现同一主机上的进程间通信。虽然使用127.0.01环回地址也可以通过网络实现同一主机的进程间通信，但UNIX域套接字更可靠、效率更高
  Docker守护进程（Docker daemon）使用了UNIX域套接字，容器中的进程可以通过它与Docker守护进程进行通信。MySQL同样提供了域套接字进行访问的方式。
  ❏ UNIX域套接字更加高效，它不用进行协议处理，不需要计算序列号，也不需要发送确认报文，只需要读写数据即可。 ❏ UNIX域套接字是可靠的，不会丢失数据，普通套接字是为不可靠通信设计的。 ❏ UNIX域套接字的代码可以非常简单地修改为普通套接字。
  启动后会在当前目录生成一个名为tmp.sock的UNIX域套接字文件，它读取客户端写入的内容并输出到终端。server.c的代码如下面代码
  SIGQUIT的默认行为是dump当前的线程堆栈，那为什么调用VirtualMachine.attach没有输出调用栈堆栈呢？ 假设目标进程为12345, Attach的详细过程如下。 1）Attach端先检查临时文件目录是否有．java_pid12345文件。 这个文件是一个UNIX域套接字文件，由Attach成功以后的目标JVM进程生成。如果这个文件存在，说明正在Attach中，可以用这个socket进行下一步的通信。如果这个文件不存在则创建一个．attach_pid12345文件，
  如果从socket的角度来看，VirtualMachine.attach方法相当于三次握手建立连接，Virtual-Machine.loadAgent则是握手成功之后发送数据，VirtualMachine.detach相当于四次挥手断开连接。

  ### 7.4 小结

  本章介绍了Java Instrumentation相关的概念，重点介绍两种不同的Instrumentation方式，即JVM启动时加载的premain方法和启动后动态Attach的agentmain方法。文章最后分析了Attach API的底层实现原理。这一章的知识是后面APM、软件破解章节的基础，希望你可以熟练掌握。下一章我们将介绍字节码相关的典型应用场景。

  ## 第8章 JSR269插件化注解处理原理

  Lombok框架就是基于JSR 269来实现的，通过简单的注解减少了手动实现get、set、toString等方法，消除了大量的冗余代码。 通过阅读本章你会学到如下知识： ❏ JSR 269的基本概念。 ❏ 抽象语法树操作有关的核心类Names、JCTree、TreeMaker的使用。 ❏ 自定义注解处理实现自动添加get、set方法。 ❏ JSR 269在Lombok和ButterKnife框架上的应用。

  ### 8.1 JSR 269 简介

  JDK1.6引入了JSR 269规范，允许开发者在编译期间对注解进行处理，可以读取、修改、添加抽象语法树中的内容。只要有足够的想象力，利用JSR269可以完成很多Java语言不支持的特性，甚至创造新的语法糖。
  实现注解处理器的第一步是继承AbstractProcessor类，实现它的process方法

  ### 8.3 JSR 269 在常用框架上的应用

11. Lombok使用示例 Lombok提供的注解类比较多，常用的注解如下所示。 ❏ @Getter、@Setter：为字段生成get和set方法。 ❏ @ToString：生成toString方法。 ❏ @EqualsAndHashCode：生成equals和hascode方法。 ❏ @NoArgsConstructor：生成无参构造器方法。 ❏ @AllArgsConstructor：生成一个全参构造函数。 ❏ @Slf4j、@Log4j：自动生成日志框架对象。 ❏ @Data：为字段生成get、set方法，并生成equals、hashCode、toString方法。

   ## 第9章 字节码的应用

   本章会介绍字节码在cglib、Fastjson、Dubbo、JaCoCo、Mock这些框架上的应用。

   ### 9.1 cglib动态代理原理分析

   如果说ASM是字节码改写的标准，那么cglib则是动态代理事实上的标准
   ❏ Spring：为AOP框架提供方法拦截。 ❏ MyBatis：用来生成Mapper接口的动态代理实现类。 ❏ Hibernate：用来生成持久化相关的类。 ❏ Guice、EasyMock、jMock等。 本节会分为cglib动态代理核心API介绍和实现原理两部分，接下来先来看看cglib核心API介绍。
   代码一开始新建了一个匿名内部类实现MethodInterceptor接口，在intercept方法的开始和结束处增加日志打印，在开始和结束中间使用methodProxy.invokeSuper调用父类方法。接下来调用Enhancer.create方法传入Person.class对象和MethodInterceptor对象，返回Person子类对象。最后使用person子类对象调用目标方法
   MethodInterceptor接口只有一个intercept方法，所有的代理方法都会调用这个intercept方法，而不是原方法。这个方法的第一个参数obj是被代理的原对象，第二个参数method是当前拦截的方法，第三个参数是方法的参数集合，最后一个参数proxy是一个MethodProxy类型的变量，表示被调用方法的代理。调用proxy对象的invokeSuper方法相当于调用原有方法。MethodInterceptor作为一个桥梁连接了目标对象和代理对象
   前面提到cglib动态代理是通过生成子类的方式来实现的，接下来我们看看生成的子类的庐山真面目。设置系统属性让cglib将生成的文件保存到磁盘中，如下所示。 ￼  System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "/path/to/￼  cglib-debug-location");
   可以看到cglib生成了一个Person的子类Person$$EnhancerByCGLIB$$a1da8fe5，覆写了doJob方法。此方法会调用MethodInterceptor的intercept方法，由intercept方法先输出“>>>>>before intercept”，随后使用invokeSuper方法调用父类（也即真正的Person类）的doJob方法，最后输出“>>>>>end intercept”。
   JDK动态代理使用反射机制来调用被拦截的方法，效率较低。cglib使用了一种FastClass的机制来规避反射调用。它的原理是对被代理类的方法增加索引，通过索引值可以直接定位到具体的方法
   可以看到FastClass的原理不过是针对具体的代理类生成了代理方法的索引值，在invoke方法通过switch-case语句找到对应的目标方法，让目标对象直接调用目标方法，规避了反射调用。

   ### 9.2 字节码在Fastjson上的应用

   ❏ 实现了类似StringBuilder的工具类SerializeWriter，减少了很多次数组越界检查，比Java内置的StringBuilder字符串拼接效率更好。 ❏ 独创的提升JSON反序列化性能的快速匹配算法，是性能超越其他框架的关键。 ❏ 在SerializeWriter、JSONScanner类中使用ThreadLocal来缓存byte[]/char[]，减少了内存分配和GC次数。 ❏ 使用ASM动态生成序列化、反序列化字节码，避免了反射的开销。
   可以看到对于序列化，Fastjson也是使用ASM为对应的JavaBean生成了一个对应的类，规避了反射获取类字段的操作。

   ### 9.3 字节码在Dubbo上的应用

   性能测试中，性能Javassit > cglib > JDK。

   ### 9.4 字节码在JaCoCo代码覆盖率上的应用

   JaCoCo这个名字是Java Code Coverage的缩写，即Java代码覆盖率。它支持多种维度覆盖率统计
   JaCoCo是通过字节码注入探针代码来实现测试覆盖率统计的，它有Offline、On-The-Fly这两种字节码插桩方式。 ❏ Offline：在生成目标文件之前先对字节码文件进行插桩，然后执行插桩后的字节码文件生成覆盖率报告。 ❏ On-The-Fly：通过指定JVM的启动参数javaagent，使用Agent的Jar包来启动Instrumentation代理程序，在类加载前利用ASM对字节码文件进行修改，JVM执行修改过的字节码文件生成测试覆盖率报告

   ### 9.5 字节码在Mock上的应用

   利用子类继承的方式无法实现静态、final、私有方法的Mock
   使用自定义类加载器和javassist字节码操作来实现Mock静态方法、final方法、私有方法的功能。

   ### 9.6 小结

   ❏ cglib使用ASM生成了目标代理类的一个子类，在子类中扩展父类方法，达到代理的功能，因此要求代理的类不能是final的。 ❏ Fastjson使用ASM生成了实例Bean反序列化类，彻底去掉了反射的开销，使性能更上一层楼。 ❏ Dubbo使用Javassist生成接口的代理类。 ❏ Jacoco使用ASM在代码分支中插入探针字节码实现覆盖率统计功能。 ❏ Mockito和EasyMock使用ByteBuddy、cglib生成子类的方式来实现Mock的功能，PowerMock利用Javassist字节码可以实现静态、final、私有方法的Mock。

   ### 10.2 软件破解

   jar包本质上就是一个zip压缩包，用unzip命令将jar包解压到一个临时文件夹tmp中，对应的目录结构如下所示。
   “return true;”语句对应的字节码语句如下所示。 ￼  ICONST_1￼  IRETURN

   ### 10.3 软件防破解

   可以看到0 xCA与0 xFF经过两次XOR运算以后，得到原值0 xCA。介绍完加解密的运算后，接下来看实际的代码实现。 1）首先新建一个MyService.java文件，使用javac编译为MyService.class文件。 ￼  public class MyService {￼  public MyService() {￼  System.out.println("init MyService");￼  }￼  } 2）将MyService.class文件进行一次XOR运算，保存为“MyService.class_”文件。 3）复制MyService.class_ 文件并放到项目的encrypt_classes目录。 4）实现一个自定义ClassLoader，代码如下所示。
   JNI是Java Native Interface（Java本地接口）的缩写，是JVM提供的Java代码调用native模块的一种特性，通过JNI可以让Java和C/C++ 代码互相调用。 接下来还是以上一个小节中的自定义ClassLoader为例，使用C/C++ 代码替换类文件中Java的解密逻辑

   ### 11.1 全链路分布式跟踪介绍

   每个请求都有对应的Trace ID、Span ID、Parent ID, Trace ID用来标识一次用户请求，所有链路上的子过程节点都共用这一个Trace ID, Span ID用来标识一次处理过程，Parent ID用来标识处理过程的父节点

   #### 11.1.2 OpenTracing基本术语

   下面介绍几种常用的OpenTracing基本术语。 1. Tracer Tracer表示一次调用链，由多个Span组成。 2. Span Span表示一段处理阶段，可以是一个方法调用，一次HTTP请求，一次数据库操作。一个Span会包含下面这些状态值。 ❏ Span名，一般是函数名，或者某个关键操作的名字。 ❏ SpanContext上下文，包括Trace ID、SpanID等。 ❏ 开始、结束时间戳。

   11.2 见微知著之APM
   APM是Application Performance Managment的缩写，字面意思是应用性能管理
   我们可以通过浏览器获取关键的业务指标：页面加载时间、首屏时间、页面渲染时间等。在chrome console里输入window.performance.timing可以获取到本次访问各阶段详细的耗时
   12.1 dex文件结构
   除此之外，class文件采用大端字节序存储，dex文件默认采用小端字节序存储。
   前4个字节是“dex\n”，紧随其后的4个字节表示dex文件的版本号，示例文件的版本号是035\0。 2．校验和（checksum） 接下来的4个字节表示dex文件的校验码。提到文件的校验码一般都会想到CRC32、SHA128算法等，dex文件用的是一个称为Adler-32的算法。Adler-32算法由Mark Adler在1995提出，这个算法与CRC32一样，都是用于计算数据的校验和，防止数据被篡改。它的安全性比CRC32要弱，但是计算却要快很多，在rsync和zlib中都有应用
   3．签名（signature） 接下来的20个字节表示dex文件的签名，由SHA-1算法生成，用于判断dex是否被篡改和校验合法性。计算的区域是除去魔数、校验码、签名以外的所有文件内容，也就是除文件头部32字节以外的所有数据。SHA-1算法生成的哈希值由20个字节表示，不同的文件计算的哈希值冲突的可能性非常小。 4．文件大小（file_size） 接下来的4个字节表示dex文件的长度，
   6．大小端字节序（endian_tag） 表示文件内容字节序，默认值为0 x12345678，表示小端字节序，如果这个值为0x78563412，则表示大端字节序
   LEB128是Little Endian Based 128的缩写，是一种小端字节序、变长的编码格式，分为unsigned LEB128和signed LEB128两种类型。
   ❏ 在Java中，方法的描述符用形如“（参数类型列表）返回值类型；”的方式表示，比如String foo（int x）的方法描述符为（I）Ljava/lang/String;。 ❏ 在Android中，方法的ShortyDescriptor的表示方式是“返回值类型+参数类型”，如果没有方法参数则只包含返回值类型即可。引用类型直接用字母“L”代替，而不用像Java那样用“L全限定名；”的方式表示方法描述符。
   ❏ direct_methods表示非虚方法的列表，每个元素类型为encoded_method，本例中非虚方法的个数为3个，这三个方法分别是默认构造器＜init>、isEmpty和main方法。 ❏ virtual_methods表示虚方法的列表，每个元素类型为encoded_method，这里虚方法的数量为0。
   code_item各部分结构释义如下。 ❏ registers_size：表示方法需要用到寄存器个数，这里的值为0 x0002，该方法所需的寄存器个数为2。 ❏ ins_size：方法入参所占空间大小，这里值为0 x0001。 ❏ outs_size：方法内部调用其他方法传参需要占用的空间大小，这里值为0 x0001。 ❏ tries_size：表示方法try_item的个数，因为代码中没有try语句，这里的值等于0。 ❏ debug_info_off：方法调试信息（行号、局部变量信息）的偏移量，这里的值为740（0x02E4）。
   dex字节码把判断对象是否等于null翻译为if-eqz或者if-nez，这与Java字节码区别较大。 ❏ 第2行：使用invoke-virtual指令调用v1寄存器对应对象s的length方法。 ❏ 第3行：调用move-result指令将上一个命令的返回值存储到寄存器v0中，这个寄存器被当作返回值。

   ### 12.2 Android字节码

   Java虚拟机（比如HotSpot）是基于栈的虚拟机，而Android虚拟机则是基于寄存器的，这两种架构各有优劣。 ❏ 基于栈的虚拟机：程序运行时需要频繁入栈、出栈，实现同样的功能需要的指令数量会更多。 ❏ 基于寄存器的虚拟机：数据的访问通过寄存器可以直接访问，速度更快。这种方式下，指令必须指定源和目标寄存器，指令会更大。ARM架构决定了dex字节码可用的寄存器个数最多可以有64k个。
   对比Java字节码的8条指令，dex字节码指令只有3条。不同的是dex字节码包含了4个寄存器v0、v1、v2、v3，其中v0表示返回值，v1表示this, v2表示入参x, v3表示入参y。
   ❏ add-int v0, v2, v3：将v2和v3相加并将结果保存到v0中，这一步是计算x+y，并将结果保存到v0寄存器中。 ❏ mul-int/lit8v0, v0, #int 2：将v0的值与2相乘并将结果放到寄存器v0中。 ❏ return v0：将此存器v0当作返回值结果返回。
   dex字节码的指令风格类似于汇编语言，汇编语言有Intel和AT&T两种语法风格，dex指令格式倾向于Intel风格，将目标操作数放在最前面，赋值方向从右往左，以add-int指令为例，运算赋值过程如下
   ❏ 第1行：if-ltz的指令格式为“if-ltz vx, target”，表示如果vx的值小于0则跳转到target处。这一行指令是将寄存器v1与0做比较，如果v1小于0则跳转到0003处继续执行。 ❏ 第2行：调用return指令返回v1寄存器的值。 ❏ 第3行，处理v0小于0的条件分支，neg-int指令的格式为“neg-int v1, v0”，将 -v0的值存储到寄存器v1中，这里“neg-int v1, v1”表示将 -v1的值赋值给v1
   在Java字节码中，switch底层实现会根据case的稀疏程度，分别使用tableswitch、lookupswitch指令来实现。在dex字节码中也是一样，根据case值稀疏的程度采用packed-switch和sparse-switch这两个指令来实现。
   ❏ 第0000行：调用静态方法tryItOut1（），如果有异常抛出，就去tries数组中依次遍历查找是否有匹配的异常，如果无异常则继续执行。 ❏ 第0003行：调用return方法返回。 ❏ 第0004行：对应MyException1异常处理流程，move-exception指令格式为“move-exception vx”，将抛出的异常对象引用赋值给寄存器vx。 ❏ 第0005行：调用handleException静态方法。 ❏ 第0008行：跳转到0003行，退出方法。 ❏ 第0009行：对应MyException2异常处理流程。 ❏ 第000a行：调用handleException2静态方法。 ❏ 第000d行：跳转到0003行，退出方法。
   dex字节码维护了一个tries数组，作用相当于Java字节码的异常表，当出现异常时去这个数组遍历查找，如果命中则跳转到相应的handler进行处理，如果遍历完数组依然没有命中，则继续往上抛出。

   ### 12.3 Gradle插件编写

   构建独立的Gradle插件项目需要在main目录新建resources/META-INF/gradle-plugins/apmplugin.properties文件，这个文件必须存在才能让插件被项目识别。这个．properties文件的文件名是app在build.gradle中引入时使用的名字，比如apply plugin: 'apmplugin'。这个．properties文件的作用是指明插件的入口文件
   为了能让其他项目访问到这个plugin，需要将项目打包到maven仓库，这里选择了本地maven仓库，在build.gradle中新增下面这段配置，这样执行uploadArchives任务时，就可以将打包好的plugin生成到本地的一个路径中