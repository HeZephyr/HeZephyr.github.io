# Git教程

## git简介

### 什么是版本控制

版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。 除了项目源代码，你可以对任何类型的文件进行版本控制。
有了它你就可以将某个文件回溯到之前的状态，甚至将整个项目都回退到过去某个时间点的状态，你可以比较文件的变化细节，查出最后是谁修改了哪个地方，从而找出导致怪异问题出现的原因，又是谁在何时报告了某个功能缺陷等等。
而版本控制系统（VCS）则是一种软件，可以帮助软件团队的开发人员协同工作，并存档他们工作的完整历史记录。
目前版本控制系统有如下三种：

* **本地版本控制系统**：即通过用文件目录形式保存每个项目版本，其中目录名会备注一些版本信息、修改时间等。其最大的好处就是简单，但特别容易犯错，不利于管理，容易覆盖重要的文件，而且不适合协同工作。
	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_19%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png)

* **集中式版本控制系统（Centralized Version Control Systems，CVCS）**：集中化的版本控制系统都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新。这么做虽然解决了本地版本控制系统无法让在不同系统上的开发者协同工作的诟病，但也还是存在下面的问题：如果中央服务器宕机，则其他人无法使用；必须联网才能工作。
	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_19%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125214815381.png)

* **分布式版本控制系统（Distributed Version Control Systems，DVCS）**：在这类系统中，像 Git、Mercurial、Bazaar 以及 Darcs 等，客户端并不只提取最新版本的文件快照， 而是把代码仓库完整地镜像下来，包括完整的历史记录。 这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。 因为每一次的克隆操作，实际上都是一次对代码仓库的完整备份。
	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125214822958.png)
	分布式版本控制系统可以不用联网就可以工作，因为每个人的电脑上都是完整的版本库，当你修改了某个文件后，你只需要将自己的修改推送给别人就可以了。但是，在实际使用分布式版本控制系统的时候，很少会直接进行推送修改，而是使用一台充当“中央服务器”的东西。这个服务器的作用仅仅是用来方便“交换”大家的修改，没有它大家也一样干活，只是交换修改不方便而已。
	分布式版本控制系统的优势不单是不必联网这么简单，后面我们还会看到 Git 极其强大的分支管理等功能。

### 什么是git

Git是一款免费、开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。其是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。
官网地址为：[https://git-scm.com/](https://git-scm.com/)，在上面有权威的git介绍以及git下载地址。
SVN是集中式版本控制系统，而Git是分布式版本控制系统，Git与SVN的区别可参考[Git与SVN的区别](https://blog.csdn.net/ThinkWon/article/details/101449611)。
git有以下优点：

* 适合分布式开发，强调个体；公共服务器压力和数据量都不会太大；

* 速度快、灵活；

* 任意两个开发者之间可以很容易的解决冲突；

* 离线工作。

### git的几个核心概念

* 工作区（workspace）：仓库的目录。工作区是独立于各个分支的。在当前仓库中，新增，更改，删除文件这些动作，都发生在工作区里面。
* 暂存区（index/stage）：数据暂时存放的区域，类似于工作区写入版本库前的缓存区。暂存区是独立于各个分支的。一般存放在 .git 目录下的 index 文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。
* 版本库（Repository）：也可以叫仓库区，实际上就是我们的本地仓库。就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本
* 版本结构：树结构，树中每个节点代表一个代码版本。
* 远程仓库（Remote Repository）： 托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换。目前流行的远程仓库有：Github、Gitee。

### git工作流程

一般工作流程如下：

* 从远程仓库中克隆 Git 资源作为本地仓库；
* 从本地仓库中checkout代码然后进行代码修改；
* 在提交本地仓库前先将代码提交到暂存区；
* 提交修改，提交到本地仓库；本地仓库中保存修改的各个历史版本；
* 在需要和团队成员共享代码时，可以将修改代码push到远程仓库。
	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_8%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png)

## git安装配置

### windows平台安装

主要学linux，这里列出其他blog的windows平台安装教程：[安装教程](https://blog.csdn.net/u013295518/article/details/78746007#3windows%E4%B8%8B%E5%AE%89%E8%A3%85git)

### Linux平台安装

首先，我们可以尝试着输入`git`，看看系统有没有安装Git：

```powershell
$ git
The program &#39;git&#39; is currently not installed. You can install it by typing:
sudo apt-get install git
```

如果出现这个，则说明Git还没有安装，如果我们使用得是Debian或者Ubuntu Linux，通过命令：`sudo apt-get install git`就可以直接完成Git的安装，非常简单。

### Mac平台安装

主要学linux，这里列出其他blog的Mac平台安装配置教程：[安装配置教程](https://blog.csdn.net/u013295518/article/details/78746007#3windows%E4%B8%8B%E5%AE%89%E8%A3%85git)

### 配置

安装好git之后，就需要对git进行配置操作了，需要配置自己的用户名和Email。配置的命令如下：

```shell
$ git config --global user.name &#34;用户名&#34;
$ git config --global user.email &#34;邮箱&#34;
```

如果你需要检查你的配置信息，可以使用`git config --list`命令来列出所有配置信息：
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/de917787862a490fb51e48830453beb9-20231125214837273.png)

## git基本命令

### 建立本地仓库

(1). 创建一个目录，并将其初始化为本地仓库
`git init 本地仓库名`
(2). 使用当前目录作为本地仓库
`git init`
(3). 将远程仓库克隆下来作为本地仓库
此命令支持多种协议，但我一般是通过SSH协议，其内部实现是通过SSH，所以进行这步操作之前我们需要确保在远程仓库添加了SSH公钥，如果没有添加需要在本地主机通过`ssh-keygen`，然后会生成ssh公钥和密钥，我们将公钥添加到远程仓库即可。
`git clone  git@服务器名:仓库路径`
该格式和`scp`命令一致，@前面表示用户名，这个一般都是git，后面表示服务器名，可以是IP地址，也可以是域名，例如`github.com`，`:`后面表示仓库路径。不过不需要担心，这个远程仓库会给出。
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_12%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125214844308.png)

### 提交、修改和删除

(1). 将文件提交到暂存区

```powershell
# 将指定文件添加到暂存区
git add file1 file2 ... fileN 
# 将指定目录添加到暂存区
git add dir1 dir2 ... dirN
# 添加当前目录下的所有文件到暂存区
git add .
```

(2). 将文件从暂存区中撤出，但不会撤销文件的更改
`git restore --staged`
(3). 将不在暂存区的文件撤销更改，需要和(2)作区分，两者作用完全不一样。

```powershell
git restore file
或
git checkout -- file
```

(4). 比较暂存区和工作区文件之间的差异

```powershell
# 显示暂存区和工作区之间的差异
git diff [file]
# 显示暂存区和上一次提交的差异
git diff --cached [file]
或
git diff --staged [file]
# 显示两次提交的差异
 
```

该file可以省略，如果省略，则是比较暂存区中的所有文件。注意，如果暂存区没有内容，则是比较`HEAD`指针指向的版本库内容。
(5). 查看仓库状态
`git status`
 该命令用于查看在你上次提交之后是否有对文件进行再次修改，如果我们需要获取简短的输出结果，可添加`-s`参数来实现。
(6). 提交暂存区内容到本地仓库
暂存区的内容可以通过`git commit`来提交到本地仓库。
`git commit [file1] [file2]...[fileN] -m [备注信息]`
其中file1等是可以直接省略的，如果意味着提交所有信息，而`-m`是参数，后面接备注信息。当然，在进行`git commit`之前，我们需要通过`git add`命令将修改添加到暂存区。
(7). 回退版本
我们可以通过`git reset`命令来回退版本，可以指定退回某一次提交的版本。具体如下：

```powershell
# 将代码库回退到上一个版本
git reset --hard HEAD^ 
或
git reset --hard HEAD~
# 向上回滚两次，一次类推
git reset --hard HEAD^^
# 向上回滚n次。
git rest --hard HEAD~n
# 回滚到某一特定版本，用版本号实现，版本号唯一标识一个版本
git reset --hard 版本号
```

 (8). 删除文件
 在Git中，删除文件也是一种修改操作，删除文件有三种形式：

 &gt; 利用`rm file `实现删除，此形式只是会删除工作区的文件，并没有删除版本库的文件，如果还需要删除版本库的文件还需要执行下列命令，这样就可以实现工作区和版本库的文件：


```powershell
git add file # 加入暂存区
git commit -m  &#34;delete file&#34;
```

&gt;利用`git rm file`实现删除，会删除工作区的文件，并且将此次删除加入暂存区。但需要注意要删除的文件是没有修改过的，如果需要删除修改过的，需要加入`-f`，当然，这个时候我们也还没有删除版本库的文件，只是我们只需要执行`git commit -m &#34;delete file&#34;`就可以。

利用`git rm --cached file`实现删除，只会删除暂存区的文件，但会保留工作区的文件，并且会将这次删除放入暂存区，然后我们执行`git commit -m &#34;delete file&#34;`就可以实现删除暂存区和版本库的文件。
(9). 移动或者重命名文件
`git mv [file] [newFile]`可以用来移动或者重命名一个文件、目录或者软连接。如果新文件名已经存在，还需要添加`-f`参数。

### 查看日志

(1). 查看当前分支的所有版本
`git log`，我们可以用这个命令历史提交记录，当然这个命令还有许多参数供我们使用：

* `--oneline`：查看历史记录的简洁版本。
* `--graph`：查看历史中什么时候出现了分支、合并。开启拓扑图选项。
* `--reverse`：逆向显示所有日志。
* `--author=user`：指定查看user提交的部分。
* `--since、--before、--after`等：指定日期。
	(2). 查看HEAD指针的移动历史（包括被回滚的版本）
	`git reflog`

### 分支管理

几乎每一种版本控制系统都以某种形式支持分支。使用分支意味着你可以从开发主线上分离开来，然后在不影响主线的同时继续工作。
有人把 Git 的分支模型称为必杀技特性，而正是因为它，将 Git 从版本控制系统家族里区分出来。
(1).  创建分支
`git branch (branchName)`
(2). 切换分支
`git checkout (branchName)`
当你切换分支的时候，Git 会用该分支的最后提交的快照替换你的工作目录的内容， 所以多个分支不需要多个目录。
添加`-b`参数可以创建并切换分支。
(3). 合并分支
`git merge (branchName)` 
将branch_name合并到当前分支上。
(4) 删除分支
`git branch -d (branchName)`
删除本地仓库的`branchName`分支。
(5). 列出分支
`git branch`
没有参数时，`git branch`会列出你在本地的分支。
(6). 合并冲突
合并并不仅仅是简单的文件添加、移除的操作。Git也会合并修改，当两个分支对同一个文件都进行了修改，那么就会产生合并冲突，我们需要去手动修改它。然后需要使用`git add`命令来告诉Git冲突已经解决了。

### 远程操作

(1). 将本地仓库关联到远程仓库
`git remote add [short-name] [url]`
其中`short-name`指定一个方便使用的简写，为远程仓库的别名。例如`git remote add origin git@git.acwing.com:unique_pursuit/test.git `即可添加远程仓库。
(2). 查看当前配置有哪些远程库 
`git remote `
执行时添加上`-v`参数可查看到每个别名的实际链接地址。
(3). 删除远程仓库
`git remote rm name`
其中name为仓库的别名。
(4). 修改仓库名
`git remote rename old_name new_name`
(5). 查看主机的详细信息
`git remote   show &lt;主机名&gt;`
(6) 设置本地分支与远程仓库分支对应
`git push --set-upstream &lt;远程主机名&gt; &lt;branchName&gt;`
设置本地的`branchName`分支对应远程仓库的`branchName`分支，远程主机名为`git clone`设置的仓库别名。
`git branch --set-upstream-to=origin/branchName1 branchName2`
将远程仓库的branchName1分支与本地的branchName2分支对应。
(7). 将本地当前分支推送到远程主机
`git push &lt;远程主机名&gt; &lt;本地分支名&gt;:&lt;远程分支名&gt;`
如果本地分支名和远程分支名相同，则可以直接使用下面的命令。
 `git push &lt;远程主机名&gt; &lt;branchName&gt;`
 将本地的`branchName`分支推到远程仓库，在此之前需要先设置与远程仓库对应分支。
 如果当前分支与多个主机存在追踪关系，需要使用`-u`参数指定一个默认主机，这样后面可以不加任何参数使用`git push `。
(8). 删除远程仓库的`branchName`分支
`git push -d &lt;远程主机名&gt; branchName`
(9). 将远程仓库的分支与本地仓库的分支合并
`git pull &lt;远程主机名&gt; &lt;远程分支名&gt;:&lt;本地分支名&gt;`
如果本地分支名和远程分支名相同，则可以使用下面的命令。
`git push &lt;远程分支名&gt; &lt;branchName&gt;`

### 保存和恢复进度

我们有时会遇到这样的情况，正在dev分支开发新功能，做到一半时有人过来反馈一个bug，让马上解决，但是新功能做到了一半你又不想提交，又想保存它，这个时候就可以使用`git stash`命令先把进度保存起来。
(1). 保存当前工作进度
`git stash`
将工作区和暂存区尚未提交的修改存入栈中。再运行`git status`可以发现是一个干净的工作区，没有任何改动。使用`git stash save &#39;message&#39;`可以添加一些注释
(2). 恢复工作进度
恢复最新的进度到工作区。git会默认把工作区和暂存区的改动都恢复到工作区。
`git stash pop`
恢复最新的进度到工作区和暂存区。
`git stash pop --index`
恢复指定的进度到工作区。
`git stash pop stash_id`
  其中stash_id是通过`git stash list`获取的。
  通过`git stash pop`恢复进度后，会删除当前进度。
  还有一个`git stash apply`命令除了不删除进度，其他和`git stash pop`一样。
(3). 显示保存进度的列表
`git stash list`
(4) 删除进度
删除所有进度
`git stash clear`
 删除一个存储的进度，如果不指定stash_id，则默认删除最新的进度。
 `git stash drop [stash_id]`

 ## Git用法思维导图

![image-20231124095105416](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20231124095105416.png)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/02.git%E6%95%99%E7%A8%8B/  

