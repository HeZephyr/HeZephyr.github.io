# 二分查找的奇技淫巧

## 二分搜索升天词

转自：[labuladong](https://zhuanlan.zhihu.com/p/79553968)

![二分搜索升天词](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/v2-a56776ec0125789e6169e715f3a4a22d_720w.webp)

## 手写二分查找模板

二分模板一共有两个，分别适用于不同情况。
算法思路：假设目标值在闭区间$[l, r]$中， 每次将区间长度缩小一半，当$l = r$时，我们就找到了目标值。

### 版本1

当我们将区间$[l, r]$划分成$[l, mid]$和$[mid &#43; 1, r]$时，其更新操作是$r = mid$或者$l = mid &#43; 1$;，计算$mid$时不需要加$1$。

```cpp
bool check(int x) {
    ...
    ...
}
int binary_search_1(int l, int r) {
    while (l &lt; r) {
        int mid = (l &#43; r) &gt;&gt; 1;
        if (check(mid)) {
            r = mid;
        } else {
            l = mid &#43; 1;
        }
    }
    return l;
}
```

### 版本2

当我们将区间$[l, r]$划分成$[l, mid-1]$和$[mid, r]$时，其更新操作是$r = mid - 1$或者$l = mid$;，此时为了防止死循环，计算$mid$时需要加$1$。

```cpp
bool check(int x) {
    ...
    ...
}
int binary_search_2(int l, int r) {
    while (l &lt; r) {
        int mid = (l &#43; r &#43; 1) &gt;&gt; 1;
        if (check(mid)) { // check为判断函数，我们自己手写的规则
            l = mid;
        } else {
            r = mid - 1;
        }
    }
    return l;
}
```

## 利用C&#43;&#43;自带的lower_bound和upper_bound函数

### 自带函数源码

`lower_bound`：返回一个迭代器，迭代器指向[first, last)不小于val的第一个元素。**如果范围内的所有元素比较小于val，则函数返回last。**

```cpp
template &lt;class ForwardIterator, class T&gt;
ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last, const T&amp; val)
{
  ForwardIterator it;
  iterator_traits&lt;ForwardIterator&gt;::difference_type count, step;
  count = distance(first,last);
  while (count&gt;0)
  {
    it = first; step=count/2; advance (it,step);
    if (*it&lt;val) {                 // or: if (comp(*it,val)), for version (2)
      first=&#43;&#43;it;
      count-=step&#43;1;
    }
    else count=step;
  }
  return first;
}
```

`upper_bound`：返回一个迭代器，迭代器指向[first, last)大于val的第一个元素。**如果范围内的所有元素比较都不大于val，则函数返回last。**

```cpp
template &lt;class ForwardIterator, class T&gt;
ForwardIterator upper_bound (ForwardIterator first, ForwardIterator last, const T&amp; val)
{
  ForwardIterator it;
  iterator_traits&lt;ForwardIterator&gt;::difference_type count, step;
  count = std::distance(first,last);
  while (count&gt;0)
  {
    it = first; step=count/2; std::advance (it,step);
    if (!(val&lt;*it))                 // or: if (!comp(val,*it)), for version (2)
      { first=&#43;&#43;it; count-=step&#43;1;  }
    else count=step;
  }
  return first;
}
```

根据源码，我们很容易就可以使用它们，不用自己手写了，但如果需要自定义比较规则，实际上就是需要我们实现一个check函数即可。

### 进阶—自定义比较规则

在 C&#43;&#43; 中有很多情况下，我们需要自定义比较器，无非就是三种情况：

1. 对一个自定义的 `struct` 重写它的 `operator &lt;` 方法
2. 定义一个``Comparator`函数
3. 定义一个`Comparator`结构体对象

* **自定义结构体**

  如果我们自定义了一个`struct`，然后想要对其排序又不想额外写一个比较器，那么最好实现它的 `operaotr &lt;` 方法。

  ```cpp
  struct node{
      string s;
      node(string s) {
          this-&gt;s = s;
      }
      bool operator &lt; (const node &amp;a) const { // 注意这两个const，必须要加上，否则会报错，前者const是能接收非const和const的实参，后者const是表明该函数不会修改类成员变量。
          return this-&gt;s.size() &lt; a.s.size();
      }
  };
  ```

  这样，我们就可以使用了。如下：

  ```cpp
  vector&lt;node&gt; v(10, node(&#34;111&#34;));
  int pos = lower_bound(v.begin(), v.end(), node(&#34;111&#34;)) - v.begin();
  ```

* **函数比较器**

  可以通过编写一个外部的比较器函数，实现 `&lt;` 功能。

  ```cpp
  struct node{
      string s;
      node(string s) {
          this-&gt;s = s;
      }
  };
  bool cmp(const string &amp;s1, const string &amp;s2) {
      return s1.size() &lt; s2.size();
  }
  ```

  这个通常用于自定义排序，其他的会报错，应该是不支持函数比较器，读者可自行尝试。

  ```cpp
  vector&lt;string&gt; v(10, &#34;111&#34;);
  sort(v.begin(), v.end(), cmp);
  ```

* **函数对象比较器**

  所谓函数对象是指实现了 `operator ()` 的类或者结构体。可以用这样的一个对象来代替函数作为比较器。

  ```cpp
  struct cmper {
      bool operator() (const string &amp;s1, const string &amp;s2) const {
          return s1.size() &lt; s2.size();
      }
  };
  ```

  这个通常用于自定义排序，其他的会报错，应该是不支持函数比较器，读者可自行尝试。

  ```cpp
  vector&lt;string&gt; v(10, &#34;111&#34;);
  sort(v.begin(), v.end(), cmper());
  ```


---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE/  

