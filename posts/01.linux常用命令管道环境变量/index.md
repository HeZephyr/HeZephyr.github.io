# Linux常用命令、管道、环境变量

## Linux常用命令

### 系统状况

`top`：查看所有进程的信息（Linux的任务管理器）。

* 打开后输入`M`：按使用内存排序；
* 打开后输入`P`：按使用CPU排序；
* 打开后输入`q`：退出。

`df -h`：查看硬盘使用情况。
`free -h`：查看内存使用情况。
`du -sh`：查看当前目录占用的硬盘空间。
`ps aux`：查看所有进程。
`kill -9 pid`：杀死编号为`pid`的进程。
`kill -s SIGTERM pid`：传递某个具体的信号。
`netstat -nt`：查看所有网络连接。
`w`：列出当前登录的用户。
`ping www.baidu.com`：测试网络连接，检查是否联网。

### 文件权限

`chmod`：修改文件权限

* `chmod &#43;x filename`：给`filename`添加可执行权限；
* `chmod -x filename`：去掉`filename`的可执行权限；
* `chmod abc filename`:其中a，b，c各为一个数字，表示User、Group以及Other的权限。r=4，w=2，x=1，为读，写，可执行。
	如设置所有人对该文件都可读可写可执行，则设置`chmod 777 filename`。

### 文件检索

`find &lt;path&gt;(文件路径) -name &#39;*.py&#39;`：搜索path路径下的所有`py`文件。
`grep xxx`：从`stdin`中读入若干行数据，如果某行中包含`xxx`，则输出该行，否则忽略该行。
`wc`：统计行数、单词数、字节数。

* 既可以从`stdin`中直接读取内容，也可以在命令行参数中传入文件名列表。
* `wc -l`：统计行数。
* `wc -w`：统计单词数。
* `wc -c`：统计字节数。

`tree`：展示当前目录的文件结构。

* `tree path`：展示某个目录的文件结构。
* `tree -a`：显示隐藏文件。

`ag xxx`：搜索当前目录下的所有文件，检索`xxx`字符串。
`cut`：分割一行内容。

* 从`stdin`中读入多行数据。
* `echo $PATH | cut -d &#39;:&#39; -f 3, 5`：输出`PATH`用`:`分割后的第3、5列数据。
* `echo $PATH | cut -d &#39;:&#39; -f 3-5`：输出`PATH`用`:`分割后的第3-5列数据。
* `echo $PATH | cut -c 3, 5`：输出`PATH`的第3、5个字符。
* `echo $PATH | cut -c 3-5`：输出`PATH`的第3-5个字符。

`sort`：将每行内容按字典序排序。

* 可以从`stdin`中读取多行数据。
* 可以从命令行参数中读取文件名列表。

`xargs`：将`stdin`中的数据用空格或回车分割成命令行参数，作为其他命令使用。

* `find . -name &#39;*.py&#39; | xargs cat | wc -l`：统计当前目录下所有python文件的总行数。

### 查看文件内容

`more`：浏览文件内容。

* 回车或空格：下一行。
* `b`：上一页。
* `q`：退出。

`less`：和`more`类似，功能更全。

* 回车：下一行。
* `y`：上一行。
* `Page Down`：下一页。
* `Page Up`：上一页。
* `q`：退出。

`head -3 xxx`：显示`xxx`的前3行内容。

* 同时支持从`stdin`读入内容。

`tail -3 xxx`：显示`xxx`末尾3行内容。

* 同时支持从`stdin`读入内容。

### 用户相关

`history`：展示当前用户的历史操作。内容存放在`~/bash_history`中。

### 工具

`md5Sum`：计算`md5`的哈希值。

* 也可以从`stdin`中读入内容，也可以在命令行参数中传入文件名列表。

`time command`：统计`command`命令的执行时间。
`ipython3`：交互式python环境。可以当作计算器，或者批量管理文件。
 `i command`：`!`表示执行`shell`脚本命令。
`watch -n 0.1 command`：每隔0.1s就执行一次`command`命令。
`tar`：压缩文件。

* `tar -zcvf xxx.tar.gz /path`：压缩。
* `tar -zxvf xxx.tar.gz `：解压缩。 

`diff xxx yyy`：查看文件`xxx`和`yyy`的不同点。

### 安装软件

`sudo command`：以`root`身份运行`command`命令。
`apt-get install xxx`：安装`xxx`软件。
`pip install xxx --user --upgrade`：安装python包。

## 管道

### 管道命令

管道命令操作符是`|`，,它只能处理经由前面一个指令传出的正确输出信息，对错误信息信息没有直接处理能力。然后传递给下一个命令，作为标准的输入。
下图为管道命令的输出说明：
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125214459645.png)
【指令1】正确输出，作为【指令2】的输入，然后【指令2】的输出作为【指令3】的输入 ，【指令3】输出就会直接显示在屏幕上面了。
通过管道之后，我们发现【指令1】和【指令2】的正确输出不显示在屏幕上面，只显示指令3的输出。

其类似于之前学习的文件重定向，可以将前面一个命令的`stdout`重定向下一个命令的`stdin`。但与文件重定向有很大区别：文件重定向左边为命令，右边为文件；管道左右两边均为命令，左边有`stdout`，右边有`stdin`。

值得注意的点：

* 管道命令仅能处理`stdout`，忽略`stderr`。
* 管道右边的命令必须能接受`stdin`。
* 多个管道命令可以串联。

### 实例

* 统计当前目录下所有python文件的总行数
	统计总行数，在前面常用命令学习中，我们已经会了：`wc -l`，统计当前目录所有的python文件，也易得为：`find . -name &#39;*.py&#39;`。那么我们需要解决的问题则是将所有python文件选出来得到其内容再统计。我们则可能会这样：`find . -name &#39;*.py&#39; | cat | wc -l`。但`find . -name &#39;*.py&#39;`得到的是字符串，我们还需要利用`xargs`将字符串分割作为命令行参数，这样即可达到效果。
	即：`find . -name &#39;*.py&#39; | xargs cat | wc -l`。

## 环境变量

### 概念

Linux系统中会用很多环境变量来记录配置信息。环境变量类似于全局变量，可以被各个进程访问到。我们可以通过修改环境变量来方便地修改系统配置。

### 查看环境变量

* 列出当前环境下的所有环境变量：

```powershell
env # 显示当前用户的变量；
set # 显示当前shell的变量，包含当前用户的变量；
export # 显示当前导出成用户变量的shell变量。
```

* 输出某个环境变量的值：`echo $PATH`

### 修改环境变量

修改环境变量我们可以先将修改命令放到`~/.bashrc`文件中。修改完之后需执行`source ~/.bashrc`，来将修改应用到当前的`bash`环境下。

为何将修改命令放到`~/.bashrc`，就可以确保修改会影响未来所有的环境呢？

* 每次启动`bash`，都会先执行`~/.bashrc`。
* 每次`ssh`登录远程服务器，都会启动一个`bash`命令行给我们。
* 每次`tmux`新开一个`pane`，都会启动一个`bash`命令行给我们。
* 未来所有的新开环境都会加载我们修改的内容。

### 常见环境变量

`HOME`：用户的家目录
`PATH`：可执行文件（命令）的存储路径。路径与路径之间用`:`分隔。当某个可执行文件同时出现多个路径中时，会选择从左到右数第一个路径中的执行。下列所有存储路径的环境变量，均采用从左到右的优先顺序。
`LD_LIBRARY_PATH`：用于指定动态链接库(.so文件)的路径，其内容是以冒号分隔的路径列表。
`C_INCLUDE_PATH`：C语言的头文件路径，内容是以冒号分隔的路径列表。
`CPLUS_INCLUDE_PATH`：CPP的头文件路径，内容是以冒号分隔的路径列表。
`PYTHONPATH`：Python导入包的路径，内容是以冒号分隔的路径列表。
`JAVA_HOME`：jdk的安装目录。
`CLASSPATH`：存放Java导入类的路径，内容是以冒号分隔的路径列表。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.linux%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4%E7%AE%A1%E9%81%93%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F/  

