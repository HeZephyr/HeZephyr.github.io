# Dijkstra算法教程


**PS：此算法不能用于求负权图，要求所有边的权重都为非负值。**
***
## 简介

&gt;迪杰斯特拉算法(Dijkstra)是由荷兰计算机科学家狄克斯特拉于1959 年提出的，因此又叫狄克斯特拉算法。这是从一个顶点到其余各顶点的最短路径算法，解决的是有权图中最短路径问题。**迪杰斯特拉算法主要特点是从起始点开始，采用贪心算法的策略，每次遍历到始点距离最近且未访问过的顶点的邻接节点，直到扩展到终点为止**。
***
## 算法思想与原理

&gt;dijkstra算法思想是基于贪心算法思想的。**所谓贪心算法即始终保持当前迭代解为当前最优解。**意思就是在已知的条件下或是当前拥有的全部条件下保证最优解，若在此后的迭代中由于加入了新的条件使得产生了更优解则替代此前的最优解。通过不断的迭代不断保证每次迭代的结果都是当前**最优解**，那么当迭代到最后一轮时得到的就会是**全局最优解**。 由于下一轮迭代会参考上一轮的最优解，因此**每一轮的迭代的工作量基本一致**，降低了整体工作的复杂性。

在最短路径的问题中，局部最优解即**当前的最短路径**或者说是在当前的已知条件下起点到其余各点的最短距离。关键就在于已知条件，这也是Dijkstra算法最精妙的地方。我们来解释一下。

对于Dijkstra算法，我们假设初始集合（也就是初始条件）不存在任何顶点的，即所有顶点之间是不存在任何路径的，即我们认为所有顶点之间的距离都是无穷大。那么开始加入新的条件，因为我们已知源点距源点距离最小，所以加入进去，并加入它的边，**在该条件下，更新该源点到其余顶点的最短距离，选出没有加入到已知集合的距源点距离最小的点，此点最短距离也被确定了（因为其他路径都比这条路径大，无法通过其他路径间接到达这个顶点使得路径更小），然后加入该点与其余还未加入已知条件顶点的边，并以该点迭代刷新最短距离**。再重复以上操作，直至所有顶点都加入已知条件集合。
***
## 具体步骤

1.  选择起点$start$与终点$end$；
2. 所有点除起点外加入未知集合，并将起点加入已知集合，即至标志位为真，意为已确定该点到源点的最短路径；
3. 初始化计算，更新起点与其他各点的耗费$dis(start,n)$;
4. 在未知集合中，选择dis(start,n)中值最小的点x，将x加入已知集合。
5. **对于剩余顶点中，计算$dis(start,n)&gt;dis(start,x)&#43;dis(x,n)$
若真则$dis(start,n)=dis(start,x)&#43;dis(x,n)$，此时start与n点路径经过x点。循环直至goal点加入已知列表，取得$dis(start,goal)$即为最短距离。**
***
## 动态展示
![dijkstra](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/20210322130709747-20231125215629571-20231125215643983.gif)

## 一般代码实现（以邻接矩阵为例）
### 基本数据

```cpp
const int inf=0x3f3f3f3f; //代表无穷大。
const int maxn=100;//最大顶点数
int n,m;//n个顶点，m条边。
bool visited[maxn];//判断是否确定到源点的最终最短距离。
int graph[maxn][maxn];//带权图
int dis[maxn];//顶点到源点的最短距离。
int start,goal;//起点与目标点。
```

### 初始化

```cpp
void init(){
	memset(visited,false,sizeof(visited));
	for(int i=1;i&lt;=n;i&#43;&#43;){
		dis[i]=graph[start][i];//初始化dis数组。
	}
}
```

### dijkstra算法核心

```cpp
void dijkstra(){
	//源点为源点start。
	int minn;//记录每趟最短路径中最小的路径值。 
	int pos;//记录得到的minn所对应的下标。
	init();//调用初始化函数。
	visited[start]=true;
	for(int i=1;i&lt;=n;i&#43;&#43;){
		//将n个顶点依次加入判断。
		minn=inf;
		for(int j=1;j&lt;=n;j&#43;&#43;){
			if(!visited[j]&amp;&amp;dis[j]&lt;minn){
				minn=dis[j];
				pos=j;
			}
		}
		//经过这趟for循环后我们找到的就是我们想要的点，可以确定这点到源点的最终最短距离了。
		visited[pos]=true;//我们将此点并入已知集合。
		//接下来就是更新dis数组了，也就是当前最短距离，这都是针对还没有并入已知集合的点。
		for(int j=1;j&lt;=n;j&#43;&#43;){
			if(!visited[j]&amp;&amp;dis[j]&gt;dis[pos]&#43;graph[pos][j])
				dis[j]=dis[pos]&#43;graph[pos][j];
		}
	}
	//退出循环后，所有的点都已并入已知集合中，得到的dis数组也就是最终最短距离了。
	cout&lt;&lt;dis[goal]&lt;&lt;endl;//输出目标点到源点的最短路径长度。
}
```

### 主函数与头文件等

```cpp
#include&lt;bits/stdc&#43;&#43;.h&gt;

using namespace std;

int main()
{
    while(cin&gt;&gt;n&gt;&gt;m){
		memset(graph,inf,sizeof(graph));
		int u,v,w;
		for(int i=0;i&lt;m;i&#43;&#43;){
			cin&gt;&gt;u&gt;&gt;v&gt;&gt;w;
//			graph[u][v]=w;//有向图
			graph[u][v]=graph[v][u]=w;//无向图
		}
		cin&gt;&gt;start&gt;&gt;goal;//输入起点与终点。
		dijkstra();
    }
    return 0;
}
```

## 拓展
如果你学会了dijkstra，那恭喜你成功突破了一关。但是，没有优化的dijkstra算法时间复杂度为$O(n^2)$，如果顶点很多边很少呢等等卡邻接矩阵的题，那么建议你还是要学一下dijkstra的优化版了。详情点击：[Dijkstra算法优化~~你一定可以看懂的四种进阶优化](https://blog.csdn.net/hzf0701/article/details/107674290?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163080838116780269879915%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&amp;request_id=163080838116780269879915&amp;biz_id=0&amp;utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-107674290.pc_v2_rank_blog_default&amp;utm_term=dijkstra&amp;spm=1018.2226.3001.4450)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/02.dijkstra%E7%AE%97%E6%B3%95%E6%95%99%E7%A8%8B/  

