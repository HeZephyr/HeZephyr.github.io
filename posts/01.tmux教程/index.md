# Tmux教程

## tmux使用教程

### tmux安装

```bash
# Ubuntu 或 Debian
$ sudo apt-get install tmux

# CentOS 或 Fedora
$ sudo yum install tmux

# Mac
$ brew install tmux
```

### tmux简介

tmux是一个非常优秀的终端复用器（terminal multiplexer），其可以使用一系列的终端session，它使您可以在一个终端中的多个程序之间轻松切换、分离它们（它们继续在后台运行）并将它们重新附加到不同的终端。

session：我们知道，在windows命令行中我们打开一个终端窗口，然后输入命令实现与计算机的交流。而这种用户与计算机的这种临时的交互就叫做session。

tmux的主要功能有：

* 分屏，即允许在单个窗口中访问多个会话。这对我们同时运行多个命令行程序非常有用。
* 允许断开Terminal连接后，继续运行进程。这样，我们可以随时随地放心的进行操作，即使工作进行到一半突然断开，我们打开tmux仍可以重新进入到操作现场。
* 会话共享（适用于结对编程或者远程教学）。我们可以将tmux的会话地址分享给其他人，这样他们就可以通过SSH接入该会话。更骚的是，如果你要给其他人演示远程服务器的操作，不必一直盯着屏幕，我们可以借助tmux，他就可以完全进入你的会话。

当然，tmux的功能还有很多，相比于其他的终端复用器，tmux更易用也更强大，这也是我们为什么要学习tmux的原因。

### tmux结构

tmux是采用C/S模型构建的，我们输入tmux即相当于开启了一个服务器，此时默认将新建一个session，然后再session中新建一个window，window中新建一个pane。其联系为：一个tmux可以包含多个session，一个session可以包含多个window，一个window可以包含多个pane。

以下即是实例：
        tmux:
            session 0:
                window 0:
                    pane 0
                    pane 1
                    pane 2
                    ...
                window 1
                window 2
                ...
            session 1
            session 2
            ...

![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/a3ea3ebfc8cbe63d5716d61f0057dbae-20231125215213708.png)

### tmux操作

注：以下操作大多数都会给出命令和对应的快捷键，如果用于学习，建议两种操作都要学会，但在平常操作中，使用快捷键是更方便的。

#### 前缀键

tmux提供了很多的快捷键。特别的是所有的快捷键都需要通过前缀键唤起，默认的快捷键为`ctrl &#43; b`。我们可以通过`tmux list-keys`命令来查看所有快捷键指令。

当然如果觉得`ctrl &#43; b`不好用，习惯用其他的键，这里提供修改前缀键的方法如下：

```linux
# 首先打开文件~/.tmux.conf
vim ~/.tmux.conf
# 设置前缀为Ctrl &#43; a。
set -g prefix C-a
# 解除Ctrl &#43; b与前缀的对应关系。
unbind C-b
# 绑定ctrl &#43; a 成为新的指令前缀。
bind C-a send-prefix
#然后让它生效。我们也可以直接重启让它生效。
source ~/.tmux.conf
```

#### 会话(session)操作

* 新建会话：`tmux new -s sessionName`。

	这样，我们就可以创建一个名为sessionName的会话了。当然我们也可以直接简写为`tmux`，这样会新建一个无名称的会话。但为了我们方便管理，所以建议还是指定会话名称。

* 断开当前会话：`tmux detach`。

	这样，我们就可以断开当前会话，但是会话 还是会在后台运行。当然，我们还可以使用快捷键：先按`ctrl &#43; b`，再按 `ctrl &#43; d`即可退出。

	注意：这个并没有真正的关闭会话。

* 进入已存在的会话：`tmux a -t sessionName`或者直接输入`tmux a`。

	前者可以指定进入的sessionName会话，而后者是默认进入第一个会话。其中的`a`命令实际上就是`attach`，我们可以简写为`a`。

* 关闭会话：`tmux kill -session -t sessionName`或者`tmux kill -server`。

	我们可以使用tmux的`kill`命令，其中kill命令有四种：`kill -pane, kill -server, kill -session, kill window`。根据需求我们使用自己需要的命令，需要注意的是，根据前面的联系，我们关闭了`session`，那么其中的所有`pane`和`window`也都不存在了，同理关闭`server`,那么所有`session`也都不存在了。所以这里即是使用`kill -session`和`kill -server`，前者关闭`sessionName`这个会话，后者关闭所有的会话。

* 查看所有的会话：`tmux ls`。

	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20210901141903522.png)

* 切换会话：`switch`命令用于切换会话。

	```linux
	# 使用会话编号来切换
	tmux switch -t 0
	# 使用会话名称来切换
	tmux switch -t sessionName
	```

* 重命名会话：`tmux rename-session -t sessionName  new Name`。

* 常用会话操作快捷键。

	|   前缀   |   指令   |            功能            |
	| :------: | :------: | :------------------------: |
	| `Ctrl&#43;b` |    d     |        断开当前会话        |
	| `Ctrl&#43;b` |    D     |      选择要断开的会话      |
	| `Ctrl&#43;b` |    $     |       重命名当前会话       |
	| `Ctrl&#43;b` |    s     | 列出当前会话用于选择并切换 |
	| `Ctrl&#43;b` |    r     |      强制重载当前会话      |
	| `Ctrl&#43;b` | ctrl &#43; z |        挂起当前会话        |

* tmux中复制/粘贴文本的通用方法。

	(1).按下`Ctrl &#43; b`前缀键，然后按`[`；

	(2).用鼠标选中文本，然后被选中的文本会被自动复制到tmux的剪贴板；

	(3).按下`Ctrl &#43; b`后前缀键，然后按`]`，会自动将剪贴板的内容粘贴到光标处。

#### 窗口(window)操作

* 新建窗口：`tmux new-window -n windowName`。

	如此我们可创建一个新的名为windowName的窗口。当然我们也可以直接输入`tmux new-window`来创建一个没有指定名称的窗口。 创建一个新窗口后，原窗口我们是看不到了的，但依然存在。

* 切换窗口：`tmux select-window -t windowName`。

* 重命名窗口：`tmux rename-window windowName`。

* 常用窗口操作快捷键。

	|   前缀   |     指令     |                 功能                 |
	| :------: | :----------: | :----------------------------------: |
	| `Ctrl&#43;b` |      c       |               新建窗口               |
	| `Ctrl&#43;b` |      &amp;       |  关闭当前窗口，需要输入`y or n`确认  |
	| `Ctrl&#43;b` | &lt;number&gt;编号 |         切换到指定编号的窗口         |
	| `Ctrl&#43;b` |      p       |           切换到上一个窗口           |
	| `Ctrl&#43;b` |      n       |           切换到下一个窗口           |
	| `Ctrl&#43;b` |      w       |        显示窗口列表用于切换。        |
	| `Ctrl&#43;b` |  ,（逗号）   |            重命名当前窗口            |
	| `Ctrl&#43;b` |      .       |  修改当前窗口编号（适用于窗口排序）  |
	| `Ctrl&#43;b` |      f       | 快速定位到窗口（输入关键字匹配窗口） |

#### 面板(pane)操作

* 划分面板。

	注意，此划分是针对当前选中的面板，面板可以无限划分。

	```linux
	# 划分成上下两个面板
	tmux split-window
	#划分成左右两个面板
	tmux split-window -h
	```

	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20210901153031880.png)

* 切换面板。

	`tmux select-pane`命令用来移动光标位置，这个更严谨的说应该是切换光标，因为我们的光标正好可以选中面板，即可以通过鼠标点击选中。当然，前者过于简单且没有逼格，我们可以通过快捷键或者命令来实现。

	```linux
	# 光标切换到上方面板
	tmux select-pane -U
	# 光标切换到下方面板
	tmux select-pane -D
	# 光标切换到左方面板
	tmux select-pane -L
	# 光标切换到右方面板
	tmux select-pane -R
	```

* 交换面板位置。

	`tmux swap-pane`可用于交换面板位置。同切换面板差不多的形式。

	```linux
	# 上移
	tmux swap-pane -U
	# 下移
	tmux swap-pane -D
	# 左移
	tmux swap-pane -L
	# 右移
	tmux swap-pane -R
	```

* 常用面板操作快捷键。

	|   前缀   |           指令           |                             描述                             |
	| :------: | :----------------------: | :----------------------------------------------------------: |
	| `Ctrl&#43;b` |           `&#34;`            |              当前面板上下一分为二，下侧新建面板              |
	| `Ctrl&#43;b` |           `%`            |              当前面板左右一分为二，右侧新建面板              |
	| `Ctrl&#43;b` |           `x`            |          关闭当前面板（关闭前需输入`y` or `n`确认）          |
	| `Ctrl&#43;b` |           `z`            |   最大化当前面板，再重复一次按键后恢复正常（v1.8版本新增）   |
	| `Ctrl&#43;b` |           `!`            | 将当前面板移动到新的窗口打开（原窗口中存在两个及以上面板有效） |
	| `Ctrl&#43;b` |           `;`            |                   切换到最后一次使用的面板                   |
	| `Ctrl&#43;b` |           `q`            |  显示面板编号，在编号消失前输入对应的数字可切换到相应的面板  |
	| `Ctrl&#43;b` |         `方向键`         |                       移动光标切换面板                       |
	| `Ctrl&#43;b` | `hjkl(分别代表左下上右)` |                       移动光标切换面板                       |
	| `Ctrl&#43;b` |           `{`            |                       向前置换当前面板                       |
	| `Ctrl&#43;b` |           `}`            |                       向后置换当前面板                       |
	| `Ctrl&#43;b` |         `Ctrl&#43;o`         |                顺时针旋转当前窗口中的所有面板                |
	| `Ctrl&#43;b` |           `o`            |                         选择下一面板                         |
	| `Ctrl&#43;b` |         `空格键`         |                  在自带的面板布局中循环切换                  |
	| `Ctrl&#43;b` |       `Alt&#43;方向键`       |              以5个单元格为单位调整当前面板边缘               |
	| `Ctrl&#43;b` |      `Ctrl&#43;方向键`       |  以1个单元格为单位调整当前面板边缘（Mac下被系统快捷键覆盖）  |
	| `Ctrl&#43;b` |           `t`            |                           显示时钟                           |

## 参考文献

[Tmux 使用教程-阮一峰](https://www.ruanyifeng.com/blog/2019/10/tmux.html)

[Tmux使用手册-路易斯](http://louiszhai.github.io/2017/09/30/tmux/#%E5%AF%BC%E8%AF%BB)

[tmux使用教程-小可七](https://zhuanlan.zhihu.com/p/98384704)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.tmux%E6%95%99%E7%A8%8B/  

