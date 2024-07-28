# 【矩阵论】Chapter 3—线性映射和线性变换知识点总结复习


## 线性映射及其矩阵表示

* 映射定义

	设$A,B$是两个集合，如果存在一个规则$f$，使得对于$A$中的元素$x$都有$B$中唯一的元素$y$与之对应，则称$f$是从$A$到$B$的映射，记作：$f:A\rightarrow B$。在映射$f:A\rightarrow B$中，$A$的元素$x$被映射到$B$的元素$y$，我们通常写作$f(x)=y$，

	如果$\forall x_1,x_2\in A,x_1\neq x_2,f(x_1)\neq f(x_2)$，则称映射$f:A\rightarrow B$是**单射**的；

	如果$\forall y\in B,\exist x\in A,f(x)=y$，则称映射$f:A\rightarrow B$是**满射**的；

	如果映射$f:A\rightarrow B$既满足单射又满足满射，则称映射$f:A\rightarrow B$是**双射**的。

* 线性映射定义

	设$V,W$是在数域$F$上的向量空间，如果$\forall v_1,v_2\in V,\forall \alpha_1,\alpha_2\in F$有$\sigma(\alpha_1v_1&#43;\alpha_2v_2)=\alpha_1\sigma(v_1)&#43;\alpha_2\sigma(v_2)$，则从$V$到$W$的映射$\sigma$称为线性映射。

* 线性映射定理

	设$\sigma,\gamma$是线性空间$V$到$W$的线性映射，则：

	1. $\sigma(0)=0$

	2. $\forall x\in V_1,\sigma(-x)=-\sigma(x)$

	3. 如果$x_1,\cdots,x_n$是$V_1$的一组向量，$k_1,\cdots,k_n\in F$，则有

		$\sigma(k_1x_1&#43;\cdots&#43;k_nx_n)=k_1\sigma(x_1)&#43;\cdots&#43;k_n\sigma(x_n)$

	4. 如果$x_1,\cdots,x_n$是$V_1$的一组线性相关向量，则$\sigma(x_1),\cdots,\sigma(x_n)$是$V_2$中的一组线性相关向量；并且当且仅当$\sigma$是一一映射时，$V_1$中的线性无关向量组的像（&lt;font color=&#34;red&#34;&gt;像即是线性映射的值域&lt;/font&gt;）是$V_2$中的线性无关向量组。

	5. 如果$v_1,\cdots,v_n$是$V$的一组基，且$\sigma(v_i)=\gamma(v_i)(1\leq i\leq n)$，则$\sigma=\gamma$。&lt;font color=&#34;red&#34;&gt;说明线性映射由基像组唯一确定。&lt;/font&gt;

* 线性映射运算

	设$V_1$到$V_2$的所有线性映射组成的集合记为$\varphi(V_1,V_2)$，类似地，$\varphi(V_1,V_3),\varphi(V_2,V_3)$分别表示$V_1$到$V_3$的所有线性映射组成的集合和$V_2$到$V_3$的所有线性映射组成的集合

	设$\sigma,\gamma \in \varphi(V_1,V_2)$，定义它们的和$\sigma&#43;\gamma$为$(\sigma&#43;\gamma)(x)=\sigma(x)&#43;\gamma(x),\forall x\in V_1$。

	1. $\sigma,\gamma \in \varphi(V_1,V_2)$，则$\sigma&#43;\gamma \in \varphi(V_1,V_2)$
	2. $\sigma\in \varphi(V_1,V_2),\gamma \in \varphi(V_2,V_3)$，则$\sigma \gamma \in \varphi(V_1,V_2)$

	线性映射的加法适合交换律和结合律，乘法适合结合律，标量乘法适合结合律，分配律。

* 重要定理

	设$\sigma \in \varphi(V_1,V_2)$，如果$\sigma$是可逆映射，则$\sigma^{-1}\in \varphi(V_2,V_1)$。

* 线性映射的矩阵表示

	设$\sigma:U\rightarrow V$是一个线性映射，$[u_1,\cdots,u_n]$是$U$的一组基，$\sigma$完全由$\sigma(u_1),\cdots,\sigma(u_n)$确定，如果$u=x_1u_1&#43;\cdots,x_nu_n$，则$\sigma(u)=x_1\sigma(u_1)&#43;\cdots&#43;x_n\sigma(u_n)$。

	设$v_1,\cdots,v_m$是$V$的一组基，则
	![image-20240728210304150](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/image-20240728210304150.png)
	故$[\sigma(u_1),\cdots,\sigma(u_n)]=[v_1,\cdots,v_m]A$，其中![](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex.png)。
    矩阵$A$称为线性映射$\sigma$在$U$的基$[u_1,\cdots,u_n]$和$V$的基$[v_1,\cdots,v_n]$下的表示矩阵。

* 重要定理

	设设$\sigma$为数域$F$上线性空间$U$到$V$的线性映射，其中$u_1,\cdots,u_n$是$U$的一组基，$v_1,\cdots,v_m$是$V$的一组基，$\sigma$在这对基下的矩阵是$A$，$\forall \alpha =\sum_{i=1}^nx_iu_i$，有$\sigma(\alpha)=\sum_{i=1}^my_iv_i$，则$[y_i,\cdots,y_m]^T=A[x_1,\cdots,x_n]$。

* 线性映射在不同基下的矩阵之间的关系

	&lt;font color=&#34;red&#34;&gt;同一个线性映射在不同基下的矩阵一般是不同的&lt;/font&gt;

	设$\sigma$为数域$F$上$n$维线性空间$U$到$n$维线性空间$V$的线性映射，其中$u_1,\cdots,u_n$和$u_1&#39;,\cdots,u&#39;_n$是$U$的两组基，由$u_1,\cdots,u_n$到$u_1&#39;,\cdots,u&#39;_n$的过渡矩阵是$Q$，$v_1,\cdots,v_m$和$v_1&#39;,\cdots,v_m&#39;$是$V$的两组基，由$v_1,\cdots,v_m$到$v_1&#39;,\cdots,v_m&#39;$的过渡矩阵是$P$，$\sigma$在基$u_1,\cdots,u_n$与基$v_1,\cdots,v_m$下的矩阵是$A$，而在基$u_1&#39;,\cdots,u&#39;_n$与基$v_1&#39;,\cdots,v_m&#39;$的矩阵为$B$，则$B=P^{-1}AQ$。

	推导：

	因为：
	![image-20240728210330175](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/image-20240728210330175.png)
	则把式子代入得到：
	![image-20240728210404961](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/image-20240728210404961.png)
	因为线性映射$\sigma$的矩阵由基唯一确定，所以$B=P^{-1}AQ$。

* 相抵

	设$A,B\in F^{m\times n}$，如果存在数域$F$上的$m$阶非奇异矩阵$P$和$n$阶非奇异矩阵$Q$使得$B=PAQ$，则称$A$与$B$相抵（等价）。

	&lt;font color=&#34;red&#34;&gt;如果$A$与$B$相抵，则它们可作为$n$维线性空间$U$到$m$维线性空间$V$的同一线性映射在两对基所对应的矩阵。&lt;/font&gt;

	相抵的充分必要条件是它们有相同的秩。

## 线性映射的值域（像）和核

* 值域（像）和核的定义

	设$\sigma$为数域$F$上线性空间$U$到$V$的线性映射，令$R(\sigma)=I_m(\sigma)=\{\sigma(x)| x\in U\}$，$Ker(\sigma)=N(\sigma)=\{x\in U|\sigma(x)=0\}$。

	称$R(\sigma)$是线性映射$\sigma$的值域（也称像），$Ker(\sigma)$是线性映射$\sigma$的核。

	易知$R(\sigma)$是$V$的一个子空间，$Ker(\sigma)$是$U$的一个子空间。

* 值域（像）和核理解

	值域（像）是映射所能到的空间，它包含了所有在映射过程中真实映射到的点，描述了映射的覆盖范围。值域（像）是目标空间 $W$的一个子空间。

	核是映射的零空间，它包含了所有被映射到零的输入向量，描述了映射的非单射性，即存在映射到同一个元素的不同输入。核是定义在$V$上的一个子空间。

* 定理

	设$\sigma$为数域$F$上$n$维线性空间$U$到$n$维线性空间$V$的线性映射，其中$u_1,\cdots,u_n$是$U$的一组基，$v_1,\cdots,v_m$是$V$的一组基，$\sigma$在这对基下的矩阵是$A$，则

	1. $R(\sigma)=span(\sigma(u_1),\cdots,\sigma(u_n))$
	2. $rank(\sigma)=rank(A)$
	3. $dim(R(\sigma))&#43;dim(Ker(\sigma))=n$

* 一般求法

	$R(\sigma)=R[x])_3$

	$Ker(\sigma)=\{0\}$

## 线性变换

* 定义

	设$V$是数域$F$上的线性空间，$V$到自身的线性映射称为$V$上的线性变换。

* $n$维线性空间$V$上的线性变换与矩阵之间的关系

	设$\sigma$是在$V$上的线性变换，$v_1,\cdots,v_n$是一组基，则
	![image-20240728213739259](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/image-20240728213739259.png)
	故$[\sigma(v_1),\cdots,\sigma(v_n)]=[v_1,\cdots,v_m]A$，其中。矩阵$A$称为线性变换$\sigma$在$U$的基$[v_1,\cdots,v_n]$下的表示矩阵。

* 重要定理

	设$n$维线性空间$V$上线性变换$\sigma$在基$v_1,\cdots,v_n$和$v_1&#39;,\cdots,v_n&#39;$下的矩阵分别为$A$和$B$，由基$v_1,\cdots,v_n$到基$v_1&#39;,\cdots,v_n&#39;$的过渡矩阵为$P$，则$B=P^{-1}AP$

	推导：

	因为
	![image-20240728213814264](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/image-20240728213814264.png)
	则代入得到
	![image-20240728213934744](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/image-20240728213934744.png)

	所以$AP=PB$，左乘$P^{-1}$，得$B=P^{-1}AP$。

* 相似

	设$A,B\in F^{m\times n}$，如果存在可逆矩阵$P\in F^{n\times n}$使得$B=P^{-1}AB$，则称$A$与$B$相似。

## 酉变换和正交变换

* 定义

	设$V$是$n$维酉（欧式）空间（一个在复数（实数）域上的内积空间），$\sigma:V\rightarrow V$是线性变换，如果
	$$
	\forall x\in V,||\sigma(x)||=||x||
	$$
	$\sigma$就称为酉（正交）变换

* 定理

	1. 设$V$是$n$维酉（欧式）空间（一个在复数（实数）域上的内积空间），如果$\sigma:V\rightarrow V$是酉（正交）变换，则
		$$
		\forall x,y\in V,(\sigma(x),\sigma(y))&gt;=(x,y)
		$$

2. &lt;font color=&#34;red&#34;&gt;即酉（正交变换）保持向量的内积。&lt;/font&gt;

3. 如果$v_1,\cdots,v_n$是$V$的一组标准正交基，则$\sigma(v_1),\cdots,\sigma(v_n)$也是$V$的一组标准正交基。

4. $\sigma$在$V$的任意一组标准正交基下的矩阵是酉（正交）矩阵。

5. 设$v=[v_1,\cdots,v_n]$是酉（欧式）空间$V$的一组标准正交基，$A$维$\sigma: V\rightarrow V$在基$v$的表示矩阵为$A$，则$\sigma$是一个酉（正交）变换当且仅当$A^HA=I(A^T=I)$。

	即，$A$的列向量组成了$C^{n}(R^n)$的标准正交基。

## 同态和同构

* 定义

	设$V$和$W$是在相同数域$F$上的两个向量空间，$\sigma:V\rightarrow W$是线性变换（也称为同态）。如果$\sigma$是一一对应的，则称为同构。

	如果存在从$V$到$W$的同构，则称$V$与$W$同构。

	&lt;font color=&#34;red&#34;&gt;对于同构$\sigma:  V\rightarrow W,ker(\sigma)=\{0\} \space and \space \sigma(V)=W$。&lt;/font&gt;

* 定理

	1. 设$V$和$W$是在相同数域$F$上的两个向量空间，$\sigma$是从$V$到$W$的同构，$S$为$V$的子空间，则$\dim(S)=\dim(\sigma(S))$。&lt;font color=&#34;red&#34;&gt;即，两个同构空间有相同的维数（充要条件）。&lt;/font&gt;
	2. 设$\sigma$是从$V$到$W$的同构，则$\sigma^{-1}$是从$W$到$V$的同构
	3. 数域 $F$ 上任意一个 $n$维 向量空间$V$同构于向量空间 $F^n$。

* 性质

	同构具有如下性质：

	1. 自反性
	2. 对称性
	3. 传递性

## 不变子空间

* 定义

	设$\sigma:V\rightarrow V$是线性变换，如果$V$的子空间$S$满足$\forall x\in S, \sigma(x)\in S$，即$\sigma(x)\subset S$，则称$S$是一个不变子空间。

	&lt;font color=&#34;red&#34;&gt;当说到不变子空间时，要指明是在什么映射下是不变的。&lt;/font&gt;利用$\sigma$-不变子空间，我们可以简化$\sigma$的表示矩阵。

* 矩阵的不变子空间

	设$A\in F^{n\times n}$，$\sigma_A:F^n\rightarrow F^n$被定义为：$\sigma_A(x)=Ax$，$F^n$的子空间$S$如果满足$\forall x\in S, Ax\in S$，则称$S$是$\sigma $-不变子空间。

* 定理

	1. 设$\sigma:V\rightarrow V$是线性变换，则两个$\sigma$-不变子空间的交、和、直和也是$\sigma$-不变子空间。
	2. 设$\sigma$是在向量空间$V$上的线性变换，$W=span\{x_1,\cdots,x_k\}$是$V$的$\sigma$-不变子空间当且仅当$\sigma(x_i)\in W(i=1,2,\cdots,k)$。
	3. 设$\sigma$是数域$F$上$n$维向量空间$V$上的线性变换，则$\sigma$可以对角化的充要条件是$V$可以分解成$\sigma$的&lt;font color=&#34;red&#34;&gt;一维&lt;/font&gt;不变子空间的直和。
	4. 设$\sigma$是数域$F$上$n$维向量空间$V$上的线性变换，则$\sigma$在$V$的一组基下的矩阵为形如![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex-20240728214118238.png)
   的块上三角矩阵的充要条件是$\sigma$的非平凡的不变子空间。
	5. 设$\sigma$是数域$F$上$n$维向量空间$V$上的线性变换，则$\sigma$在$V$的一组基下的矩阵为块对角巨好着呢的充要条件是$V$可以分解成$\sigma$的&lt;font color=&#34;red&#34;&gt;若干个非平凡不变子空间&lt;/font&gt;的直和。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/03.%E7%BA%BF%E6%80%A7%E6%98%A0%E5%B0%84%E5%92%8C%E7%BA%BF%E6%80%A7%E5%8F%98%E6%8D%A2/  

