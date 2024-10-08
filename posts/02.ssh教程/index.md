# SSH教程

## SSH基本知识

### SSH是什么

SSH为Secure Shell的缩写，是一种网络协议，用于加密两台计算机之间的通信，保证不被窃听或篡改，并且支持各种身份验证机制。在事务中，它主要用户保证远程登录贺远程通信的安全，任何网络服务都可以用这个协议来加密。

### SSH架构

SSH的软件架构是C/S模式，即客户端-服务器模式。在这个架构中，SSH软件分成了两个部分：向服务器发出请求的部分，称为客户端，OpenSSH的实现为ssh；接受客户端发出的请求的部分，称为服务器，OpenSSH的实现为sshd。
其中大写的SSH表示协议，小写的ssh表示客户端软件。
OpenSSH的客户端是二进制程序ssh。它在Linux/Unix系统的位置是`/usr/local/bin/ssh`，windows系统的位置是`/Program Files/OpenSSH/bin/ssh.exe`。Linux系统一般都自带ssh，如果没有则需要自己安装。安装命令如下：

```powershell
# Ubuntu 和 Debian
sudo apt install openssh-client
# CentOS 和 Fedora
sudo dnf install openssh-clients
```

## ssh登录

### 基本用法

远程登录服务器有以下几种方式：

```powershell
ssh hostname # 没有指定用户名，将使用客户端的当前用户名
ssh user@hostname # 指定用户名
ssh user@hostname -p 22 # 登录端口为22，使用-p可以登录到某一特定的端口。
```

其中`user`为用户名，`hostname`为IP地址或域名
第一次登录时会提示：

```powershell
The authenticity of host &#39;123.57.47.211 (123.57.47.211)&#39; can&#39;t be established.
ECDSA key fingerprint is SHA256:iy237yysfCe013/l&#43;kpDGfEG9xxHxm0dnxnAbJTPpG8.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

上述这段文字告诉用户，这台服务器的指纹是陌生的，让用户选择是否要继续连接（输入yes或no）。
所谓“服务器指纹”，指的是 SSH 服务器公钥的哈希值。每台 SSH 服务器都有唯一一对密钥，用于跟客户端通信，其中公钥的哈希值就可以用来识别服务器。

输入yes，然后回车即可。这样会将服务器的信息记录在`~/.ssh//known_hosts`文件中，然后输入密码即可登录到远程服务器。
在`~/.ssh/known_hosts`文件中保存了本机连接过的所有服务器的公钥指纹信息，每次通过ssh连接一台服务器，系统会通过该文件判断当前需要连接的服务器是否为陌生主机。而如果之前ssh登陆过的服务器的密钥发生了更改，那么登录就会收到提示`Host key verification failed`，由于发生改变了，所以西我们需要删除之前的主机的指纹信息，添加新的。这个可以使用`ssh-keygen -R &lt;hostname&gt;`来进行删除。

如果我们需要退出登录，输入`logout`或`exit`都可以实现，也可以使用快捷键`Ctrl&#43;d`退出。

### 配置文件

创建文件`~/.ssh/config`，此为用户的个人配置文件，其保存相关配置信息。我们可以在文件中输入：

```powershell
Host myServer1
    HostName IP地址或域名
    User 用户名
Host myServer2
    HostName IP地址或域名
    User 用户名
```

之后再使用服务器时，就可以直接使用别名`myServer1`、`myServer2`进行登录。
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_19%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125215546561.png)
SSH客户端的全局配置文件为`/etc/ssh/ssh_config`，其优先级低于用户个人配置文件。除了配置文件，`~/.ssh`还有一些用户个人的密钥文件和其他文件

### 密钥登录

* **创建密钥**
	首先在本台主机上创建密钥，输入`ssh-keygen`指令，然后一直回车即可。该命令会为本机生成公私钥对，通过密钥来进行远程登录的验证机制使用了密码学中的非对称加密体系。执行命令后，会在`~/.ssh/`文件夹下生成两个新的文件：

	    1. `id_rsa`：私钥
	    2. `id_rsa.pub`：公钥

	  注意，一定要保护好生成的私钥`id_rsa`，一旦暴露，利用上述指令可以重新生成新的密钥。
	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125215546692.png)

* **免密登录**
	如果想要免密登录某个服务器，我们只需要将刚才生成的公钥保存在需要登录的远程服务器`~/.ssh/authorized_keys`文件中。例如我想免密登录到`myServer1`服务器，则将`~/.ssh/id_rsa.pub`公钥中的内容，复制到`myServer1`中的`~/.ssh/authorized_keys`文件中。也可以使用如下命令一键添加公钥：
	`ssh-copy-id myServer1`。
	第一次应用`ssh-copy-id`系统会提示输入远程服务器的密码，之后登录时就可以免密登录了。
	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125215556717.png)

一般来说，应用密钥登录比使用密码登录更加安全，所以启用密钥登录之后，最好关闭服务器的密码登录。如果需要在远程服务器上关闭密码登录。具体方法是先远程登录到服务器中，然后找到sshd的配置文件`/etc/ssh/sshd_config`，最后将`PasswordAuthentication`这一项设置为`no`。
`PasswordAuthentication no`
sshd是服务器运行的后台进程，当我们修改配置文件以后，需要重新启动sshd，然后修改才能生效。可应用下面语句重启远程服务器上的`ssh`和`sshd`服务。

```powershell
sudo systemctl restart ssh.service
sudo systemctl restart sshd.service
```

* **执行远程命令**
	免密登录后，我们可以在命令行下直接执行远程命令：`ssh user@hostname command`。这样的命令会使SSH在登录成功后，立刻在远程主机上执行命令`comand`。但并不是真正的登录，执行完命令后还在原主机。
	测试：`ssh myServer1 ls`
	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/2f5876c7802840d48072d1687b483a4c.png)

## scp传文件

### scp命令简介

`scp`是secure copy的缩写，相当于`cp`命令&#43;SSH。它的底层是SSH协议，默认端口是22，相当于先使用`ssh`命令登录远程主机，然后再进行拷贝操作。
`scp`主要用于以下三种复制操作：

* 本地复制到远程。
* 远程复制到本地。
* 两个远程系统之间的复制。

使用`scp`传输数据时，文件和密码都是加密的，不会泄露敏感信息。

### 基本语法

`scp`的语法类似`cp`的语法。

```powershell
scp source destination
```

上面命令中，`source`是文件当前的位置，`destination`是文件所要复制到的位置。它们都可以包含用户名和主机名。例如：

```powershell
scp myServer1:main.cpp main.cpp
```

上面命令即是将远程主机（myServer1）用户主目录下的`main.cpp`复制为本机当前目录下的`main.cpp`。注意主机与文件之间要使用`:`分隔。
`scp`会先用SSH登录到远程主机，然后在加密连接之中复制文件。客户端发起连接后，会提示用户输入密码。其中用户名和主机名若省略则是代表当前主机下的当前用户名。

`scp`支持一次复制多个文件。`scp source1 source2 destination`。如果存在同名文件，`scp`会在没有警告的情况下覆盖同名文件。

## 参考文献

[Linux基础课](https://www.acwing.com/activity/content/57/)
[菜鸟教程](https://www.runoob.com/)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/02.ssh%E6%95%99%E7%A8%8B/  

