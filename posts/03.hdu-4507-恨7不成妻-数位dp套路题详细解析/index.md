# HDU 4507 恨7不成妻 （数位DP套路题）

不会数位$DP$的这里指路一篇介绍非常详细的数位$DP$的$blog$:[点这里。](https://unique-pure.github.io/pages/243023/)

* **链接**
	[恨7不成妻](http://acm.hdu.edu.cn/showproblem.php?pid=4507)

* **题面**

	&gt;单身!
	&gt;依然单身！
	&gt;吉哥依然单身！
	&gt;DS级码农吉哥依然单身！
	&gt;所以，他生平最恨情人节，不管是214还是77，他都讨厌！
	&gt;吉哥观察了214和77这两个数，发现：
	&gt;$2&#43;1&#43;4=7$　
	&gt;$7&#43;7=7*2$
	&gt;$77=7*11$
	&gt;最终，他发现原来这一切归根到底都是因为和7有关！所以，他现在甚至讨厌一切和7有关的数！什么样的数和7有关呢？如果一个整数符合下面3个条件之一，那么我们就说这个整数和7有关——
	&gt;　　　1、整数中某一位是7；
	&gt;　　　2、整数的每一位加起来的和是7的整数倍；
	&gt;　　　3、这个整数是7的整数倍；
	&gt;
	&gt;　现在问题来了：吉哥想知道在一定区间内和7无关的数字的平方和。
	&gt;　Input
	&gt;　输入数据的第一行是case数T(1 &lt;= T &lt;= 50)，然后接下来的T行表示T个case;每个case在一行内包含两个正整数L, R(1 &lt;= L &lt;= R &lt;= 10^18)。
	&gt;　Output
	&gt;　请计算[L,R]中和7无关的数字的平方和，并将结果对10^9 &#43; 7 求模后输出。
	&gt;　Sample Input
	&gt;
	&gt;```c&#43;&#43;
	&gt;3
	&gt;1 9
	&gt;10 11
	&gt;17 17
	&gt;```
	&gt;
	&gt;　Sample Output
	&gt;
	&gt;```c&#43;&#43;
	&gt;236
	&gt;221
	&gt;0
	&gt;```

* **解题思路**

	根据题意我们做出预处理，利用闫式$DP$分析法分析如下：

![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/2f5f90205e029c3471d7dc4d0dea96de.png)

  以上只是简单分析，我们还并没有真正的进行状态转移和计算，那么根据题意，首先是需要知道整数的每一位加起来的和是$7$的整数倍以及该整数是$7$的整数倍，这个好处理，在我们的前面的题中有类似的题型，这已经在我们的$f$数组的第三维和第四维了。所以难点就在于怎么处理整数的平方和。我们看下面的公式推导：

 &lt;font color=&#34;red&#34;&gt;我们用$jA$来表示$i$位数，而其中的$A$为$i-1$位数。设这个状态有$t$个符合要求的数，分别是$A_1$~$A_t$。&lt;/font&gt; 那么，平方和易得为：

  $(jA_1)^2&#43;(jA_2)^2&#43;(jA_3)^2&#43;...&#43;(jA_{t-1})^2&#43;(jA_t)^2$

  （我们分割表示将$A$提取出来。）

  $=(j*10^{i-1}&#43;A_1)^2&#43;(j*10^{i-1}&#43;A_2)^2&#43;(j*10^{i-1}&#43;A_3)^2&#43;...&#43;(j*10^{i-1}&#43;A_{t-1})^2&#43;(j*10^{i-1}&#43;A_t)^2$

  （平方和公式）

  $=t*(j*10^{i-1})^2&#43;2*(j*10^{i-1})*(A_1&#43;...&#43;A_t)&#43;(A_1^2&#43;...&#43;A^2)$

  这样，在这个式子中，由于$j$已知，所以我们发现$f$数组需要保存三个值。$A$的$0$次方之和，也就是符合要求的数，$A$的$1$次方之和，也就是符合要求的除去$j$的$i-1$位数相加，$A$的$2$次方之和，也就是符合要求的除去$j$的$i-1$位数平方相加。我们分别用$s_0,s_1,s_2$

  分别代表上述的三个值。

  那么这里我们需要怎么求$s_1$，如下：

  注：这里的$s_1$为$i&#43;1$位的$s_1$，而它存储的就是$i$位的$A$。

  $jA_1&#43;...&#43;jA_t$

  $=j*10^{i-1}&#43;(A_1&#43;...&#43;A_t)$

  所以我们的$f$应该是一个结构体数组，它需要存取$s_0,s_1,s_2$。那么预处理根据上述分析其实就简单了。那么就按照数位$DP$的套路解决这道题即可。需要注意这道题好多坑点，多取模，足够细心才可以解决。（调$Bug$调了好久。快绝望了。）


* **代码**

```cpp
/**
  *@filename:恨7不成妻
  *@author: pursuit
  *@csdn:unique_pursuit
  *@email: 2825841950@qq.com
  *@created: 2021-05-12 21:19
**/
#include &lt;bits/stdc&#43;&#43;.h&gt;

using namespace std;

typedef long long ll;
const int N = 20;
const ll P = 1e9&#43;7;

//需要满足三个性质。
//1.不含7.
//2.各位数字之和模7不为0.an-1&#43;...&#43;a0%7!=0. 
//3.该数模7不为0.an-1*pow(10,n-1)&#43;...&#43;a0&#43;pow(10,0)%7!=0

struct F{
    ll s0,s1,s2;//s0为符合要求的数。s1为符合要求的数1次方之和，s2为符合要求的数的2次方之和。
}f[N][10][7][7];//f[i][j][k][u]表示总共有i位数且最高位是j，该数值模7为k，各位数数字之和模7为u的所有数的s0,s1,s2.
//进行初始化。
int t;//测试数。
ll l,r;
ll power7[N],power9[N];//power7[i]存储10^i余7的余数，power9[i]存储10^i余P的余数。
ll mod(ll x,ll y){
    return (x%y&#43;y)%y;
}
void init(){
    //确定初始值，位数为1的情况。
    for(int j=0;j&lt;10;j&#43;&#43;){
        if(j==7)continue;
        //根据性质排除不符合要求的。
        F &amp;v=f[1][j][j%7][j%7];//这里用引用减少代码量。
        v.s0&#43;&#43;;
        v.s1&#43;=j;
        v.s2&#43;=j*j;
    }
    ll power = 10;//辅助作用，表示10的i-1次方。
    for(int i=2;i&lt;N;i&#43;&#43;,power*=10){
        for(int j=0;j&lt;10;j&#43;&#43;){
            if(j==7)continue;//排除不符合要求的数。
            for(int k=0;k&lt;7;k&#43;&#43;){
                for(int u=0;u&lt;7;u&#43;&#43;){
                    for(int q=0;q&lt;10;q&#43;&#43;){
                        //枚举i-1的最高位。
                        if(q==7)continue;
                        F &amp;x=f[i][j][k][u],y=f[i-1][q][mod(k-j*(power%7),7)][mod(u-j,7)];
                        //s0,s1,s2都是通过公式就算得到。
                        x.s0=mod(x.s0&#43;y.s0,P);
                        x.s1=mod(x.s1&#43;1LL*j%P*(power%P)%P*y.s0%P&#43;y.s1,P);
                        x.s2=mod(x.s2&#43;
                            1LL*j%P*y.s0%P*(power%P)%P*j%P*(power%P)%P&#43;
                            1LL*y.s1%P*2%P*j%P*(power%P)%P&#43;y.s2,P);
                    }
                }
            }
        }
    }
    //这里处理为了方便以及降低时间复杂度。
    power7[0]=1,power9[0]=1;
    for(int i=1;i&lt;N;i&#43;&#43;){
        power7[i]=power7[i-1]*10%7;
        power9[i]=power9[i-1]*10%P;
    }
}
F get(int i,int j,int k,int u){
    //因为f[i][j][k][u]是本身模7等于k，且各位数之和模7等于u的，所以我们需要找出符合条件的集合。
    ll s0=0,s1=0,s2=0;
    for(int x=0;x&lt;7;x&#43;&#43;){
        for(int y=0;y&lt;7;y&#43;&#43;){
            if(x==k||y==u)continue;
            F v=f[i][j][x][y];
            s0=mod(s0&#43;v.s0,P);
            s1=mod(s1&#43;v.s1,P);
            s2=mod(s2&#43;v.s2,P);
        }
    }
    return {s0,s1,s2};
}
ll dp(ll n){
    if(!n)return 0;//0的平方和为0.
    vector&lt;int&gt; a;
    ll temp=n%P;//备份一个n，供后面判断n使用。
    while(n)a.push_back(n%10),n/=10;
    ll last_a=0,last_b=0;//这里我们需要存储前缀的本身值和前缀的个位数之和。
    ll ans=0;//答案。
    for(int i=a.size()-1;i&gt;=0;i--){
        int x=a[i];
        for(int j=0;j&lt;x;j&#43;&#43;){
            //走左分支。
            if(j==7)continue;
            //我们需要将符合条件的数筛出来，这里要用到一个get函数。
            //求得本身模7不等于a，并且各位数之和模7不等b的集合，此时就可以使用预处理出来的结构体
            int k=mod(-last_a*power7[i&#43;1],7),u=mod(-last_b,7);
            F v=get(i&#43;1,j,k,u);
            //cout&lt;&lt;v.s0&lt;&lt;&#34; &#34;&lt;&lt;v.s1&lt;&lt;&#34; &#34;&lt;&lt;v.s2&lt;&lt;endl;
            //根据公式求解s2.
            //j就是last_a.
            ans=mod(ans&#43;
                1LL*(last_a%P)*(last_a%P)%P*(power9[i&#43;1]%P)%P*(power9[i&#43;1]%P)%P*v.s0%P&#43;
                1LL*2*last_a%P*(power9[i&#43;1]%P)%P*v.s1%P&#43;
                v.s2,P);
            //cout&lt;&lt;ans&lt;&lt;endl;
        }
        //判断x。
        if(x==7)break;
        //走右分支更新。
        last_a=last_a*10&#43;x;
        last_b&#43;=x;
        //判断自己本身是否符合要求。
        if(!i&amp;&amp;last_a%7&amp;&amp;last_b%7){
            ans=mod(ans&#43;temp*temp%P,P);
        }
    }
    return ans;
}
int main(){
    init();
    cin&gt;&gt;t;
    while(t--){
        cin&gt;&gt;l&gt;&gt;r;
        cout&lt;&lt;mod(dp(r)-dp(l-1),P)&lt;&lt;endl;
    }
    return 0;
}
/*
1
1 1000000000000000000
*/
```

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/03.hdu-4507-%E6%81%A87%E4%B8%8D%E6%88%90%E5%A6%BB-%E6%95%B0%E4%BD%8Ddp%E5%A5%97%E8%B7%AF%E9%A2%98%E8%AF%A6%E7%BB%86%E8%A7%A3%E6%9E%90/  

