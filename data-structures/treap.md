## 随机二叉搜索树

如果将一个含 n 个元素的集合插入到一棵二叉搜索树中，所得到的树可能会相当不平衡，从而导致查找时间很长。然而，随机构造二叉搜索树是趋向于平衡的，因此，一般来说，要为一组固定的元素建立一棵平衡树，可以采用的一种策略就是先随机排列这些元素，然后按照排列的顺序将它们插入到树中。

如果没法同时得到所有的元素，应该怎样处理呢？如果一次收到一个元素，是否仍然能用它们来随机建立一棵二叉搜索树？我们将通过考察一个数据结构来正面回答这个问题。一棵 treap 树是一棵更改了结点排序方式的二叉搜索树，下图显示了一个例子。通常，树内的每个结点 x 都有一个关键字值 x.key，另外，还要为每个结点指定 x.priority，它是一个独立选取的随机数。假设所有的优先级都是不同的，而且所有的关键字也是不同的。treap 树的结点被排列成让关键字遵循二叉搜索树的性质，且优先级遵循最小堆性质：

* 如果 v 是 u 的左孩子，则 v.key < u.key。

* 如果 v 是 u 的右孩子，则 v.key > u.key。

* 如果 v 是 u 的孩子，则 v.priority > u.priority。

这两个性质的结合就是这种树被称为 treap 树的原因：它同时具有二叉搜索树和堆的特征。

```java
class Treap {
    Node root;
    
    class Node {
        int key;
        int priority;
        Node left;
        Node right;

        Node(int key) {
            this.key = key;
            this.priority = Util.randomInt(0, 100);
        }
    }
}
```

### 搜索

Treap 的搜索和普通二叉搜索树的搜索方式完全一样，下面是一个递归实现：

```java
Node search(int key) {
    return search(root, key);
}

Node search(Node node, int key) {
    if (node == null || key == node.key) {
        return node;
    }
    if (key < node.key) {
        return search(node.left, key);
    } else {
        return search(node.right, key);
    }
}
```

由于树高是 O(lgn)，因此搜索的时间复杂度也是 O(lgn)。

### 插入

首先以普通二叉搜索树的方式插入一个新结点，这个结点被赋予了一个随机的 priority 值，所以最大堆性质可能会被破坏，因此需要调整树的结构以维持最大堆性质。树的结构调节是通过旋转来实现的：

![](../assets/images/part2/red-black-tree3.png)

```java
Node leftRotate(Node p) {
    Node r = p.right;
    p.right = r.left;
    r.left = p;
    return r;
}

Node rightRotate(Node p) {
    Node l = p.left;
    p.left = l.right;
    l.right = p;
    return l;
}
```

先将新结点插入到树中，然后我们沿着树根方向回溯，遍历每个祖先结点。如果新结点在左子树上并且当前祖先结点的左孩子 priority 更大，执行右旋；如果新结点在右子树上并且当前祖先结点的右孩子 priority 更大，执行左旋。以下是插入的递归实现：

```java
void insert(int key) {
    root = insert(root, key);
}

Node insert(Node node, int key) {
    if (node == null) {
        return new Node(key);
    }
    if (key < node.key) {
        node.left = insert(node.left, key);
        if (node.left.priority > node.priority) {
            node = rightRotate(node);
        }
    } else {
        node.right = insert(node.right, key);
        if (node.right.priority > node.priority) {
            node = leftRotate(node);
        }
    }
    return node;
}
```

### 删除

Treap 的删除和插入有所不同，令被删除的结点为 x，删除分为以下几种情况：

1. 如果 x 是一个叶结点，则直接删除。

2. 如果 x 有一个子结点为空，则用 x 的另一个子结点替代 x。

3. 如果 x 的左右孩子都不为空，那么找到 x 的左右孩子中 priority 较大的孩子 y。  
   如果 y 是 x 的右孩子，那么对 x 执行左旋，产生新的 x，并在新 x 的左子树上递归调用 delete。
   如果 y 是 x 的左孩子，那么对 x 执行右旋，产生新的 x，并在新 x 的右子树上递归调用 delete。

```java
void delete(int key) {
    root = delete(root, key);
}

Node delete(Node node, int key) {
    if (node == null) {
        return null;
    }
    if (key < node.key) {
        node.left = delete(node.left, key);
    } else if (key > node.key) {
        node.right = delete(node.right, key);
    } else {
        if (node.left == null) {
            node = node.right;
        } else if (node.right == null) {
            node = node.left;
        } else {
            if (node.left.priority < node.right.priority) {
                node = leftRotate(node);
                node.left = delete(node.left, key);
            } else {
                node = rightRotate(node);
                node.right = delete(node.right, key);
            }
        }
    }
    return node;
}
```
