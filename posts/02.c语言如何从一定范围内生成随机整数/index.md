# 如何使用C语言从一定范围内生成随机整数？

[出处](https://stackoverflow.com/questions/2509679/how-to-generate-a-random-integer-number-from-within-a-range)

如何使用C语言生成随机数？例如希望生成一段特定范围的随机数，如$[1,6]$来模拟掷骰子。

&gt; 到目前为止，所有答案在数学上都是错误的。返回 `rand() % N` 并不统一给出 `[0, N)` 范围内的数字，除非 `N` 除以 `rand()` 返回的区间长度（即是 2 的幂）。此外，我们不知道 `rand()` 的模数是否独立：它们有可能是 `0, 1, 2, ...` ，这是均匀的但不是很随机。唯一似乎合理的假设是 `rand()` 给出泊松分布：任何两个相同大小的非重叠子区间均可能且独立。对于一组有限的值，这意味着均匀分布，并且还确保 `rand()` 的值很好地分散。
&gt;
&gt; 这意味着更改 `rand()` 范围的唯一正确方法是将其分为多个框；例如，如果 `RAND_MAX == 11` 并且您想要 `1..6` 的范围，则应将 `{0,1}` 分配给 1，将 `{2,3}` 分配给 2，依此类推在。这些是不相交的、大小相等的区间，因此是均匀且独立分布的。
&gt;
&gt; 使用浮点除法的建议在数学上是合理的，但原则上存在舍入问题。也许 `double` 的精度足以使其工作；也许不是。我不知道，也不想弄清楚；无论如何，答案取决于系统。
&gt;
&gt; 正确的方法是使用整数运算。也就是说，您想要如下所示的内容：
&gt;
&gt; ```c
&gt; #include &lt;stdlib.h&gt; // For random(), RAND_MAX
&gt; 
&gt; // Assumes 0 &lt;= max &lt;= RAND_MAX
&gt; // Returns in the closed interval [0, max]
&gt; long random_at_most(long max) {
&gt;   unsigned long
&gt;     // max &lt;= RAND_MAX &lt; ULONG_MAX, so this is okay.
&gt;     num_bins = (unsigned long) max &#43; 1,
&gt;     num_rand = (unsigned long) RAND_MAX &#43; 1,
&gt;     bin_size = num_rand / num_bins,
&gt;     defect   = num_rand % num_bins;
&gt; 
&gt;   long x;
&gt;   do {
&gt;    x = random();
&gt;   }
&gt;   // This is carefully written not to overflow
&gt;   while (num_rand - defect &lt;= (unsigned long)x);
&gt; 
&gt;   // Truncated division is intentional
&gt;   return x/bin_size;
&gt; }
&gt; ```
&gt;
&gt; 为了获得完全均匀的分布，循环是必要的。例如，如果给你 0 到 2 之间的随机数，而你只想要 0 到 1 之间的数字，你就继续拉，直到没有 2 为止；不难检查这是否以相同的概率给出 0 或 1。 这里使用 `random()` 而不是 `rand()` 因为它具有更好的分布（如 `rand()` 的手册页所述）。
&gt;
&gt; 如果你想获得默认范围 `[0, RAND_MAX]` 之外的随机值，那么你必须做一些棘手的事情。也许最方便的方法是定义一个函数 `random_extended()` 提取 `n` 位（使用 `random_at_most()` ）并返回 `[0, 2**n)` ，然后应用 `random_at_most()` 用 `random_extended()` 代替 `random()` （并用 `2**n - 1` 代替 `RAND_MAX` ）来提取随机值小于 `2**n` ，假设您有一个可以保存这样的值的数字类型。最后，当然，您可以使用 `min &#43; random_at_most(max - min)` 获取 `[min, max]` 中的值，包括负值。


这段代码通过以下方式确保生成的随机数在闭区间 [0, max] 内均匀分布：

1. **确定随机数的范围和数量**：
	- `num_bins` 表示生成的随机数的数量，即 `max` 的值加一，因为随机数生成器的范围是左闭右开的。
	- `num_rand` 表示随机数生成器的最大值加一，表示总的可能的随机数的数量。
2. **计算每个区间的大小**：
	- `bin_size` 表示每个区间的大小，即将随机数的总数量除以可能的随机数的数量，以确保每个随机数的概率相等。
3. **处理不能均匀分配的情况**：
	- `defect` 表示随机数总数量除以可能的随机数数量的余数，用于处理不能均匀分配的情况。
4. **生成随机数并检查范围**：
	- 使用 `do-while` 循环来生成随机数，并检查生成的随机数是否在指定的范围内，如果不在范围内则重新生成，直到生成的随机数在指定范围内。
5. **返回均匀分布的随机数**：
	- 将生成的随机数除以区间大小得到的结果，即在闭区间 [0, max] 内均匀分布的随机数。

通过以上步骤，代码确保了生成的随机数在指定范围内且分布均匀。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://lruihao.cn/posts/02.c%E8%AF%AD%E8%A8%80%E5%A6%82%E4%BD%95%E4%BB%8E%E4%B8%80%E5%AE%9A%E8%8C%83%E5%9B%B4%E5%86%85%E7%94%9F%E6%88%90%E9%9A%8F%E6%9C%BA%E6%95%B4%E6%95%B0/  

