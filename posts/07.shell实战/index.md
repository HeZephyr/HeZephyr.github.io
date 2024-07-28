# Shell 23道例题实战

## 统计文件的行数

&gt; 编写一个`shell`脚本以输出一个文本文件`nowcoder.txt`中的行数

```shell
#!/bin/bash

# 方法1：使用 wc -l 和 awk
# 统计行数并使用 awk 提取第一个字段，即行数
lines=$(wc -l nowcoder.txt | awk &#39;{print $1}&#39;)
echo &#34;使用 wc -l 和 awk：$lines 行&#34;

# 方法2：通过输入流传递文件内容给 wc -l
# 使用 &lt; 操作符
lines=$(wc -l &lt; nowcoder.txt)
echo &#34;通过输入流：$lines 行&#34;

# 方法3：使用 cat 和管道传递给 wc -l
# 使用 cat 命令和管道
lines=$(cat nowcoder.txt | wc -l)
echo &#34;通过管道：$lines 行&#34;

# 方法4：使用 sed 统计行数
# 使用 sed 的 -n &#39;$=&#39; 选项
lines=$(sed -n &#39;$=&#39; nowcoder.txt)
echo &#34;使用 sed：$lines 行&#34;
```

## 打印文件的最后5行

&gt; 查看日志的时候，经常会从文件的末尾往前查看，请你写一个`bash shell`脚本以输出一个文本文件`nowcoder.txt`中的最后5行。

```shell
#!/bin/bash

# 查看文件的前5行
echo &#34;前5行：&#34;
head -5 nowcoder.txt
echo &#34;&#34;

# 查看文件的后5行
echo &#34;后5行：&#34;
tail -5 nowcoder.txt
echo &#34;&#34;

# 查看文件的第5行到第20行
echo &#34;第5行到第20行：&#34;
sed -n &#39;5,20p&#39; nowcoder.txt
```

## 输出 0 到 500 中 7 的倍数

&gt; 写一个 `bash`脚本以输出数字 $0$ 到 $500$ 中 $7$ 的倍数$(0 7 14 21...)$的命令

```shell
#!/bin/bash

# 方法1：使用 Bash 的扩展语法的 for 循环
echo &#34;方法1：使用 Bash 的扩展语法的 for 循环&#34;
for item in {0..500..7}
do 
    echo $item
done

echo &#34;&#34; # 分隔行

# 方法2：使用 seq 命令 
# seq [选项]... 首部 增量 尾部
echo &#34;方法2：使用 seq 命令&#34;
seq 0 7 500

echo &#34;&#34; # 分隔行

# 方法3：使用 while 循环
echo &#34;方法3：使用 while 循环&#34;
# 初始化变量
i=0
# 使用 while 循环
while [ $i -le 500 ]
do
    # 输出当前的 7 的倍数
    echo $i
    
    # 增加 7
    i=$((i &#43; 7))
done
```

## 输出第5行的内容

&gt; 编写一个`bash`脚本以输出一个文本文件`nowcoder.txt`中第$5$行的内容。

```shell
#!/bin/bash

# head 命令拿到前五行，再通过通道，通过tail取出来最后一行，即第五行
head -n 5 nowcoder.txt | tail -n 1
#!/bin/bash

# 使用sed 命令中的 p选项，打印第五行
sed -n 5p nowcoder.txt
```

## 打印空行的行号

&gt; 编写一个`shell`脚本以输出一个文本文件`nowcoder.txt`中空行的行号（空行可能连续，从1开始输出）

```shell
#!/bin/bash

# 使用 grep 命令匹配所有空行，并且输出匹配的行号。-n 选项表示输出匹配行的行号，&#39;^$&#39; 匹配空行。使用 cut 命令以 : 作为分隔符，提取每行的第一个字段，即行号。
grep -n &#39;^$&#39; nowcoder.txt | cut -d&#39;:&#39; -f1

# 使用 awk 命令，NF 表示当前行的字段数，NR 表示当前行号。当字段数为0时，即当前行为空行，{ print NR } 输出当前行的行号。
awk &#39;NF == 0 { print NR }&#39; nowcoder.txt

# 使用 sed 命令匹配所有空行，并输出匹配行的行号。-n 选项表示只输出指定的行，/^$/ 匹配空行，=表示输出匹配行的行号。
sed -n &#39;/^$/=&#39; nowcoder.txt
```

## 去掉空行

&gt; 写一个 `bash`脚本以去掉一个文本文件`nowcoder.txt`中的空行

```shell
#!/bin/bash

# 使用 grep 命令匹配所有非空行。-v 选项表示反转匹配，&#39;^$&#39; 匹配空行。
grep -v &#39;^$&#39; nowcoder.txt

# 使用 sed 命令删除匹配空行的行。/^$/ 匹配空行，d 命令删除匹配的行
sed &#39;/^$/d&#39; nowcoder.txt &gt; nowcoder_no_empty_lines.txt

# 使用 awk 命令，NF 表示字段数，NF 为真时表示非空行。
awk &#39;NF&#39; nowcoder.txt
```

## 打印字母数小于8的单词

&gt; 写一个`bash`脚本以统计一个文本文件`nowcoder.txt`中字母数小于8的单词。

```shell
#!/bin/bash

# 使用 awk 命令遍历每个单词，NF 表示当前行的单词数，length($i) 表示当前单词的字母数，如果字母数小于8，则打印当前单词。
awk &#39;{ for (i=1; i&lt;=NF; i&#43;&#43;) if (length($i)&lt;8) print $i }&#39; nowcoder.txt

# 使用 grep 命令匹配字母数小于8的单词。-o 选项表示只输出匹配的内容，\b 表示单词边界，\w\{1,7\} 匹配字母数在1到7之间的单词。
grep -o &#39;\b\w\{1,7\}\b&#39; nowcoder.txt
```

## 统计所有进程占用内存百分比的和

&gt; 假设 `nowcoder.txt` 内容如下：
&gt;
&gt; ```c
&gt; USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
&gt; root         1  0.0  0.4  77744  8332 ?        Ss    2021   1:15 /sbin/init noibrs splash
&gt; root         2  0.0  0.0      0     0 ?        S     2021   0:00 [kthreadd]
&gt; root         4  0.0  0.0      0     0 ?        I&lt;    2021   0:00 [kworker/0:0H]
&gt; daemon     486  0.0  0.1  28340  2372 ?        Ss    2021   0:00 /usr/sbin/atd -f
&gt; root       586  0.0  0.3  72308  6244 ?        Ss    2021   0:01 /usr/sbin/sshd -D
&gt; root     12847  0.0  0.0   4528    68 ?        S&lt;   Jan03   0:13 /usr/sbin/atopacctd
&gt; root     16306  1.7  1.2 151964 26132 ?        S&lt;sl Apr15 512:03 /usr/local/aegis/aegis_client/aegis_11_25/AliYunDun
&gt; root     24143  0.0  0.4  25608  8652 ?        S&lt;Ls 00:00   0:03 /usr/bin/atop -R -w /var/log/atop/atop_20220505 600
&gt; root     24901  0.0  0.3 107792  7008 ?        Ss   15:37   0:00 sshd: root@pts/0
&gt; root     24903  0.0  0.3  76532  7580 ?        Ss   15:37   0:00 /lib/systemd/systemd --user
&gt; root     24904  0.0  0.1 111520  2392 ?        S    15:37   0:00 (sd-pam)
&gt; ```
&gt;
&gt; 以上内容是通过`ps aux`命令输出到`nowcoder.txt`文件中的，请你写一个脚本计算一下所有进程占用内存大小的和。

```shell
#!/bin/bash

# 使用awk命令过滤到第一行并累加$4
awk &#39;BEGIN { sum=0 } NR &gt; 1 { sum&#43;=$4 } END { print sum }&#39; nowcoder.txt

# 使用while循环读取，并用if跳过第一行，使用bc进行浮点数加法运算
sum=0
cnt=1
while read -r line;
do
    if [ $cnt -gt 1 ]; then
        mem=$(echo $line | awk &#39;{ print $4 }&#39;)
        sum=$(echo &#34;$sum&#43;$mem&#34; | bc)
    fi
    cnt=$((cnt&#43;1))
done
echo $sum
```

&gt; ```shell
&gt; #!/bin/bash
&gt; 
&gt; sum=0
&gt; tail -n &#43;2 nowcoder.txt | while read -r line;
&gt; do
&gt;     mem=$(echo $line | awk &#39;{ print $4 }&#39;)
&gt;     sum=$(echo &#34;$sum&#43;$mem&#34; | bc)
&gt;     echo $sum
&gt; done
&gt; echo $sum
&gt; 
&gt; ```
&gt;
&gt; 在 `Bash` 中，&lt;font color=&#34;red&#34;&gt;管道中的命令会在子 shell 中执行，因此变量修改不会影响主 shell 中的变量&lt;/font&gt;。这就是为什么看到 `sum` 在循环内部被正确更新，但在循环外部仍然是初始值 `0`。

## 统计每个单词出现的个数

&gt; 写一个`bash`脚本以统计一个文本文件`nowcoder.txt` 中每个单词出现的个数。
&gt;
&gt; 为了简单起见，你可以假设：
&gt; `nowcoder.txt`只包括小写字母和空格，每个单词只由小写字母组成，单词间由一个或多个空格字符分隔。

```bash
#!/bin/bash

# 将空格转换为换行符，以便每个单词占一行
tr -s &#39; &#39; &#39;\n&#39; &lt;nowcoder.txt |
	# 对单词进行排序
	sort |
	# 统计每个单词的出现次数
	uniq -c |
	# 调整输出格式为&#34;单词 词频&#34;
	awk &#39;{ print $2, $1 }&#39; |
	# 按词频升序排序，-k2,2 意味着只使用第二列进行排序，表示按数值进行排序（默认情况按字典序排序）
	sort -k2,2n 
	
# 使用 awk 统计每个单词的出现次数
# NF 表示当前行的字段数，即单词数
# 使用一个关联数组 cnt 存储每个单词出现的次数

awk &#39;{
    for (i=1; i&lt;=NF; i&#43;&#43;)  # 遍历当前行的每个单词
        cnt[$i] &#43;= 1        # 将单词加入关联数组 cnt，统计出现次数
}
END {
    for (x in cnt)           # 遍历关联数组 cnt
        print x, cnt[x]       # 输出单词和对应的出现次数
}&#39; nowcoder.txt | sort -k2,2n
```

## 第二列是否有重复

&gt; 给定一个`nowcoder.txt`文件，其中有3列信息，如下：
&gt;
&gt; ```txt
&gt; 20201001 python 99
&gt; 20201002 go 80
&gt; 20201002 c&#43;&#43; 88
&gt; 20201003 php 77
&gt; 20201001 go 88
&gt; 20201005 shell 89
&gt; 20201006 java 70
&gt; 20201008 c 100
&gt; 20201007 java 88
&gt; 20201006 go 97
&gt; ```
&gt;
&gt; 编写一个`shell`脚本来检查文件第二列是否有重复，且有几个重复，并提取出重复的行的第二列信息（先按次数排序，如果次数相同，按照单词字母顺序排序），输入如下：
&gt;
&gt; ```txt
&gt; 2 java
&gt; 3 go
&gt; ```

```bash
#!/bin/bash

cat nowcoder.txt | 
awk &#39;{ print $2 }&#39; | 
sort | uniq -c | 
awk &#39;{ print $1, $2 }&#39; | # 重新格式化输出
sort -k1,1n -k2,2 | # 按照出现次数和字母顺序排序
grep -v &#39;1&#39;	# 过滤出现次数不为 1 的行
```

## 转置文件的内容

&gt; 写一个`bash`脚本来转置文本文件`nowcoder.txt`中的文件内容。
&gt; 文件中每行列数相同，并且每个字段由空格分隔

```shell
#!/bin/bash

# 读取文件并使用 awk 转置文件内容
awk &#39;
{
    # 遍历当前行的每一个字段
    for (i = 1; i &lt;= NF; i&#43;&#43;) {
        a[NR, i] = $i  # 将每个字段存储在一个二维数组中，a[行号, 列号] = 值
    }
}
NF &gt; p { p = NF }  # 如果当前行的字段数大于 p，则更新 p 为当前行的字段数

END {
    # 遍历每一列（由最大字段数 p 确定）
    for (i = 1; i &lt;= p; i&#43;&#43;) {
        # 遍历每一行（由总行数 NR 确定）
        for (j = 1; j &lt;= NR; j&#43;&#43;) {
            printf(&#34;%s%s&#34;, a[j,i], (j==NR ? &#34;&#34; : &#34; &#34;))  # 输出数组中对应的字段值，并在每个字段后添加空格，除非是最后一个字段
        }
        printf(&#34;\n&#34;)  # 每一列输出完之后换行
    }
}&#39; nowcoder.txt
```

## 打印每一行出现的数字个数

&gt; 写一个`bash`脚本，统计一个文本文件`nowcoder.txt`中每一行出现的`1~5`数字的个数，并且计算一下整个文档中一共出现了几个`1~5`数字的总数。

```shell
#!/bin/bash

# 使用 awk 读取文件并统计每行中包含的特定数字（1, 2, 3, 4, 5）的数量
awk -F &#34;[1,2,3,4,5]&#34; &#39;
BEGIN { 
    sum = 0  # 初始化 sum 变量，用于存储总和
} {
    # 打印当前行号 NR 以及当前行中包含的特定数字的数量 (NF - 1)
    print(&#34;line&#34; NR &#34; number: &#34; (NF - 1))
    
    # 将当前行中包含的特定数字的数量累加到 sum
    sum &#43;= (NF - 1)
} END {
    # 打印总和
    print(&#34;sum is &#34; sum)
}&#39; nowcoder.txt
```

## 去掉所有包含this的句子

&gt; 编写一个`shell`脚本以实现如下功能：去掉输入中含有`this`的语句，把不含`this`的语句输出

```shell
#!/bin/bash

# -v 反转匹配
grep -v &#34;this&#34; nowcoder.txt

# sed 命令 -&gt; d 删除 -&gt; // 包含要搜索的字符串
sed &#39;/this/d&#39; nowcoder.txt

# awk 命令，$0为当前行的所有内容，!~ 是 awk 的模式匹配运算符，表示模式不匹配
awk &#39;$0!~/this/ {print $0}&#39; nowcoder.txt
```

## 求平均值

&gt; 写一个`bash`脚本以实现一个需求，求输入的一个数组的平均值
&gt;
&gt; 第`1`行为输入的数组长度`N`
&gt;
&gt; 第`2~N`行为数组的元素，如以下为:
&gt; 数组长度为`4`，数组元素为`1 2 9 8`

```shell
#!/bin/bash

awk &#39;BEGIN {
    sum = 0
}{
    if (NR == 1) {
        N = $1  # 将第一行的数字数量保存到变量 N 中
    } else {
        sum &#43;= $1  # 对随后的数字进行累加求和
    }
} END {
    printf(&#34;%.3f&#34;, sum / N)  # 输出平均值，保留三位小数
}&#39;
```

## 去掉不需要的单词

&gt; 写一个`bash`脚本以实现一个需求，去掉输入中含有`B`和`b`的单词。

```shell
#!/bin/bash

# 使用 sed -n 命令打印不包含 &#39;B&#39; 和 &#39;b&#39; 的行
# /^[^bB]*$/ 表示匹配不包含 &#39;B&#39; 和 &#39;b&#39; 的行，^ 表示行开头，[^bB] 表示不包含 &#39;B&#39; 和 &#39;b&#39; 的任何字符，* 表示零次或多次重复，$ 表示行结尾
sed -n &#39;/^[^bB]*$/p&#39; nowcoder.txt

# 使用 grep -E -v 命令排除包含 &#39;B&#39; 和 &#39;b&#39; 的行
# -E 选项启用扩展的正则表达式，-v 选项表示反转匹配
grep -E -v &#34;[bB]&#34; nowcoder.txt

# 使用 awk 命令，遍历每个单词，如果不包含 &#39;B&#39; 和 &#39;b&#39;，则输出该单词
awk &#39;{
    for (i = 1;i &lt;= NF; i&#43;&#43;) {
        if ($i !~ /b|B/) {  # 使用正则表达式匹配单词中不包含 &#39;B&#39; 和 &#39;b&#39; 的部分
            printf(&#34;%s &#34;, $i)  # 输出不包含 &#39;B&#39; 和 &#39;b&#39; 的单词
        }
    }
}&#39; nowcoder.txt
```

## 判断输入的是否为IP地址

&gt; 写一个脚本统计文件`nowcoder.txt`中的每一行是否是正确的`IP`地址。
&gt;
&gt; 如果是正确的`IP`地址输出：`yes`
&gt;
&gt; 如果是错误的`IP`地址，且是四段号码的话输出：`no`，否则的话输出：`error`

```shell
#!/bin/bash

awk -F &#34;.&#34; &#39;{
    flag = &#34;error&#34;
    if (NF == 4) {
        flag = &#34;yes&#34;
        for (i = 1; i &lt;= NF; i&#43;&#43;) {
            if ($i &gt; 255) {
                flag = &#34;no&#34;;
                break;
            }
        }
    }
    printf(flag&#34;\n&#34;)
}&#39; nowcoder.txt
```

## 将字段逆序输出文件的每行

&gt; 编写一个`shell`脚本，将文件`nowcoder.txt`中每一行的字段逆序输出，其中字段之间使用英文冒号`:`相分隔。
&gt;
&gt; 假设`nowcoder.txt`内容如下：
&gt;
&gt; ```txt
&gt; nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
&gt; root:*:0:0:System Administrator:/var/root:/bin/sh
&gt; ```
&gt;
&gt; 你的脚本应当输出
&gt;
&gt; ```txt
&gt; /usr/bin/false:/var/empty:Unprivileged User:-2:-2:*:nobody
&gt; /bin/sh:/var/root:System Administrator:0:0:*:root
&gt; ```

```shell
#!/bin/bash

awk -F &#34;:&#34; &#39;{
    for (i = 1; i &lt;= NF; i&#43;&#43;) {
        temp[i] = $i
    }
    for (i = NF; i &gt;= 1; i--) {
        printf(&#34;%s%s&#34;, temp[i], (i == 1 ? &#34;\n&#34; : &#34;:&#34;))
    }
}&#39; nowcoder.txt
```

## 域名进行计数排序处理

&gt; 假设有一些域名，存储在`nowcoder.txt`里，现在需要写一个`shell`脚本，将域名取出并根据域名进行计数排序处理（降序）。
&gt;
&gt; 假设`nowcoder.txt`内容如下：
&gt;
&gt; ```html
&gt; http://www.nowcoder.com/index.html
&gt; http://www.nowcoder.com/1.html
&gt; http://m.nowcoder.com/index.html
&gt; ```
&gt;
&gt; 你的脚本应该输出：
&gt;
&gt; ```html
&gt; 2 www.nowcoder.com
&gt; 1 m.nowcoder.com
&gt; ```

```shell
#!/bin/bash

awk -F &#39;/&#39; &#39;{
    print($3)
}&#39; nowcoder.txt |
sort |
uniq -c |
sort -r |
awk &#39;{
    print($1&#34; &#34;$2)
}&#39;
```

## 打印等腰三角形

&gt; 编写一个`shell`脚本，输入正整数`n`，打印边长为`n`的等腰三角形。
&gt;
&gt; 示例：
&gt;
&gt; 输入：`5`
&gt;
&gt; 输出：
&gt;
&gt; ```txt
&gt;     *
&gt;    * *
&gt;   * * *
&gt;  * * * *
&gt; * * * * *
&gt; ```

```shell
#!/bin/bash

read n
for ((i = 1; i &lt;= n; i&#43;&#43;))
do
    for ((j = 1; j &lt;= n - i; j&#43;&#43;))
    do
        printf &#34; &#34;
    done
    for ((j = 1; j &lt;= i; j&#43;&#43;))
    do
        if [[ $j -eq $i ]]; then
            printf &#34;*&#34;
        else
            printf &#34;* &#34;
        fi
    done
    printf &#34;\n&#34;
done
```

## 打印只有一个数字的行

&gt; 假设有一个`nowcoder.txt`，编写脚本，打印只有一个数字的行。

```shell
#!/bin/bash

awk -F &#34;[0-9]&#34; &#39;{
    if (NF == 2) {
        print($0)
    }
}&#39;
```

## 格式化输出

&gt; 有一个文件`nowcoder.txt`，里面的每一行都是一个数字串，编写一个`shell`脚本对文件中每一行的数字串进行格式化：每$3$个数字加入一个逗号（,）。
&gt;
&gt; 例如：数字串为“123456789”，那么需要格式化为123,456,789。

```shell
#!/bin/bash

# 使用-F分割数字串
awk -F &#34;&#34; &#39;{
    for (i = 1; i &lt;= NF; i&#43;&#43;) {
        printf($i)
        if ((NF - i) % 3 == 0 &amp;&amp; i != NF) {
            printf(&#34;,&#34;)
        }
    }
    printf(&#34;\n&#34;)
}&#39;
```

## 处理文本

&gt; 有一个文本文件`nowcoder.txt`，假设内容格式如下：
&gt;
&gt; ```text
&gt; 111:13443
&gt; 222:13211
&gt; 111:13643
&gt; 333:12341
&gt; 222:12123
&gt; ```
&gt;
&gt; 现在需要编写一个`shell`脚本，按照以下的格式输出：
&gt;
&gt; ```text
&gt; [111]
&gt; 13443
&gt; 13643
&gt; [222]
&gt; 13211
&gt; 12123
&gt; [333]
&gt; 12341
&gt; ```

```shell
#!/bin/bash

awk -F &#34;:&#34; &#39;{
    cnt[$1] = cnt[$1] $2 &#34;\n&#34;
} END {
    for (i in cnt) {
        printf(&#34;[%s]\n%s&#34;, i, cnt[i])
    }
}&#39; nowcoder.txt
```

## Nginx日志分析1-IP访问次数统计

&gt; 假设 `Nginx` 的日志存储在 `nowcoder.txt` 里，内容如下：
&gt;
&gt; ```c
&gt; 192.168.1.20 - - [21/Apr/2020:14:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [21/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [21/Apr/2020:21:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.23 - - [21/Apr/2020:22:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [22/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [22/Apr/2020:15:26:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:08:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:09:20:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:14:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:15:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:16:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:15:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; ```
&gt;
&gt; 现在需要编写 Shell 脚本统计出 2020 年 4 月 23 号访问 IP 的对应次数，并且按照次数降序排序。你的脚本应该输出：
&gt;
&gt; ```c
&gt; 5 192.168.1.22
&gt; 4 192.168.1.21
&gt; 3 192.168.1.20
&gt; 2 192.168.1.25
&gt; 1 192.168.1.24
&gt; ```

```shell
#!/bin/bash

# 通过grep过滤，再统计排序
grep &#34;23/Apr/2020&#34; nowcoder.txt |
awk &#39;{ print $1 }&#39; |
sort |
uniq -c |
sort -r |
awk &#39;{ print $1, $2 }&#39;
```

## Nginx日志分析2-统计某个时间段的IP访问量

&gt; 假设 `Nginx` 的日志存储在 `nowcoder.txt` 里，内容如下：
&gt;
&gt; ```txt
&gt; 192.168.1.20 - - [21/Apr/2020:14:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [21/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [21/Apr/2020:21:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.23 - - [21/Apr/2020:22:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [22/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [22/Apr/2020:15:26:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:08:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:09:20:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:14:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:15:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:16:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:22:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:23:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; ```
&gt;
&gt; 现在需要编写 `Shell` 脚本统计 2020年04月23日20点至23点去重后的 IP 访问量，你的脚本应该输出：
&gt;
&gt; ```c
&gt; 5
&gt; ```
&gt;
&gt; 输出说明：2020年04月23日20点至23点，共有 192.168.1.24、192.168.1.25、192.168.1.20、192.168.1.21、192.168.1.22 共 5 个 IP 访问了。

```shell
#!/bin/bash

grep &#34;23/Apr/2020:2[0-3]&#34; nowcoder.txt |
awk &#39;{ print $1 }&#39; |
sort |
uniq |
wc -l
```

## nginx日志分析3-统计访问3次以上的IP

&gt; 假设`nginx`的日志我们存储在nowcoder.txt里，格式如下：
&gt;
&gt; ```txt
&gt; 192.168.1.20 - - [21/Apr/2020:14:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [21/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [21/Apr/2020:21:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.23 - - [21/Apr/2020:22:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [22/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [22/Apr/2020:15:26:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:08:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:09:20:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:14:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:15:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:16:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:22:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:23:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; ```
&gt;
&gt; 现在需要编写`shell`脚本统计访问3次以上的IP，你的脚本应该输出：
&gt;
&gt; ```txt
&gt; 6 192.168.1.22
&gt; 5 192.168.1.21
&gt; 4 192.168.1.20
&gt; ```

```shell
#!/bin/bash

awk &#39;{ print $1 }&#39; nowcoder.txt |
sort |
uniq -c |
sort -r |
awk &#39;{
    if ($1 &gt; 3) {
        print $1, $2
    }
}&#39;
```

## Nginx日志分析4-查询某个IP的详细访问情况

&gt; 假设`Nginx`的日志存储在`nowcoder.txt`里，内容如下：
&gt;
&gt; ```txt
&gt; 192.168.1.20 - - [21/Apr/2020:14:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [21/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [21/Apr/2020:21:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.23 - - [21/Apr/2020:22:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [22/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [22/Apr/2020:15:26:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:08:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:09:20:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:14:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:15:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:15:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:16:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:22:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:23:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; ```
&gt;
&gt; 现在需要编写`shell`脚本查询192.168.1.22的详细访问次数情况，按访问频率降序排序。你的脚本应该输出：
&gt;
&gt; ```c
&gt; 4 /1/index.php
&gt; 2 /3/index.php
&gt; ```

```shell
#!/bin/bash

grep &#34;192.168.1.22&#34; |
awk &#39;{ print $7 }&#39; |
sort |
uniq -c |
sort -r |
awk &#39;{ print $1, $2 }&#39;
```

## nginx日志分析5-统计爬虫抓取404的次数

&gt; 假设`nginx`的日志存储在`nowcoder.txt`里，内容如下：
&gt;
&gt; ```
&gt; 192.168.1.20 - - [21/Apr/2020:14:12:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 301 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [21/Apr/2020:15:00:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 500 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [21/Apr/2020:21:21:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.23 - - [21/Apr/2020:22:10:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [22/Apr/2020:15:00:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 200 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [22/Apr/2020:15:26:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:08:05:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (compatible; Baiduspider/2.0; &#43;http://www.baidu.com/search/spider.html)&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:09:20:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 200 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:14:12:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 200 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:15:00:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:15:00:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (compatible; Baiduspider/2.0; &#43;http://www.baidu.com/search/spider.html)&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:15:00:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 200 490 &#34;-&#34; &#34;Mozilla/5.0 (compatible; Baiduspider/2.0; &#43;http://www.baidu.com/search/spider.html)&#34;
&gt; 192.168.1.24 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 200 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 300 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 500 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:22:10:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:23:59:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; ```
&gt;
&gt; 现在需要编写`shell`脚本统计百度爬虫抓取404的次数，你的脚本应该输出
&gt;
&gt; ```txt
&gt; 2
&gt; ```

```shell
#!/bin/bash

grep &#34;404&#34; | grep &#34;www.baidu.com&#34; | wc -l
```

## Nginx日志分析6-统计每分钟的请求数

&gt; 假设`Nginx`的日志存储在nowcoder.txt里，内容如下：
&gt;
&gt; ```txt
&gt; 192.168.1.20 - - [21/Apr/2020:14:12:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [21/Apr/2020:15:00:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [21/Apr/2020:21:21:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.23 - - [21/Apr/2020:22:10:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [22/Apr/2020:15:00:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [22/Apr/2020:15:26:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:08:05:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Baiduspider&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:09:20:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:10:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:14:12:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:15:00:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:15:00:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Baiduspider&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:16:15:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.24 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /2/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.25 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /3/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.20 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:20:27:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.22 - - [23/Apr/2020:22:10:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; 192.168.1.21 - - [23/Apr/2020:23:59:49 &#43;0800] &#34;GET /1/index.php HTTP/1.1&#34; 404 490 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0&#34;
&gt; ```
&gt;
&gt; 现在需要编写`Shell`脚本统计每分钟的请求数，并且按照请求数降序排序。你的脚本应该输出：
&gt;
&gt; ```c
&gt; 5 20:27
&gt; 4 15:00
&gt; 2 22:10
&gt; 2 14:12
&gt; 2 10:27
&gt; 1 23:59
&gt; 1 21:21
&gt; 1 16:15
&gt; 1 15:26
&gt; 1 09:20
&gt; 1 08:05
&gt; ```

```shell
#!/bin/bash

awk -F &#34;:&#34; &#39;{ print $2&#34;:&#34;$3 }&#39; |
sort |
uniq -c |
sort -r |
awk &#39;{ print $1, $2 }&#39;
```

## netstat练习1-查看各个状态的连接数

&gt; 假设netstat命令运行的结果我们存储在nowcoder.txt里，格式如下：
&gt;
&gt; ```txt
&gt; Proto Recv-Q Send-Q Local Address           Foreign Address         State
&gt; tcp        0      0 0.0.0.0:6160            0.0.0.0:*               LISTEN
&gt; tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN
&gt; tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
&gt; tcp        0      0 172.16.56.200:41856     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49822     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49674     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:42316     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:44076     172.16.240.74:6379      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49656     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58248     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:50108     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41944     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:35548     100.100.32.118:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:39024     100.100.45.106:443      TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41788     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58260     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41812     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41854     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58252     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:49586     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41754     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50466     120.55.222.235:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:38514     100.100.142.5:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:49832     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:52162     100.100.30.25:80        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50372     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50306     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49600     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41908     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:60292     100.100.142.1:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:37650     100.100.54.133:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41938     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49736     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41890     172.16.34.144:3306      ESTABLISHED
&gt; udp        0      0 127.0.0.1:323           0.0.0.0:*
&gt; udp        0      0 0.0.0.0:45881           0.0.0.0:*
&gt; udp        0      0 127.0.0.53:53           0.0.0.0:*
&gt; udp        0      0 172.16.56.200:68        0.0.0.0:*
&gt; udp6       0      0 ::1:323                 :::*
&gt; raw6       0      0 :::58                   :::*                    7
&gt; ```
&gt;
&gt; 现在需要编写shell脚本查看系统tcp连接中各个状态的连接数，并且按照连接数降序输出。你的脚本应该输出如下：
&gt;
&gt; ```txt
&gt; ESTABLISHED 22
&gt; TIME_WAIT 9
&gt; LISTEN 3
&gt; ```

```shell
#!/bin/bash

grep &#34;tcp&#34; |
awk &#39;{ print $6 }&#39; |
sort |
uniq -c |
sort -nr |
awk &#39;{ print $2, $1 }&#39;
```

## netstat练习2-查看和3306端口建立的连接

&gt; 假设netstat命令运行的结果我们存储在nowcoder.txt里，格式如下：
&gt;
&gt; ```txt
&gt; Proto Recv-Q Send-Q Local Address           Foreign Address         State
&gt; tcp        0      0 0.0.0.0:6160            0.0.0.0:*               LISTEN
&gt; tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN
&gt; tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
&gt; tcp        0      0 172.16.56.200:41856     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49822     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49674     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:42316     172.16.34.143:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:44076     172.16.240.74:6379      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49656     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58248     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:50108     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41944     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:35548     100.100.32.118:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:39024     100.100.45.106:443      TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41788     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58260     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41812     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41854     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58252     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:49586     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41754     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50466     120.55.222.235:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:38514     100.100.142.5:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:49832     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:52162     100.100.30.25:80        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50372     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50306     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49600     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41908     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:60292     100.100.142.1:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:37650     100.100.54.133:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41938     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49736     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41890     172.16.34.144:3306      ESTABLISHED
&gt; udp        0      0 127.0.0.1:323           0.0.0.0:*
&gt; udp        0      0 0.0.0.0:45881           0.0.0.0:*
&gt; udp        0      0 127.0.0.53:53           0.0.0.0:*
&gt; udp        0      0 172.16.56.200:68        0.0.0.0:*
&gt; udp6       0      0 ::1:323                 :::*
&gt; raw6       0      0 :::58                   :::*                    7
&gt; ```
&gt;
&gt; 现在需要你查看和本机3306端口建立连接并且状态是`established`的所有IP，按照连接数降序排序。你的脚本应该输出
&gt;
&gt; ```txt
&gt; 10 172.16.0.24
&gt; 9 172.16.34.144
&gt; 1 172.16.34.143
&gt; ```

```shell
#!/bin/bash

grep &#34;tcp.*ESTABLISHED&#34; |
awk &#39;{ print $5 }&#39; |
awk -F &#34;:&#34; &#39;$2 == 3306 { print $1 }&#39; |
sort |
uniq -c |
sort -nr |
awk &#39;{ print $1, $2 }&#39;
```

## netstat练习3-输出每个IP的连接数

&gt; 假设`netstat`命令运行的结果我们存储在`nowcoder.txt`里，格式如下：
&gt;
&gt; ```txt
&gt; Proto Recv-Q Send-Q Local Address           Foreign Address         State
&gt; tcp        0      0 0.0.0.0:6160            0.0.0.0:*               LISTEN
&gt; tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN
&gt; tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
&gt; tcp        0      0 172.16.56.200:41856     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49822     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49674     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:42316     172.16.34.143:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:44076     172.16.240.74:6379      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49656     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58248     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:50108     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41944     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:35548     100.100.32.118:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:39024     100.100.45.106:443      TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41788     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58260     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41812     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41854     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58252     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:49586     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41754     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50466     120.55.222.235:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:38514     100.100.142.5:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:49832     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:52162     100.100.30.25:80        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50372     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50306     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49600     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41908     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:60292     100.100.142.1:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:37650     100.100.54.133:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41938     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49736     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41890     172.16.34.144:3306      ESTABLISHED
&gt; udp        0      0 127.0.0.1:323           0.0.0.0:*
&gt; udp        0      0 0.0.0.0:45881           0.0.0.0:*
&gt; udp        0      0 127.0.0.53:53           0.0.0.0:*
&gt; udp        0      0 172.16.56.200:68        0.0.0.0:*
&gt; udp6       0      0 ::1:323                 :::*
&gt; raw6       0      0 :::58                   :::*                    7
&gt; ```
&gt;
&gt; 现在需要你输出每个IP的连接数，按照连接数降序排序。你的脚本应该输出
&gt;
&gt; ```txt
&gt; 172.16.0.24 10
&gt; 172.16.34.144 9
&gt; 100.100.142.4 3
&gt; 0.0.0.0 3
&gt; 172.16.34.143 1
&gt; 172.16.240.74 1
&gt; 120.55.222.235 1
&gt; 100.100.54.133 1
&gt; 100.100.45.106 1
&gt; 100.100.32.118 1
&gt; 100.100.30.25 1
&gt; 100.100.142.5 1
&gt; 100.100.142.1 1
&gt; ```

```shell
#!/bin/bash

grep &#34;tcp&#34; |
awk &#39;{ print $5 }&#39; |
awk -F &#34;:&#34; &#39;{ print $1 }&#39; |
sort |
uniq -c |
sort -nr |
awk &#39;{ print $2, $1 }&#39;
```

## netstat练习4-输出和3306端口建立连接总的各个状态的数目

&gt; 假设`netstat`命令运行的结果我们存储在`nowcoder.txt`里，格式如下：
&gt;
&gt; ```txt
&gt; Proto Recv-Q Send-Q Local Address           Foreign Address         State
&gt; tcp        0      0 0.0.0.0:6160            0.0.0.0:*               LISTEN
&gt; tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN
&gt; tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
&gt; tcp        0      0 172.16.56.200:41856     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49822     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49674     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:42316     172.16.34.143:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:44076     172.16.240.74:6379      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49656     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58248     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:50108     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41944     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:35548     100.100.32.118:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:39024     100.100.45.106:443      TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41788     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58260     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41812     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41854     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:58252     100.100.142.4:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:49586     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41754     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50466     120.55.222.235:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:38514     100.100.142.5:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:49832     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:52162     100.100.30.25:80        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50372     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:50306     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49600     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41908     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:60292     100.100.142.1:80        TIME_WAIT
&gt; tcp        0      0 172.16.56.200:37650     100.100.54.133:80       TIME_WAIT
&gt; tcp        0      0 172.16.56.200:41938     172.16.34.144:3306      ESTABLISHED
&gt; tcp        0      0 172.16.56.200:49736     172.16.0.24:3306        ESTABLISHED
&gt; tcp        0      0 172.16.56.200:41890     172.16.34.144:3306      ESTABLISHED
&gt; udp        0      0 127.0.0.1:323           0.0.0.0:*
&gt; udp        0      0 0.0.0.0:45881           0.0.0.0:*
&gt; udp        0      0 127.0.0.53:53           0.0.0.0:*
&gt; udp        0      0 172.16.56.200:68        0.0.0.0:*
&gt; udp6       0      0 ::1:323                 :::*
&gt; raw6       0      0 :::58                   :::*                    7
&gt; ```
&gt;
&gt; 现在需要你输出和本机3306端口建立连接的各个状态的数目，按照以下格式输出
&gt; `TOTAL_IP`表示建立连接的ip数目
&gt;
&gt; `TOTAL_LINK`表示建立连接的总数目
&gt;
&gt; ```txt
&gt; TOTAL_IP 3
&gt; ESTABLISHED 20
&gt; TOTAL_LINK 20
&gt; ```

```shell
#!/bin/bash

# 统计包含3306且协议为tcp的总IP数量
TOTAL_IP=$(grep &#34;tcp&#34; nowcoder.txt |
	awk &#39;{ print $5 }&#39; |
	awk -F: &#39;$2 == 3306 {print $1, $2 }&#39; |
	sort |
	uniq |
	wc -l)

echo &#34;TOTAL_IP $TOTAL_IP&#34;

# 统计包含3306且状态为ESTABLISHED且协议为tcp的数量
ESTABLISHED=$(awk &#39;/3306/ { if ($6 == &#34;ESTABLISHED&#34; &amp;&amp; $1 == &#34;tcp&#34;) print $5 }&#39; nowcoder.txt |
	wc -l)

echo &#34;ESTABLISHED $ESTABLISHED&#34;

# 统计包含3306且协议为tcp的连接数量
TOTAL_LINK=$(awk &#39;/3306/ { if ($1 == &#34;tcp&#34;) print $5 }&#39; nowcoder.txt |
	wc -l)

echo &#34;TOTAL_LINK $TOTAL_LINK&#34;
```

## 业务分析-提取值

&gt; 假设我们的日志`nowcoder.txt`里，内容如下
&gt;
&gt; ```txt
&gt; 12-May-2017 10:02:22.789 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Server version:Apache Tomcat/8.5.15
&gt; 12-May-2017 10:02:22.813 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:May 5 2017 11:03:04 UTC
&gt; 12-May-2017 10:02:22.813 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Server number:8.5.15.0
&gt; 12-May-2017 10:02:22.814 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:Windows, OS Version:10
&gt; 12-May-2017 10:02:22.814 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:x86_64
&gt; ```
&gt;
&gt; 现在需要你提取出对应的值，输出内容如下
&gt;
&gt; ```txt
&gt; serverVersion:Apache Tomcat/8.5.15
&gt; serverName:8.5.15.0
&gt; osName:Windows
&gt; osVersion:10
&gt; ```

```shell
#!/bin/bash

grep -o &#34;Server version:.*&#34; nowcoder.txt |
	awk -F &#34;:&#34; &#39;{print &#34;serverVersion:&#34; $2}&#39;
grep -o &#34;Server number:.*&#34; nowcoder.txt |
	awk -F &#34;:&#34; &#39;{print &#34;serverName:&#34; $2}&#39;
grep -o &#34;OS Name:.*&#34; nowcoder.txt |
	awk -F &#34;[:,]&#34; &#39;{print &#34;osName:&#34; $2}&#39;
grep -o &#34;OS Version:.*&#34; nowcoder.txt |
	awk -F &#34;:&#34; &#39;{print &#34;osVersion:&#34; $2}&#39;
```

## ps分析-统计VSZ,RSS各自总和

&gt; 假设命令运行的结果我们存储在`nowcoder.txt`里，格式如下：
&gt;
&gt; ```txt
&gt; USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
&gt; root         1  0.0  0.1  37344  4604 ?        Ss    2020   2:13 /sbin/init
&gt; root       231  0.0  1.5 166576 62740 ?        Ss    2020  15:15 /lib/systemd/systemd-journald
&gt; root       237  0.0  0.0      0     0 ?        S&lt;    2020   2:06 [kworker/0:1H]
&gt; root       259  0.0  0.0  45004  3416 ?        Ss    2020   0:25 /lib/systemd/systemd-udevd
&gt; root       476  0.0  0.0      0     0 ?        S&lt;    2020   0:00 [edac-poller]
&gt; root       588  0.0  0.0 276244  2072 ?        Ssl   2020   9:49 /usr/lib/accountsservice/accounts-daemon
&gt; message&#43;   592  0.0  0.0  42904  3032 ?        Ss    2020   0:01 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
&gt; root       636  0.0  0.0  65532  3200 ?        Ss    2020   1:51 /usr/sbin/sshd -D
&gt; daemon     637  0.0  0.0  26044  2076 ?        Ss    2020   0:00 /usr/sbin/atd -f
&gt; root       639  0.0  0.0  29476  2696 ?        Ss    2020   3:29 /usr/sbin/cron -f
&gt; root       643  0.0  0.0  20748  1992 ?        Ss    2020   0:26 /lib/systemd/systemd-logind
&gt; syslog     645  0.0  0.0 260636  3024 ?        Ssl   2020   3:17 /usr/sbin/rsyslogd -n
&gt; root       686  0.0  0.0 773124  2836 ?        Ssl   2020  26:45 /usr/sbin/nscd
&gt; root       690  0.0  0.0  19472   252 ?        Ss    2020  14:39 /usr/sbin/irqbalance --pid=/var/run/irqbalance.pid
&gt; ntp        692  0.0  0.0  98204   776 ?        Ss    2020  25:18 /usr/sbin/ntpd -p /var/run/ntpd.pid -g -u 108:114
&gt; uuidd      767  0.0  0.0  28624   192 ?        Ss    2020   0:00 /usr/sbin/uuidd --socket-activation
&gt; root       793  0.0  0.0 128812  3148 ?        Ss    2020   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
&gt; www-data   794  0.0  0.2 133376  9120 ?        S     2020 630:57 nginx: worker process
&gt; www-data   795  0.0  0.2 133208  8968 ?        S     2020 633:02 nginx: worker process
&gt; www-data   796  0.0  0.2 133216  9120 ?        S     2020 634:24 nginx: worker process
&gt; www-data   797  0.0  0.2 133228  9148 ?        S     2020 632:56 nginx: worker process
&gt; web        955  0.0  0.0  36856  2112 ?        Ss    2020   0:00 /lib/systemd/systemd --user
&gt; web        956  0.0  0.0  67456  1684 ?        S     2020   0:00 (sd-pam)
&gt; root      1354  0.0  0.0   8172   440 tty1     Ss&#43;   2020   0:00 /sbin/agetty --noclear tty1 linux
&gt; root      1355  0.0  0.0   7988   344 ttyS0    Ss&#43;   2020   0:00 /sbin/agetty --keep-baud 115200 38400 9600 ttyS0 vt220
&gt; root      2513  0.0  0.0      0     0 ?        S    13:07   0:00 [kworker/u4:1]
&gt; root      2587  0.0  0.0      0     0 ?        S    13:13   0:00 [kworker/u4:2]
&gt; root      2642  0.0  0.0      0     0 ?        S    13:17   0:00 [kworker/1:0]
&gt; root      2679  0.0  0.0      0     0 ?        S    13:19   0:00 [kworker/u4:0]
&gt; root      2735  0.0  0.1 102256  7252 ?        Ss   13:24   0:00 sshd: web [priv]
&gt; web       2752  0.0  0.0 102256  3452 ?        R    13:24   0:00 sshd: web@pts/0
&gt; web       2753  0.5  0.1  14716  4708 pts/0    Ss   13:24   0:00 -bash
&gt; web       2767  0.0  0.0  29596  1456 pts/0    R&#43;   13:24   0:00 ps aux
&gt; root     10634  0.0  0.0      0     0 ?        S    Nov16   0:00 [kworker/0:0]
&gt; root     16585  0.0  0.0      0     0 ?        S&lt;    2020   0:00 [bioset]
&gt; root     19526  0.0  0.0      0     0 ?        S    Nov16   0:00 [kworker/1:1]
&gt; root     28460  0.0  0.0      0     0 ?        S    Nov15   0:03 [kworker/0:2]
&gt; root     30685  0.0  0.0  36644  2760 ?        Ss    2020   0:00 /lib/systemd/systemd --user
&gt; root     30692  0.0  0.0  67224  1664 ?        S     2020   0:00 (sd-pam)
&gt; root     32689  0.0  0.0  47740  2100 ?        Ss    2020   0:00 /usr/local/ilogtail/ilogtail
&gt; root     32691  0.2  0.5 256144 23708 ?        Sl    2020 1151:31 /usr/local/ilogtail/ilogtail
&gt; ```
&gt;
&gt; 现在需要你统计`VSZ`，`RSS`各自的总和（以M兆为统计），输出格式如下
&gt;
&gt; ```txt
&gt; MEM TOTAL
&gt; VSZ_SUM:3250.8M,RSS_SUM:179.777M
&gt; ```

```shell
#!/bin/bash

awk &#39;{
    sum_vsz = sum_vsz &#43; $5
    sum_rss = sum_rss &#43; $6
}END{
    print(&#34;MEM TOTAL \n&#34; &#34;VSZ_SUM:&#34; sum_vsz/1024 &#34;M,&#34; &#34;RSS_SUM:&#34; sum_rss/1024 &#34;M&#34;)}&#39;
```



---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/07.shell%E5%AE%9E%E6%88%98/  

