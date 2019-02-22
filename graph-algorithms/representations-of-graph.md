## 图的表示

对于图 G = (V, E)，可以用两种标准表示方法表示。一种表示法将图作为邻接链表的组合，另一种表示法则将图作为邻接矩阵来看待。两种表示法都既可以表示无向图，也可以表示有向图。邻接链表因为在表示稀疏图（边的条数 E 远小于 V<sup>2</sup> 的图）时非常紧凑而成为通常的选择，大多数图算法都是假定作为输入的图是以邻接链表方式进行表示的。不过，在稠密图（E 接近 V<sup>2</sup> 的图）的情况下，我们可能倾向于使用邻接矩阵表示法，另外，如果需要快速判断两个结点之间是否有边相连，可能也需要使用邻接矩阵表示法。

对于图 G = (V, E) 来说，其邻接链表表示由一个包含 V 条链表的数组 adj 所构成，每个结点有一条链表。对于每个结点 u ∈ V，邻接链表 adj[u] 包含所有与结点 u 之间有边相连的结点 v，即 adj[u] 包含图 G 中所有与 u 邻接的结点（也可以说，该链表里包含指向这些结点的指针）。

对于邻接链表表示，先给出顶点和边的定义：

```java
public class Vertex {
    int id;
    String label;
    
    Vertex(int id) {
        this.id = id;
    }

    Vertex(int id, String label) {
        this.id = id;
        this.label = label;
    }
}

public class Edge {
    int startId;
    int endId;
    int weight;
    
    Edge(int startId, int endId) {
        this.startId = startId;
        this.endId = endId;
    }

    Edge(int startId, int endId, int weight) {
        this.startId = startId;
        this.endId = endId;
        this.weight = weight;
    }
}
```

![](../assets/images/graph-algorithms/graph1.png)

无向图的两种表示。(a) 一个有 5 个结点和 7 条边的无向图 G。(b) G 的邻接链表表示。(c) G 的邻接矩阵表示。

```java
public class Graph {
    int V;
    int E;
    Vertex[] vertices;
    LinkedList<Edge>[] adj;
    
    Graph(int V) {
        this.V = V;
        E = 0;
        vertices = new Vertex[V];
        adj = new LinkedList[V];
        for (int i = 0; i < V; i++) {
            vertices[i] = new Vertex(i);
            adj[i] = new LinkedList<>();
        }
    }
    
    Graph(int V, String[] labels) {
        this.V = V;
        E = 0;
        vertices = new Vertex[V];
        adj = new LinkedList[V];
        for (int i = 0; i < V; i++) {
            vertices[i] = new Vertex(i, labels[i]);
            adj[i] = new LinkedList<>();
        }
    }
    
    void addEdge(int startId, int endId) {
        Edge e = new Edge(startId, endId);
        adj[startId].add(e);
        adj[endId].add(e);
        E += 2;
    }
    
    void addEdge(int startId, int endId, int weight) {
        Edge e = new Edge(startId, endId, weight);
        adj[startId].add(e);
        adj[endId].add(e);
        E += 2;
    }
}
```

![](../assets/images/graph-algorithms/graph2.png)

有向图的两种表示。(a) 一个有 6 个结点和 8 条边的有向图 G。(b) G 的邻接链表表示。(c) G 的邻接矩阵表示。

```java
public class Digraph {
    int V;
    int E;
    Vertex[] vertices;
    String[] labels;
    LinkedList<Edge>[] adj;
    
    Digraph(int V) {
        this.V = V;
        E = 0;
        vertices = new Vertex[V];
        adj = new LinkedList[V];
        for (int i = 0; i < V; i++) {
            vertices[i] = new Vertex(i);
            adj[i] = new LinkedList<>();
        }
    }
    
    Digraph(int V, String[] labels) {
        this.V = V;
        E = 0;
        vertices = new Vertex[V];
        this.labels = labels;
        adj = new LinkedList[V];
        for (int i = 0; i < V; i++) {
            vertices[i] = new Vertex(i, labels[i]);
            adj[i] = new LinkedList<>();
        }
    }
    
    void addEdge(int startId, int endId) {
        Edge e = new Edge(startId, endId);
        adj[startId].add(e);
        E++;
    }

    void addEdge(int startId, int endId, int weight) {
        Edge e = new Edge(startId, endId, weight);
        adj[startId].add(e);
        E++;
    }
}
```

如果 G 是一个有向图，则对于边 (u, v) 来说，结点 v 将出现在链表 adj[u] 里，因此，所有邻接链表的长度值和等于 E。如果 G 是一个无向图，则对于边 (u, v) 来说，结点 v 将出现在链表 adj[u] 里，结点 u 将出现在链表 adj[v] 里，因此，所有邻接链表的长度值和等于 2E。但不管有向图还是无向图，邻接链表表示法的存储空间需求均为 Θ(V + E)，这正是我们所希望的数量级。

对邻接链表稍加修改，即可以用来表示权重图。权重图是图中每条边都带有一个相关的权重的图，该权重值通常由一个 w: E -> R 的权重函数给出。例如，设 G = (V, E) 为一个权重图，其权重函数为 w，我们可以直接将边 (u, v) ∈ E 的权重值 w(u, v) 存放在结点 u 的邻接链表里。从这种意义上来说，邻接链表表示法的鲁棒性很高，可以对其进行简单修改来支持许多其它的图变种。

邻接链表的一个潜在缺陷是无法快速判断一条边 (u, v) 是否是图中的一条边，唯一的办法是在邻接链表 adj[u] 里面搜索结点 v。邻接矩阵表示则克服了这个缺陷，但付出的代价是更大的存储空间消耗（存储空间的渐进数量级更大）。

对于邻接矩阵表示来说，我们通常会将图 G 中的结点编号为 1, 2, ..., V，这种编号可以是任意的。在进行此种编号后，图 G 的邻接矩阵表示由一个 V × V 的矩阵 A = (a<sup>ij</sup>) 表示，该矩阵满足下述条件：

![](../assets/images/graph-algorithms/graph3.png)

不管一个图有多少条边，邻接矩阵的空间需求皆为 Θ(V<sup>2</sup>)。

从上图中可以看到，无向图的邻接矩阵是一个对称矩阵。由于在无向图中，边 (u, v) 和边 (v, u) 是同一条边，无向图的邻接矩阵 A 就是自己的转置，即 A = A<sup>T</sup>。在某些应用中，可能只需要存放对角线及其以上的这部分邻接矩阵（即半个矩阵），从而将图存储空间需求减少几乎一半。

与邻接链表表示法一样，邻接矩阵也可以用来表示权重图。例如，如果 G =(V, E) 为一个权重图，其权重函数为 w，则我们直接将边 (u, v) ∈ E 的权重 w(u, v) 存放在邻接矩阵的第 u 行第 v 列记录上。对于不存在的边，则在相应的行列记录上存放值 null。不过，对于许多问题来说，用 0 或者 ∞ 来表示一条不存在的边可能更为便捷。

虽然邻接链表表示法和邻接矩阵表示法在渐进意义下至少是一样空间有效的，但邻接矩阵表示法更为简单，因此在图规模比较小时，我们可能更倾向于使用邻接矩阵表示法，而且，对于无向图来说，邻接矩阵还有一个优势：每个记录项只需要 1 位的空间。
