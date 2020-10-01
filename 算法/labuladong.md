### 1. 必读系列



#### 1.1 数据结构存储方式

数据结果物理存储方式只有两种:

+ 数组

在物理上连续存储,因此可以通过索引随机访问元素,相对链表更节约存储空间, 因为内存连续所以分配时必须一次性分配, 在扩容缩容时需要重新分配空间, 再把数据复制过去, 时间复杂度O(N), 如果在数组中间插入 / 删除, 每次必须移动后面的元素以保持连续, 时间复杂度O(N)

+ 链表

元素在物理上不连续, 靠指针指向下一个元素的位置, 不存在容量的问题, 如果知道某个元素的前驱和后继, 操作指针即可删除 / 插入元素, 时间复杂度O(1), 正因为元素不连续, 所以无法随机访问, 只能遍历查找元素, 时间复杂度O(N)



散列表, 栈, 队列, 堆, 树, 图等数据结构都是构建在数组与链表之上的上层数据结构

> 例如:
>
> 图的两种表示法
>
> + 邻接表, 就是链表, 图比较稀疏的话使用邻接表节约空间
> + 邻接矩阵, 就是二维数组, 判断连通性比较快, 并可以借助矩阵运算解决一些问题
>
> 树
>
> + 用数组实现就是堆
>
>  因为堆是一个完全二叉树, 可以通过索引操作元素而不需要额外的指针.
>
> + 用链表实现就是普通的树
>
> 构建在链表树的结构之上, 又衍生出各种巧妙的上层结构, 如 二叉搜索树, AVL树, 红黑树, 线段树, 区间树, B树



#### 1.2 数据结构的遍历

对任意数据结构, 我们可以划分为线性, 非线性

+ for / while (迭代线性数据结构)
+ 递归(非线性数据结构)

``` java
/**
 * 线性数据结构的遍历
 */
// 数组
for (int i = 0; i < arr.length; ++i)
    ;
// 链表
class ListNode<T> {
    T val;
    ListNode<T> next;
}
for (ListNode p = head; p != null; p = p.next)
    ;


/**
 * 非线性数据结构遍历
 */
// 二叉树
class TreeNode<T> {
    T val;
    TreeNode<T> left, right;
}
void traverse(TreeNode root) {
    // 前序 sout(root.val)
    traverse(root.left);
    // 中序 sout(root.val)
    traverse(root.right);
    // 后序 sout(root.val)
}

// 多叉树 / 图
// 可能存在环导致递归溢出, 添加boolean[]变量即可
void traverse(TreeNode node) {
    for (TreeNode child : node.children())
        traverse(child);
}
```



#### 1.3 DP

**首先, DP问题一般形式是求最值**, 比如最长递增子序列, 最小编辑距离, 最长回文串

既然是求最值, 那么肯定要穷举所有可行解, 再比较找最值

**但是, DP的穷举有点特别, 因为DP问题存在「重叠子问题」**, 如果暴力穷举的话效率极其低下, 所以需要「备忘录」 / 「DP table」来优化穷举过程, 避免不必要的计算

**并且, DP问题一定具备「最优子结构」**, 这样才能通过子问题最值求原问题最值

**另外, DP的关键是「状态转移方程」, 这是最难的**, 状态转移一定是朝着已知方向递进, 否则递归是无限进行下去的

**==我们需要明确 「base case」,「状态」,「比较后的最值选择」, 以此定义 dp数组含义==**



按以上说明, 结果是

``` python
# 初始化 base case
dp[0][0][...] = base case

# 状态转移
for 状态1 in 状态1所有值:
    for 状态2 in 状态2所有值:
        for ...:
            dp[状态1][状态2][...] = 求最值(选择1, 选择2, ...)
```



**==带备忘录递归的解法是自顶向下, 动态规划的解法是自底向上. 时间复杂度O(N)==**

##### 1.3.1 Fibonacci

``` java
// 自顶向下的递归
int fib(int n) {
    if (n < 1) return 0;
    int[] dp = new int[n + 1];
    // 以一个不可能出现的值为初始化
    Arrays.fill(dp, -1);
    return fib(n, dp);
}
int fib(int n, int[] dp) {
    if (n == 1 || n == 2) return n;
    if (dp[n] != -1) return dp[n];
    dp[n] = fib(n-1, dp) + fib(n-2, dp);
    return dp[n];
}

// 自底向上的动态规划
int fib(int n) {
    int[] dp = new int[n + 1];
    dp[1] = dp[2] = 1;
    for (int i = 3; i <= n; ++i) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}

/**
 * 根据Fibonacci状态转移方程, 发现当前状态只和之前的两个状态有关
 * 所以无需dp[n + 1]的空间存储
 */
int fib(int n) {
    int first = 1, second = 1;
    for (int i = 3; i <= n; ++i) {
        int sum = first + second;
        first = second;
        second = sum;
    }
    return second;
}
```



##### 1.3.2 凑零钱

给你 `k` 种面值的硬币，面值分别为 `c1, c2 ... ck`，每种硬币的数量无限，再给一个总金额 `amount`，问你**最少**需要几枚硬币凑出这个金额，如果不可能凑出，算法返回 -1 。算法的函数签名如下：

```
// coins 中是可选硬币面值，amount 是目标金额
int coinChange(int[] coins, int amount);
```

比如说 `k = 3`，面值分别为 1，2，5，总金额 `amount = 11`。那么最少需要 3 枚硬币凑出，即 11 = 5 + 5 + 1。

你认为计算机应该如何解决这个问题？显然，就是把所有肯能的凑硬币方法都穷举出来，然后找找看最少需要多少枚硬币。



> 解法说明:
>
> 1. 确定 base case. 目标金额为0时, 0个硬币解决
> 2. 确定[状态], 也就是原问题和子问题中会变化的变量. 由于硬币数量无限, 只有金额会不断的向base case靠近, 所以唯一[状态]是amout
> 3. 确定[选择], 也就是导致 [状态]产生变化的行为, 状态转移方程
> 4. 明确[dp数组]的定义. 我们这里讲的是自顶向下的解法, 所以会有一个递归的dp函数, 一般来说函数的参数就是状态转移中变化的量, 也就是上述步骤中的[状态], 函数的返回值就是我们需要的计算量

``` python
# 伪代码如下
def coin(coins: List[int], amount: int):
    # 备忘录
    memo = dict()
    # 凑出金额n,至少需要dp[n]个硬币
    def dp(n):
        if n in memo: return memo[n]
        # base case
        if n == 0: return 0
    	if n < 0: return -1
    	res = float('INF')
        # 做选择
        for coin in coins:
            sub = dp(n - coin)
            # 子问题无解则跳过
            if (sub == -1): continue
            res = min(res, 1 + sub)
        # 记入备忘录
        memo[n] = res
        return memo[n]
    # 结果
    return dp(amount) 
```