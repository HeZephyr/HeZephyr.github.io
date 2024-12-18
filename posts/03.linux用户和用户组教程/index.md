# Linux用户和用户组教程

## 用户和用户组介绍

### 用户

任何操作系统都存在“用户”的概念，Linux也不例外。Linux系统是一个多用户多任务的分时操作系统，&lt;font color=&#34;red&#34;&gt;即Linux系统支持多个用户在同一时间内登录，不同用户可以执行不同的任务，并且互不影响。&lt;/font&gt;每个用户账号都拥有一个用户名和各自的口令（即密码）。在登录系统时，只有正确输入用户名和密码，才能进入系统和自己的主目录。

在Linux中，用户分为两大类、三小类：分别为**系统管理员**（一般为root）和**普通用户** 。普通用户中，又划分为两类，分别为系统用户和登录用户。

* **系统管理员**：即超级管理员，可以操作系统中任意文件和命令，拥有最高的管理权限。
* **普通用户**
	* 登录用户：为管理员手动添加的用户，&lt;font color=&#34;red&#34;&gt;默认仅拥有操作自身家目录中文件及目录的权限，以及进入与浏览相关目录文件的权限（如/etc、/var/log等），但没有创建、修改、删除等权限。&lt;/font&gt;
	* 系统用户：一般为系统安装后默认存在的，且默认情况下不能登录系统，它们的存在主要是为了满足系统进程对文件属主的需求。
		Tips：在部署某些服务是，也可以手动添加某些系统用户。



Linux系统使用UID（User ID）来标识不同用户，说白了，其实Linux并不认识你的用户名称，它只认识用户名对应的UID。其中UID是16bit的二进制数字，所以换算成十进制，UID的范围是0~65535，Linux根据用户类别，对UID划分做了规定：

* 0（系统管理员）：，当UID是0时，代表这个用户为超级管理员，所以当你想要其他的用户也有root权限时，将该用户的UID改为0即可。&lt;font color=&#34;red&#34;&gt;但一般来说，用户的UID应当是独一无二的，其他用户不应当有相同的UID数值，只有UID等于0时可以例外。&lt;/font&gt;
* 1~499（系统账号）：该范围内的UID是保留给系统使用的 ID，其实 1~65534 之间的账号并没有不同， 也就是除了 0 之外，其它的 UID 并没有不一样，预设 500 以下给系统作为保留账号只是一个习惯。这样的好处是，以有名的 DNS 服务器的启动服务『 named 』为例，这个程序的预设所有人 named 的账号 UID 是 25 ，当你自定义的账号也是 25 时，会造成系统冲突！为了杜绝这样的问题，养成好习惯，保留 500 以前的 UID 给系统使用！&lt;font color=&#34;red&#34;&gt; 一般来说， 1到99 会保留给系统预设的账号，另外 100~499 则保留给一些服务来使用。&lt;/font&gt;
* 500~65535（登录用户）：给一般使用者用的。事实上，目前的 linux 核心 (2.6.x 版)已经可以支持到 4294967295 (2^32-1) 这么大的 UID 号码。

### 用户组

用户组是具有相同特征用户的逻辑集合。简单的理解，有时我们需要让多个用户具有相同的权限，比如查看、修改某一个文件的权限，一种方法是分别对多个用户进行文件访问授权，如果有 10 个用户的话，就需要授权 10 次，那如果有 100、1000 甚至更多的用户呢？

显然，这种方法不太合理。最好的方式是建立一个组，让这个组具有查看、修改此文件的权限，然后将所有需要访问此文件的用户放入这个组中。那么，所有用户就具有了和组一样的权限，这就是用户组。&lt;font color=&#34;red&#34;&gt;将用户分组是 Linux 系统中对用户进行管理及控制访问权限的一种手段，通过定义用户组，很多程序上简化了对用户的管理工作。&lt;/font&gt;

Linux对用户组也有三种划分方式：

* 第一种组类别

	* 管理员组
	* 普通用户组（包括系统用户组和登录用户组）

* 第二种组类别

	* 用户的基本组（主组）：用户必须有且只能有一个基本组。
	* 用户的附加组  （附属组）：用户可以有0个、1个或多个附加组。

	基本组和附加组就比如，每个人有一个用来安家的房子（基本组），还可以有N个用于投资的房子（附属组）。

* 第三种组类别

	* 私有组：每新建一个用户，如果不指定-g参数，都会自动创建一个和用户名同名的组，且组内只包含用户本身。

	- 公共组：组内可包含多个用户。

Linux系统也是使用GID（Group ID）来标识不同组。用户和用户组的对应关系有以下 4 种：

1. 一对一：一个用户可以存在一个组中，是组中的唯一成员；
2. 一对多：一个用户可以存在多个用户组中，此用户具有这多个组的共同权限；
3. 多对一：多个用户可以存在一个组中，这些用户具有和组相同的权限；
4. 多对多：多个用户可以存在多个组中，也就是以上 3 种关系的扩展。

下图形象的表示了用户和用户组的4种对应关系。
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/2ab64ef22dfa44d29b2760570e2a8870-20231125214644832.png)

&lt;font color=&#34;red&#34;&gt;注意：每一个用户组也有一个口令，当我们将用户添加到指定组时需要该用户组的密码。&lt;/font&gt;

### 文件权限

在Linux操作系统中，任何文件都归属于某一特定的用户，其作为多用户系统，如何区分不同用户对文件的权限成了不可避免的问题。例如，小 A 希望个人文件不被其他用户读取，而如果不对文件进行权限设置，共享了主机资源的小 B 也可以读取小 A 的个人文件，这是不合理的，不同用户对不同文件所拥有的权限应该不尽相同。

因此，Linux 以 “用户与用户组” 的概念，建立用户与文件权限之间的联系，保证系统能够充分考虑每个用户的隐私保护，很大程度上保障了 Linux 作为多用户系统的可行性。&lt;font color=&#34;red&#34;&gt;从文件权限的角度出发，“用户与用户组” 引申为三个具体的对象——**文件所有者**、**用户组成员**、**其他人**。每一个对象对某一个文件的持有权限是不同的。&lt;/font&gt;

#### 文件所有者（User）

当一个用户创建了一个文件，这个用户就是这个文件的文件所有者。文件所有者对文件拥有最高权限，除非文件所有者开放权限，否则其他人无法对文件执行查看、修改等操作。**这也是 Linux 系统能够保护用户隐私的最关键的原因。**在文件所有者占有文件之后，需要文件所有者对其他用户开放权限，其他用户才能查看、修改文件。

如果仅区分 “文件所有者” 和 “其他用户”，那么文件所有者对其他用户开放权限后，所有其他用户均能查看、修改文件。但是，若文件所有者希望仅对部分用户开放，那么仅仅区分 “用户所有者” 和 “其他用户” 显然不满足需求。这就引入了 “用户组的概念”。

#### 用户组成员（Group）

将 “其他用户” 区分为用户组成员和其他人后，若文件所有者希望对部分用户开放权限，而对其他人继续保持私有，则只需要将这部分用户与文件所有者划入一个用户组。这样，这部分用户就成了与文件所有者同组的用户组成员。&lt;font color=&#34;red&#34;&gt;用户可以对用户组成员开放文件权限，用户组成员则具备了查看、修改文件的权限，而对其他无关用户保持私有。&lt;/font&gt;

用户组成员在团队开发中非常有帮助。例如，团队成员之间保持文件资源共享，但对非团队成员保持私有，这就需要将文件所有者与团队成员用户划分为同一个用户组，再对用户组成员开放权限即可。

需要注意的是，**一个用户可在多个用户组中**。

#### 其他人（Others）

顾名思义，就是与文件所有者没有任何联系的用户，即不是文件所有者也不是所在文件所属用户组。

### 超级管理员（root）

由于Linux系统中，root具有最高权限，可以执行任何想要执行的操作，也正因为如此，处于安全考虑，一般情况下不推荐使用 root 用户进行日常使用。root 用户所在的用户组称为 “root组”，处于 root 组的普通用户，能够通过 sudo 命令获取 root 权限，即我们是可以通过sudo权限来操作文件的。

### AAA基础

AAA指的是Authentication、Authorization、Accounting，即认证、授权和审计。

- 认证：验证用户是否可以获得权限，是AAA的第一步，即验证身份；
- 授权：授权用户可以使用那些服务或资源，即身份验证成功后，赋予这个身份相应的权限；
- 审计：记录用户的操作情况，在Linux中，日志就是审计的一种手段。

Linux的用户和组管理可以说是基于AAA进行的，首先用户登录输入用户名密码，就是认证的过程；其次，在用户登录成功后，所拥有的权限各不相同，这就是

授权；最后，用户的操作历史会记录在日志中，这是审计。



## 用户和用户组文件

### 用户账号文件— /etc/passwd

`/etc/passwd`文件是Linux系统安全的关键文件之一，只有系统管理员才可以修改此文件。该文件用于用户登录等操作时校验用户的登录名、加密的口令数据项、用户ID（UID）、默认的用户组ID（GID）、用户信息、用户主目录及登录后使用的shell。该文件种每一行保存一个用户的资料，而用户数据按域以冒号&#39;:&#39;分割。格式如下。

```
username:password:uid:gid:userinfo:home:shell
```

具体含义如表所示。

|    域    |                         含义                         |
| :------: | :--------------------------------------------------: |
| username |                        登录名                        |
| password |                    加密的用户口令                    |
|   uid    |                        用户ID                        |
|   gid    |                       用户组ID                       |
| userinfo |                       用户信息                       |
|   home   |                  分配给用户的主目录                  |
|  shell   | 用户登录后将执行的shell（若为空格泽默认为“/bin/sh”） |

其中关于用户主目录，每个用户都需要保存专属于自己的配置文件及其他文档，这是以免用户间相互干扰。除root账户外（root账户的主目录为&#34;/root&#34;），大多数Linux默认将用户主目录安置在&#34;/home&#34;目录下，并把每个用户的主目录命名为其上机使用的登录名。

![image-20220503204800893](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/842dc2752e8996482acbbf0c83c50545.png)

如图，acs的登录主目录为&#34;/home/acs&#34;。通常，“~”被指向当前用户的登录子目录。

&lt;font color=&#34;red&#34;&gt;注意：用户主目录被安排在“/home”下完全是认为决定的。系统并不关心我们到底把用户主目录安排在什么地方，因为每个用户的位置是在账号文件中定义说明的。所以，用户可以自行调整，灵活使用&lt;/font&gt;

关于shell，Shell是用户与Linux系统之间的接口。Linux的Shell有许多种，每种都有不同的特点。当用户登录进入系统时，会启动一个Shell程序，默认是bash。系统管理员可以根据系统情况和用户习惯为用户指定某个Shell。如果不指定Shell，那么系统使用bash为默认的登录Shell，即这个字段的值为/bin/bash。用户的登录Shell也可以指定为某个特定的程序（此程序不是一个命令解释器）。



我们通过查看`/etc/passwd`文件，可以得到如下完整的系统账号文件。

![image-20220503205339447](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/62794c219ba89bdea97a4871ac63f61e.png)

我们发现，第二列都为x。我们可以继续往下看。

### 用户影子文件—/etc/shadow

实际上，Linxu使用不可逆的加密算法（如MD5）来加密口令，由于加密算法是不可逆的，所以黑客从密文是得不到明文的。但`/etc/passwd`文件时全局可读的，且加密的算法是公开的，如果在passwd中显示密文，黑客据此可以破解口令。Linux系统目前广泛采用了“shadow（影子）文件”机制。将加密的口令转移到“/etc/shadow”文件中。`/etc/shadow`文件只为root超级用户可读，而相应的`etc/passwd`文件的密文域泽显示为一个x，从而最大限度地减少了密文泄露的机会。x表示该账户需要密码才能登录，为空时，账户无须密码即可登录。

和`/etc/passwd`类似，`/etc/shadow`文件中每条记录用冒号“：”分隔，形成9个域，格式如下。

```
username:password:lastchg:min:max:warn:inactive:expire:flag
```

|    域    |                         含义                          |
| :------: | :---------------------------------------------------: |
| username |                      用户登陆名                       |
| password |                    加密的用户口令                     |
| lastchg  |    表示从1970年1月1日起到上次修改口令所经过的天数     |
|   min    |          表示两次修改口令之间至少经过的天数           |
|   max    | 表示口令还会有效的最大天数，如果是99999则表示永不过期 |
|   warn   |       表示口令失效前多少天内系统向用户发出警告        |
| inactive |           表示禁止登录前用户名还有效的天数            |
|  expire  |               表示用户被禁止登录的时间                |
|   flag   |                   保留域，暂未使用                    |

下图为系统中实际影子文件的例子。

![image-20220503210639011](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/6d4cb873977925e048a456514530a915-20231125214650485.png)

### 创建用户的默认设置文件—/etc/login.defs

/etc/login.defs 文件用于在创建用户时，对用户的一些基本属性做默认设置，例如指定用户 UID 和 GID 的范围，用户的过期时间，密码的最大长度，等等。

&lt;font color=&#34;red&#34;&gt;需要注意的是，该文件的用户默认配置对 root 用户无效。并且，当此文件中的配置与 /etc/passwd 和 /etc/shadow 文件中的用户信息有冲突时，系统会以/etc/passwd 和 /etc/shadow 为准。&lt;/font&gt;

其中设置项含义如下表所示。

|          设置项          |                             含义                             |
| :----------------------: | :----------------------------------------------------------: |
| MAIL_DIR /var/spool/mail | 创建用户时，系统会在目录 /var/spool/mail 中创建一个用户邮箱，比如 lamp 用户的邮箱是 /var/spool/mail/lamp。 |
|   PASS_MAX_DAYS 99999    | 密码有效期，99999 是自 1970 年 1 月 1 日起密码有效的天数，相当于 273 年，可理解为密码始终有效。 |
|     PASS_MIN_DAYS 0      | 表示自上次修改密码以来，最少隔多少天后用户才能再次修改密码，默认值是 0。 |
|      PASS_MIN_LEN 5      | 指定密码的最小长度，默认不小于 5 位，但是现在用户登录时验证已经被 PAM 模块取代，所以这个选项并不生效。 |
|     PASS_WARN_AGE 7      | 指定在密码到期前多少天，系统就开始通过用户密码即将到期，默认为 7 天。 |
|       UID_MIN 500        | 指定最小 UID 为 500，也就是说，添加用户时，默认 UID 从 500 开始。注意，如果手工指定了一个用户的 UID 是 550，那么下一个创建的用户的 UID 就会从 551 开始，哪怕 500~549 之间的 UID 没有使用。 |
|      UID_MAX 60000       |                指定用户最大的 UID 为 60000。                 |
|       GID_MIN 500        | 指定最小 GID 为 500，也就是在添加组时，组的 GID 从 500 开始。 |
|      GID_MAX 60000       |                   用户 GID 最大为 60000。                    |
|     CREATE_HOME yes      | 指定在创建用户时，是否同时创建用户主目录，yes 表示创建，no 则不创建，默认是 yes。 |
|        UMASK 077         |               用户主目录的权限默认设置为 077。               |
|   USERGROUPS_ENAB yes    | 指定删除用户的时候是否同时删除用户组，准备地说，这里指的是删除用户的初始组，此项的默认值为 yes。 |
|  ENCRYPT_METHOD SHA512   | 指定用户密码采用的加密规则，默认采用 SHA512，这是新的密码加密模式，原先的 Linux 只能用 DES 或 MD5 加密。 |

如果我们想修改默认配置即可修改配置项的值即可。

### 用户组账号文件—/etc/group

我们知道，`/etc/passwd`文件中包含着每个用户的用户组ID（GID），但如果我们需要找一个用户组中的所有用户，通过`/etc/passwd`难免有些复杂，需要从头到尾寻找同组用户。而`/etc/group`文件包含关于用户组信息，GID被映射到用户分组的名称及同一分组中的其他成员，这样找同组用户以及配置用户组就方便了许多。`/etc/group`文件对用户组的许可权限的控制并不是必要的，这是因为Linux系统用来自于`/etc/passwd`文件的UID、GID来决定文件存取权限。即使`/etc/group`文件不存在于系统中，具有相同的GID用户也能以用户组的许可权限共享文件。

`/etc/group`文件记录格式如下。

```
group_name:group_password:group_id:group_members
```

其中，各个域的含义如下表。

| 域             | 含义                     |
| -------------- | ------------------------ |
| group_name     | 用户组名                 |
| group_password | 加密后的用户组口令       |
| group_id       | 用户组ID（GID）          |
| group_members  | 以逗号分隔的成员用户清单 |

以下是一个`/etc/group`文件的实例。

![image-20220503212923479](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/ea98544d0c79f65fa03cec6026358ae3-20231125214657037.png)

### 用户组影子文件—/etc/gshadow

和用户账号文件passwd一样，为了应对黑客对其进行的暴力攻击，用户组文件也采用一种将组口令与组的其他信息相分离的安全机制——gshadow。`/etc/shadow`文件记录格式如下。

```
group_name:group_password:group_members
```

其中，各个域的含义如下表。

|       域       |           含义           |
| :------------: | :----------------------: |
|   group_name   |         用户组名         |
| group_password |    加密后的用户组口令    |
| group_members  | 以逗号分隔的成员用户清单 |

## 用户和用户组管理

### 用户管理

#### 使用useradd命令添加用户

Linux使用useradd命令添加用户或更新新创建用户的默认信息。其命令格式如下。

```
useradd [option] username
```

可使用的选项如下：

* `-c comment`：描述新用户账号，通常为用户全名。
* `-d home_dir`：设置用户主目录。默认值为用户的登录名，并放在&#34;/home&#34;目录下。
* `-g group`：指定用户所属的基本组。
* `-G group`：指定用户所属的附加组。
* `-u uid`：设置用户的ID。
* `-s shell类型`：设定用户使用的登录shell类型。
* `-k dir`：设置框架目录，创建用户时该目录下的文件都被复制到主目录。
* `-e expire_date`：设置账号过期时间。
* `-f inactivity`：设置口令失效时间。
* `-n`：不为用户创建私有用户组。
* `-p password`：为新建用户指定登录密码。此处的 password 是对应登录密码经 MD5 加密后所得到的密码值，不是真实密码原文，因此在实际应用中，该参数选项使用较少，通常单独使用 passwd 命令来为用户设置登录密码。
* `-r`：创建一个用户 ID 小于 500 的系统账户，默认不创建对应的主目录。
* `-m`：若主目录不存在，则创建它。通常与`-r`结合，可为系统用户主目录。
* `-M`：不创建主目录。

实例1：创建一个普通用户，名为hzf，其中uid为6666，用户主目录指定在/hzf/。

```shell
useradd -u 6666 -d /hzf/ hzf
```

实例2：创建一个系统账户，名为mysystem，其中为系统用户创建主目录，指定密码为12345678

&gt;首先需要加密密码串，这里使用md5sum工具。
&gt;
&gt;利用md5sum加密字符串的方法
&gt;
&gt;\# md5sum     //然后回车
&gt;
&gt;12345678      //输入12345678，然后按两次ctrl&#43;d。
&gt;
&gt;这个时候就会得到一串密文。
&gt;
&gt;使用以下命令即可创建该系统用户
&gt;
&gt;```
&gt;useradd -r -m -p 1234567825d55ad283aa400af464c76d713c07ad mysystem
&gt;```

查看`/etc/passwd`即可看到我们创建得用户信息。

![image-20220504142637032](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/99982b32e60cffbb552501758e033046.png)

#### 使用usermod命令修改用户信息

对于已创建好的用户，可使用 usermod 命令来修改和设置账户的各项属性，包括登录名，主目录，用户组，登录 shell 等，该命令格式如下。

```
usermod [option] username
```

常用的选项包括`-c, -d, -m, -g, -G, -s, -u以及-o等`，这些选项的意义与`useradd`命令中的选项一样，可以为用户指定新的资源值。

另外，有些系统还可以使用`-l`修改用户名。用法为：`usermod -l newusername`

使用`-L`可以锁定用户账号，临时禁止用户登录。用法为：`usermod -L username`，Linux锁定用户，是通过在密码文件 shadow 的密码字段前加 “！” 来标识该用户被锁定。

但如果我们是用root用户登录，再用`su`命令切换到被锁定的账号是可以进去的。

使用`-U`可以解锁用户账号。用法为：`usermod -U username`。

#### 使用userdel命令删除用户

要删除用户，可以使用userdel命令删除，命令格式如下。

```
userdel [-r] username
```

其中`-r`参数可选，若带上参数，表示在删除账户的同时，一并删除用户的主目录。

#### 使用passwd命令管理用户口令

用户管理的一项重要内容是用户口令的管理。用户账号刚创建时没有口令，但是被系统锁定，无法使用，必须为其指定口令后才可以使用，即使是指定空口令。

指定和修改用户口令的Shell命令是`passwd`。超级用户可以为自己和其他用户指定口令，普通用户只能用它修改自己的口令。命令的格式如下。

```
passwd [option] username
```

可使用的选项如下：

* `-l`：锁定口令，即禁用账号。
* `-u`：口令解锁。
* `-d`：使账号无口令。这样，下次登录的时候，系统就不再允许该用户登录了。
* `-f`：强迫用户下次登陆时修改口令。

如果不指定用户名，则表示修改自己的口令。

### 用户组管理

#### 使用groupadd命令创建用户组

增加一个新的用户组使用groupadd命令。命令格式如下。

```shell
groupadd [option] groupname
```

可使用的选项如下：

* `-r`：表示创建系统用户组，该类用户组的 GID 值小于 500；若没有 - r 参数，则创建普通用户组，其 GID 值大于或等于 500。
* `-g gid`：指定新用户组的标识号GID。
* `-o`：一般与-g选项同时使用，表示新用户组的GID可以与系统已有用户组的GID相同。

如果没有指定选择参数，新组的组标识号是在当前已有的最大组标识号的基础上加1。

#### 使用groupmod修改

修改用户组的属性使用groupmod命令。命令格式如下。

```shell
groupmod [option] groupname
```

可使用的选项如下：

- `-g gid`：为用户组指定新的组标识号。
- `-o`：与-g选项同时使用，用户组的新GID可以与系统已有用户组的GID相同。
- `-n newgroupname`： 将用户组的名字改为新名字

#### 使用groupdel命令删除用户组

使用groupdel命令可以删除用户组。命令格式如下。

```shell
groupdel groupname
```

&lt;font color=&#34;red&#34;&gt;在删除用户组时，被删除的用户组不能是某个账户的私有用户组，否则将无法删除，若要删除，则应先删除引用该私有用户组的账户，然后再删除用户组。&lt;/font&gt;

#### 使用gpasswd命令管理用户组

为了避免系统管理员（root）太忙碌，无法及时管理群组，我们可以使用 gpasswd 命令给群组设置一个群组管理员，代替 root 完成将用户加入或移出群组的操作。gpasswd命令格式如下。

```shell
gpasswd [option] groupname
```

可选择的选项如下：

|     选项     |                             功能                             |
| :----------: | :----------------------------------------------------------: |
|              |      选项为空时，表示给群组设置密码，仅 root 用户可用。      |
| -A user1,... | 将群组的控制权交给 user1,... 等用户管理，也就是说，设置 user1,... 等用户为群组的管理员，仅 root 用户可用。 |
| -M user1,... |       将 user1,... 加入到此群组中，仅 root 用户可用。        |
|      -r      |              移除群组的密码，仅 root 用户可用。              |
|      -R      |             让群组的密码失效，仅 root 用户可用。             |
|   -a user    |                  将 user 用户加入到群组中。                  |
|   -d user    |                  将 user 用户从群组中移除。                  |

实例如下。

![image-20220504165806305](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/bba82a11cf576dfbe9b1b405c7fa91ab-20231125214704213.png)

#### 使用newgrp命令切换用户的有效组

我们知道，每个用户可以属于一个初始组（用户是这个组的初始用户），也可以属于多个附加组（用户是这个组的附加用户）。既然用户可以属于这么多用户组，那么用户在创建文件后，默认生效的组身份是哪个呢？

当然是初始用户组的组身份生效，因为初始组是用户一旦登陆就获得的组身份。也就是说，用户的有效组默认是初始组，因此所创建文件的属组是用户的初始组。那么，既然用户属于多个用户组，能不能改变用户的初始组呢？使用命令 newgrp 就可以。此命令基本格式如下。

```shell
newgrp groupname
```

newgrp 命令可以从用户的附加组中选择一个群组，作为用户新的初始组。



## 其他相关命令

### id命令查看用户的UID和GID

id 命令可以查询用户的UID、GID 和附加组的信息。命令比较简单，格式如下。

```shell
id username
```

实例如下：

![image-20220504161152456](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/e7235be54ea1e48c59c3689b2856f75c.png)

### su命令临时切换用户身份

su 是最简单的用户切换命令，通过该命令可以实现任何身份的切换，包括从普通用户切换为 root 用户、从 root 用户切换为普通用户以及普通用户之间的切换。

&lt;font color=&#34;red&#34;&gt;普通用户之间切换以及普通用户切换至 root 用户，都需要知晓对方的密码，只有正确输入密码，才能实现切换；从 root 用户切换至其他用户，无需知晓对方密码，直接可切换成功。&lt;/font&gt;

su命令格式如下。

```shell
su [option] username
```

可使用的选项如下：

* `-l或-`：带这个参数就好像是重新 login 为该使用者一样，大部份环境参数都是以该使用者为主，并且工作目录也会改变，如果没有指定 USER ，内定是 root。
* `-c &lt;command&gt;` ：仅切换用户执行一次命令，执行后自动切换回来，该选项后通常会带有要执行的命令。
* `-s &lt;shell&gt;`： 指定要执行的 shell （bash csh tcsh 等），预设值为 /etc/passwd 内的该使用者shell
* `-h`：显示说明文件。
* `-V`：显示版本资讯。
* `-m或-p`：执行su时不改变工作环境。

![image-20220504162626296](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/8015a145aca6fb66062f3a9d8973ed72.png)

### whoami和who am i命令

whoami 命令和 who am i 命令是不同的 2 个命令，前者用来打印当前执行操作的用户名，后者则用来打印登陆当前 Linux 系统的用户名。

我们可以看一下操作实例来感受区别：

![image-20220504163557031](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/608b588d94af27af4ad729454707411b.png)

在未切换用户身份之前，whoami 和 who am i 命令的输出是一样的，但使用 su 命令切换用户身份后，使用 whoami 命令打印的是切换后的用户名，而 who am i 命令打印的仍旧是登陆系统时所用的用户名。

### users和groups命令

users命令格式如下。

```shell
users [option]
```

* 如果没有参数，则显示当前登录系统的所有用户的用户列表。每个显示的用户名对应一个登录会话。如果一个用户有不止一个登录会话，那他的用户名将显示相同的次数。
* `--help`：显示命令的帮助信息。
* `--version`：显示命令的版本信息。

groups命令格式如下。

```shell
groups [option] [groupname]
```

* 如果没有参数，查看当前登录用户的组内成员。如果指定了groupname，则显示该group的成员。
* `--help`：显示命令的帮助信息。
* `--version`：显示命令的版本信息。

## 高级操作示例

### 通过更改用户和组的配置文件，直接添加或修改用户和组

为了更深入了解用户和组的相关配置文件，可以手动更改配置文件以达到命令的执行效果。

首先我们需要了解Linux中的/etc/skel目录。skel是skeleton的缩写，意为骨骼、框架。故此目录的作用是在建立新用户时，用于初始化用户根目录。系统会将此目录下的所有文件、目录都复制到新建用户的主目录，并且将用户属主与用户组调整为与此主目录相同。所以可将用户配置文件预置到/etc/skel目录下，比如说.bashrc、.profile与.vimrc等。

注意：

* 如果在新建用户时，没有自动建立用户根目录，则无法调用到此框架目录。
* 如果不想以默认的/etc/skel目录作为框架目录，可以在运行useradd命令时使用`-k`指定新的框架目录。
* 如果不想在每次新建用户时，都重新指定新的框架目录，可以通过修改/etc/default/useradd配置文件来改变默认的框架目录。修改SKEL变量的值即可。原来为`SKEL=/etc/skel`。

实际操作步骤如下：

1. 编辑/etc/group文件，添加组test，其中GID为1500。

	`echo &#39;test:x:1500&#39; &gt;&gt; /etc/group`

2. 创建用户的主目录。

	我们需要将框架目录中的文件放到主目录中。同时还需要修改好主目录对其他用户都没有任何访问权限。

	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/3d9ce011a1404b37a0688da4860e9456.png)


3. 编辑/etc/passwd文件，添加用户test，UID为1500，其中基本组ID为test组的GID，其家目录为/home/test。

	`echo &#39;test:x:1500:1500::/home/test:/bin/bash&#39; &gt;&gt; /etc/passwd`

4. 修改/home/test目录及其内部所有文件的属主为test，属组为test。

	![image-20220504191453653](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/505e991a41cc9c3b49b6fb662b94eae5.png)

5. 修改test用户的密码并尝试登录。

	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/699e451be8c44c7c8b38179abae26fec.png)


### Linux批量添加用户



我们可以使用useradd&#43;passwd命令配合shell脚本来实现该功能。

首先我们将需要创建的用户名写入一个文本文件，其中每行代表一个用户名：

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/c197c4ecea8d4a52815a3671c2db21e4.png)


然后实际上我们的思路就是提取出文件中的用户名然后自动执行useradd命令，再执行passwd自动填入初始密码。编写的shell脚本文件如下：

```shell
#! /bin/bash

for username in $(more username.txt)
do
if [ -n $username ]
then
        useradd -m $username # 执行useradd命令
        echo $username&#34;123456&#34; | passwd --stdin $username
        echo &#34;User $username&#39;s password is changed!&#34;
else
        echo &#34;The username is null!&#34;
fi
done
```


![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/34bdf7a7e0714f0aa3ce0e831f11a6eb.png)

测试登录，登录成功！

![image-20220504193147760](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/a825fa4fb9da3a81cf4bf2cf6eeea224-20231125214638090.png)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/03.linux%E7%94%A8%E6%88%B7%E5%92%8C%E7%94%A8%E6%88%B7%E7%BB%84%E6%95%99%E7%A8%8B/  

