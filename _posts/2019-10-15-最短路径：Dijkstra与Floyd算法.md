---
layout: post
title: Dijkstra与Floyd算法求最短路径
date: 2019-10-15
tags: Study_Notes
---

# 图论问题

## 定义

所谓最短路径问题是指：如果从图中某一顶点（源点）到达另一顶点（终点）的路径可能不止一条，如何找到一条路径使得沿此路径上各边的权值总和（称为路径长度）达到最小。

-----

**下面我们介绍两种比较常用的求最短路径算法：**

## Dijkstra算法

他的算法思想是按路径长度递增的次序一步一步并入来求取，是贪心算法的一个应用，用来解决单源点到其余顶点的最短路径问题。

### 算法思想

首先，我们引入一个辅助向量D，它的每个分量D[i]表示当前找到的从起始节点v到终点节点vi的最短路径的长度。它的初始态为：若从节点v到节点vi有弧，则D[i]为弧上的权值，否则D[i]为∞，显然，长度为D[j] = Min{D[i] | vi ∈V}的路径就是从v出发最短的一条路径，路径为(v, vi)。
那么，下一条长度次短的最短路径是哪一条呢？假设次短路径的终点是vk，则可想而知，这条路径或者是(v, vk)或者是(v, vj, vk)。它的长度或者是从v到vk的弧上的权值，或者是D[j]和从vj到vk的权值之和。

一般情况下，假设S为已知求得的最短路径的终点集合，则可证明：一下条最短路径（设其终点为x）或者是弧(v, x)或者是中间只经过S中的顶点而最后到达顶点x的路径。这可用反证法来证明，假设此路径上有一个顶点不在S中，则说明存在一条终点不在S中而长度比此路径短的路径。但是这是不可能的。因为，我们是按路径常度的递增次序来产生个最短路径的，故长度比此路径端的所有路径均已产生，他们的终点必定在S集合中，即假设不成立。

因此下一条次短的最短路径的长度是：D[j] = Min{D[i] \ vi ∈ V - S}，其中，D[i]或者是弧(v, vi)的权值，或者是D[k](vk ∈ S)和弧(vk, vi)上权值之和。

![img](/images/posts/lsjg/75.png)

### 算法结构
```
初始化: S ← { v0 };  
               dist[j] ← a[0][j],   j = 1, 2, …, n-1; // n顶点个数
(1) 求出最短路径的长度：
         dist[k] ← min{ dist[i] },  i  V- S ; 
(2) S ← S U { k };
(3) 修改：  
      dist[i] ← min{ dist[i], dist[k] +a[k][i] },
               对于每一个 i  V- S ; 
(4) 重复（2）和（3），直至S = V时算法结束。

```



### 算法描述

假设现要求取如下示例图所示的顶点V0与其余各顶点的最短路径：
![img](/images/posts/lsjg/74.png)

我们使用Guava的ValueGraph作为该图的数据结构，每个顶点对应一个visited变量来表示节点是在V中还是在S中，初始时S中只有顶点V0。然后从nodes集合中遍历找出从V0出发到各节点路径最短的节点，并将该节点并入S中（即修改该节点的visited属性为true），此时就找到了一个顶点的最短路径。然后，**我们看看新加入的顶点是否可以到达其他顶点，并且看看通过该顶点到达其他点的路径长度是否比从V0直接到达更短,如果是，则修改这些顶点的权值**(即if (D[j] + arcs[j][k] < D[k]) then D[k] = D[j] + arcs[j][k])。然后又从{V - S}中找最小值，重复上述动作，直到所有顶点都并入S中。


第一步，我们通过ValueGraphBuilder构造图的实例，并输入边集：
```java
MutableValueGraph<String, Integer> graph = ValueGraphBuilder.directed()
        .nodeOrder(ElementOrder.insertion())
        .expectedNodeCount(10)
        .build();
graph.putEdgeValue(V0, V2, 10);
graph.putEdgeValue(V0, V4, 30);
graph.putEdgeValue(V0, V5, 100);
graph.putEdgeValue(V1, V2, 5);
graph.putEdgeValue(V2, V3, 50);
graph.putEdgeValue(V3, V5, 10);
graph.putEdgeValue(V4, V3, 20);
graph.putEdgeValue(V4, V5, 60);

return graph;
```
初始输出结果如下：
```java
nodes: [v0, v2, v4, v5, v1, v3], 
edges: {<v0 -> v5>=100, <v0 -> v4>=30, <v0 -> v2>=10, 
<v2 -> v3>=50, <v4 -> v5>=60, <v4 -> v3>=20, <v1 -> v2>=5, 
<v3 -> v5>=10}
```

为了不破坏graph的状态，我们引入一个临时结构来记录每个节点运算的中间结果：
```java
private static class NodeExtra {
    public String nodeName; //当前的节点名称
    public int distance; //开始点到当前节点的最短路径
    public boolean visited; //当前节点是否已经求的最短路径（S集合）
    public String preNode; //前一个节点名称
    public String path; //路径的所有途径点
}
```

第二步，我们首先将起始点V0并入集合S中，因为他的最短路径已知为0：

```java
startNode = V0;
NodeExtra current = nodeExtras.get(startNode);
current.distance = 0; //一开始可设置开始节点的最短路径为0
current.visited = true; //并入S集合
current.path = startNode;
current.preNode = startNode;
```
第三步，在当前状态下找出起始点V0开始到其他节点路径最短的节点：
```java
NodeExtra minExtra = null; //路径最短的节点信息
int min = Integer.MAX_VALUE;
for (String notVisitedNode : nodes) {
    //获取节点的辅助信息
    NodeExtra extra = nodeExtras.get(notVisitedNode); 
    
    //不在S集合中，且路径较短
    if (!extra.visited && extra.distance < min) {
        min = extra.distance;
        minExtra = extra;
    }
}
```
第四步，将最短路径的节点并入集合S中：

```java
if (minExtra != null) { //找到了路径最短的节点
    minExtra.visited = true; //并入集合S中
    //更新其中转节点路径
    minExtra.path = nodeExtras.get(minExtra.preNode).path + " -> " + minExtra.nodeName; 
    current = minExtra; //标识当前并入的最短路径节点
}
```
第五步，更新与其相关节点的最短路径中间结果：

```java
/**
 * 并入新查找到的节点后，更新与其相关节点的最短路径中间结果
 * if (D[j] + arcs[j][k] < D[k]) D[k] = D[j] + arcs[j][k]
 */
//只需循环当前节点的后继列表即可(优化)
Set<String> successors = graph.successors(current.nodeName); 
for (String notVisitedNode : successors) {
    NodeExtra extra = nodeExtras.get(notVisitedNode);
    if (!extra.visited) {
        final int value = current.distance 
            + graph.edgeValueOrDefault(current.nodeName,
                notVisitedNode, 0); //D[j] + arcs[j][k]
        if (value < extra.distance) { //D[j] + arcs[j][k] < D[k]
            extra.distance = value;
            extra.preNode = current.nodeName;
        }
    }
}
```
第六步，输出起始节点V0到每个节点的最短路径以及路径的途径点信息

```java
Set<String> keys = nodeExtras.keySet();
for (String node : keys) {
    NodeExtra extra = nodeExtras.get(node);
    if (extra.distance < Integer.MAX_VALUE) {
        Log.i(TAG, startNode + " -> " + node + ": min: " + extra.distance
                + ", path: " + extra.path); //path在运算过程中更新
    }
}
```

实例图的输出结果为：
```java
 v0 -> v0: min: 0, path: v0
 v0 -> v2: min: 10, path: v0 -> v2
 v0 -> v3: min: 50, path: v0 -> v4 -> v3
 v0 -> v4: min: 30, path: v0 -> v4
 v0 -> v5: min: 60, path: v0 -> v4 -> v3 -> v5
```

### 算法伪码
```c
void Dijkstra ( int n,  int v ){
   //a[i][j]:(vi,vj)的权，dist[j]:当前求到的从顶点v 到顶点 j 的最短  
  // 路径长度, 同时用数组path[j]:存放求到的最短路径, 0  j < n
    for (  i = 0; i < n; i++) {				
        dist[i] =a[v][i];     //dist数组初始化
        S[i] = 0;					
        if ( i != v && dist[i] < MAXNUM) path[i] = v;
        else path[i] = -1;	        //path数组初始化	
    }    
    S[v] = 1; //顶点 v 加入顶点集合
    dist[v] = 0;	
    for ( i = 0; i < n-1; i++ ) {			
        u=当前不在集合S中且具有最短路径的顶点;
            S[u] = 1;  //将顶点u加入集合S
            for ( int w = 0; w < n; w++ )  {  //修改 
        if ( !S[w] && a[u][w] < MAXNUM &&
                dist[u] + a[u][w] < dist[w] ) {
            dist[w] = dist[u] + a[u][w]; 
            path[w] = u;
        } 
        }
    } 	 	  

// 算法的时间复杂度：O(n2)

```

### 代码实现
```java
import android.util.ArrayMap;
import android.util.Log;

import com.google.common.graph.ElementOrder;
import com.google.common.graph.MutableValueGraph;
import com.google.common.graph.ValueGraphBuilder;

import java.util.Map;
import java.util.Set;


public class Dijkstra {
    private final static String TAG = "Dijkstra";

    private static final String V0 = "v0";
    private static final String V1 = "v1";
    private static final String V2 = "v2";
    private static final String V3 = "v3";
    private static final String V4 = "v4";
    private static final String V5 = "v5";

    private static final String A = "A";
    private static final String B = "B";
    private static final String C = "C";
    private static final String D = "D";
    private static final String E = "E";



    public static void test() {
        MutableValueGraph<String, Integer> graph = buildGraph1();
        Log.i(TAG, "graph: " + graph);

        testDijkstra(graph, V0);


        MutableValueGraph<String, Integer> graph2 = buildGraph2();
        Log.i(TAG, "graph: " + graph2);

        testDijkstra(graph2, A);
    }

    private static void testDijkstra(MutableValueGraph<String, Integer> graph, String startNode) {
        Set<String> nodes = graph.nodes();
        Map<String, NodeExtra> nodeExtras = new ArrayMap<>(nodes.size());
        /**
         * 初始化extra
         */
        for (String node : nodes) {
            NodeExtra extra = new NodeExtra();
            extra.nodeName = node;
            /*初始最短路径：存在边时，为边的权值；没有边时为无穷大*/
            final int value = graph.edgeValueOrDefault(startNode, node, Integer.MAX_VALUE);
            extra.distance = value;
            extra.visited = false;
            if (value < Integer.MAX_VALUE) {
                extra.path = startNode + " -> " + node;
                extra.preNode = startNode;
            }
            nodeExtras.put(node, extra);
        }

        /**
         * 一开始可设置开始节点的最短路径为0
         */
        NodeExtra current = nodeExtras.get(startNode);
        current.distance = 0;
        current.visited = true;
        current.path = startNode;
        current.preNode = startNode;
        /*需要循环节点数遍*/
        for (String node : nodes) {
            NodeExtra minExtra = null;
            int min = Integer.MAX_VALUE;
            /**
             * 找出起始点当前路径最短的节点
             */
            for (String notVisitedNode : nodes) {
                NodeExtra extra = nodeExtras.get(notVisitedNode);
                if (!extra.visited && extra.distance < min) {
                    min = extra.distance;
                    minExtra = extra;
                }
            }

            /**
             * 更新找到的最短路径节点的extra信息（获取的标志、路径信息）
             */
            if (minExtra != null) {
                minExtra.visited = true;
                minExtra.path = nodeExtras.get(minExtra.preNode).path + " -> " + minExtra.nodeName;
                current = minExtra;
            }

            /**
             * 并入新查找到的节点后，更新与其相关节点的最短路径中间结果
             * if (D[j] + arcs[j][k] < D[k]) D[k] = D[j] + arcs[j][k]
             */
            Set<String> successors = graph.successors(current.nodeName); //只需循环当前节点的后继列表即可
            for (String notVisitedNode : successors) {
                NodeExtra extra = nodeExtras.get(notVisitedNode);
                if (!extra.visited) {
                    final int value = current.distance + graph.edgeValueOrDefault(current.nodeName,
                            notVisitedNode, 0);
                    if (value < extra.distance) {
                        extra.distance = value;
                        extra.preNode = current.nodeName;
                    }
                }
            }
        }

        /**
         * 输出起始节点到每个节点的最短路径以及路径的途径点信息
         */
        Set<String> keys = nodeExtras.keySet();
        for (String node : keys) {
            NodeExtra extra = nodeExtras.get(node);
            if (extra.distance < Integer.MAX_VALUE) {
                Log.i(TAG, startNode + " -> " + node + ": min: " + extra.distance
                        + ", path: " + extra.path);
            }
        }
    }

    private static class NodeExtra {
        public String nodeName; //当前的节点名称
        public int distance; //开始点到当前节点的最短路径
        public boolean visited; //当前节点是否已经求的最短路径
        public String preNode; //前一个节点名称
        public String path; //路径的所有途径点
    }

    private static MutableValueGraph<String, Integer> buildGraph1() {
        MutableValueGraph<String, Integer> graph = ValueGraphBuilder.directed()
                .nodeOrder(ElementOrder.insertion())
                .expectedNodeCount(10)
                .build();
        graph.putEdgeValue(V0, V2, 10);
        graph.putEdgeValue(V0, V4, 30);
        graph.putEdgeValue(V0, V5, 100);
        graph.putEdgeValue(V1, V2, 5);
        graph.putEdgeValue(V2, V3, 50);
        graph.putEdgeValue(V3, V5, 10);
        graph.putEdgeValue(V4, V3, 20);
        graph.putEdgeValue(V4, V5, 60);

        return graph;
    }

    private static MutableValueGraph<String, Integer> buildGraph2() {
        MutableValueGraph<String, Integer> graph = ValueGraphBuilder.directed()
                .nodeOrder(ElementOrder.insertion())
                .expectedNodeCount(10)
                .build();

        graph.putEdgeValue(A, B, 10);
        graph.putEdgeValue(A, C, 3);
        graph.putEdgeValue(A, D, 20);
        graph.putEdgeValue(B, D, 5);
        graph.putEdgeValue(C, E, 15);
        graph.putEdgeValue(C, B, 2);
        graph.putEdgeValue(D, E, 11);

        return graph;
    }

    private static <K, V> String format(Map<K, V> values) {
        StringBuilder builder = new StringBuilder();
        builder.append("{");
        Set<K> keys = values.keySet();
        for (K key : keys) {
            builder.append(key).append(":")
                    .append(values.get(key)).append(",");
        }
        if (builder.length()  > 1) {
            builder.deleteCharAt(builder.length() - 1);
        }
        builder.append("}");
        return builder.toString();
    }


    private static <T> String format(Iterable<T> iterable) {
        StringBuilder builder = new StringBuilder();
        builder.append("{");
        for (T obj : iterable) {
            builder.append(obj).append(",");
        }
        if (builder.length()  > 1) {
            builder.deleteCharAt(builder.length() - 1);
        }
        builder.append("}");
        return builder.toString();
    }
}
```




## Floyd算法
Floyd算法是一个经典的动态规划算法。是解决任意两点间的最短路径(称为多源最短路径问题)的一种算法，可以正确处理有向图或负权的最短路径问题。（动态规划算法是通过拆分问题规模，并定义问题状态与状态的关系，使得问题能够以递推（分治）的方式去解决，最终合并各个拆分的小问题的解为整个问题的解。）


### 算法思想

从任意节点i到任意节点j的最短路径不外乎2种可能：

* 直接从节点i到节点j，
* 从节点i经过若干个节点k到节点j。

所以，我们假设arcs(i,j)为节点i到节点j的最短路径的距离，对于每一个节点k，我们检查arcs(i,k) + arcs(k,j) < arcs(i,j)是否成立，如果成立，证明从节点i到节点k再到节点j的路径比节点i直接到节点j的路径短，我们便设置arcs(i,j) = arcs(i,k) + arcs(k,j)，这样一来，当我们遍历完所有节点k，arcs(i,j)中记录的便是节点i到节点j的最短路径的距离。（由于动态规划算法在执行过程中，需要保存大量的临时状态（即小问题的解），因此它天生适用于用矩阵来作为其数据结构，因此在本算法中，我们将不使用Guava-Graph结构，而采用邻接矩阵来作为本例的数据结构）
![img](/images/posts/lsjg/78.png)
### 算法结构
```c
void floyd( ) {
   for ( i = 1; i <= n; i++ )    //矩阵a与path初始化
        for ( j = 1; j <= n; j++ ) {
            if ( i!= j && a[i][j] < MAXINT ) 
                    path[i][j] = i;	    // i 到 j 有路径 	
             else path[i][j] = 0;	    // i 到 j 无路径	
        }
    for (  k = 1; k <=n; k++ )     //产生a(k)及path(k)
        for ( i = 1; i <= n; i++ )
	        for ( j = 1; j <= n; j++ )
                if ( a[i][k] + a[k][j] < a[i][j] ) { 
                        a[i][j] = a[i][k] + a[k][j];
		                path[i][j] = k;
                    }      //缩短路径长度(i-- k -- j )
} 	
```

### 算法描述

假设现要求取如下示例图所示任意两点之间的最短路径：

![img](/images/posts/lsjg/76.png)

我们以一个4x4的邻接矩阵（二维数组arcs[ ][ ]）作为图的数据结构。比如1号节点到2号节点的路径的权值为2，则arcs[1][2] = 2，2号节点无法直接到达4号节点，则arcs[2][4] = ∞(Integer.MAX_VALUE)，则可构造如下矩阵：
![img](/images/posts/lsjg/77.png)

根据以往的经验，如果要让任意两个顶点（假设从顶点a到顶点b）之间的距离变得更短，唯一的选择就是引入第三个顶点（顶点k），并通过顶点k中转（a -> k ->b）才可能缩短顶点a到顶点b之间的距离。于是，现在的问题便分解为：求取某一个点k，使得经过中转节点k后，使得两点之间的距离可能变短，且还可能需要中转两个或者多个节点才能使两点之间的距离变短。比如图中的4号节点到3号节点（4 -> 3）的距离原本是12（arcs[4][3] = 12），如果在只通过1号节点时中转时（4 -> 1 ->3），距离将缩短为11（arcs[4][1] + arcs[1][3] = 5 + 6 = 11）。其实1号节点到3号节点也可以通过2号节点中转，使得1号到3号节点的路程缩短为5（arcs[1][2] + arcs[2][3] = 2 + 3 = 5），所以如果同时经过1号和2号两个节点中转的话，从4号节点到3号节点的距离会进一步缩短为10。于是，延伸到一般问题：

* 当不经过任意第三节点时，其最短路径为初始路径，即上图中的邻接矩阵所示。
* 当只允许经过1号节点时，求两点之间的最短路径该如何求呢？只需判断arcs[i][1]+arcs[1][j]是否比arcs[i][j]要小即可。arcs[i][j]表示的是从i号顶点到j号顶点之间的距离，arcs[i][1] + arcs[1][j]表示的是从i号顶点先到1号顶点，再从1号顶点到j号顶点的路程之和。


循环遍历一遍二维数组，便可以获取在仅仅经过1号节点时的最短距离，实现如下：

```java
for (int i = 1; i <= vexCount; i++) {
    for (int j = 1; j < vexCount; j++) {
        if (arcs[i][1] + arcs[1][j] < arcs[i][j]) {
            arcs[i][j] = arcs[i][1] + arcs[1][j]; 
        }
    }
}
```

由于上述代码更新了两点之间经过1号节点的最短距离arcs[i][j]，因此，数组中每两个节点之间对应距离都是最短的。由于此时arcs[i][j]的结果已经保存了中转1号节点的最短路径，此时如果继续并入2号节点为中转节点，则是任意两个节点都经过中转节点1号节点和2号节点的最短路径，因为运算完中转1号节点时，arcs[i][j]的结果已经更新为中转1号节点的最短路径了。更一般的，继续并入下一个中转节点一直到vexCount个时，arcs[i][j]的结果保存的就是整个图中两点之间的最短路径了。这就是Floyd算法的描述，变成代码就是下面几行行：

```java
for (int k = 1; k <= vexCount; k++) { //并入中转节点1,2,...vexCount
    for (int i = 1; i <= vexCount; i++) {
        for (int j = 1; j < vexCount; j++) {
            if (arcs[i][k] + arcs[k][j] < arcs[i][j]) {
                arcs[i][j] = arcs[i][k] + arcs[k][j];
                path[i][j] = path[i][k]; //这里保存当前是中转的是哪个节点的信息
            }
        }
    }
} 
```


对应到示例图的中间运算结果如下：
```java
print array step of 1: //并入1号节点的结果
    0     2     6     4 
    ∞     0     3     ∞ 
    7     9     0     1 
    5     7    11     0 

print array step of 2: //并入2号节点的结果
    0     2     5     4 
    ∞     0     3     ∞ 
    7     9     0     1 
    5     7    10     0 

print array step of 3: //并入3号节点的结果
    0     2     5     4 
   10     0     3     4 
    7     9     0     1 
    5     7    10     0 

print array step of 4: //并入4号节点（图最终两两节点之间的最短路径值）
    0     2     5     4 
    9     0     3     4 
    6     8     0     1 
    5     7    10     0 
```
虽然此时已求得了节点的最短路径，但结果却不能明显的表达最终最短路径是中转了哪些节点，因此这里对应到动态规划算法中的强项——算法过程中可以完全记录所有的中间结果。我们再定义一个二位数组path[][]，其大小规模对应arcs[][]，初始结果path[i][j] = j，表示节点i到节点j最后的中转节点是j。在运算中是在判断arcs[i][k]+arcs[k][j]比arcs[i][j]要小时，我们进一步更新为：path[i][j] = path[i][k]，即当前最短路径的最后中转节点是path[i][k]对应的节点（如果只允许中专一个节点时即为k，但中转多个节点时，需要对应上一步的中转节点，因此这里要指明是path[i][k]而不是k）。
于是我们通过向前递推path[][]数组，直到path[i][j]是目标节点。则可输出其中转节点，输出函数实现如下：
```java
private void printPath(int arcs[][], int path[][], int vexCount) {
    int temp;
    for (int i = 1; i <= vexCount; i++) {
        StringBuilder builder = new StringBuilder();
        for (int j = 1; j <= vexCount; j++) { //遍历打印任意亮点的路径
            builder.append(i).append("->").append(j)
                .append(", weight: "). append(arcs[i][j])
                    .append(":").append(i);
            temp = path[i][j];
            while(temp != j) {
                builder.append("->").append(temp);
                temp = path[temp][j];
            }
            builder.append("->").append(j).append("\n");
        }
        Log.i(TAG, builder.toString());
    }
}
```

对应示例图的最短路径的中转节点结果输出如下：
```java
 1->1, weight: 0, path: 1->1
 1->2, weight: 2, path: 1->2
 1->3, weight: 5, path: 1->2->3
 1->4, weight: 4, path: 1->4
 2->1, weight: 9, path: 2->3->4->1
 2->2, weight: 0, path: 2->2
 2->3, weight: 3, path: 2->3
 2->4, weight: 4, path: 2->3->4
 3->1, weight: 6, path: 3->4->1
 3->2, weight: 8, path: 3->4->1->2
 3->3, weight: 0, path: 3->3
 3->4, weight: 1, path: 3->4
 4->1, weight: 5, path: 4->1
 4->2, weight: 7, path: 4->1->2
 4->3, weight: 10, path: 4->1->2->3
 4->4, weight: 0, path: 4->4
```

### 代码实现
```java
import android.util.Log;

public class Floyd {
    private static final String TAG = "Floyd";
    public static void test() {
        int vexCount = 4;
        int[][] arcs = new int[vexCount + 1][vexCount + 1];
        int[][] path = new int[vexCount + 1][vexCount + 1];

        /**
         * 初始化图的邻接矩阵
         */
        for (int i = 1; i <= vexCount; i++) {
            for (int j = 1; j <= vexCount; j++) {
                if (i != j) {
                    arcs[i][j] = Integer.MAX_VALUE;
                } else {
                    arcs[i][j] = 0;
                }
                path[i][j] = j;
            }
        }

        /**
         * 输入图的边集
         */
        arcs[1][2] = 2;
        arcs[1][3] = 6;
        arcs[1][4] = 4;
        arcs[2][3] = 3;
        arcs[3][1] = 7;
        arcs[3][4] = 1;
        arcs[4][1] = 5;
        arcs[4][3] = 12;
        print(arcs, vexCount, 0);

        /**
         * floyd核心算法：
         * if arcs[i][k] + arcs[k][j] < arcs[i][j] then
         *      arcs[i][j] = arcs[i][k] + arcs[k][j]
         */
        for (int k = 1; k <= vexCount; k++) {
            for (int i = 1; i <= vexCount; i++) {
                for (int j = 1; j <= vexCount; j++) {
                    if (arcs[i][k] < Integer.MAX_VALUE && arcs[k][j] < Integer.MAX_VALUE) {
                        final int d = arcs[i][k] + arcs[k][j];
                        if (d < arcs[i][j]) { //经过k点时i到j的距离比不经过k点的距离更短
                            arcs[i][j] = d; //更新i到j的最短距离
                            path[i][j] = path[i][k]; //更新i到j经过的最后一个点为k点
                        }
                    }
                }
            }
            print(arcs, vexCount, k);
        }

        printPath(arcs, path, vexCount);
    }

    private static void print(int arcs[][], int vexCount, int index) {
        Log.d(TAG, "print array step of " + index + ":");
        for (int i = 1; i <= vexCount; i++) {
            StringBuilder builder = new StringBuilder();
            for (int j = 1; j <= vexCount; j++) {
                if (arcs[i][j] < Integer.MAX_VALUE) {
                    builder.append(String.format("%5d", arcs[i][j])).append(" ");
                } else {
                    builder.append(String.format("%5s", "∞")).append(" ");
                }
            }
            builder.append("\n");
            Log.d(TAG, builder.toString());
        }
    }

    private static void printPath(int arcs[][], int path[][], int vexCount) {
        Log.d(TAG, "print path:");
        int temp;
        for (int i = 1; i <= vexCount; i++) {
            StringBuilder builder = new StringBuilder();
            for (int j = 1; j <= vexCount; j++) {
                builder.append(i).append("->").append(j)
                    .append(", weight: "). append(arcs[i][j]).append(":").append(i);
                temp = path[i][j];
                while(temp != j) {
                    builder.append("->").append(temp);
                    temp = path[temp][j];
                }
                builder.append("->").append(j).append("\n");
            }
            Log.d(TAG, builder.toString());
        }
    }
}
```