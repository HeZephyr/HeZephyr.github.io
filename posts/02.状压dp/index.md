# 状压DP学习总结&#43;经典例题精解

## 前言

&gt; 学了这么久，说真的，动态规划是一个特别难的领域，而状压$DP$我感觉是其中一个比较难的分支，其中的状态定义、状态转移、状态计算都是难点。如果要完全搞懂状压$DP$是需要花很多时间去吸收去实践的，所以建议读者多刷$DP$题。同时，学习本文的先修知识为二进制位运算操作、基础动态规划和动态规划的分析。这里指路一篇二进制讲解$blog$:[点这里](https://unique-pure.github.io/pages/41f1cd/)。

## 状态压缩

我们知道状态压缩，顾名思义，&lt;font color=&#34;red&#34;&gt;就是需要考虑的状态非常多，我们如果用平常的思想去表示状态，那是非常不现实的，在时间和空间上都不允许，我们使用某种方法，以最小的代价表示某种状态。&lt;/font&gt; 那么，这通常是用进制来表示状态的，而选择几进制则根据要求使用的对象的点的状态有几种。一般来说，只有$0$和$1$，我们则是用二进制来表示，当然也有其他进制的题，在例题中会列举，需要我们灵活变通，主要谈二进制。

那么如何用二进制表示状态呢？我们发现，二进制上是按位分的，那么我们每一位可以看成一个点，而点上的取值则为该点的状态或者选择。例如$00001001$这个状态则表示第一个点和第四个点状态为$1$，其余的点状态为$0$。所以按照这种思想，我们能抽象的表示出一个很复杂的状态，实现了时间和空间的优化。

知道了这个，我们就按照正常动态规划的思想去写这类题目即可。

## 使用场景

由上我们知道，状态压缩其实是有适用环境的：

1. **状态需要有一定的状态单元。** 即一个状态应该是保存一个集合，其中的元素值对应着$0$或$1$，例如我们常见的棋盘，我们可以用$0$或$1$来表示棋子的放置状态。而整个集合即是一个$01$串，即二进制数，我们通常用十进制表示。那么我们再进行状态转移或者判断的时候，需要先将十进制转化为二进制，再将二进制转化为十进制。
2. **题目中限制的集合大小不会超过$20$。** 这是最显著的特征，为什么呢？我们知道如果用二进制表示状态，那么集合大小为$20$的二进制状态有$2^{20} - 1$，已经达到$1e7$的数量级了。
3. **具有动态规划的特性。** 对于动态规划，一般都是要求最优化某个值，具有最优子结构的性质。同时也需要满足状态转移的特性，而不是前一个状态毫无关系的。

## 常用模板

下面的模板适用于大多数题目，特殊题目需要灵活变动，总之，多刷题自然就都会了。

```cpp
int n;
int maxn = 1 &lt;&lt; n;//总状态数。
    //枚举已有的集合数。按照状态转移的顺序，一般从小编号到大编号。
    for(int i = 1; i &lt;= m; &#43;&#43; i){
        //枚举当前集合中的状态。
        for(int j = 0; j &lt; maxn; &#43;&#43; j){
            //判断当前集合是否处于合法状态，通常我们需用一个数组提前处理好。如g数组;
            if(当前状态是否合格){
                for(int k = 0; k &lt; maxn; &#43;&#43; k){
                    //枚举上一个集合的状态。
                    if(上一个集合的状态是否合格 &#43; 上一个集合的状态和当前状态的集合是否产生了冲突){
                        列写状态转移方程。
                    }
                }
            }
        }
    }
}
```

## 经典例题

### [5.1 USACO06NOV Corn Fields G](https://www.luogu.com.cn/problem/P1879)

* **题面**

	&gt;农场主John新买了一块长方形的新牧场，这块牧场被划分成M行N列(1 ≤ M ≤ 12; 1 ≤ N ≤ 12)，每一格都是一块正方形的土地。John打算在牧场上的某几格里种上美味的草，供他的奶牛们享用。
	&gt;
	&gt;遗憾的是，有些土地相当贫瘠，不能用来种草。并且，奶牛们喜欢独占一块草地的感觉，于是John不会选择两块相邻的土地，也就是说，没有哪两块草地有公共边。
	&gt;
	&gt;John想知道，如果不考虑草地的总块数，那么，一共有多少种种植方案可供他选择？（当然，把新牧场完全荒废也是一种方案）
	&gt;
	&gt;**输入格式**
	&gt;
	&gt;第一行：两个整数M和N，用空格隔开。
	&gt;
	&gt;第2到第M&#43;1行：每行包含N个用空格隔开的整数，描述了每块土地的状态。第i&#43;1行描述了第i行的土地，所有整数均为0或1，是1的话，表示这块土地足够肥沃，0则表示这块土地不适合种草。
	&gt;
	&gt;**输出格式**
	&gt;
	&gt;一个整数，即牧场分配总方案数除以100,000,000的余数。
	&gt;
	&gt;**输入**
	&gt;
	&gt;```
	&gt;2 3
	&gt;1 1 1
	&gt;0 1 0
	&gt;```
	&gt;
	&gt;**输出**
	&gt;
	&gt;```
	&gt;9
	&gt;```

* **解题思路**

	我们先作出规定，定义$n$代表的是行，$m$代表的是列。那么牧场大小就是$n\times m$。我们看到数据范围,$n,m$都特别小，同时所求为方案数，这很符合状压DP的适用条件。&lt;font color=&#34;red&#34;&gt;那么对于每一行，我们就可以看成一个未知集合，而集合的大小自然就是列$m$。对于每一个单元，其取值范围为$0,1$，而$1$代表放置奶牛，$0$代表不放置奶牛，所以我们自然可以用二进制表示，那么状态总数就是$(1 &lt;&lt; m) - 1$。&lt;/font&gt; 对于每一个状态，我们需要判断是否合格，而其中明确不能选择两块相邻的土地，在集合内，即相邻位不能全为$1$，所以我们可以预处理$g$数组，处理方式即为:`g[i] = !(i &amp; (i &lt;&lt; 1))`；同样，我们还应该知晓土地的状况，因为毕竟只有土地肥沃才可以放置奶牛，则我们可以通过一个$st$数组判断，集合与集合之间，我们也需要考虑相邻位不能全为$1$，所以在枚举上一个集合的状态也需要严格判断。对于状态定义，我们可以用$f[i][j]$表示第$i$行且状态为$j$的方案数。对于状态转移，假设上一行状态为$k$，则状态转移方程为：

	$f[i][j] &#43;= f[i - 1][k]$

	具体见$AC$代码。

* **AC代码**

```cpp
/**
  *@filename:Corn Field G
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-28 16:50
**/
#include &lt;bits/stdc&#43;&#43;.h&gt;

using namespace std;

typedef long long ll;
const int N = 10 &#43; 5,M = 10 &#43; 5;
const int P = 1e8;

int n,m;//n行m列的土地。
int a[N][M],st[N];//a代表土地，st代表每一行的土地状况。
bool g[1 &lt;&lt; N];//g得到所有状态中的合法状态。
int f[N][1 &lt;&lt; N];//f[i][j]表示的则是第i行且状态为j的方案数，是由上一行转移过来的，所以我们定义上一行的状态为k。
//则状态转移方程为f[i][j] &#43;= f[i - 1][k];//其中j和k必须满足条件。
void solve(){
}
int main(){
    scanf(&#34;%d%d&#34;, &amp;n, &amp;m);
    for(int i = 1; i &lt;= n; &#43;&#43; i){
        for(int j = 1; j &lt;= m; &#43;&#43; j){
            scanf(&#34;%d&#34;, &amp;a[i][j]);
        }
    }
    //得到每一行的土地状况。
    for(int i = 1; i &lt;= n; &#43;&#43; i){
        for(int j = 1; j &lt;= m; &#43;&#43; j){
            st[i] = (st[i] &lt;&lt; 1) &#43; a[i][j];
        }
    }
    //得到所有状态中的合法状态。
    int maxn = 1 &lt;&lt; m;//总状态。
    f[0][0] = 1;//初始化，这种也算一种。
    for(int i = 0; i &lt; maxn; &#43;&#43; i){
        g[i] = !( i &amp; (i &lt;&lt; 1));//由于不能相邻，所以我们左移判断是否符合条件。 
    }
    for(int i = 1; i &lt;= n; &#43;&#43; i){
        //枚举每一行。
        for(int j = 0; j &lt; maxn; &#43;&#43; j){
            //枚举每一行的状态，判断此状态是否符合条件。1.不能相邻。2.是全部状态的子集。
            if(g[j] &amp;&amp; (j &amp; st[i]) == j){
                //如果符合条件。则我们去判断上一行是否符合。
                for(int k = 0; k &lt; maxn; &#43;&#43; k){
                    //枚举上一行状态。注意，这里我们无需判断上一行状态是否存在，因为不存在即为0.
                    //只需要判断j和k是否存在相邻草地。
                    if(!(j &amp; k)){
                        f[i][j] = (f[i][j] &#43; f[i - 1][k]) % P;
                    }
                }
            }
        }
    }
    int ans = 0;
    for(int j = 0; j &lt; maxn; &#43;&#43; j){
        ans = (ans &#43; f[n][j]) % P;
    }
    printf(&#34;%d\n&#34;, ans);
    solve();
    return 0;
}
```

### [5.2 吃奶酪](luogu.com.cn/problem/P1433)

* **题面**

	&gt; 房间里放着$n$块奶酪。一只小老鼠要把它们都吃掉，问至少要跑多少距离？老鼠一开始在 $(0,0)$点处。
	&gt;
	&gt; **输入格式**
	&gt;
	&gt; 第一行有一个整数，表示奶酪的数量 n。
	&gt;
	&gt; 第 22 到第 (n &#43; 1)行，每行两个实数，第 $(i &#43; 1)$行的实数分别表示第 $i$ 块奶酪的横纵坐标 $x_i, y_i$。
	&gt;
	&gt; **输出格式**
	&gt;
	&gt; 输出一行一个实数，表示要跑的最少距离，保留 2位小数。
	&gt;
	&gt; **输入 #1**
	&gt;
	&gt; ```
	&gt; 4
	&gt; 1 1
	&gt; 1 -1
	&gt; -1 1
	&gt; -1 -1
	&gt; ```
	&gt;
	&gt; **输出 #1**
	&gt;
	&gt; ```
	&gt; 7.41
	&gt; ```
	&gt;
	&gt; **数据规模与约定**
	&gt;
	&gt; 对于全部的测试点，保证 $1\leq n\leq 15，|x_i|, |y_i| \leq 200$，小数点后最多有 3位数字。
	&gt;
	&gt; **提示**
	&gt;
	&gt; 对于两个点$$ (x_1,y_1)，(x_2, y_2)$$，两点之间的距离公式为$\sqrt{(x_1-x_2)^2&#43;(y_1-y_2)^2}$。

* **解题思路**

	同样，根据数据量等信息我们很容易发现这是一个状压$DP$。奶酪的状态无非两种$0,1$，而根据题意我们的集合数量只有$1$个，集合大小自然是奶酪的数量，而奶酪有$n$个，所以我们的集合情况也有$(1 &lt;&lt; n)-1$种，同样在此题我们需要先初始化好奶酪的合法状态，用$g$数组表示，更严格的说，$g[i]$表示第$i$个奶酪所在的二进制中的位置，用十进制数表示 。由于我们还需要计算距离，所以我们需要将每个点之间的距离也求出来，用$dist$数组预处理奶酪之间的距离以及起点与各个奶酪的距离。 &lt;font color=&#34;red&#34;&gt;那么在状态定义上，我们可以用$f[i][j]$表示当前为$i$状态，且处于第$j$个奶酪的最小距离，故状态转移方程易知为：&lt;/font&gt;

	&lt;font color=&#34;red&#34;&gt;$f[i][j]=min(f[i][j],f[i-g[j]][k]&#43;dist[k][j])$&lt;/font&gt;

	据此，题目可解，具体看代码。

* **AC代码**

```cpp
/**
  *@filename:吃奶酪
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-30 14:20
**/
#include &lt;bits/stdc&#43;&#43;.h&gt;
#define x first 
#define y second
using namespace std;

typedef long long ll;
typedef pair&lt;double,double&gt; pdd;
const int N = 16;
const int P = 1e9&#43;7;

int n;
pdd a[N];
double dist[N][N];//dist[i][j]表示第i个奶酪到第j个奶酪的距离。
int g[1 &lt;&lt; N];//奶酪的状态。
double f[1 &lt;&lt; N][N];//f[i][j]表示当前为i状态，且处于第j个奶酪的最小距离。
pdd st;
double get(pdd a,pdd b){
    return sqrt(pow(a.x - b.x,2) &#43; pow(a.y - b.y,2));
}
void solve(){
    //先计算距离。
    st.x = 0,st.y = 0;
    for(int i = 1; i &lt;= n; &#43;&#43; i){
        dist[0][i] = get(st,a[i]);
    }
    for(int i = 1; i &lt;= n; &#43;&#43; i){
        for(int j = 1; j &lt;= n; &#43;&#43; j){
            dist[i][j] = get(a[i],a[j]);
        }
    }
     int maxn = 1 &lt;&lt; n;
    //初始化奶酪的状态。
    g[1] = 1;
    for(int i = 2; i &lt;= n; &#43;&#43; i){
        g[i] = g[i - 1] &lt;&lt; 1;
    }
    //初始化最大值。
    fill(f[0],f[0] &#43; (1 &lt;&lt; N) * N,0x3f3f3f3f);
    //确定只吃了一个奶酪的距离。
    for(int i = 1; i &lt;= n; &#43;&#43; i){
        f[g[i]][i] = dist[0][i];
    }
    f[0][0] = 0;//最开始自然为0，0.
    for(int i = 0; i &lt; maxn ; &#43;&#43; i){
        //枚举所有状态。
        for(int j = 1; j &lt;= n; &#43;&#43; j){
            if(i &amp; g[j]){
                //该状态如果包含此奶酪就跳过。
                for(int k = 1; k &lt;= n; &#43;&#43; k){
                    if(k != j &amp;&amp; i &amp; g[k]){
                        //说明符合条件。
                        f[i][j] = min(f[i][j],f[i - g[j]][k] &#43; dist[k][j]);//进行状态转移。
                    }
                }
            }
        }
    }
    double maxx = 0x3f3f3f3f;
    for(int i = 1; i &lt;= n; &#43;&#43; i){
        maxx = min(maxx,f[maxn - 1][i]);
    }
    printf(&#34;%.2lf\n&#34;,maxx);
}
int main(){
    scanf(&#34;%d&#34;, &amp;n);
    for(int i = 1; i &lt;= n; &#43;&#43; i){
        scanf(&#34;%lf%lf&#34;, &amp;a[i].x, &amp;a[i].y);
    }
    solve();
    return 0;
}
```

### [5.3  USACO13NOV No Change G](https://www.luogu.com.cn/problem/P3092)

* **题面**

	&gt;约翰到商场购物，他的钱包里有K(1 &lt;= K &lt;= 16)个硬币，面值的范围是1..100,000,000。
	&gt;
	&gt;约翰想按顺序买 N个物品(1 &lt;= N &lt;= 100,000)，第i个物品需要花费c(i)块钱，(1 &lt;= c(i) &lt;= 10,000)。
	&gt;
	&gt;在依次进行的购买N个物品的过程中，约翰可以随时停下来付款，每次付款只用一个硬币，支付购买的内容是从上一次支付后开始到现在的这些所有物品（前提是该硬币足以支付这些物品的费用）。不幸的是，商场的收银机坏了，如果约翰支付的硬币面值大于所需的费用，他不会得到任何找零。
	&gt;
	&gt;请计算出在购买完N个物品后，约翰最多剩下多少钱。如果无法完成购买，输出-1
	&gt;
	&gt;**输入格式**
	&gt;
	&gt;\*Line 1: Two integers, K and N.
	&gt;
	&gt;\* Lines 2..1&#43;K: Each line contains the amount of money of one of FJ&#39;s coins.
	&gt;
	&gt;\* Lines 2&#43;K..1&#43;N&#43;K: These N lines contain the costs of FJ&#39;s intended purchases.
	&gt;
	&gt;**输出格式**
	&gt;
	&gt;\* Line 1: The maximum amount of money FJ can end up with, or -1 if FJ cannot complete all of his purchases.
	&gt;
	&gt;**输入 #1**
	&gt;
	&gt;```
	&gt;3 6 
	&gt;12 
	&gt;15 
	&gt;10 
	&gt;6 
	&gt;3 
	&gt;3 
	&gt;2 
	&gt;3 
	&gt;7 
	&gt;```
	&gt;
	&gt;**输出 #1**
	&gt;
	&gt;```
	&gt;12 
	&gt;```
	&gt;
	&gt;说明/提示
	&gt;
	&gt;FJ has 3 coins of values 12, 15, and 10. He must make purchases in sequence of value 6, 3, 3, 2, 3, and 7.
	&gt;
	&gt;FJ spends his 10-unit coin on the first two purchases, then the 15-unit coin on the remaining purchases. This leaves him with the 12-unit coin.

* **解题思路**

	此题状态定义比较简单，因为实际上我们只在乎了硬币的花费，这已经是一个集合了，花费为$1$不花费为$0$，而其他并不用在乎。所以我们完全可以用$f[i]$表示在$i$状态下能够购买的最大物品数。此题难点在于状态转移。同样在此题我们需要先初始化好硬币的合法状态，用$g$数组表示，更严格的说，$g[i]$表示第$i$个硬币所在的二进制中的位置。用十进制数表示。为了方便处理，我们需要用前缀和来优化本题，因为在处理过程中我们随时都要计算当前已有的总价值能够换取多少物品。 &lt;font color=&#34;red&#34;&gt;同样，在状态转移方面，我们需要根据前面的状态得到后者的状态，而由于我们是从小到大枚举状态的，故一定可以利用前面的状态而不会出现前面状态不是最优解，我们对于每一种状态，我们可以排除一个硬币获取前面的最优解，即枚举该状态已有的硬币，通过异或排除，最后利用二分查找所能购买的最大值得到最优解。&lt;/font&gt; 说起来有点乱，详情可见$AC$代码。

* **AC代码**

```cpp
/**
  *@filename:No_Change_G
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-28 17:14
**/
#include &lt;bits/stdc&#43;&#43;.h&gt;

using namespace std;

typedef long long ll;
const int N = 1000000 &#43; 5,K = 20;
const int P = 1e9&#43;7;

int n,k;
int g[K];//每个硬币的状态。
int w[K];//硬币的价值
int sum[N];//物品价值的前缀和。
int ans;
int f[1 &lt;&lt; K];//f[i]表示在i状态下能够购买的最大物品数。
void solve(){
    //初始化硬币状态。
    g[1] = 1;
    for(int i = 2; i &lt;= k; &#43;&#43; i){
        g[i] = g[i -1] &lt;&lt; 1;
    }
    int maxn = 1 &lt;&lt; k;//得到
    for(int i = 0; i &lt; maxn; &#43;&#43; i){
        //枚举每一种状态。
        for(int j = 1; j &lt;= k; &#43;&#43; j){
            //枚举所有的硬币。
            if(i &amp; g[j]){
                //说明该硬币在当前状态使用过。
                int te = f[i ^ g[j]];//获取该状态不使用j能获得的物品数。
                te = upper_bound(sum &#43; 1,sum &#43; n &#43; 1,sum[te] &#43; w[j]) - sum;//这里需要减1.
                f[i] = max(f[i],te - 1);
            }
        }
    }
    ll maxx = -2,temp;
    for(int i = 0; i &lt; maxn; &#43;&#43; i){
        if(f[i] == n){
            //说明该状态能够将所有物品都买完。
            temp = 0;
            for(int j = 1; j &lt;= k; &#43;&#43; j){
                if(i &amp; g[j]){
                    temp &#43;= w[j];
                }
            }
            maxx = max(maxx,ans - temp);
        }
    }
    if(maxx &lt; 0)printf(&#34;%d\n&#34;, -1);
    else printf(&#34;%d\n&#34;,maxx);
}
int main(){
    scanf(&#34;%d%d&#34;, &amp;k , &amp;n);
    for(int i = 1; i &lt;= k; &#43;&#43; i){
        scanf(&#34;%d&#34;, &amp;w[i]);
        ans &#43;= w[i];//求硬币总价值。
    }
    for(int i = 1; i &lt;= n; &#43;&#43; i){
        scanf(&#34;%d&#34;, &amp;sum[i]);
        sum[i] &#43;= sum[i - 1];
    }
    solve();
    return 0;
}
```

### [5.4 SCOI2005互不侵犯](https://www.luogu.com.cn/problem/P1896)

* **题面**

	&gt;在N×N的棋盘里面放K个国王，使他们互不攻击，共有多少种摆放方案。国王能攻击到它上下左右，以及左上左下右上右下八个方向上附近的各一个格子，共8个格子。
	&gt;
	&gt;**输入格式**
	&gt;
	&gt;只有一行，包含两个数N，K （ 1 &lt;=N &lt;=9, 0 &lt;= K &lt;= N * N）
	&gt;
	&gt;**输出格式**
	&gt;
	&gt;所得的方案数
	&gt;
	&gt;**输入 #1**
	&gt;
	&gt;```
	&gt;3 2
	&gt;```
	&gt;
	&gt;**输出 #1**
	&gt;
	&gt;```
	&gt;16
	&gt;```

* **解题思路**

	这道题跟$5.1$例题有点相似，只不过这里多了左上左下右上右下这几个点，处理方法一样，我们需要知道每个状态的放置国王数，所以这我们需要预处理。定义状态$f[i][j][k]$表示在第$i$行且处于$j$状态时已经放置了$k$个国王。其他的处理方式和$5.1$相同，这里不作叙述，可见$AC$代码。

* **AC代码**

	

```cpp
/**
  *@filename:互不侵犯
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-31 16:32
**/
#include &lt;bits/stdc&#43;&#43;.h&gt;

using namespace std;

typedef long long ll;
const int N = 10;
const int P = 1e9&#43;7;

int n,m;
int tot,num[1 &lt;&lt; N];//num[i]表示第i种可行状态的国王所放数量。
ll f[N][1 &lt;&lt; N][N * N];//f[i][j][k]表示前i行，当前处于j状态且已经放置了k个国王。
int get(int x){
    //获取该状态有多少国王。
    int sum = 0;
    for(int i = 0; i &lt; n; &#43;&#43; i){
        sum &#43;= (x &amp; 1);
        x &gt;&gt;= 1;
    }
    return sum;
}
void init(){
    int maxn = 1 &lt;&lt; n;
    for(int i = 0; i &lt; maxn; &#43;&#43; i){
        if(i &amp; (i &lt;&lt; 1))continue;//说明存在相邻的。
        num[i] = get(i);
        f[1][i][num[i]] = 1;
    }
}
void solve(){
    init();
    int maxn = 1 &lt;&lt; n;
    for(int i = 2; i &lt;= n; &#43;&#43; i){
        for(int j = 0; j &lt; maxn; &#43;&#43; j){
            //枚举所有状态。
            if(j &amp; (j &lt;&lt; 1))continue;
            for(int k = 0; k &lt; maxn; &#43;&#43; k){
                //枚举上一行的所有状态。
                if(((k &amp; (k &lt;&lt; 1)) || j &amp; k || ((j &lt;&lt; 1) &amp; k) || (j &amp; (k &lt;&lt; 1)))){
                    continue;
                }
                for(int cnt = num[j]; cnt &lt;= m; &#43;&#43; cnt){
                    f[i][j][cnt] &#43;= f[i - 1][k][cnt - num[j]];
                }
            }
        }
    }
    ll ans = 0;
    for(int  i= 0; i &lt; maxn; &#43;&#43; i){
        ans &#43;= f[n][i][m];
    }
    printf(&#34;%lld\n&#34;,ans);
}
int main(){
    scanf(&#34;%d %d&#34;, &amp;n, &amp;m);
    solve();
    return 0;
}
```


---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/02.%E7%8A%B6%E5%8E%8Bdp/  

