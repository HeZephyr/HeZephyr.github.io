# 内存API

[github地址](https://github.com/unique-pure/ostep/tree/main/Virtualization/09.Memory%20API)

1. 首先，编写一个名为 null.c 的简单程序，该程序创建一个指向整数的指针，将其设置为 NULL，然后尝试取消引用它。将其编译为名为 null 的可执行文件。当你运行这个程序时会发生什么？

	```c
	#include &lt;stdio.h&gt;
	
	int main() {
	    int *p = NULL;
	    printf(&#34;Value of p: %d\n&#34;, *p);
	    return 0;
	}
	```

	```shell
	❯ gcc null.c -o null
	❯ ./null
	[1]    67350 segmentation fault  ./null
	```

2. 接下来，在编译程序时加入符号信息（使用 -g 标志）。这样做可以在可执行文件中加入更多信息，使调试器可以访问更多有用的变量名等信息。在调试器下运行程序，键入 `gdb null`，然后在 gdb 运行后键入 run。gdb 会显示什么？

	```shell
	&gt; gcc null.c -g -o null
	&gt; gdb null
	GNU gdb (Ubuntu 12.1-0ubuntu1~22.04) 12.1
	Copyright (C) 2022 Free Software Foundation, Inc.
	License GPLv3&#43;: GNU GPL version 3 or later &lt;http://gnu.org/licenses/gpl.html&gt;
	This is free software: you are free to change and redistribute it.
	There is NO WARRANTY, to the extent permitted by law.
	Type &#34;show copying&#34; and &#34;show warranty&#34; for details.
	This GDB was configured as &#34;x86_64-linux-gnu&#34;.
	Type &#34;show configuration&#34; for configuration details.
	For bug reporting instructions, please see:
	&lt;https://www.gnu.org/software/gdb/bugs/&gt;.
	Find the GDB manual and other documentation resources online at:
	    &lt;http://www.gnu.org/software/gdb/documentation/&gt;.
	
	For help, type &#34;help&#34;.
	Type &#34;apropos word&#34; to search for commands related to &#34;word&#34;...
	Reading symbols from null...
	(gdb) run
	Starting program: /home/zfhe/null
	[Thread debugging using libthread_db enabled]
	Using host libthread_db library &#34;/lib/x86_64-linux-gnu/libthread_db.so.1&#34;.
	
	Program received signal SIGSEGV, Segmentation fault.
	0x0000555555555161 in main () at null.c:5
	5	    printf(&#34;Value of p: %d\n&#34;, *p);
	```

3. 最后，在这个程序上使用 `valgrind` 工具。我们将使用 `valgrind` 中的 `memcheck` 工具来分析发生的情况。运行时输入以下内容：`valgrind --leak-check=yes ./null`。运行时会发生什么？你能解释该工具的输出吗？

	```shell
	&gt; valgrind --leak-check=yes ./null
	==1316115== Memcheck, a memory error detector
	==1316115== Copyright (C) 2002-2017, and GNU GPL&#39;d, by Julian Seward et al.
	==1316115== Using Valgrind-3.18.1 and LibVEX; rerun with -h for copyright info
	==1316115== Command: ./null
	==1316115==
	==1316115== Invalid read of size 4
	==1316115==    at 0x109161: main (null.c:5)
	==1316115==  Address 0x0 is not stack&#39;d, malloc&#39;d or (recently) free&#39;d
	==1316115==
	==1316115==
	==1316115== Process terminating with default action of signal 11 (SIGSEGV)
	==1316115==  Access not within mapped region at address 0x0
	==1316115==    at 0x109161: main (null.c:5)
	==1316115==  If you believe this happened as a result of a stack
	==1316115==  overflow in your program&#39;s main thread (unlikely but
	==1316115==  possible), you can try to increase the size of the
	==1316115==  main thread stack using the --main-stacksize= flag.
	==1316115==  The main thread stack size used in this run was 8388608.
	==1316115==
	==1316115== HEAP SUMMARY:
	==1316115==     in use at exit: 0 bytes in 0 blocks
	==1316115==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
	==1316115==
	==1316115== All heap blocks were freed -- no leaks are possible
	==1316115==
	==1316115== For lists of detected and suppressed errors, rerun with: -s
	==1316115== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
	[1]    1316115 segmentation fault (core dumped)  valgrind --leak-check=yes ./null
	```

	输出的解释：

	1. `Invalid read of size 4`：程序尝试读取一个无效的内存地址，该地址的大小为 4 字节。这表明在程序执行过程中发生了一次无效的内存读取操作。
	2. `Address 0x0 is not stack&#39;d, malloc&#39;d or (recently) free&#39;d`：Valgrind 报告了试图访问的内存地址是 0x0，即空指针。程序尝试读取了一个空指针，这是非法操作，因为空指针通常不指向任何有效的内存位置。
	3. `Process terminating with default action of signal 11 (SIGSEGV)`：由于程序尝试访问无效的内存地址导致了段错误，程序被终止，并且默认行为是发送信号 11 (SIGSEGV)。
	4. `Access not within mapped region at address 0x0`：Valgrind 报告了访问的内存地址不在映射区域内，即程序试图访问未分配或未初始化的内存区域。
	5. `HEAP SUMMARY` 和 `ERROR SUMMARY`：Valgrind 提供了堆内存使用情况的总结和错误摘要。在这个例子中，堆内存中没有分配任何内存块，也没有发生内存泄漏。
	6. `All heap blocks were freed -- no leaks are possible`：Valgrind 告诉我们在程序结束时所有的堆内存都已被释放，因此不存在内存泄漏的可能性。

4. 编写一个简单的程序，使用 `malloc()` 分配内存，但在退出前忘记释放内存。程序运行时会发生什么？你能用 gdb 查找出任何问题吗？使用 `valgrind`（同样使用 --leak-check=yes 标志）如何？

	```c
	#include &lt;stdio.h&gt;
	#include &lt;stdlib.h&gt;
	
	void malloc_but_nofree() {
	    int *p = (int *)malloc(sizeof(int));
	}
	int main() {
	    malloc_but_nofree();
	    return 0;
	}
	```

	程序正常运行正常结束，`gdb`也无法发现内存泄漏，但`valgrind`可以。

5. 编写一个程序，使用 `malloc` 创建一个大小为 100 的名为 data 的整数数组，然后将 data[100] 设置为零。运行这个程序时会发生什么？使用 valgrind 运行此程序时会发生什么？程序正确吗？

	```c
	#include &lt;stdio.h&gt;
	#include &lt;stdlib.h&gt;
	
	int main() {
	    // create size of 100 array
	    int *p = (int *)malloc(100);
	    // set p[100] = 0
	    p[100] = 0;
	    return 0;
	}
	```

	程序正常运行。

	```shell
	==1316673== Memcheck, a memory error detector
	==1316673== Copyright (C) 2002-2017, and GNU GPL&#39;d, by Julian Seward et al.
	==1316673== Using Valgrind-3.18.1 and LibVEX; rerun with -h for copyright info
	==1316673== Command: ./a.out
	==1316673==
	==1316673== Invalid write of size 4
	==1316673==    at 0x10916D: main (in /home/zfhe/a.out)
	==1316673==  Address 0x4a921d0 is 224 bytes inside an unallocated block of size 4,194,032 in arena &#34;client&#34;
	==1316673==
	==1316673==
	==1316673== HEAP SUMMARY:
	==1316673==     in use at exit: 100 bytes in 1 blocks
	==1316673==   total heap usage: 1 allocs, 0 frees, 100 bytes allocated
	==1316673==
	==1316673== 100 bytes in 1 blocks are definitely lost in loss record 1 of 1
	==1316673==    at 0x4848899: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
	==1316673==    by 0x10915E: main (in /home/zfhe/a.out)
	==1316673==
	==1316673== LEAK SUMMARY:
	==1316673==    definitely lost: 100 bytes in 1 blocks
	==1316673==    indirectly lost: 0 bytes in 0 blocks
	==1316673==      possibly lost: 0 bytes in 0 blocks
	==1316673==    still reachable: 0 bytes in 0 blocks
	==1316673==         suppressed: 0 bytes in 0 blocks
	==1316673==
	==1316673== For lists of detected and suppressed errors, rerun with: -s
	==1316673== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
	```

6. 创建一个程序，分配一个整数数组（如上所述），释放它们，然后尝试打印该数组的一个元素的值。程序能运行吗？当你使用 valgrind 时会发生什么？

	```c
	#include &lt;stdio.h&gt;
	#include &lt;stdlib.h&gt;
	
	int main() {
	    // create size of 100 array
	    int *p = (int *)malloc(100);
	    free(p);
	    // use p
	    p[0] = 0;
	    return 0;
	}
	```

	程序能运行。

	```shell
	&gt; valgrind --leak-check=yes ./a.out
	==1316773== Memcheck, a memory error detector
	==1316773== Copyright (C) 2002-2017, and GNU GPL&#39;d, by Julian Seward et al.
	==1316773== Using Valgrind-3.18.1 and LibVEX; rerun with -h for copyright info
	==1316773== Command: ./a.out
	==1316773==
	==1316773== Invalid write of size 4
	==1316773==    at 0x109193: main (in /home/zfhe/a.out)
	==1316773==  Address 0x4a92040 is 0 bytes inside a block of size 100 free&#39;d
	==1316773==    at 0x484B27F: free (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
	==1316773==    by 0x10918E: main (in /home/zfhe/a.out)
	==1316773==  Block was alloc&#39;d at
	==1316773==    at 0x4848899: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
	==1316773==    by 0x10917E: main (in /home/zfhe/a.out)
	==1316773==
	==1316773==
	==1316773== HEAP SUMMARY:
	==1316773==     in use at exit: 0 bytes in 0 blocks
	==1316773==   total heap usage: 1 allocs, 1 frees, 100 bytes allocated
	==1316773==
	==1316773== All heap blocks were freed -- no leaks are possible
	==1316773==
	==1316773== For lists of detected and suppressed errors, rerun with: -s
	==1316773== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
	```

7. 现在向 free 传递一个有趣的值（例如，上面分配的数组中间的指针）。会发生什么？你需要工具来发现这类问题吗？

	```c
	#include &lt;stdio.h&gt;
	#include &lt;stdlib.h&gt;
	
	int main() {
	    // create size of 100 array
	    int *p = (int *)malloc(100);
	    // free the pointer in the middle of the array
	    free(p &#43; 50);
	    return 0;
	}
	```

	```shell
	test.c: In function ‘main’:
	test.c:8:5: warning: ‘free’ called on pointer ‘p’ with nonzero offset 200 [-Wfree-nonheap-object]
	    8 |     free(p &#43; 50);
	      |     ^~~~~~~~~~~~
	test.c:6:21: note: returned from ‘malloc’
	    6 |     int *p = (int *)malloc(100);
	      |                     ^~~~~~~~~~~
	&gt; ./a.out
	free(): invalid pointer
	[1]    1316922 IOT instruction (core dumped)  ./a.out
	```

	编译时提供warning，运行时直接报错。可以不通过工具就能发现这个问题。

8. 了解内存分配的其他接口。例如，创建一个简单的类似向量的数据结构和使用 realloc() 管理向量的相关例程。使用数组存储矢量元素；当用户向矢量添加条目时，使用 realloc() 为其分配更多空间。这样的向量性能如何？与链表相比如何？使用 valgrind 查找错误。

	```c
	#include &lt;stdio.h&gt;
	#include &lt;stdlib.h&gt;
	
	typedef struct vect {
	    int *data;          // pointer to the data
	    size_t length;
	    size_t population;
	} Vector;
	
	Vector *new_vector(size_t length);  // Allocate memory for a new Vector
	void delete(Vector *v);             // Free memory allocated for a Vector
	void add(Vector *v, int elem);      // Add an element to the Vector    
	int get(Vector *v, int index);      // Get the element at the specified index from the Vector
	
	int main(int argc, char *argv[]) {
	    Vector *vector = new_vector(0);
	    if (vector == NULL) 
	        return EXIT_FAILURE;
	
	    printf(&#34;Vector has %d spaces and %d items stored\n&#34;, 
	            (int) vector-&gt;length, (int) vector-&gt;population);
	    
	    add (vector, 5);
	    printf(&#34;Vector has %d spaces and %d items stored\n&#34;, 
	            (int) vector-&gt;length, (int) vector-&gt;population);
	
	    printf(&#34;The value at index %d is %d\n&#34;, 
	            0, vector-&gt;data[0]);
	    delete(vector);
	    return 0;
	}
	
	Vector *new_vector(size_t length) {
	    Vector *v = (Vector *) malloc(sizeof(Vector));
	    if (v == NULL) {
	        fprintf(stderr, &#34;Failed to allocate memory\n&#34;);
	        return NULL;
	    }
	    v-&gt;length = length;
	    v-&gt;population = 0;
	    v-&gt;data = (int *) calloc(length, sizeof(int));
	    if (v-&gt;data == NULL) {
	        fprintf(stderr, &#34;Failed to allocate memory\n&#34;);
	        return NULL;
	    }
	    return v;
	}
	
	void delete(Vector *v) {
	    free(v-&gt;data);
	    free(v);
	}
	
	void add(Vector *v, int elem) {
	    if (v-&gt;population &gt;= v-&gt;length) { // need more space
	        v-&gt;data = realloc(v-&gt;data, (v-&gt;length &#43; 1) * sizeof(int));
	        v-&gt;length&#43;&#43;;
	    }
	    *(v-&gt;data &#43; v-&gt;population&#43;&#43;) = elem;
	}
	
	int get(Vector *v, int index) {
	    return v-&gt;data[index];
	}
	```

	随机访问快速，空间利用率高，但插入和删除元素较慢，且内存需要重新分配开销，而链表只需要单独为每个节点开辟内存。

	

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/09.%E5%86%85%E5%AD%98api/  

