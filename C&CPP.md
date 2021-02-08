## 1 翁恺C











## 2 韩顺平C





C内存布局

> 内存自底向上增大
>
> + 代码段
> + 全局段
> + 堆
> + 堆栈

```cpp

// 全局变量，存储在全局段
int ga = 10;
int gb = 10;

int main() {
  int la = 10;
  int lb = 10;
  
  // 静态变量，存储在全局段
  static int sa = 10;
  static int sb = 10;
  
  // stdlib.h
  int *p = (int*)malloc(4);
  int *p2 = (int*)malloc(4);
  
  count << "局部变量与局部常量地址 la " << &la << " lb " << lb << endl;
  
  count << "全局变量全局常量与静态变量地址 ga= " << &ga << " gb= " << gb << "sa= " << sa << "sb= " << sb << endl;
  
}
```

``` mermaid
graph TD

A[堆栈]-->B[堆]
B --> C[全局段]
C --> D[代码段]
```





## 3 C++基础





### 文件操作

> 步骤
>
> 1. 引入头文件「`fstream / ofstream / ifstream`」
> 2. 创建输入输出流对象
> 3. 文件路径与打开方式
> 4. 读写文件
> 5. 关闭文件



| unsinged int | desc                                   |
| ------------ | -------------------------------------- |
| ios::in      | 以读方式打开                           |
| ios::out     | 以写方式打开                           |
| ios::app     | 追加                                   |
| ios::trunc   | 如果文件存在，删除后创建               |
| ios::binary  | 操作二进制文件「文本文件、二进制文件」 |
| ios::ate     | 读写，直接定位到文件尾                 |

> 可以通过 | 合并多个操作
>
> `ios::in | ios::binary`



## 4 C++ 面向对象



### 4.1 C++内存模型

> 1. 代码段『存放CPU执行的机器指令，只读、共享』
>
> 2. 全局段『存放全局变量、静态变量、常量（字符串常量，const修饰的全局常量），程序结束后该区域内存由OS释放』
>
> 3. 堆区『由程序员分配和释放，使用new, delete』
>
>    `int *p = new int(10)`
>
>    其中p分配在栈（&p获取p的地址），10分配在堆（p指向堆区中的10）
>
> 4. 堆栈『不要返回局部变量的指针 / 引用，该区域内存由编译器管理』

`#define N 200`

`const int a = 200`

宏定义在预处理阶段替换，const修饰的常量在编译阶段替换

> 1. 预处理：宏定义展开，头文件展开，条件编译
> 2. 编译：检查语法，将预处理后文件编译生成汇编文件
> 3. 汇编：将汇编文件生成目标文件（二进制文件）
> 4. 链接：将目标文件链接为可执行程序



## 5 C++提高

1. 泛型编程
2. STL



### 模板编程

> `template<typename T>  / template<class T>`





### STL

> 标准模板库
>
> 6大组件：
>
> + **容器**「vector、list、deque、set、map」
> + **算法**「sort、find、copy、for_each」
> + **迭代器**
> + **仿函数**「重载()，行为类似函数，可作为算法的某种策略」
> + 适配器
> + 空间配置器



#### 迭代器

> 



#### 容器



##### vector

``` cpp

#include <vector>
#include <algorithm>

// 提供给for_each回调
void myPrint(int val) {
  cout << val;
}

int main() {
  vector<int> v;
  v.push_back(10);
  v.push_back(20);
  v.push_back(30);
  
  // 遍历方式一
  for (int t : v) {
    cout << t;
  }
  cout << endl;
  
  // 迭代器遍历
  for (auto it = v.begin(); it != v.end(); b++) {
    cout << *b
  }
  cout << endl;
  
  // for_each遍历
  for_each(v.begin(), v.end(), myPrint)
}
```





##### list





##### deque

