## 算法导论 -- 麻省理工

渐近符号 **Ω O Θ**

规模在30左右 O(nlogn) >= On<sup>2</sup>

### 主定理

``` properties
# 主定理
T(n) = aT(n / b) + f(n)
有a个相同的子问题，每个子问题的规模是 n / b
fn是非递归的代价

# 约束条件
a >= 1 子问题至少需要递归一次
b > 1	规模必须递减，否则结果是infinity
f(n)渐近趋正（fn总是正数）： 存在常数n0，当n >= n0时，fn > 0
子问题规模都相同

# 结果，比较递归函数与非递归函数的增长量级

n ^ log b a 是递归函数的叶节点的数量（宽 * 高）
以二叉树为例：
	w: 2T(n / 2) + Θ(1) (case 1)
	h: 1T(n / 2) + Θ(1) (case 1)
w * h = n * log n

Tn = min { f(n), n ^ logb a }

# ================================
1. f(n) < n ^ log b a
Tn = Θ(n ^ log b a)

2. fn 基本等于 n ^ log b a
Tn = Θ((n ^ log b a) * (k + 1)logn) 
n的以b为底a的对数次幂 * log n的k（k可为0）+ 1次幂

3. fn >  n ^ log b a
fn = Ω(n ^ log b (a + ε))
ε > 0 ，这样fn 才会大于 n ^ log b a
Tn = Θ(fn)
# =================================

# Ex1
Tn = 4T(n / 2) + n
	a = 4, b = 2 => n ^ log b a > fn
Tn = Θ(n2)

#Ex2
Tn = 4T(n / 2) + n2
	n ^ log b a == fn
Tn = O(n2 * log n)

#Ex3
Tn = 4T(n / 2) + n3
	n^log b a < n3
Tn = O(n3)

# 画出递归树，证明主定理
# 树的高度 log b n，每一次递归有a的规模
a ^ h == a ^ log b n == n ^ log b a
```

