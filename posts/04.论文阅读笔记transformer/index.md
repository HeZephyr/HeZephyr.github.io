# 【论文阅读笔记】Attention Is All You Need

## 引言

循环神经网络(RNN)，特别是长短期记忆和门控循环神经网络（**编码器-解码器**架构），已成为序列建模和转换问题（如语言建模和机器翻译）的先进方法，众多研究在不断推动其发展。

![RNN做机器翻译](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/RNN)

但RNN通常&lt;font color=&#34;red&#34;&gt;沿输入和输出序列的符号位置&lt;/font&gt;进行计算，其固有的顺序性导致训练示例内难以并行化，在序列长度较长时，由于内存限制跨示例的批处理，这一问题更加突出。尽管近期通过一些技术改进了计算效率，&lt;font color=&#34;red&#34;&gt;但顺序计算的基本限制仍未改变&lt;/font&gt;。

且RNN使用共享权值矩阵，在面临较长序列时，会有梯度消失的问题(也可以说是后面词的梯度会覆盖前面的梯度)。即使后序的LSTM和GRU对这一部分做了改进，但也无法完全解决该问题。

&lt;font color=&#34;red&#34;&gt;注意力机制&lt;/font&gt;已成为各种序列建模和转换模型的重要组成部分，能在不考虑输入或输出序列距离的情况下对依赖关系进行建模，但在大多数情况下与循环网络结合使用。

作者提出了 Transformer 模型，该模型摒弃了循环单元和卷积单元，完全依赖注意力机制来建立输入和输出之间的全局依赖关系，允许更多的并行化。

## 背景

&lt;font color=&#34;red&#34;&gt;减少序列计算量和加速计算&lt;/font&gt;是序列处理模型中的基本思想。ByteNet和ConvS2S通过使用卷积神经网络并行计算，计算量和序列中位置的距离相关。ConvS2S是线性关系，而ByteNet是对数关系，使得长距离关系学习困难。Transformer将这个过程减少到常数规模，尽管降低了有效分辨率，但**多头注意力机制**弥补了这一点。

自注意力机制（内部注意力机制）为&lt;font color=&#34;red&#34;&gt;序列的不同位置分配权重，并学习表示向量&lt;/font&gt;，已在阅读理解、文本摘要等任务中表现出色。

端到端记忆网络通常基于循环注意力机制，已用于简单语言翻译等任务。而Transformer 是&lt;font color=&#34;red&#34;&gt;第一个完全依赖自注意力&lt;/font&gt;来计算其输入和输出表示的转换模型。

## 模型架构

 Transformer中沿用了非常经典的编码器-解码器架构，编码器将输入的序列$(x_1,\dots,x_n)$转化成一个表示向量$\boldsymbol{z}=(z_1,\dots,z_n)$，而编码器依据向量$\boldsymbol{z}$逐步生成输出序列$(y_1,\dots,y_m)$，并且模型在每个步骤中都是**自回归**的，会将先前生成的符号作为生成下一个的额外输入，例如这一步要生成$y_t$，要将$(y_1,\dots,y_{t-1})$都拿到也作为输入。

同时Transformer模型在编码器和解码器中都使用堆叠自注意力机制和逐点全连接层，如下图的左半部分和右半部分所示。

&lt;img src=&#34;https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Transformer_Architecture&#34; alt=&#34;image-20240707194838925&#34; style=&#34;zoom: 33%;&#34; /&gt;

### 编码器堆叠和解码器堆叠

编码器由 $6$ 个相同的层堆叠而成。每个层包含两个子层：&lt;font color=&#34;red&#34;&gt;多头自注意力机制和逐位置全连接前馈神经网络&lt;/font&gt;。每个子层都使用了残差连接，然后进行层规范化（**防止模型过拟合**）。假设每一层的输入是$x$，那么每一层的输出结果可以表示为：
$$
\text{LayerNorm}(x&#43;\text{Sublayer(x)})
$$
其中

* $\text{SubLayer}$是当前子层本身实现的运算函数，比如注意力运算和全连接运算；
* 模型中的所有子层以及嵌入层的输出维度均为 $d_{\text{model}} = 512$（便于残差连接）。

解码器同样由 6 个层组成。其结构与编码器类似，但多了一个**对编码器输出进行关注的多头注意力子层**。并且在**自注意力子层中进行了修改**，以防止信息左向流动。这种掩码机制，结合输出嵌入向量向后偏移一个位置，确保位置 $i$ 的预测仅依赖于位置小于 $i$ 的已知输出。

### 注意力机制

注意力机制就是对一系列的query和一系列的key-value对，我们需要确定对于每个query而言不同 value 的重要程度，而这个权重是根据 query 和key 的相关度计算得到的。

#### 缩放的点积注意力机制

Transformer模型中使用的是缩放的点积注意力机制。在这种机制中，注意力计算涉及query和key的维度为  $d_k$ \)，以及value的维度为 $d_v$ 。通过引入一个**缩放因子**$\sqrt{d_k}$，可以控制注意力分布的稳定性和计算效率。

&lt;img src=&#34;https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Scaled_Dot_Product_Attention&#34; alt=&#34;image-20240707205349089&#34; style=&#34;zoom:33%;&#34; /&gt;

我们用向量化的方式可以将这种注意力机制的计算过程表示成：
$$
\text{Attention}(Q,K,V)=\text{softmax}(\frac{QK^T}{\sqrt{d_k}})V
$$
与传统的注意力机制相比，缩放的点积注意力计算速度更快。缩放参数用于调节注意力计算的规模，以确保对于不同大小的输入，注意力权重的计算结果都能保持在合理的范围内。特别是在$\text{softmax}$函数的应用中，由于指数函数的快速增长特性，&lt;font color=&#34;red&#34;&gt;缩放可以有效防止某些权重过大&lt;/font&gt;，而其他权重接近零的情况，确保了计算结果的平稳性和有效性。

#### 多头注意力机制

#### 

Transformer采用了多头注意力机制，将query、key和value进行$h$次投影，然后对$h$个投影并行计算注意力，再将这些结果组合并线性投影生成最终的多头注意力输出。多头注意力使模型能够共同关注不同表示子空间和不同位置的信息。

&lt;img src=&#34;https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Multi_Head_Attention&#34; alt=&#34;image-20240707214205509&#34; style=&#34;zoom:50%;&#34; /&gt;

多头注意力机制的计算公式为：
$$
\operatorname{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O
$$
其中每个头的计算方式为：

$$
\text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
$$
参数的维度如下：

- $( Q, K, V)$ 的输入维度为 $( d_{model} )$（通常为$512$）
- $W_i^Q \in \mathbb{R}^{d_{model} \times d_k},  W_i^K \in \mathbb{R}^{d_{model} \times d_k}, W_i^V \in \mathbb{R}^{d_{model} \times d_v}, W^O \in \mathbb{R}^{hd_v \times d_{model}}$

作者在Transformer中设置 $h = 8$  个头，每个头的维度为 $d_k = d_v = \frac{d_{model}}{h} = 64$。

#### Transformer中注意力机制的应用

* 在解码器的“编码器-解码器注意力层”中，query 来自上一个解码器层，而 key 和值 value 来自编码器的输出。这使得解码器中的每个位置都可以参与到输入序列所有位置的注意力计算中。

1. **编码器-解码器注意力层**： 在这一层中，query 来自上一个解码器，而记忆 key 和 value 来自编码器的输出。这使得解码器中的每个位置都可以关注输入序列中的所有位置。
2. **编码器中的自注意力层**： 编码器包含自注意力层。在这种自注意力机制中，&lt;font color=&#34;red&#34;&gt;query, key 和 value 都来自同一位置，即编码器前一层的输出&lt;/font&gt;。这样，编码器中的每个位置都可以关注编码器前一层中的所有位置，从而捕捉输入序列中不同位置之间的全局依赖关系。
3. **解码器中的自注意力层**： 解码器中的自注意力层的key，value和query也是同源的。但为了保持自回归属性，防止序列中前面的内容被后面的内容所影响，解码器在自注意力计算中加入了掩码机制。具体来说，通过在缩放的点积注意力中，将$\operatorname{softmax}$输入中对应非法连接的值设置为$-\infin$，来屏蔽这些连接。

### 逐位置全连接前馈神经网络

### Transformer中的逐位置全连接前馈神经网络

在Transformer中，逐位置全连接前馈神经网络用于增强模型对序列中每个位置信息的处理能力，这种网络结构包含两个线性变换层和$\operatorname{ReLU}$激活函数，用于每个位置独立地进行相同的操作。其数学表示如下：

$$
\text{FFN}(x) = \max(0, x W_1 &#43; b_1) W_2 &#43; b_2
$$


虽然不同位置的线性变换相同，但各层使用的参数不同。其中，$x$ 是输入向量，维度为 $d_{\text{model}} = 512$ ，而内层的维度为$d_{ff}=2048$，$W_1$ 和 $W_2$ 是两个线性变换的权重矩阵，$b_1$ 和 $b_2$ 是相应的偏置向量。这个结构类似于`kernel size`为1的卷积操作。

### 嵌入和Softmax

和其他序列转换模型类似，Transformer 使用学习得到的嵌入将输入的token和输出的token转换为维度为 $ d_{\text{model}} $ 的向量。Transformer 还使用学习得到的线性变换和$\text{softmax}$函数将解码器的输出转换为预测的下一个token的概率。在Transformer中两个嵌入层和$\text{softmax}$之前的线性变换层之间共享相同的权重矩阵（**减少模型的参数数量，降低过拟合的风险**），同时在嵌入层中，我们将这些权重乘以  $\sqrt{d_{\text{model}}}$（**缩放嵌入权重，防止梯度消失**）。

### 位置编码

在Transformer模型中，由于没有循环和卷积单元，为了处理序列数据的位置信息，引入了位置编码。位置编码是为序列中的每个位置添加特定的向量表示，以便模型能够区分不同位置的token。在Transformer中的位置编码使用了sin函数和cos函数，这种方法不同于传统的学习得到的位置嵌入，而是采用固定的函数形式，即

![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/test.image.latex-20240729215814533.png)


这里的$\text{pos}$表示位置而$i$表示维度，也就是对于位于$\text{pos}$位置的token的嵌入向量第$i$维加上这样一个值。

## 为什么使用自注意力机制

在处理序列数据时，长距离依赖关系的学习是一个关键挑战。论文中对比了使用自注意力、循环单元和卷积单元等不同模型结构时的计算量、时间复杂度和最大依赖路径长度。其中，最大依赖路径长度指的是任意两个输入输出位置之间信息传递的最长路径。

![image-20240707224337637](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Maximum_Path_Lengths)

在传统的循环神经网络（RNN）中，信息是逐步传递的，因此全局视角下，任意两个位置之间的最长信息传递路径往往是以序列长度 $n$ 为量级的。这种逐步传递导致了RNN在捕捉长距离依赖时可能面临的挑战，尤其是在处理长序列时效果不佳。

相比之下，&lt;font color=&#34;red&#34;&gt;Transformer利用自注意力机制直接将每个位置与所有其他位置进行关联，避免了逐步传递的过程&lt;/font&gt;，使得任意两个位置之间的信息传递路径变得极为直接和高效。这种直接的路径传递方式使得Transformer能够更有效地捕捉到长距离的依赖关系，而不受序列长度的限制。

因此，Transformer凭借其独特的注意力机制，实现了“Attention is all you need”的理念，强调了在序列建模中注意力机制的重要性和效果。它不仅仅是一种模型结构的创新，更是在解决长距离依赖问题上的一次重大突破。Transformer的成功不仅在于其高效的信息传递路径，还在于其能够在更大范围内捕捉和利用序列中的关联信息，从而提升了序列建模任务的性能和效果。

## 训练

在论文中，训练Transformer模型涉及到几个关键的优化和正则化策略。

### 优化器

模型的训练使用了Adam优化器，并采用了一种自适应的学习率。学习率 $\text{lr}$ 的计算方式如下所示：
![img](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/lr_learn_compute)

这种自适应学习率的设计有助于在训练初期快速提升学习率，以加速模型收敛，而在训练后期逐渐降低学习率，以更细致地调整模型参数，提升训练的稳定性和效果。

### 正则化

Transformer模型采用了多种正则化方法，以提升泛化能力和训练稳定性：

1. **残差连接**：在每个子层之间都使用了残差连接，这种连接方式有助于减少梯度消失问题，并简化了模型的训练和优化过程。

2. **Dropout**：在输入嵌入向量和位置编码相加后的层中使用了Dropout，选择的Dropout概率为0.1。Dropout通过随机地将部分神经元的输出置为零，有助于防止模型过拟合，并增强泛化能力。

3. **标签平滑处理**：这是一种用于改善模型训练和提高评价指标（如BLEU分数）的技术。标签平滑处理通过将真实标签替换为一个分布更平滑的目标分布，从而减少模型对训练数据中特定标签的过度自信，提升泛化能力和性能评估的一致性。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/04.%E8%AE%BA%E6%96%87%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0transformer/  

