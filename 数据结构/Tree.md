## Binary Tree

+ **BST** 二叉搜索树在极端情况下退化为链表

> **insert**
>
> **print**

+ **AVL**

``` java
/**
 * 节点定义
 */
public class AvlNode {
    // private int height;
    private int val;
    private Node left, right;
    public AvlNode(int _val) { this.val = _val; }
}
```

``` java
/**
 * 树定义
 */
public class AvlTree {
    private AvlNode root;
    public AvlTree() { }
    // 指定根节点
    public AvlTree(int rootVal) { this.root = new Node(rootVal); }
    // 中序遍历
    public void printInorder() { this.printInorder0(root); }
    private void printInorder0(AvlNode node) {
        // 判断是否需要执行，否则不需要开辟栈空间
        if (null != node.left) { printInorder0(node.left); }
        System.out.print(node.val + " ");
        if (null != node.right) { printInorder0(node.right); }
    }
}
```

+ **insert例程**

``` java
/**
 * insert之后执行旋转
 */
public AvlNode insert(int val) {
    if (null == root) {
        root = new AvlNode(val);
        return root;
    }
    return this.insert0(root, val);
}
private AvlNode insert0(AvlNode node, val) {
    
}
```

> + **left rotate**
>
> 右子树的右子节点添加值
>
> + **right rotate**
>
> 左子树的左子节点添加值
>
> + **left-right double rotate**
>
> 左子树的右子节点添加值
>
> + **right-left double rotate**
>
> 右子树的左子节点添加值

+ **红黑树**

    > **性质**：
    >
    > + 节点是红色或黑色
    > + 根节点是黑色
    > + 红色节点的子节点都是黑色，从叶子节点到根节点的路径上不能有连续的红色节点
    > + 所有叶子节点黑色
    >
    > **三种变换**
    >
    > + 改变颜色
    >
    > 父节点是红色，且父节点的兄弟节点是红色
    >
    > ​	将父节点和父节点的兄弟节点变为黑色，父节点的父节点（指针指向该节点）变为红色，
    >
    > + 左旋
    >
    > 父节点是红色，父节点的兄弟节点是黑色，且当前节在右子树
    >
    > + 右旋
    >
    > 父节点是红色，父节点的兄弟节点是黑色，且当前节点在左子树 
    >
    > 1. 把父节点变为黑色
    > 2. 父节点的父节点变为红色
    > 3. 以父节点的父节点右旋
    >
    > **insert**
    >
    > 所有插入节点为红色 
    >
    > 

## B Tree