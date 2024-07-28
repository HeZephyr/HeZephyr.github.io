# 【矩阵论】Chapter 9—广义逆矩阵知识点总结复习


&lt;font color=&#34;red&#34;&gt;Hermite标准型实际上就是行最简行&lt;/font&gt;

## 广义逆矩阵定义

* 广义逆矩阵$G$的定义：对任意$m\times n$矩阵的$A$，如果存在某个$n\times m$的矩阵$G$，满足`Penrose`方程的一部分或全部，则称$G$为$A$的广义逆矩阵

	&gt; `Penrose`方程的四个条件：
	&gt;
	&gt; 1. $AGA=A$;
	&gt; 2. $GAG=G$;
	&gt; 3. $(AG)^T=AG$;
	&gt; 4. $(GA)^T=GA$
	&gt;
	&gt; 满足第$i$个条件，则把$G$记为$A^{(i)}$，这类矩阵的全体记为$A\{i\}$，所以$A^{i}\in A\{i\}$
	&gt;
	&gt; 类似，满足第$i,j$个条件：$A^{i,j}\in A\{i,j\}$
	&gt;
	&gt; 根据以上，满足$1$个，$2$个，$3$个，$4$个`Penrose`方程的广义逆矩阵有$C_4^1&#43;C_4^2&#43;C_4^3&#43;C_4^4=4&#43;6&#43;4&#43;1=15$，但应用最多的，也就是我们所学的以下四种：
	&gt;
	&gt; 1. 减号逆或者$g$逆：$A^-=A^{(1)}$
	&gt; 2. 最小二乘广义逆：$A_l^-=A^{(1,3)}$
	&gt; 3. 极小范数广义逆：$A_m^-=A^{(1,4)}$
	&gt; 4. 加号逆或`Moore-Penrose`广义逆：$A^&#43;=A^{(1,2,3,4)}$

## 减号逆

* $A^-$的性质

	设$A$为$m\times n$矩阵，$P$和$Q$分别是$m$阶和$n$阶非奇异方阵，且$B=PAQ$，$A^-$为A的减号逆，则：

	1. $rank(A)\leq rank(A^-)$
	2. $AA^-$和$A^-A$是幂等矩阵，并且$rank(AA^-)=rank(A^-A)=rank(A)$
	3. $Q^{-1}A^-P^{-1}\in B\{1\}$
	4. $A^T\{1\}=\{G^T|G\in A\{1\}\}$

* （`Penrose`定理）

	设$A,B,C$分别为$m\times n,p\times q,m\times q$矩阵，则矩阵方程：
	$$
	AXB=C
	$$
	有解的充分必要条件是：
	$$
	AA^-CB^-B=C
	$$
	并且在有解的情况下，其通解为：
	$$
	X=A^-CB^-&#43;Y-A^-AYBB^-
	$$
	其中$Y\in R^{n\times p}$是任意的矩阵。

* 求解$A^-$

	$A$为$m\times n$矩阵
	![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex-20240728233037081.png)
	，其中$P,Q,E_r$通过对$A$进行如下操作得到，$G_{12},G_{21},G_{22}$均为常数矩阵，每一项均用$g_{ij}$表示常数，且$Q,P$维度均为$m\times n$，$E_r$是一个$r \times r$ 的对角矩阵，其中$r$是矩阵$A$的秩，$G_{12}$维度为$m \times (n-r)$ ，$G_{21}$维度为$(m-r) \times n$，$G_{22}$是一个$ (m-r) \times (n-r)$的矩阵。

	例子![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex-20240728233212366.png)

	初等行变换化为行最简阶梯形矩阵，则![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex-20240728233255489.png)
	
    ![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex-20240728233335693.png)
	再进行列变换化为$E_r$得到![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex-20240728233427741.png)
	
    ![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex-20240728233458270.png)
	则
    ![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex-20240728233541449.png)
	令$g_{ij}=0$，则![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex-20240728233612330.png)

* 利用$A^-$求解线性方程组$Ax=b$

	$Ax=b$有解的充分必要条件是$AA^-b=b$，这时特解$x_0=A^-b$，通解$x=A^-b&#43;(I-A^-A)y,\forall y\in C^n$

	这里不给出$A^-$，感兴趣的读者可以自己去实现，具体的算法如下：

	&gt; 1. **构造水平增广矩阵**： 将原矩阵和单位矩阵水平拼接，形成增广矩阵。
	&gt; 2. **初等行变换**： 利用初等行变换将增广矩阵转化为最简行阶梯形式。
	&gt; 3. **提取$P$**：变换后的单位矩阵就是$P$
	&gt; 4. **构造垂直增广矩阵**：再将最简行阶梯形与单位矩阵垂直拼接，形成增广矩阵。
	&gt; 5. **初等列变换，提取$G$**：变换后的单位矩阵就是$G$

## 最小二乘广义逆

* 定理1

	设$A\in  C^{m\times n}$，$G\in A\{1,3\}$的充分必要条件是$G$满足
	$$
	A^HAG=A^H
	$$


	这即为$A\{1,3\}$（最小二乘广义逆）的通式

* 定理2

	设$A\in  C^{m\times n}$，$A_l^-$是$A$的任一最小二乘广义逆，则
	$$
	A\{1,3\}=\{G\in C^{n\times m}|AG=AA_l^-\}
	$$

* 

* 定理3

	设$A$是$m\times n$矩阵，则$G\in A\{1,3\}$（即G为最小二乘广义逆）的充分必要条件为$x=Gb$是&lt;font color=&#34;red&#34;&gt;不相容线性方程组&lt;/font&gt;$Ax=b$的最小二乘解。

* 利用$A_l^-$求解线性方程组$Ax=b$

	$x$是不相容线性方程组$Ax=b$的最小二乘解当且仅当$x$是相容线性方程组
	$$
	Ax=AA_l^-b
	$$
	的解，并且$Ax=b$的最小二乘解的通式为$x=A_l^-b&#43;(I-A^-A)y,\forall y\in C^n$

## 极小范数广义逆

* 定理1

	设$A\in  C^{m\times n}$，$G\in A\{1,4\}$的充分必要条件是$G$满足
	$$
	GAA^H=A^H
	$$


	这即为$A\{1,4\}$（极小范数广义逆）的通式

* 定理2

	设$A\in  C^{m\times n}$，$A_m^-$是$A$的任一极小范数广义逆，则
	$$
	A\{1,4\}=\{G\in C^{n\times m}|GA=A_m^-A\}
	$$

* 定理$3$

	设$A$是$m\times n$矩阵，则$G\in A\{1,4\}$（即G为极小范数广义逆）的充分必要条件为$x=Gb$是&lt;font color=&#34;red&#34;&gt;相容&lt;/font&gt;线性方程组$Ax=b$的极小范数解。

* 利用$A_m^-$求解线性方程组$Ax=b$

	设$A$是$m\times n$矩阵，则$G\in A\{1,4\}$的充分必要条件为$x=Gb$是相容线性方程组$Ax=b$的极小范数解，即$x=A_m^-b$为相容线性方程组$Ax=b$的极小范数解

&lt;font color=&#34;red&#34;&gt;注意：极小范数解是唯一的，而最小二乘解不唯一&lt;/font&gt;

## Moore-Penrose（加号逆）

![image-20231127225725016](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20231127225725016.png)

* $A^&#43;$的性质

	* &lt;font color=&#34;red&#34;&gt;$A^&#43;$存在且唯一&lt;/font&gt;
	* $A^&#43;=A_m^-AA_l^-$

* 定理1

	设$A$是$m\times n$矩阵，则$G$是加号逆$A^&#43;$的充分必要条件为$x=Gb$是不相容线性方程组$Ax=b$的极小最小二乘解。

* &lt;font color=&#34;red&#34;&gt;重点&lt;/font&gt;

	因为加号逆满足四个条件，所以它也是减号逆、最小二乘广义逆、极小范数广义逆。所以：

	1. 当$b\in R(A)$时，$Ax=b$的通解为：
		$$
		x=A^&#43;b&#43;(I-A^&#43;A)y,\forall y\in R^n
		$$

	2. 当$b\in R(A)$时，$Ax=b$的极小范数解为：
		$$
		x=A^&#43;b
		$$
		&lt;font color=&#34;red&#34;&gt;极小范数解是唯一的&lt;/font&gt;

	3. 对于$\forall b$，$Ax=b$的最小二乘解为：
		$$
		x=A^&#43;b&#43;(I-A^&#43;A)y,\forall y\in R^n
		$$

	4. 对于$\forall b$，$Ax=b$的具有极小范数的最小二乘解为：
		$$
		x=A^&#43;b
		$$

* 求解$A^&#43;$

	1. 若$A$为行满秩矩阵，则：$A^&#43;=A^H(AA^H)^{-1}$
	2. 若$A$为列满秩矩阵：则：$A^&#43;=(A^HA)^{-1}A^H$
	3. 否则利用满秩分解求解：$A^&#43;=G^&#43;F^&#43;=G^H(GG^H)^{-1}(F^HF)^{-1}F^H$

	Python代码如下：

	```py
	import numpy as np
	from sympy import Matrix, Symbol
	
	def get_A_plus(A):
	    A_plus = None
	    # 判断A是行满秩还是列满秩，如果都不是则利用满秩分解求解A_plus
	    if A.rank() == A.rows:
	        print(&#34;A为行满秩矩阵&#34;)
	        A_plus = A.H * (A * A.H).inv()
	    elif A.rank() == A.cols:
	        print(&#34;A为列满秩矩阵&#34;)
	        A_plus = (A.H * A).inv() * A.H
	    else:
	        print(&#34;A为非满秩矩阵&#34;)
	        # 利用满秩分解求解A_plus，full_rank在另一篇矩阵论复习博客中
	        F, G = full_rank(A)
	        A_plus = G.H * ((G * G.H).inv()) * ((F.H * F).inv()) * F.H
	    return A_plus
	```

	

* 利用$A^&#43;$求解线性方程组$Ax=b$

	1. $Ax=b$有解（相容）的充要条件是$AA^&#43;b=b$
	2. $x=A^&#43;b&#43;(I-A^&#43;A)y,\forall y\in C^n$是相容方程组$Ax=b$的通解，或是不相容方程组$Ax=b$的全部最小二乘解
	3. $x_0=A^&#43;b$是相容方程组$Ax=b$的唯一极小范数解，或是不相容方程组$Ax=b$的唯一极小范数最小二乘解

	Python代码如下：

	```py
	import numpy as np
	from sympy import Matrix, Symbol
	
	def get_solution(A, b):
	    A_plus = get_A_plus(A)
	    # 单位矩阵
	    I = Matrix(np.eye(A_plus.rows))
	    print(&#34;I:&#34;, I)
	    # 生成符号列表
	    symbols_list = [Symbol(f&#39;y{i&#43;1}&#39;) for i in range(A_plus.rows)]
	    # 生成符号矩阵
	    symbols_matrix = Matrix(symbols_list)
	    print(&#34;symbols_matrix:&#34;, symbols_matrix)
	
	    if A * A_plus * b == b:
	        print(&#34;Ax = b有解&#34;)
	        print(&#34;通解为：&#34;)
	        print(A_plus.rows)
	        print(A_plus * b &#43; (I - A_plus * A) * symbols_matrix)
	        print(&#34;唯一极小范数解为：&#34;)
	        print(A_plus * b)
	    else:
	        print(&#34;Ax = b无解&#34;)
	        print(&#34;全部最小二乘解为：&#34;)
	        print(A_plus.rows)
	        print(A_plus * b &#43; (I - A_plus * A) * symbols_matrix)
	        print(&#34;唯一极小范数最小二乘解：&#34;)
	        print(A_plus * b)
	get_solution(A, b)
	```

	

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/09.%E5%B9%BF%E4%B9%89%E9%80%86%E7%9F%A9%E9%98%B5/  

