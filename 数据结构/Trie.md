> 前缀树，字典树
>
> Ternary Search Trie
>
> ``` js
> /*               d
>  *           /   |   \
>  *          a    k     z
>  *             / | \
>  *      左边小于父节点，右边大于父节点
>  */
> ```
>
> 

``` java
// Node定义
static class Node {
    boolean isWord;
    Map<Character, Node> next;
    public Node() { this(false); }
    public Node(boolean isWord) {
        this.isWord = isWord;
        this.next = new TreeMap<>();
    }
}

public void insert(String word) {
    Node p = root;
    for (int i = 0; i < word.length(); i++) {
        char c = word.charAt(i);
        if (!p.next.containsKey(c)) {
            p.next.put(c, new Node());
        }
        p = p.next.get(c);
    }
    p.isWord = true;
}

public boolean search(String word) {
    Node p = root;
    for (int i = 0; i < word.length(); i++) {
        char c = word.charAt(i);
        if (p.next.get(c) == null) 
            return false;
        p = p.next.get(c);
    }
    return p.isWord;
}

public boolean startsWith(String prefix) {
    Node p = root;
    for (int i = 0; i < prefix.length(); i++) {
        char c = prefix.charAt(i);
        if (p.next.get(c) == null)
            return false;
        p = p.next.get(c);
    }
    return true;
}
```

