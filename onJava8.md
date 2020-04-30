### 18-String

| `.`            | 任意字符                                       |
| -------------- | ---------------------------------------------- |
| `[abc]`        | 包含`a`、`b`或`c`的任何字符（和`a              |
| `[^abc]`       | 除`a`、`b`和`c`之外的任何字符（否定）          |
| `[a-zA-Z]`     | 从`a`到`z`或从`A`到`Z`的任何字符（范围）       |
| `[abc[hij]]`   | `a`、`b`、`c`、`h`、`i`、`j`中的任意字符（与`a |
| `[a-z&&[hij]]` | 任意`h`、`i`或`j`（交）                        |
| `\s`           | 空白符（空格、tab、换行、换页、回车）          |
| `\S`           | 非空白符（`[^\s]`）                            |
| `\d`           | 数字（`[0-9]`）                                |
| `\D`           | 非数字（`[^0-9]`）                             |
| `\w`           | 词字符（`[a-zA-Z_0-9]`）                       |
| `\W`           | 非词字符（`[^\w]`）                            |



| 逻辑操作符 | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| `XY`       | `Y`跟在`X`后面                                               |
| `X         | Y`                                                           |
| `(X)`      | 捕获组（capturing group）。可以在表达式中用`\i`引用第i个捕获组 |



| 边界匹配符 | 含义             |
| ---------- | ---------------- |
| `^`        | 一行的开始       |
| `$`        | 一行的结束       |
| `\b`       | 词的边界         |
| `\B`       | 非词的边界       |
| `\G`       | 前一个匹配的结束 |



#### 量词

量词描述了一个模式捕获输入文本的方式：

- **贪婪型**： 量词总是贪婪的，除非有其他的选项被设置。贪婪表达式会为所有可能的模式发现尽可能多的匹配。导致此问题的一个典型理由就是假定我们的模式仅能匹配第一个可能的字符组，如果它是贪婪的，那么它就会继续往下匹配。
- **勉强型**： 用问号来指定，这个量词匹配满足模式所需的最少字符数。因此也被称作懒惰的、最少匹配的、非贪婪的或不贪婪的。
- **占有型**： 目前，这种类型的量词只有在 Java 语言中才可用（在其他语言中不可用），并且也更高级，因此我们大概不会立刻用到它。当正则表达式被应用于 `String` 时，它会产生相当多的状态，以便在匹配失败时可以回溯。而“占有的”量词并不保存这些中间状态，因此它们可以防止回溯。它们常常用于防止正则表达式失控，因此可以使正则表达式执行起来更高效。

| 贪婪型   | 勉强型    | 占有型    | 如何匹配                    |
| -------- | --------- | --------- | --------------------------- |
| `X?`     | `X??`     | `X?+`     | 一个或零个`X`               |
| `X*`     | `X*?`     | `X*+`     | 零个或多个`X`               |
| `X+`     | `X+?`     | `X++`     | 一个或多个`X`               |
| `X{n}`   | `X{n}?`   | `X{n}+`   | 恰好`n`次`X`                |
| `X{n,}`  | `X{n,}?`  | `X{n,}+`  | 至少`n`次`X`                |
| `X{n,m}` | `X{n,m}?` | `X{n,m}+` | `X`至少`n`次，但不超过`m`次 |

应该非常清楚地意识到，表达式 `X` 通常必须要用圆括号括起来，以便它能够按照我们期望的效果去执行



matches()：永远匹配整个字符串

find()：是指找子串

matches() 和 find() 会相互影响,可以使用 reset 让它恢复到最初始的状态

lookingAt()：每次都从头上还是找



### 19-类型信息

类字面常量（Test.class）不仅可以应用于普通类，也可以应用于接口、数组以及基本数据类型。另外，对于基本数据类型的包装类，还有一个标准字段 `TYPE`。`TYPE` 字段是一个引用，指向对应的基本数据类型的 `Class` 对象，如下所示：

| ...等价于...  |                |
| ------------- | -------------- |
| boolean.class | Boolean.TYPE   |
| char.class    | Character.TYPE |
| byte.class    | Byte.TYPE      |
| short.class   | Short.TYPE     |
| int.class     | Integer.TYPE   |
| long.class    | Long.TYPE      |
| float.class   | Float.TYPE     |
| double.class  | Double.TYPE    |
| void.class    | Void.TYPE      |

#### 类初始化

1. **加载**，这是由类加载器执行的。该步骤将查找字节码（通常在 classpath 所指定的路径中查找，但这并非是必须的），并从这些字节码中创建一个 `Class` 对象。
2. **链接**（**验证，准备，解析**）。在链接阶段将验证类中的字节码，为 `static` 字段分配存储空间，并且如果需要的话，将解析这个类创建的对其他类的所有引用。
3. **初始化**。如果该类具有超类，则先初始化超类，执行 `static` 初始化器和 `static` 初始化块。

> 直到第一次引用一个 `static` 方法（**构造器隐式地是 `static`**）或者非常量（**非编译器常量**）的 `static` 字段，才会进行类初始化。



#### 添加泛型语法的原因只是为了提供编译期类型检查（将运行时检查提前到编译时）

> 泛型语法用于Class对象时，newInstance将返回该Class对象的确切类型，而不仅仅不是泛型语法返回的Object，但这在使用泛型通配符时可能会受限
>
> **Integer is Numbe，但 Class<Integer> is not a Class<Number>**

` Class<? super Test> clazz = ...`

`Object obj = clazz.newInstance()`	// 由于编译器不知道应该返回Test的哪种父类，所以只能返回Object



#### RTTI（编译器在编译时会打开并检查 `.class` 文件）

1. 传统的类型转换，如 “`(Shape)`”，由 RTTI 确保转换的正确性，如果执行了一个错误的类型转换，就会抛出一个 `ClassCastException` 异常。
2. 代表对象类型的 `Class` 对象. 通过查询 `Class` 对象可以获取运行时所需的信息.
3. TTI 在 Java 中还有第三种形式，那就是关键字 `instanceof`



#### 反射（.class不可用，运行时环境打开并检查）

> 类 `Class` 支持*反射*的概念， `java.lang.reflect` 库中包含类 `Field`、`Method` 和 `Constructor`（每一个都实现了 `Member` 接口）。这些类型的对象由 JVM 在运行时创建，以表示未知类中的对应成员。然后，可以使用 `Constructor` 创建新对象，`get()` 和 `set()` 方法读取和修改与 `Field` 对象关联的字段，`invoke()` 方法调用与 `Method` 对象关联的方法。此外，还可以调用便利方法 `getFields()`、`getMethods()`、`getConstructors()` 等，以返回表示字段、方法和构造函数的对象数组。**因此，匿名对象的类信息可以在运行时完全确定，编译时不需要知道任何信息。**

``` java
// 查看该类所有方法
private static Pattern p = Pattern.compile("\\w+\\.");
public static void main(String[] args) {
    if (args.length < 1) {
        System.out.println(usage);
        System.exit(0);
    }
    int lines = 0;
    try {
        Class<?> c = Class.forName(args[0]);
        Method[] methods = c.getMethods();
        Constructor[] ctors = c.getConstructors();
        if (args.length == 1) {
            for (Method method : methods)
                System.out.println(
                p.matcher(
                    method.toString()).replaceAll(""));
            for (Constructor ctor : ctors)
                System.out.println(
                p.matcher(ctor.toString()).replaceAll(""));
            lines = methods.length + ctors.length;
        } else {
            for (Method method : methods)
                if (method.toString().contains(args[1])) {
                    System.out.println(p.matcher(
                        method.toString()).replaceAll(""));
                    lines++;
                }
            for (Constructor ctor : ctors)
                if (ctor.toString().contains(args[1])) {
                    System.out.println(p.matcher(
                        ctor.toString()).replaceAll(""));
                    lines++;
                }
        }
    } catch (ClassNotFoundException e) {
        System.out.println("No such class: " + e);
    }
}
```



#### JDK动态代理（只能代理接口）

``` java
// 接口
interface Inter {
    void proxyMethod();
}

// 接口的实现类
class Impl implements Inter {
    @Override
    public void proxyMethod() {
        return;
    }
}

// JDK动态代理需实现InvocationHandler
class Proxyer implements InvocationHandler {
    private Object p;
    public Proxyer(Object p) { this.p = p }
    // 重写invoke()
    @Override
    public Object invoke(Object proxy, Method m, Object[] args) throw throwable {
        return m.invoke(proxy, args);
    }
}

class Demo {
    main(String[] args) {
        // 类加载器，接口的实现类，代理对象
        Inter proxy = (Integer)Proxy.newInstance(Proxys.class.getClassLoader(), 
                         new class[] {Impl.class},
                         new Proxyer(new Impl()));
        proxy.invoke(proxyMethod());
    }
}
```

