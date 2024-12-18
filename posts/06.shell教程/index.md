# Shell 教程

## Shell概论

Shell是一个用C语言编写的程序，它诞生于Unix，是我们通过命令行与Unix/Linux交互的工具。笼统地说：Shell既是一种命令语言，又是一种程序设计语言。

而Shell脚本是一种为Shell编写的脚本程序，有的时候也被称为Shell（但二者是两个完全不同的概念！），它可以直接在命令行中执行，也可以将一套逻辑组织成一个文件，方便复用。
Acwing网站提供给的AC Terminal中的命令行就可以看成是一个“Shell脚本在逐行执行”。

Unix/Linux系统中常见的shell脚本有很多：

- Bourne Shell（`/usr/bin/sh`或`/bin/sh`）
- Bourne Again Shell（`/bin/bash`）
- C Shell（`/usr/bin/csh`）
- K Shell（`/usr/bin/ksh`）
- Shell for Root（`/sbin/sh`）

由于Linux系统中一般默认使用bash，而且其易用免费，所以我们接下来学习bash中的语法。

**脚本示例**

进入终端，新建一个`hello.sh`文件，内容如下：

```bash
#! /bin/bash
echo &#34;Hello, World!&#34;
```

**说明：**

* `sh`后缀则表示该文件是Shell脚本文件。

* `#!`告诉系统这个脚本用什么解释器来执行，后面所跟的就是你所需要用的解释器。这个一般都需要添加上，具体解释见运行。
* `echo`指令用于字符串的输出，所以运行该文件会输出`Hello,World`。

**运行方式：**

* 作为可执行文件

	该方法将`hello.sh`作为可执行程序运行，由于未指定解释器，所以使用该方法第一行一定要指定解释器。

```bash
#! /bin/bash
chmod &#43;x hello.sh #使脚本具有可执行权限；
./hello.sh #当前路径下执行
```

* 用解释器执行

	该方法直接运行解释器，此时`hello.sh`作为Shell解释器的参数。此时Shell脚本就不需要指定解释器信息，则不需要第一行的注释了。

	```bash
	bash hello.sh #当然也可以用缩写 sh hello.sh
	```

**输出：**

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/78757c2dc7064b4bae86b9c7461b3ade.png)

## Shell语法

### 注释

* **单行注释**

	每一行中`#`之后的内容均是注释。

	```shell
	# 这是一行注释
	
	echo &#34;Hello, World&#34; # 这也是一行注释
	```

* **多行注释**

	Shell中的多行注释有点特别，格式为：

	```bash
	:&lt;&lt;EOF
	第一行注释
	...
	第n行注释
	EOF
	```

	其中`EOF`可以替换成其他任意字符串，如：

	````shell
	:&lt;&lt;!
	第一行注释
	...
	第n行注释
	!
	````

	

### 变量

* **定义变量**

	定义变量时，变量名不加美元符号`$`，同时特别需要注意的一点就是变量名与等号之间不能有空格（如果有自动打空格的习惯在这里最好克制）。shell中的变量命名同样须遵循如下规则：

	- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
	- 中间不能有空格，可以使用下划线 **_**。
	- 不能使用标点符号。
	- 不能使用bash里的关键字（可用help命令查看保留关键字）。

	如下：

	```bash
	# 有效的Shell变量名称
	_var
	var123
	LF_DDFHI_X
	
	# 无效的Shell变量名称
	?var
	123abc
	echo
	```

	和现在大多数的语言一样，Shell定义变量不需要指定变量类型，如下：

	```bash
	name1=&#34;a&#34; #单引号定义字符串
	name2=&#39;a&#39; #双引号定义字符串
	name3=a   #也可以不加引号，同样表示字符串
	```

	&lt;font color=&#34;red&#34;&gt;在Bash shell中，每一个变量的值都是字符串，无论你给变量赋值用的时单引号双引号还是没有使用引号，都是使用字符串的i形式存储的，这很特殊。&lt;/font&gt;串的i形式存储的，这很特殊。&lt;/font&gt;串的i形式存储的，这很特殊。&lt;/font&gt;串的i形式存储的，这很特殊。&lt;/font&gt;

* **使用变量**

	使用变量我们需要加上`$`符号或者`${}`符号。花括号时可选的，主要是为了帮助解释器识别变量边界。

	&lt;font color=&#34;red&#34;&gt;一定要注意：只有使用变量的时候才加美元符号`$`&lt;/font&gt;变量的时候才加美元符号`$`&lt;/font&gt;变量的时候才加美元符号`$`&lt;/font&gt;

	```bash
	name=hzf
	echo $name
	echo ${name}
	echo ${name}123
	```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/375905eaca5d47a3a259edffdbdec203.png)

* **只读变量**

	我们可以使用`readOnly`或者`declare`将变量设置为只读。

	```bash
	name=hzf
	readOnly name
	#declare -r name #两种写法均可。
	name=ylf #此时会报错，因为已经设置成了只读变量。
	```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/46b734003f9a4fa8bb7ed91eb06cd1f5.png)

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/6b53d378e1594c48843cbeee61fb3816.png)


* **删除变量**

	` unset`可以删除变量。

	```bash
	name=hzf
	unset name
	echo $name
	```

	此时没有输出`hzf`，代表已经删除了，这里没有报错是因为Shell将没有定义的变量设置为空字符串。
	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/63971c3f222d4a3583f0284dae8c51c9.png)


* **变量类型**

	根据访问权限划分可以分为：

	* 自定义变量（局部变量）

		在脚本或命令中定义，仅在当前Shell示例中有效，其他Shell启动的程序不能访问局部变量。即为子进程不能访问的变量。

	* 环境变量（全局变量）

		所有程序，包括shell启动程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。即为子进程可以访问的变量。

	自定义变量转换为环境变量：

	```bash
	name=hzf#定义自定义变量
	export name
	delcare -x name#两种方法均可
	```

	环境变量转化为自定义变量

	```bash
	export name=hzf#定义环境变量。
	delcare &#43;x name#改为自定义变量。
	```

* **Shell字符串**

	字符串可以用单引号，也可以用双引号，也可以不用引号。但其中是有区别的：

	* 使用单引号字符串，其中的变量名不会输出，但可以输出转义字符。单引号字串中不能出现单独一个的单引号（对单引号使用转义符后也不行），但可成对出现，作为字符串拼接使用。

	* 不用引号的字符串，其中变量可以输出，但是不能输出转移字符。
	* 使用双引号的字符串，既可以输出变量也可以输出转义字符。

	示例：

	```bash
	name=hzf
	echo 123$name\n
	echo &#34;123$name\n&#34;
	echo &#39;123$name\n&#39;
	```

	输出：

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/b678f7010e534684b9e7356b81ce8341.png)

  

  **字符串操作**

  * 拼接字符串

	```bash
	name=xyz
	#双引号字符串拼接
	s1=&#34;Hello, $ {name} !”
	s2=&#34;Hello, &#34;$ {name}&#34;!&#34;
	echo $s1 $s2
	```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/e392df84562e4f809610f61c9ad7304a.png)


  * 获取字符串长度

	```bash
	name=hzf
	echo ${#name}
	```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/597a73a23cd54bf9a1873e28bc9aabd2.png)


  * 提取字符串

	注意，第一个字符的索引为$0$，给出的两个参数第一个为起始位置，第二个为截取长度。

	```bash
	name=&#34;Hello, World!&#34;                                                                                                        
	echo ${name}
	echo ${name:0:4}
	echo ${name:0}
	echo ${name:1}
	```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/bf80338d3b6c448ebdc49085148497fa.png)




* **文件参数变量**

	在执行Shell脚本时，可以向脚本传递参数。`$1`是第一个参数，`$2`是第二个参数，以此类推。特殊的，`$0`是文件名（包含路径）。例如：

	```bash
	echo &#34;文件名:$0&#34;                                                                                                                                              
	echo &#34;第一个参数:$1&#34;
	echo &#34;第二个参数:$2&#34;
	echo &#34;$*&#34;
	echo &#34;$@&#34;
	```

	然后我们执行的时候在后面添加参数即可。如果没有给出，那么则为空字符。

 ![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/0631d6df189a4ed19b662c44d3e456e4.png)


* **其他参数相关变量**

	| 参数         | 说明                                                         |
	| ------------ | ------------------------------------------------------------ |
	| `$#`         | 代表文件传入的参数个数，如上例中值为2                        |
	| `$*`         | 由所有参数构成的用空格隔开的字符串，如上例中值为`&#34;$1 $2&#34;`    |
	| `$@`         | 每个参数分别用双引号括起来的字符串，如上例中值为`&#34;$1&#34; &#34;$2&#34; ` |
	| `$$`         | 脚本当前运行的进程ID                                         |
	| `$?`         | 上一条命令的退出状态（注意不是stdout，而是exit code）。0表示正常退出，其他值表示错误 |
	| `$(command)` | 返回`command`这条命令的stdout（可嵌套）                      |
	| `command`    | 返回`command`这条命令的stdout（不可嵌套）                    |

### 数组

数组中可以存放多个值。Bash Shell 只支持一维数组（不支持多维数组），初始化时不需要定义数组大小（与 PHP 类似）。与大部分编程语言类似，数组元素的下标由 0 开始。

* **定义**

	Shell 数组用括号来表示，元素用&#34;空格&#34;符号分割开，语法格式如下：

	```bash
	array_name=(value1 value2 ... valuen)
	```

	如：

	```bash
	name=(张三 &#34;李四&#34; &#39;王五&#39;)
	```

	也可以直接通过定义数组中元素的值来创建数组，如：

	```bash
	name[0]=&#34;张三&#34;
	```

	这样就创建了name数组。

* **访问数组元素**

	* 访问单个元素

		语法格式为：`${array_name[index]}`

		如：

		```bash
		name[0]=&#34;123&#34;                                                                                                                                                 
		name[3]=&#34;124&#34;
		echo &#34;${name[0]}&#34;
		echo &#34;${name[3]}&#34;
		```

	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/9d41480b0709427c95be2ccaad1bc485.png)


  * 访问所有元素

	格式：

	```bash
	${array[@]}#第一种写法
	${array[*]}#第二种写法
	```

	示例：

	```bash
	array=(1 abc &#34;def&#34; yxc)
	
	echo ${array[@]}  # 第一种写法
	echo ${array[*]}  # 第二种写法
	```

	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/a2a6418d85824d1d81adbcdf60ee1e9e.png)

  * 获取数组长度

	同字符串写法：

	```bash
	array=(1 abc &#34;def&#34; yxc)
	
	echo ${#array[@]} #第一种写法
	echo ${#array[*]} #第二种写法
	```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/7383ce64177f4d47900ba723e91960d2.png)

### expr命令与基本运算符

Shell 和其他编程语言一样，支持多种运算符，包括：

- 算数运算符
- 关系运算符
- 布尔运算符
- 字符串运算符
- 文件测试运算符

但是原生Bash不支持简单的数学运算，我们需要通过其他命令来实现，如`awk`,`expr`，`expr`命令最常用，所以这里介绍`expr`。

`expr`是一款表达式计算工具，使用它能完成表达式的求值操作。格式为：

`expr 表达式`

* **表达式说明**

	* 用空格隔开每一项
	* 用反斜杠放在shell特定的字符前面（发现表达式运行错误时，可以试试转义）
	* 对包含空格和其他特殊字符的字符串要用引号括起来
	* `expr`会在`stdout`中输出结果。如果为逻辑关系表达式，则结果为真，`stdout`为1，否则为0。
	* `expr`的`exit code`：如果为逻辑关系表达式，则结果为真，`exit code`为0，否则为1。
	* 完整的表达式要被 \` \` 包含，注意这个字符不是常用的单引号，而是反引号，代表执行该命令。通常在 Esc 键下边。

* **算数表达式**

	下表列出了常用的算术运算符，假定变量 a 为 10，变量 b 为 20：

	| 运算符 | 说明                                          | 举例                           |
	| :----- | :-------------------------------------------- | :----------------------------- |
	| &#43;      | 加法                                          | `expr $a &#43; $b` 结果为 30。     |
	| -      | 减法                                          | `expr $a - $b` 结果为 -10。    |
	| *      | 乘法                                          | `expr $a \* $b` 结果为  200。  |
	| /      | 除法                                          | `expr $b / $a` 结果为 2。      |
	| %      | 取余                                          | `expr $b % $a` 结果为 0。      |
	| =      | 赋值                                          | `a=$b` 将把变量 b 的值赋给 a。 |
	| &lt;font color=&#34;red&#34;&gt;     | 相等。用于比较两个数字，相同则返回 true。     | `[ $a &lt;/font&gt; $b ]` 返回 false。    |
	| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | `[ $a != $b ]` 返回 true。     |

	**注意：**条件表达式要放在方括号之间，并且要有空格，例如: **`[$a&lt;font color=&#34;red&#34;&gt;$b]`** 是错误的，必须写成 **`[ $a &lt;/font&gt; $b ]`**。注意乘号等特殊符号需要转义。`()`可表优先级，但同样需要反斜杠转移。

	实例：

	```bash
	a=10                                                                                                                                                                         
	b=20
	
	echo &#34;a &#43; b = `expr $a &#43; $b`&#34;
	echo &#34;a - b = `expr $a - $b`&#34;
	echo &#34;a * b = `expr $a \* $b`&#34;
	echo &#34;a / b = `expr $a / $b`&#34;
	echo &#34;a % b = `expr $a % $b`&#34;
	echo &#34;a &lt;font color=&#34;red&#34;&gt; b = `expr [$a &lt;/font&gt; $b]`&#34;
	echo &#34;a != b = `expr [$a != $b]`&#34;
	a=$b
	echo &#34;$a&#34;
	```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png)

* **关系运算符**

	Shell支持正常的关系比较运算符，即`&lt;,&lt;=,&gt;,&gt;=`等。其会返回01代表结果。

	实例：

	```bash
	a=10                                                                                                                                                                         
	b=20
	echo &#34;a &lt; b : `expr $a \&lt; $b`&#34; #需要转义
	echo &#34;a &gt; b : `expr $a &#39;&gt;&#39; $b`&#34; #也可以用引号括起来。
	echo &#34;a &gt;= b : `expr $a &#39;&gt;=&#39; $b`&#34;
	```

	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/eab97235bd544c8cadf2ff8a59e81a89.png)


  同时，Shell也给出了自己特定的比较命令，如下表：

| 运算符 | 说明                                                  | 举例                         |
| :----- | :---------------------------------------------------- | :--------------------------- |
| `-eq`  | 检测两个数是否相等，相等返回 true。                   | `[ $a -eq $b ]` 返回 false。 |
| `-ne`  | 检测两个数是否不相等，不相等返回 true。               | `[ $a -ne $b ] `返回 true。  |
| `-gt`  | 检测左边的数是否大于右边的，如果是，则返回 true。     | `[ $a -gt $b ]` 返回 false。 |
| `-lt`  | 检测左边的数是否小于右边的，如果是，则返回 true。     | `[ $a -lt $b ]` 返回 true。  |
| `-ge`  | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | `[ $a -ge $b ]` 返回 false。 |
| `-le`  | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | `[ $a -le $b ]` 返回 true。  |

  实例&lt;font color=&#34;red&#34;&gt;（注：if...then是条件语句，之后会讲解）&lt;/font&gt;：

  ```bash
#!/bin/bash

a=10
b=20

if [ $a -eq $b ]
then
   echo &#34;$a -eq $b : a 等于 b&#34;
else
   echo &#34;$a -eq $b: a 不等于 b&#34;
fi
if [ $a -ne $b ]
then
   echo &#34;$a -ne $b: a 不等于 b&#34;
else
   echo &#34;$a -ne $b : a 等于 b&#34;
fi
if [ $a -gt $b ]
then
   echo &#34;$a -gt $b: a 大于 b&#34;
else
   echo &#34;$a -gt $b: a 不大于 b&#34;
fi
if [ $a -lt $b ]
then
   echo &#34;$a -lt $b: a 小于 b&#34;
else
   echo &#34;$a -lt $b: a 不小于 b&#34;
fi
if [ $a -ge $b ]
then
   echo &#34;$a -ge $b: a 大于或等于 b&#34;
else
   echo &#34;$a -ge $b: a 小于 b&#34;
fi
if [ $a -le $b ]
then
   echo &#34;$a -le $b: a 小于或等于 b&#34;
else
   echo &#34;$a -le $b: a 大于 b&#34;
fi
  ```

 ![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/3cc514dc121449c5932dec5ed5ed4935.png)


* **字符串表达式**

	即结合`expr`命令实现对字符串的操作，常见有以下：

	* `length str`：返回str的长度。
	* `index str charSet`:返回字符集charSet中任意一个字符在str中最前面字符的位置。下标从1开始，如果不存在，则返回0。
	* `substr str st len`：截取字符串str，从st位置开始，长度最大为`len`的子串。如果截取不成功，则返回空字符串。

	实例：

	```bash
	str=&#34;Hello,World!&#34;
	echo `expr length &#34;$str&#34;` #`不是单引号，而是反引号，代表执行该命令。
	echo `expr index &#34;$str&#34; llo`
	echo `expr substr &#34;$str&#34; 1 4`
	```

	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/552fb79080d240bdb6702df6ea2dd9f4.png)

* **文件测试运算符**

	文件测试运算符用于检测 Unix 文件的各种属性。

	属性检测描述如下：

	| 操作符  | 说明                                                         | 举例                      |
	| :------ | :----------------------------------------------------------- | :------------------------ |
	| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
	| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -c $file ] 返回 false。 |
	| -d file | 检测文件是否是目录，如果是，则返回 true。                    | [ -d $file ] 返回 false。 |
	| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
	| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
	| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
	| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
	| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
	| -r file | 检测文件是否可读，如果是，则返回 true。                      | [ -r $file ] 返回 true。  |
	| -w file | 检测文件是否可写，如果是，则返回 true。                      | [ -w $file ] 返回 true。  |
	| -x file | 检测文件是否可执行，如果是，则返回 true。                    | [ -x $file ] 返回 true。  |
	| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | [ -s $file ] 返回 true。  |
	| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | [ -e $file ] 返回 true。  |

### read命令

read命令可用于从标准输入中读取单行数据，当读到文件结束符时，`exit code`为1，否则为0。

参数说明：

* `-p`：后面可以接提示信息。
* `-t`：后面跟秒数，定义输入字符的等待时间，超过等待时间后会自动忽略此命令

实例：

```bash
read name
echo $name
read -p &#34;please input your name:&#34; -t 30 name #读入name的值，等待30s。
echo $name
```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/18c65cb79166418c9ad9f330b8d8de7a.png)

### echo命令

echo命令用于字符串的输出，在前面的过程中我们已经接触到了，其命令格式为：

```bash
echo string
```

我们可以用echo实现更复杂的输出格式控制。

* **显示普通字符串**

	```bash
	echo &#34;Hello, World!&#34;
	```

	这种情况下，我们不加双引号也是可以的。

* **显示转义字符**

	```bash
	echo &#34;\&#34;Hello,World!\&#34;&#34;
	```

	同样，这里的双引号也可以省略。

* **显示变量**

	```bash
	name=hzf
	echo &#34;My name is $name&#34;
	```

* **显示换行等特殊字符**

	注意，这些字符都需要通过`-e`命令开启转义才能起作用的，如：`\\ \a \b \c \d \e \f \n \r \t \v` 这些是要在有 `-e` 的时候才能起作用, 其他时候的转义是不用`- e`也能转义的。

	```bash
	echo -e &#34;Hello\n&#34; #-e开启转义。
	echo &#34;World!&#34;
	```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/bb29f0e88ad149b1b22c7a223d81ad25.png)

* **显示结果定向至文件**

	```bash
	echo &#34;Hello, World!&#34; &gt; output.txt
	```

	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/2d5d4a1ed79a4b53831c743142c8e83d.png)


* **原样输出字符串**

	前面有提及，如果想原样输出，不进行转义或者取变量，用单引号。

	```bash
	name=hzf
	echo &#39;$name&#39;
	```

	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/e16c0084b604452cbfb1d2c9f62ce270.png)


* **显示命令执行结果**

	使用反引号。

	```bash
	echo `date`
	echo `ls`
	```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/5aa567919f7d40f98f1fc020dbf34c8d.png)

### printf命令

Shell的printf命令和C语言中的printf差不多，用于格式化输出。该命令不会像echo命令一样自动添加换行。其语法为：

```bash
printf format-string [args]
```

其中format-string为格式控制字符串，[args]为参数列表。

实例：

```bash
#-表示左对齐，没有则表示右对齐，d为宽度。%s表示输出字符串，%d整型输出。
#%-10s则表示输出宽度为10的左对齐字符串。而%-4.2f则表示输出4位整数，保留2位小数。
printf &#34;%-10s %-8s %-4s\n&#34; 姓名 性别 体重kg  
printf &#34;%-10s %-8s %-4.2f\n&#34; 郭靖 男 66.1234
printf &#34;%-10s %-8s %-4.2f\n&#34; 杨过 男 48.6543
printf &#34;%-10s %-8s %-4.2f\n&#34; 郭芙 女 47.9876
```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/7543126cfccb4735ad5b5d9402d566d7.png)


* **命令格式指示符**

	| 符号  | 说明                                    |
	| ----- | --------------------------------------- |
	| %c    | ASCII字符.显示相对应参数的第一个字符    |
	| %d,%i | 十进制整数                              |
	| %E    | 浮点格式([-d].precisionE [&#43;-dd])        |
	| %e    | 浮点格式([-d].precisione [&#43;-dd])        |
	| %g    | %e或%f转换,看哪一个较短,则删除结尾的零  |
	| %G    | %E或%f转换,看哪一个较短,则删除结尾的零  |
	| %s    | 字符串                                  |
	| %u    | 不带正负号的十进制值                    |
	| %x    | 不带正负号的十六进制.使用a至f表示10至15 |
	| %%    | 字面意义的%                             |
	| %X    | 不带正负号的十六进制.使用A至F表示10至15 |

### test命令与判断符号[]

在命令行中输入`man test`，即可查看`test`命令的用法。
`test`命令可以用于判断文件类型，以及对变量做比较。`test`命令用`exit code`返回结果，而不是使用`stdout`。0表示真，非0表示假。这里可以通过`echo $?`来输出上一条命令的结果。

* **文件判断**
	命令格式：`test 测试参数 filename `
	其中常用测试参数如下表：

| 测试参数                                                     | 代表意义       |
| ------------------------------------------------------------ | -------------- |
| `-e`                                                         | 文件是否存在   |
| `-f`                                                         | 是否为文件     |
| `-d`                                                         | 是否为目录     |
| `-r`                                                         | 文件是否可读   |
| `-w`                                                         | 文件是否可写   |
| `-x`                                                         | 文件是否可执行 |
| `-s`                                                         | 是否为非空文件 |
| 测试                                                         |                |
| ![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/45e5664213814ed38ca3f365f7f6b848.png) |                |

* **整数之间的比较**
	命令格式：`test $a 关系运算符 $b`
	其中关系运算符为上文所提及的，这里不作列举。
* **字符串比较**
	相关操作如表所示：

| 测试参数                                                     | 代表意义                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `test -z str`                                                | 判断str是否为空，如果为空，返回true，否则false               |
| `test -n str`                                                | 判断str是否为非空，如果非空，返回true，否则false.其中-n可以省略 |
| `test str1 == str2`                                          | 判断str1是否等于str2                                         |
| `test str1 != str2`                                          | 判断str1是否不等于str2                                       |
| `![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_15%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png) |                                                              |

* **多重条件判定**
	即判断多个条件是否符合要求，可以嵌套多层。具体操作图表所示：

| 测试参数                                                     | 代表意义               |
| ------------------------------------------------------------ | ---------------------- |
| `-a`                                                         | 两条件是否同时成立     |
| `-o`                                                         | 两条件是否至少一个成立 |
| `!`                                                          | 取反。对返回结果取反   |
| ![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_17%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png) |                        |

* **判断符号`[]`**
	和`test`命令用法几乎一模一样，只是将需要判断的内容放入括号中。
	更常用于`if`语句中，且`[[]]`是`[]`的加强版，支持的特性也更多。
	值得注意的是一些特性：`[]`内的每一项都必须用空格隔开；中括号内的变量，最好用双引号括起来；中括号内的常数，最好用单或双引号括起来。
	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png)

### 判断语句

* `if`语句
	和python的语法有点像，命令格式如下：

```powershell
if [ 条件 ]
then
	内容
elif [ 条件 ]
then 
	内容
else
	内容
 fi
```

注意`fi`为结束标记，是一定需要加的，作为结尾闭合。
实例：
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_17%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20240609093330070.png)

* `case...esac`形式
	命令格式如下：

```powershell
case $变量名 in 
	value1)
		内容
		;; # ;;，类似break
	value2)
		内容
		;;
	*) #类似default
		内容
		;;
esac # 也是结尾闭合语句
```

测试
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20240609093330094.png)

### 循环语句

* `for...in...do...done`语句
	命令格式：

```powershell
for var in val1 val2 val3 ... valN
do
	内容
done # 结尾闭合
```

循环规则为：从左到右遍历，当变量值为列表时，则一次遍历完列表。
实例1：输出a 2 cc，每个元素一行：

```powershell
for var in a 2 cc
do 
	echo $var
done            
```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/f745418c2f304fdcba876bbd796f3238.png)

* `for((...;...;...)) do...done`语句
	命令格式：

```powershell
for ((expression; condition; expression))
do
	内容
done
```

测试：输出1-10。

```powershell
for ((i=0; i&lt;=10; &#43;&#43;i))
do
	echo $i
done 
```

* `while...do...done`语句 
	命令格式：

```powershell
while 条件
do 
	内容
done
```

实例：文件结束符为`ctrl&#43;d`，输入文件结束符后`read`指令返回false。

```powershell
while reabashd name
do
	echo $name
done
```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/cc32a004a1fd44d98c969b23c5352947.png)

* `unti...do..done`语句
	和while语句相同，while能实现的脚本until同样可以实现。但区别是until循环的退出状态为0，与while刚好相反，即while循环在条件为真时继续执行循环而until在条件为假时继续执行循环。
* `break`语句
* 跳出当前的一层循环，break不能跳出case语句。
* `continue`语句
	跳出当前循环。
* 死循环的处理方式
	如果Terminal可以打开该程序，则输入`Ctrl&#43;c`即可。否则可以直接关闭进程：使用`top`命令找到该进程的PID；输入`kill -9 PID`即可关掉此进程。

### 函数

定义格式：

```powershell
[ function ] function_name [()]{
	内容;
	[return int;]
}
```

说明：

1. 可以带function func()定义，也可以直接fun()定义，不带任何参数。
2. 参数返回，可以显示加：return返回，如果不加，将以最后一条命令运行结果作为返回值。&lt;font color=&#34;red&#34;&gt;return后跟数值n(0-255，不能超过该范围)。&lt;/font&gt; 其中函数返回值通过`$?`获取。
3. 函数体声明局部变量可以用local关键字声明。

实例：

```powershell
function add(){
	echo &#34;相加预算函数&#34;
	echo &#34;请输入第一个数&#34;
	read a
	echo &#34;请输入第二个数&#34;
	read b
	return $(($a &#43; $b))
}
add
echo &#34;结果为$?&#34;
```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/9b6f76197a1b44419f1a4bcae1691843.png)

* 函数参数
	在shell中，调用函数可以向其传递参数。在函数体内部，通过`$n`的形式来获取参数的值。例如，`$1`表示第一个参数，`$2`表示第二个参数。这个规则和上文说的文件参数相同。也是在执行的后面添加参数。

### exit命令

`exit`命令用来退出当前的shell进程，&lt;font color=&#34;red&#34;&gt;并返回一个退出状态（0-255，只有0表示成功，其他都表示失败）&lt;/font&gt;；使用`$?`即可接收这个退出状态。
`exit`命令可以接收一个整数值作为参数，代表退出状态。如果不指定，默认值为0。

### 文件重定向

每个进程默认打开3个文件描述符：

* `stdin`：标准输入，从命令行读取数据，文件描述符为0。
* `stdout`：标准输出，向命令行输出数据，文件描述符为1。
* `stderr`：标准错误输出，向命令行输出数据，文件描述符为2。
	可以用文件重定向将这三个文件重定向到其他文件中去。重定向命令列表如下：

| 命令              | 说明                                               |
| ----------------- | -------------------------------------------------- |
| `command &gt; file`  | 将输出重定向到 file。                              |
| `command &lt; file`  | 将输入重定向到 file。                              |
| `command &gt;&gt; file` | 将输出以追加的方式重定向到 file。                  |
| `n &gt; file`        | 将文件描述符为 n 的文件重定向到 file。             |
| `n &gt;&gt; file`       | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| `n &gt;&amp; m`          | 将输出文件 m 和 n 合并。                           |
| `n &lt;&amp; m`          | 将输入文件 m 和 n 合并。                           |
| `&lt;&lt; tag`          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |

* 输入输出重定向实例

```powershell
echo -e &#34;Hello \c&#34; &gt; output.txt  # 将stdout重定向到output.txt中
echo &#34;World&#34; &gt;&gt; output.txt  # 将字符串追加到output.txt中

read str &lt; output.txt  # 从output.txt中读取字符串

echo $str  # 输出结果：Hello World
```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/7870860ada2d464898a64a5de76d5787.png)

* 同时重定向stdin和stdout

创建main.sh编写脚本：

```powershell
read a
read b

echo $(expr &#34;$a&#34; &#43; &#34;$b&#34;)
```

创建input.txt，填写内容为：

```
10
20
```

执行结果如下：
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/50d92d483c144e54a2589058d8401b5b.png)

### 引入外部脚本

类似`C/C&#43;&#43;`中的include操作，`bash`也可以引入其他文件中的代码。
语法格式为：

```powershell
. filename # 注意空格
或者
source filename
```

## 参考文献

[Linux基础课](https://www.acwing.com/activity/content/57/)
[菜鸟教程](https://www.runoob.com/linux/linux-shell.html)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/06.shell%E6%95%99%E7%A8%8B/  

