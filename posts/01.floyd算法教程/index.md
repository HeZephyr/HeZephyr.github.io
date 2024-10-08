# Floyd算法教程

## 简介

Floyd算法又称为插点法，是一种利用动态规划的思想寻找给定的加权图中多源点之间最短路径的算法，可以正确处理有向图或无向图或负权（但不可存在负权回路)的最短路径问题，同时也被用于计算有向图的传递闭包。该算法名称以创始人之一、1978年图灵奖获得者、斯坦福大学计算机科学系教授罗伯特·弗洛伊德命名。

## 算法原理

在给出的一张具有权值图中，我们已知每个顶点v与每个顶点u中的最短距离(即使无法到达，我们也认为是有距离的，但距离为无穷大）。对于最短距离表示，我们用`dis[v][u]`来表示顶点v到顶点u的最短距离，一开始没进行任何操作时，则表示他们的最短距离，就是顶点v到达顶点u的直接距离。

OK，那么如果我现在有一个中转站（中转顶点temp），我先从顶点v到达顶点temp，再从顶点temp到达顶点u，则这段距离我们可以表示为`dis[v][temp]&#43;dis[temp][u]`，如果这段距离比我们目前的直接到达方式更小的话，那么我们不就可以更新我们的最短距离`dis[v][u]`了吗？那么temp这个中转顶点可以是图中的所有顶点，如果我们不断把所有顶点插入作为中转顶点，再不断更新最短距离，最后，得到的`dis[n][n]`自然是多源点之间的最短路径了。我们由上述推导也可求得我们的状态转移方程`dis[v][u]=min(dis[v][u],dis[v][temp]&#43;dis[temp][u])`，这就是Floyd算法的思想与原理，不断插入中转顶点，利用动态规划思想来实现。

这里我们还要记录最短路径的方案，针对有些要求最短路径方案的题，我们用`path[v][u]`的值来记录顶点v到顶点u的中转顶点，若为-1则表示无中转顶点。

## 邻接矩阵解题模板

存储结构：

```cpp
const int maxn=1000;//maxn表示图的最大顶点数
int graph[maxn][maxn];
int n;//图的实际顶点数
int dis[maxn][maxn];//最短距离
const int inf=0x3f3f3f3f;//代表无穷大
int path[maxn][maxn];//记录中转顶点
```

```cpp
//Floyd算法核心
void floyd(int n){
    //顶点数目。
    int i,j,k;//循环变量。
    memset(dis,inf,sizeof(dis));//调用这个函数需包含memory.h头文件
    memset(path,-1,sizeof(path));
    for(i=0;i&lt;n;i&#43;&#43;){
        for(int j=0;j&lt;n;j&#43;&#43;){
            //初始化dis。
            dis[i][j]=graph[i][j];
        }
    }
    for(k=0;k&lt;n;k&#43;&#43;){
    //不断插点
        for(i=0;i&lt;n;i&#43;&#43;){
            for(j=0;j&lt;n;j&#43;&#43;){
            //对图中所有点之间的最短距离进行更新。
                if(dis[i][j]&gt;dis[i][k]&#43;dis[k][j]){
                    path[i][j]=k;//记录中转顶点。
                    dis[i][j]=dis[i][k]&#43;dis[k][j];//更新。
                }
            }
   	    }
    }
}

```

我们现在来输出最短路径，由于我们记录了中转顶点，那么我们可以借助我们求得的`path数组`来实现。如果`path[i][j]`的值不为-1的话，说明这个值就为中转顶点，可这样最短路径就求出来了吗？显然不是，我们还要继续判断i和k之间以及k和j之间有没有中转顶点。我们可以利用递归思想来实现这个方法输出最短路径。

```cpp
void print_path(int i,int j){
    int k=path[i][j];
    if(k==-1)
        //说明没有中转顶点，直接返回.
        return;
    print_path(i,k);//寻找i和k之间
    cout&lt;&lt;k&lt;&lt;&#34;,&#34;;
    print_path(k,j);//寻找k和j之间
}
```

最后，再设置一个打印我们所求的结果的方法，比较简单，核心就是上面两个函数。

```cpp
void print_result(){
    int i,j;
    for(i=0;i&lt;n;i&#43;&#43;){
        for(j=0;j&lt;n;j&#43;&#43;){
              if(dis[i][j]==inf){
                  //我们先前说过，无法到达我们设置距离为无穷大，则他们之前没有路径
                  cout&lt;&lt;i&lt;&lt;&#34;和&#34;&lt;&lt;j&lt;&lt;&#34;之间没有路径&#34;&lt;&lt;endl;
              }
              else{
                  cout&lt;&lt;i&lt;&lt;&#34;和&#34;&lt;&lt;j&lt;&lt;&#34;的最短路径为&#34;&lt;&lt;dis[i][j]&lt;&lt;endl;
                  cout&lt;&lt;&#34;路径方案为：&#34;&lt;&lt;i&lt;&lt;&#34;,&#34;;
                  print_path(i,j);
                  cout&lt;&lt;j&lt;&lt;endl;
              }
        }
    }
}
```

***

## 邻接表解题模板

存储结构：

```cpp
const int maxn=1000;
typedef struct arc{
    //边的信息
    int adjust;//该边所指向的顶点。
    int weight;//该边的权值。
    struct arc *next;//指向一条边。
}arc,*arclink;

typedef struct vex{
    arclink firstarc;//该顶点的第一条边。
}vex,vexs[maxn];
typedef struct graph{
    vexs adj;
    int arcnum;//边数
    int vexnum;//顶点数
}graph;
graph G;
```

//初始化操作就不在这写了，最短距离还是用`dis[maxn][maxn]`表示，中转站顶点用`path[maxn][maxn]`毕竟我们的核心是解决Floyd算法。

```cpp
void floyd(){
    int i,j,k;
    //初始化dis
    for(i=0;i&lt;G.vexnum;i&#43;&#43;){
        for(j=0;j&lt;G.vexnum;j&#43;&#43;){
            if(i==j)dis[i][j]=0;
            else dis[i][j]=inf;
        }
    }
    memset(path,-1,sizeof(path));//初始化path数组
    arclink *p;//辅助作用
    for(i=0;i&lt;G.vexnum;i&#43;&#43;){
        p=G.adj[i].firstarc;
        while(p){
            //此操作是为了记录所有顶点之间的距离
            dis[i][s-&gt;adjust]=s-&gt;wight;
            s=s-&gt;next;
        }
    }
    //到了这一步，就跟上面的是一样了，因为我们已经得到了dis数组和path数组的值了。下面利用dp并记录中转顶点，同种套路。
}
```

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.floyd%E7%AE%97%E6%B3%95%E6%95%99%E7%A8%8B/  

