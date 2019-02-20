## AVL 树

AVL 树是一种高度平衡的（height balanced）二叉搜索树：对每一个结点，它的左子树与右子树的高度差至多为 1。

![](../assets/images/data-structures/avl-tree1.jpg)

上图是一棵合法的 AVL 树，因为对每个结点来说，其左子树和右子树的高度差都不超过 1。

![](../assets/images/data-structures/avl-tree2.jpg)

上图不是一棵合法的 AVL 树，因为对结点 12 和 8 来说，它们的左右子树高度差都超过了 1。

对于一棵高度为 h 的普通二叉搜索树来说，它的 search、predecessor、successor、minimum、maximum、insert 和 delete 操作的时间复杂度均为 O(h)。因此，如果搜索树的高度较低时，这些集合操作会执行得比较快。然而，如果树的形状比较偏斜，高度达到了 O(n)，这些集合操作的时间复杂度就退化到了 O(n)。AVL 树是许多平衡搜索树中的一种，可以保证每次插入和删除操作之后，树的高度 h 一直都能维持在 O(lgn)。

要实现一棵 AVL 树，需要在每个结点内维护一个额外的属性 height，代表该结点的高度。


```java
public class AVLTree {
    Node root;
    
    class Node {
        int key;
        int height;
        Node left;
        Node right;

        Node(int key) {
            this.key = key;
            height = 1;
        }
    }
}
```

### 插入操作

要在一棵 AVL 树中插入一个结点，首先以二叉搜索树的顺序把该结点放在适当的位置上，此时，这棵树可能就不再是高度平衡的，具体来说，某些结点的左子树和右子树的高度差可能会到 2。为了让树重新保持平衡，我们引入一个 balance 操作，输入一棵以 x 为根的子树，其左子树和右子树都是高度平衡的，而且它们的高度差至多是 2，即丨x.right.height - x.left.height丨<= 2，将这棵以 x 为根的子树转变为高度平衡的。

平衡操作是基于旋转的，类似于红黑树的旋转，只不过在旋转之后需要更新结点的高度。

![](../assets/images/part3/red-black-tree3.png)

```java
Node rightRotate(Node p) {
    Node l = p.left;
    p.left = l.right;
    l.right = p;
    updateHeight(p);
    updateHeight(l);
    return l;
}

Node leftRotate(Node p) {
    Node r = p.right;
    p.right = r.left;
    r.left = p;
    updateHeight(p);
    updateHeight(r);
    return r;
}

void updateHeight(Node node) {
    node.height = Math.max(getHeight(node.left), getHeight(node.right)) + 1;
}

int getHeight(Node node) {
    if (node == null) {
        return 0;
    } else {
        return node.height;
    }
}

int getDifference(Node node) {
    if (node == null) {
        return 0;
    } else {
        return getHeight(node.left) - getHeight(node.right);
    }
}
```

插入一个结点后导致树不平衡的原因，是某个子树的高度增加了 1。从新插入的结点 w 开始往根结点回溯，假设 z 是第一个不平衡的结点，y 是从回溯路径上 z 的孩子，x 是回溯路径上 y 的孩子。根据 y 和 x 的方向不同，插入后的平衡操作需要考虑 4 种情况：

**1. y 是 z 的左孩子， x 是 y 的左孩子**

令 T1, T2, T3 和 T4 为任意子树。

     z                                      y 
    / \                                   /   \
   y   T4      Right Rotate (z)          x      z
  / \          - - - - - - - - ->      /  \    /  \ 
 x   T3                               T1  T2  T3  T4
/ \
T1   T2

**2. y 是 z 的左孩子，x 是 y 的右孩子**

     z                               z                           x
    / \                            /   \                        /  \ 
   y   T4  Left Rotate (y)        x    T4  Right Rotate(z)    y      z
  / \      - - - - - - - - ->    /  \      - - - - - - - ->  / \    / \
T1   x                          y    T3                    T1  T2 T3  T4
    / \                        / \
  T2   T3                    T1   T2

**3. y 是 z 的右孩子，x 是 y 的右孩子**

  z                                y
 /  \                            /   \ 
T1   y     Left Rotate(z)       z      x
    /  \   - - - - - - - ->    / \    / \
   T2   x                     T1  T2 T3  T4
       / \
     T3  T4

**4. y 是 z 的右孩子，x 是 y 的左孩子**

   z                            z                            x
  / \                          / \                          /  \ 
T1   y   Right Rotate (y)    T1   x      Left Rotate(z)   z      y
    / \  - - - - - - - - ->     /  \   - - - - - - - ->  / \    / \
   x   T4                      T2   y                  T1  T2  T3  T4
  / \                              /  \
T2   T3                           T3   T4

```java
Node balanceInsert(Node node) {
    // If this node becomes unbalanced, then there are 4 cases

    // Left Left Case
    if (getDifference(node) == 2 && getDifference(node.left) == 1) {
        return rightRotate(node);
    }

    // Right Right Case
    if (getDifference(node) == -2 && getDifference(node.right) == -1) {
        return leftRotate(node);
    }

    // Left Right Case
    if (getDifference(node) == 2 && getDifference(node.left) == -1) {
        node.left = leftRotate(node.left);
        return rightRotate(node);
    }

    // Right Left Case
    if (getDifference(node) == -2 && getDifference(node.right) == 1) {
        node.right = rightRotate(node.right);
        return leftRotate(node);
    }

    // Return the (unchanged) node pointer
    return node;
}
```

有一个 balanceInsert 过程，我们就可以先将一个结点插入，然后沿着树根方向回溯，更新结点的高度，遇到第一个不平衡的结点，将其调整为平衡的，再往上就不会再出现不平衡的结点了，只需要更新高度即可。

```java
Node insert(Node node, int key) {
    // Perform the normal BST insertion
    if (node == null) {
        return new Node(key);
    }
    if (key < node.key) {
        node.left = insert(node.left, key);
    } else if (key > node.key) {
        node.right = insert(node.right, key);
    } else {
        // Duplicated keys not allowed
        return node;
    }

    // Update height of this ancestor node
    updateHeight(node);

    return balanceInsert(node);
}
```

### 删除操作

