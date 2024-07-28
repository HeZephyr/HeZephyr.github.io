# 解决VSCode中不能使用万能头文件的问题

由于博主最近由CB转到Vscode了，可是发现我最爱用的万能头文件`&lt;bits/stdc&#43;&#43;.h&gt;`使用不了。于是我找了各种办法，终于解决了。为了帮助到同样遇到这样问题的你们，所以在这里列出详细解决方法。

 首先，我们要知道问题根源所在，为什么引入`iostream`可以，而引入`bits/stdc&#43;&#43;.h`不行，我们点击鼠标右键对这两个头文件转到定义。
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6ZjA3MDE%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70.png)
发现尝试万能头文件的时候显示未定义，而尝试`isotream`的时候跳转到：
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6ZjA3MDE%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70-20240709223937850.png)
我们发现，这即是`iostream`头文件的定义，这里给出了它的路径。我们看看它在什么文件下。
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6ZjA3MDE%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70-20240709223937882.png)
右键选择在文件资源管理器中显示。我们看到如下：
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6ZjA3MDE%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70-20240709223937983.png)
这些都是好多头文件的定义，我们vscode引入头文件都是从这里寻找引入的。
那么我们试想，如果我们把`bits/stdc&#43;&#43;.h`头文件的定义给出，是不是就可以引入了？在官网，有`bits/stdc&#43;&#43;.h`头文件的内容，这里贴出如下：

```cpp
// C&#43;&#43; includes used for precompiling -*- C&#43;&#43; -*-
// Copyright (C) 2003-2014 Free Software Foundation, Inc. This file is part of the GNU ISO C&#43;&#43; Library.  This library is free// software; you can redistribute it and/or modify it under the// terms of the GNU General Public License as published by the// Free Software Foundation; either version 3, or (at your option)// any later version.
// This library is distributed in the hope that it will be useful,// but WITHOUT ANY WARRANTY; without even the implied warranty of// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the// GNU General Public License for more details.
// Under Section 7 of GPL version 3, you are granted additional// permissions described in the GCC Runtime Library Exception, version// 3.1, as published by the Free Software Foundation.
// You should have received a copy of the GNU General Public License and// a copy of the GCC Runtime Library Exception along with this program;// see the files COPYING3 and COPYING.RUNTIME respectively.  If not, see// &lt;http://www.gnu.org/licenses/&gt;.
/** @file stdc&#43;&#43;.h *  This is an implementation file for a precompiled header. */
// 17.4.1.2 Headers
// C
#ifndef _GLIBCXX_NO_ASSERT
#include &lt;cassert&gt;
#endif
#include &lt;cctype&gt;
#include &lt;cerrno&gt;
#include &lt;cfloat&gt;
#include &lt;ciso646&gt;
#include &lt;climits&gt;
#include &lt;clocale&gt;
#include &lt;cmath&gt;
#include &lt;csetjmp&gt;
#include &lt;csignal&gt;
#include &lt;cstdarg&gt;
#include &lt;cstddef&gt;
#include &lt;cstdio&gt;
#include &lt;cstdlib&gt;
#include &lt;cstring&gt;
#include &lt;ctime&gt;

#if __cplusplus &gt;= 201103L
#include &lt;ccomplex&gt;
#include &lt;cfenv&gt;
#include &lt;cinttypes&gt;
#include &lt;cstdalign&gt;
#include &lt;cstdbool&gt;
#include &lt;cstdint&gt;
#include &lt;ctgmath&gt;
#include &lt;cwchar&gt;
#include &lt;cwctype&gt;
#endif

// C&#43;&#43;
#include &lt;algorithm&gt;
#include &lt;bitset&gt;
#include &lt;complex&gt;
#include &lt;deque&gt;
#include &lt;exception&gt;
#include &lt;fstream&gt;
#include &lt;functional&gt;
#include &lt;iomanip&gt;
#include &lt;ios&gt;
#include &lt;iosfwd&gt;
#include &lt;iostream&gt;
#include &lt;istream&gt;
#include &lt;iterator&gt;
#include &lt;limits&gt;
#include &lt;list&gt;
#include &lt;locale&gt;
#include &lt;map&gt;
#include &lt;memory&gt;
#include &lt;new&gt;
#include &lt;numeric&gt;
#include &lt;ostream&gt;
#include &lt;queue&gt;
#include &lt;set&gt;
#include &lt;sstream&gt;
#include &lt;stack&gt;
#include &lt;stdexcept&gt;
#include &lt;streambuf&gt;
#include &lt;string&gt;
#include &lt;typeinfo&gt;
#include &lt;utility&gt;
#include &lt;valarray&gt;
#include &lt;vector&gt;

#if __cplusplus &gt;= 201103L
#include &lt;array&gt;
#include &lt;atomic&gt;
#include &lt;chrono&gt;
#include &lt;condition_variable&gt;
#include &lt;forward_list&gt;
#include &lt;future&gt;
#include &lt;initializer_list&gt;
#include &lt;mutex&gt;
#include &lt;random&gt;
#include &lt;ratio&gt;
#include &lt;regex&gt;
#include &lt;scoped_allocator&gt;
#include &lt;system_error&gt;
#include &lt;thread&gt;
#include &lt;tuple&gt;
#include &lt;typeindex&gt;
#include &lt;type_traits&gt;
#include &lt;unordered_map&gt;
#include &lt;unordered_set&gt;
#endif


```

这里还有一个小细节，我们知道/(斜杆)其实代表目录的，也就是说`stdc&#43;&#43;.h`才是头文件名，它在`bits`这个文件夹下的，所以我们要做的就是新建一个名为`bits`的文件夹。然后在vscode中新建一个文件：`stdc&#43;&#43;.h`，将我们上面万能头文件的定义复制到该文件。保存在`bits`文件目录下即可。

我们测试helloworld程序，发现可以使用。问题解决。
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6ZjA3MDE%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70-20240709223938020.png)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.%E8%A7%A3%E5%86%B3vscode%E4%B8%AD%E4%B8%8D%E8%83%BD%E4%BD%BF%E7%94%A8%E4%B8%87%E8%83%BD%E5%A4%B4%E6%96%87%E4%BB%B6%E7%9A%84%E9%97%AE%E9%A2%98/  

