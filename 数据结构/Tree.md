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

## B Tree