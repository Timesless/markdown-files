### 基础知识

+ C语言`&`作为取地址符，来取变量存储在内存地址的首地址。假设a = 55，分配内存地址为`0x0012FF7C`，那么`&a`结果为`0X0012FF7C`。那如何获取55呢？
+ C语言提供`*`，用来取出地址里面的值。`&a`代表地址，显然`*&a`可以取出55

``` c
#include <stdio.h>
int main() {
    int a = 65;		// 假设内存首地址为“0x0012ff7c”
    int addr;
    
    addr = 0x0012ff7c;	// 将a的首地址存入变量addr作为addr的值，addr分配地址应该是0x001277f8
    printf("0x%p, 0x%p, 0x%p\n", &a, addr, &addr);
    printf("%d, %d, 0x%p", a, *&a, *&addr);
    printf("%d", *(int*)addr);	// 65
    return 0;
}
```

> `*&addr`：&addr取addr变量的地址，即`0x0012ff7c`，`*&addr`则取地址的值为65
>
> `*addr`原本应该取出`0x0012ff7c`地址的值65，但它们（值和地址）对运算的反应不一样（运算符重载），`*addr`输出的是addr的存储内容，即`0x0012ff7c`
>
> `**addr`应该输出`0x0012ff7c`地址存储的值65，但编译系统并不知道*addr存储的是地址，所以将它当作整数导致编译报错，但如果我们能确定addr里面是地址，那么我们可以将整数强转为地址，即`(int *)addr`。则使用"\*(int\*)addr"取出65



#### 指针

**==如果想像对待变量一样对待addr，即使用“*” 和 “&”运算符的结果与变量a一样，就必须定义新的变量类型==**

``` c
addr = (int)&a;	// &a是0x0012ff7c
(int*)addr;	// 强制将值转换为地址
*(int*)addr;	// 这样才能取值65
```

> 由此可见，如果定义一种变量，使它存储的数据类型是地址，问题就迎刃而解了，
>
> 要使用`(int*)addr`的addr存储地址，那么就要使用“int*”来声明`addr`，即**==int\* addr==**
>
> 这时，`addr = &a`就无需强制转换了，则“*addr”就是65，与普通变量使用方法一样了
>
> **暂且将“int *”定义的变量称为指针变量**



使用新的数据类型（指针）的栗子

``` c
#include <stdio.h>
int main() {
    int a = 65;		// 同样，假设内存首地址为0x0012ff7c
    int *p;		// 声明变量p
    p = &a;		// 給p赋值为a的地址0x0012ff7c，但p分配的地址是0x0012ff78
    
    printf("%#p, %#p, %#p\n", &a, &p, p);
    printf("%d, %d, %d", a, *&a, *p);
    return 0;
}
```

**引入字符指针的栗子**

``` c
#include <stdio.h>
int main() {
    char a = 'B';	// 假设内存首地址0x0012ff7c
    int addr;
    addr = (int)&a;		// 分配地址为0x0012f78
    
    printf("0x%p, 0x%p, 0x%p\n", &a, addr, (char*)addr);
    printf("%c, %c, %c", a, *&a, *(char*)addr);
    return 0;
}
```

**声明指针类型的变量**

```c
int* p;	//整型类型指针
char* pc; // 字符类型指针
float* pf;	// 浮点类型指针
double* pd;	// 双精度类型指针
struct* ps;	// 结构类型指针

// 以下三种声明都可以。
int* p;
int * p;
int *p;
```

> 任何数据类型的指针，一律分配4字节。