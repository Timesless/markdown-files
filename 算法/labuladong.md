### 1. 必读系列



#### 1.1 数据结构存储方式

数据结果物理存储方式只有两种:

+ 数组

在物理上连续存储,因此可以通过索引随机访问元素,相对链表更节约存储空间, 因为内存连续所以分配时必须一次性分配, 在扩容缩容时需要重新分配空间, 再把数据复制过去, 时间复杂度O(N), 如果在数组中间插入 / 删除, 每次必须移动后面的元素以保持连续, 时间复杂度O(N)

+ 链表

元素在物理上不连续, 靠指针指向下一个元素的位置, 不存在容量的问题, 如果知道某个元素的前驱和后继, 操作指针即可删除 / 插入元素, 时间复杂度O(1), 正因为元素不连续, 所以无法随机访问, 只能遍历查找元素, 时间复杂度O(N)



散列表、栈、队列、堆、树、图等数据结构都是构建在数组与链表之上的上层结构

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

> 掌握树的遍历
>
> + **回溯就是多叉树的前后序遍历**
>
>     `void backtrack(int[] nums, int pos, LinkedList<Integer> trace)`
>
>     ​	`if 满足条件`
>
>     ​		`... return;`
>
>     ​	`for (int i = pos; i < nums.length; ++i)`
>
>     ​		`backtrack(nums, pos, trace)`
>
>     ​		`trace.removeLast()`
>
> + 动态规划也是遍历一棵树
>
>     `def dp(n):`
>
>     ​	`for coin in coins:`
>
>     ​		`dp(n - coin)`

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
    // 访问val;
  ;

// 链表的递归访问
traverse(node) {
  // 访问val
  traverse(node.next);
}

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



###  2 DP

> **首先, DP问题一般形式是求最值**，比如最长递增子序列，最小编辑距离，最长回文串
>
> 既然是求最值, 那么肯定要穷举所有可行解, 再比较找最值
>
> 这里和贪心就不同，贪心算法确定的直到每一步选择那个是最优解
>
> **但是, DP的穷举有点特别, 因为DP问题存在「重叠子问题」**, 如果暴力穷举的话效率极其低下, 所以需要「备忘录」 / 「DP table」来优化穷举过程, 避免不必要的计算
>
> **并且, DP问题一定具备「最优子结构」**, 这样才能通过子问题最值求原问题最值
>
> **另外, DP的关键是「状态转移方程」, 这是最难的**，状态转移一定是朝着已知方向递进，否则递归是无限进行下去的

**==我们需要明确 「base case」,「状态」,「选择（即状态转移）」, 以此定义 dp数组含义==**



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

#### 2.1 Fibonacci

> 「**状态转移方程**」
>
> 把 `f(n)` 想做一个状态 `n`，这个状态 `n` 是由状态 `n - 1` 和状态 `n - 2` 相加转移而来，这就叫状态转移
>
> 「**状态压缩**」
>
> 由于最终结果n（状态）只与n - 1，n - 2的状态有关，所以无需dp[n]的空间，这是叫状态压缩

``` java
// 暴力递归
int fib(int n) {
  if (n == 1 || n == 2) return 1;
  return fib(n - 1) + fib(n - 2);
}

// 自顶向下，带备忘录的递归
int fib(int n) {
    if (n < 1) return 0;
    int[] dp = new int[n + 1];
    // 以一个不可能出现的值为初始化
    Arrays.fill(dp, -1);
    return fib(n, dp);
}
int fib(int n, int[] dp) {
    if (n == 1 || n == 2) return 1;
  	// 说明该子项已经被计算过了
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



#### 2.2 凑零钱

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
> 2. 确定[状态], 也就是原问题和子问题中会变化的变量. 由于硬币数量无限, 只有金额会不断的向base case靠近, 所以唯一[状态]是amount
> 3. **确定[选择], 也就是导致 [状态]产生变化的行为, 状态转移方程**
> 4. 明确[dp数组]的定义. 我们这里讲的是自顶向下的解法, 所以会有一个递归的dp函数, 一般来说函数的参数就是状态转移中变化的量, 也就是上述步骤中的[状态], 函数的返回值就是我们需要的计算量



##### 自顶向下 - 备忘录递归

``` python

# 伪码框架
def coinChange(coins: List[int], amount: int) -> int:
  def coin(n):
    for coin in coins:
      # 选不选当前的硬币res 与 dp(n-coin) + 1
      res = min(res, 1 + dp(n-coin))
    return res if res != float('INF') else -1
  return coin(amount)

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



##### 自底向上 - 动态规划

> ==一般来说，DP会将阶乘，指数级时间复杂度优化为 O(N<sup>2</sup>)==
>
> dp[]**：令 当目标金额为i时，至少需要dp[i]枚硬币凑出
>
> **状态转移方程**： 取或不取当前硬币`min(rs, 1 + dp(amount - coin) + 1)`
>
> **base case**：金额-1需要-1，0需要0枚，1需要1枚硬币

``` python
def coinChange(coins: List[int], amount: int) -> int:
  # 初始化一个最大值，便于比较时取最小值
  ln = amount + 1
	dp = [ln] * ln
  dp[0] = 0
  for i in range(1, ln):
    for coin in coins:
      # 子问题无解，跳过
      if i < coin: continue
      dp[i] = min(dp[i], dp[i-coin] + 1)
  return dp[amount] if dp[amount] != ln else -1 
```



#### 2.3 状态压缩

> 常见的状态压缩：
>
> + 从O(N) 压缩为 O1
> + ON<sup>2</sup>压缩为ON
>
> 如果最终状态只依赖部分或相邻的状态，则可以根据情况进行压缩
>
> **什么叫相邻状态呢?**
>
> 比如<kbd>最长回文子序列</kbd>，最终状态为`return dp[0][n-1]`
>
> ==显然可以从O(N<sup>2</sup>)优化为ON==





### 3 回溯

> ==解决一个回溯问题，实际上是一个决策树的遍历过程==
>
> 一般来说需要思考3个问题：
>
> 1. 路径（即已做出的选择）
> 2. 选择列表（当前剩余可以做的选择）
> 3. 结束条件（达到决策树的底层）



#### 3.1 伪代码

``` python
rs = []
def backtrack(路径，选择列表):
  if 满足条件:
    rs.add(路径)
    return
  for 选择 in 选择列表:
    做选择
    backtrack(路径递进...，选择列表)
    撤销选择（path.removeLast）
```



#### 3.2 全排列

> 給定不重复的数字，请输出这些数字的全排列
>
> [1, 2, 3] => 3!
>
> 我们称下图为**决策树**，在每个节点其实都是在做决策
>
> **「路径」：记录已经做过的选择**
>
> **「选择列表」：当前可以做出的选择**
>
> **「结束条件」：遍历到树的叶节点**
>
> ==我们只要在递归之前做选择，递归之后撤销选择，就可以得到所有节点的路径和选择列表==

![img](assets/1.jpg)



``` java
// 做决策与撤销决策刚好可以对应到前、后序遍历

def traverse(TreeNode node) {
  for (TreeNode node : child) 
    // 前序遍历需要的操作
  	traverse(node)
    // 后序遍历需要的操作
}

```



#### 3.3 N Queen

> N皇后问题
>
> 給定一个N * N棋盘，放置N个皇后使它们无法相互攻击
>
> PS：皇后可攻击同一行、同一列、左上左下右上右下任意单位



``` python
rs: List[List[str]] = None
  
def nQueens(n: int) -> List[List[int]]:
  path = [0] * n
  backtrack(path, 0)
  return rs

def backtrack(path: List[int], row: int):
  # 说明此时棋盘已经放满了皇后
  if row == len(path):
    rs.append(path)
    return
  # 每次新的一行都从第0列开始放置
  for col in range(n + 1):
    if attack():
      continue
    # 做选择
    path.append(col)
    # 向结果递进
    backtrack(path, row + 1)
    # 回溯
    path.popleft()

# 可以使用对角线掩码来判断是否互相攻击    
def attack():
  pass

```



#### 3.4 N Queen的一种解法

> 递归调用带返回值，一定是需要在调用出返回的，否则只会结束当前的递归调用
>
> 请仔细思考

``` python
def backtrack(path: List[int], row: int) -> bool:
  # 说明此时棋盘已经放满了皇后
  if row == len(path):
    rs.append(path)
    return true
  # 每次新的一行都从第0列开始放置
  for col in range(n + 1):
    if attack():
      continue
    # 做选择
    path.append(col)
    # 向结果递进
    if backtrack(path, row + 1):
			return true
    # 回溯
    path.popleft()
  return false
```



### 4 BFS

> **本质是在无向无权图中查找最近距离**
>
> ==BFS每次所有节点齐头并进，后面的路径一定经过了前面的路径，所以能查寻到最短路径==
>
> + 走迷宫
> + 橘子腐烂
>
> 注意，将队列中所有节点的未访问的相邻节点加入队列，此时步数 + 1
>
> 在matrix中，可能是上下左右扩散，有可能是下右扩散，具体根据情况判断



``` java
int BFS(Node start, Node target) {
  Queue<Node> q;
  // 数组情况下可以使用visited[]
  Set<Node> visited;
  // 轮数（扩散的步数）
  int round = 0;
  // 添加起始节点
  q.offer(start);
  while (!q.isEmpty()) {
    int sz = q.size();
    // 将队列中所有节点，向四周扩散，注意是所有节点
    for (int i = 0; i < sz; ++i) {
      Node cur = q.poll();
      // 如果到达
      if (cur == target) {
        return step;
      }
      // adj 指相邻节点，如果是matrix可能是 上下左右，根据情况也可能是下右
      for(Node child : cur.adj()) {
        if (!visited.contains(cur)) {
          q.offer(child);
          visited.add(child)
        }
      }
    }
    // 所有节点都扩散之后，扩散的步数 + 1
    ++ round;
  }
  return round;
}
```



#### 4.1 二叉树的最小高度

> 給定一个二叉树，返回其最小深度（根节点到叶子节点最小的距离）
>
> ​			3
>
> ​		/     \
>
> ​    9        20
>
> ​			/
>
> ​		15
>
> rs = 2【3，9】



``` java
// 判断是否到叶节点了
if (node.left == null && node.right == null)
  // 到达叶节点
  ;
  
// 代码
int minDeepth(TreeNode root) {
  if (null == root) return 0;
  Queue<TreeNode> q;
  // 树不存在环，所以不需要visited判断
  q.offer(root);
  int minDeepth = 0;
  while(!q.isEmpty()) {
    int sz = q.size();
    for (int i = 0; i < sz; ++i) {
      TreeNode cur = q.poll();
      // 一定是最小的
      if (cur.left == null && cur.right == null)
        return minDeepth;
      // 向队列中加入cur的相邻节点
      if (cur.left != null) {
        q.offer(cur.left);
      }
      if (cur.right != null) {
        q.offer(cur.right);
      }
    }
    ++ minDeepth;
  }
  return minDeepth;
}
```



#### 4.2 解开密码锁的最少次数

> 你有一个带有四个圆形拨轮的转盘锁。每个拨轮都有10个数字： '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' 。每个拨轮可以自由旋转：例如把 '9' 变为  '0'，'0' 变为 '9' 。每次旋转都只能旋转一个拨轮的一位数字。
>
> 锁的初始数字为 '0000' ，一个代表四个拨轮的数字的字符串。
>
> 列表 deadends 包含了一组死亡数字，一旦拨轮的数字和列表里的任何一个元素相同，这个锁将会被永久锁定，无法再被旋转。
>
> 字符串 target 代表可以解锁的数字，你需要给出最小的旋转次数，如果无论如何不能解锁，返回 -1。
>

```
deadends = ["0201","0101","0102","1212","2002"], target = "0202"
输出：6
可能的移动序列为 "0000" -> "1000" -> "1100" -> "1200" -> "1201" -> "1202" -> "0202"。
注意 "0000" -> "0001" -> "0002" -> "0102" -> "0202" 这样的序列是不能解锁的，
因为当拨动到 "0102" 时这个锁就会被锁定。

```



> **优化**：将deadend列表加入到visited集合中

``` java
int minReleaseLock(List<String> deadends, int target) {
  
  Queue<String> q;
  Set<String> visited;
  q.offer('0000');
  visited.add('0000');
  
  int round = 0;
  while (!q.isEmpty()) {
    
    // 将此轮的所有密码都向上向下转动
    int sz = q.size();
    for (int i = 0; i < sz; ++i) {
      String cur = q.offer();
      if (cur == target)
        return rount;
      // if (deadend.contains(cur))
      //   continue;
      for (int j = 0; j < 4; ++j) {
        char[] arrUp = cur.toCharArray();
        char cu = arrUp[j];
        // 向上拨动
        if (cu == '0') 
          cu= '9';
        else
          cu += 1;
        arrUp[j] = cu;
        String up = new String(arrUp);
        if (deadend.contains(up))
          continue;
        if (!visited.contains(up)) {
          q.offer(up);
          visited.add(ip)
        }
      	// 向下拨动
        char[] arrDown = cur.toCharArray();
        char cd = arrDown[j];
        if (cd == '9')
          cd = '0';
        else
          cd -= 1;
        arrDown[j] = cd;
        String down = new String(arrDown);
        if (deadend.contains(down))
          continue;
        if (!visited.contains(down)) {
          q.offer(down);
          visited.add(down);
        }
      }
    }
    ++ round;
  }
  return -1;
}
```



#### 4.3 双向BFS

> 并不改变时间复杂度
>
> 从起点和终点同时向四周扩散，当两边有交集时停止。
>
> **局限性：必须知道终点在哪**
>
> 对于二叉树的最小高度无法得知终点，而转动密码锁是知道终点的

``` java
// 双向BFS，使用双队列同时poll()出元素放入set查看是否有交集，如果有则可以停止了

Queue<String> q1;
q1.offer(start)

Queue<String> q2;
q2.offer(end);
```



### 5 DFS

> 



### 6 二分查找

> 二分查找的难点在于，比较时是否应该添加等号，mid是否需要 + 1等



#### 6.1 Binary Search

``` java
int left = 0, right = nums.length - 1;
while (left <= right) {
  // 防止越界
	int mid = left + (right - left) / 2;
  if (nums[mid] == target) {
    return mid;
  } else if (nums[mid] < target) {
    left = mid + 1;
  } else {
    right = mid - 1;
  }
  // 返回该数据应该在的位置
  return -(left + 1);
}
```



#### 6.2 寻找左侧边界的二分

> 当nums[mid] == target的时候，此时不能return
>
> 收缩right = mid

``` java
if (nums[mid] == target) {
  right = mid - 1;
}
return nums[left] == target ? left : -1;
```



#### 6.3 寻找右侧边界的二分

> 当nums[mid] == target时，收缩left = mid + 1;

``` java
if (nums[mid] == target) {
  left = mid + 1;
}
return nums[right] == target ? right : -1;
```



### 7 双指针

> 双指针主要可能有以下3种情况
>
> + 对撞指针
> + 快慢指针
> + 滑动窗口



#### 7.1 滑动窗口

> 维护一个窗口，不断滑动，每次滑动更新答案

``` java
int left = 0, right = 0;
while (right < nums.length) {
  // 满足条件增大窗口
  window.add(nums[right]);
  ++ right;
  // 更新结果数据
  
  // 缩小窗口
  while (left < right) {
    window.remove(nums[left]);
    ++ left;
    // 更新结果数据
  }
}
```

