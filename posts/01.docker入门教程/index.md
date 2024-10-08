# Docker入门教程


## 初识Docker

### Docker开源项目

Docker是基于Go语言实现的云开源项目，诞生于2013年初，用于支持创建和使用 Linux容器。它的主要目标是“Build, Ship and Run Any App, Anywhere”，即通过对应用封装（Packaging）、分发（Deployment）、运行（Runtime）等生命管理，达到应用组件级别的 &lt;font color=&#34;red&#34;&gt;“一次封装、到处运行”&lt;/font&gt; 。这里的应用组件既可以是一个应用，也可以是一套数据库服务，甚至是一个操作系统或编译器。

### Linux容器技术

Docker引擎的基础是Linux容器（Linux Containers，LXC）技术。容器则是有效地将由单个操作系统管理的资源划分到孤立的组中，以便更好地在孤立间平衡有冲突的资源使用需求，而LXC项目则是借助容器设计理念，并基于一系列新的内核特性实现了更具有扩展性的虚拟化容器方案。更关键的是，LXC被集成到了主流Linux内核中，进而成为Linux系统轻量级容器技术的事实标准。
那么在LXC的基础上，Docker进一步优化了容器的使用体验。Docker提供了各种容器管理工具（如分发、版本、移植等）让用户无需关注底层的操作，可以简单明了管理和使用容器。用户操作Docker容器就像操作一个轻量级的虚拟机那样简单。

我们可以将Docker容器理解为一种沙盒，每个容器运行一个应用，不同的容器相互隔离，容器之间也可以建立通信机制。容器的创建和停止都十分迅速，容器自身对资源的需求也十分有限，远远低于虚拟机。很多时候，甚至将容器比作是应用本身也没有问题。

### 为什么要使用Docker

* **Docker容器虚拟化**
	举个简单的应用场景的例子，假设用户试图基于最常用的LAMP（Linux&#43;Apache&#43;MySQL&#43;PHP）组合来运维一个网站。按照最传统的做法，首先需要安装Apache、Mysql和PHP以及它们各自运行所依赖的环境；之后分别对它们进行配置。经过大量操作后，然后还需要进行功能测试，看是否工作正常；如果不正常，则意味着更多的时间代价和不可控的风险。可以想象，如果再加上更多的应用，事情会变得更加难以处理。更为可怕的是，如果需要进行服务器迁移，如从腾讯云迁移到阿里云，往往需要重新部署和调试。&lt;font color=&#34;red&#34;&gt;而Docker提供了一种更为聪明的方式，通过容器来打包应用，意味着迁移只需要再服务器上启动需要的容器就可以了，节约大量的宝贵时间，并降低部署过程中出现问题的风险。&lt;/font&gt;
* **Docker在开发和运维的优势**
	Docker可以在任何环境、任意时间让应用正常运行。在开发和运维中有4大优势：更快速的交付和部署；更高效的资源利用；更轻松的迁移和扩展；更简单的更新管理。
* **Docker与虚拟机的比较**

| 特性       | 容器               | 虚拟机       |
| ---------- | ------------------ | ------------ |
| 启动速度   | 秒级               | 分钟级       |
| 硬盘使用   | 一般为MB           | 一般为GB     |
| 性能       | 接近原生           | 弱于         |
| 系统支持量 | 单机支持上千个容器 | 一般为几十个 |
| 隔离性     | 安全隔离           | 完全隔离     |

### Docker的核心概念

Docker有三大核心概念，如果我们理解了这三个核心概念，就能顺利理解Docker的整个生命周期。

* **镜像（Image）**
	Docker镜像（Image）类似于虚拟机镜像，可以理解为一个面向Docker引擎的只读模板，包含了文件系统。例如一个镜像可以只包含一个完整的Ubuntu操作系统环境，可以把它称为一个Ubuntu镜像。镜像也可以安装了Apache应用程序，可以把它称为一个Apache镜像。
	镜像是创建Docker容器的基础。通过版本管理和增量的文件系统，Docker提供了一套十分简单的机制来创建和更新现有的镜像，用户甚至可以从网上下载一个已经做好的应用镜像，并通过简单的命令就可以直接使用。
* **容器（Container）**
	Docker容器（Container）类似于一个轻量级的沙箱，Docker利用容器来运行和隔离应用。容器是从镜像创建的应用运行实例，可以将其启动、开始、停止、删除，而这些容器都是相互隔离、互不可见的。&lt;font color=&#34;red&#34;&gt;我们实际是可以将容器看作是一个简易版的Linux系统环境（包括root用户权限、进程空间、用户空间和网络空间等），以及运行在其中的应用程序打包而成的应用盒子。&lt;/font&gt;
	镜像自身是只读的。容器从镜像启动的时候，Docker会在镜像的最上层创建一个可写层，镜像本身将保持不变。如果认为虚拟机是模拟运行的一整套操作系统（提供了运行态环境和其他系统环境）和跑在上面的应用，那么Docker容器就是独立运行的一个或一组应用，以及它们的必需运行环境。
* **仓库（Repository）**
	Docker仓库（Repository）类似于代码仓库，是Docker集中存放镜像文件的场所。而注册服务器（Registry）是存放仓库的地方，其上往往存放着多个仓库。每个仓库集中存放某一类镜像，往往包括多个镜像文件，通过不同的标签（tag）来进行区分。例如存放Ubuntu操作系统镜像的仓库，称为Ubuntu仓库，其中可能包括20.04等不同版本的镜像。根据所存储的镜像公开分享与否，Docker仓库可以分为公开仓库（Public）和私有仓库（Private）两种形式。目前，最大的公开仓库是Docker Hub，存放着数量庞大的镜像供用户下载，国内的公开仓库包括Docker Pool等，可提供稳定的国内访问。
	当然，用户如果不希望公开分享自己的镜像文件，Docker也支持用户在本地网络内创建一个只能自己访问的私有仓库。当用户创建了自己的镜像之后就可以使用push命令将它上传到指定的公有或者私有仓库。这样用户下次在另外一台机器上使用该镜像时，只需将其从仓库上pull下来就可以了。

### Docker安装（Linux）

[安装教程](https://blog.csdn.net/hzf0701/article/details/122810614?spm=1001.2014.3001.5501)


## 镜像具体操作

### 获取镜像

镜像是Docker运行容器的前提。使用`docker pull`命令即可从镜像仓库上下载指定镜像到本地。命令格式如下：
`docker pull [options] NAME[:TAG]`
options参数说明：

* `-a`：拉取所有tagged镜像

例如从Docker Hub的Ubuntu仓库下载一个最新版的Ubuntu操作系统的镜像

```bash
docker pull ubuntu:20.04 # 指定版本号，目前最新为20.04
docker pull ubuntu #该命令实际上下载的就是ubuntu:latest镜像。
```

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/ab1e7c77f32c4616a74bc49ee493b34f-20231125205158334.png)

上面这两条命令实际上相当于`docker pull registry.hub.docker.com/ubuntu:latest`命令，即从默认的注册服务器registry.hub.docker.com中的ubuntu仓库来下载标记为latest的镜像。
我们也可以选择其他注册服务器的仓库下载。那么这个时候我们需要在仓库前指定完整的仓库注册服务器地址。例如从Docker Pool社区的镜像源d1.dockerpool.com下载最新的ubuntu镜像。
`docker pull d1.dockerpool.com:5000/ubuntu`

### 查看镜像信息

使用`docker images`命令可以列出本地主机上的所有镜像。命令格式如下：
`docker images [options] [repository:[tag]]`
options参数说明：

* -a :列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；
* -f :显示满足条件的镜像；
* -q :只显示镜像ID。

如果没有给出仓库名，那么默认列出本地主机上已有的镜像。我们使用`docker images`可以查看到如下信息：
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/3e6c31e3a93b4d6a96177fcc77f098e1-20231125205547210.png)
可以看到几个字段信息：

* repository：来自于哪个仓库，比如ubuntu仓库。
* tag：镜像的标签信息，比如20.04或latest。用于标记来自同一个仓库的不同镜像。
* &lt;font color=&#34;red&#34;&gt;image id：镜像的ID号，这个特别重要，唯一标识镜像。&lt;/font&gt;
* created：镜像创建时间。
* size：镜像大小。

我们从图中可以发现，20.04和latest标签的镜像ID是完全一致的，说明它们实际上指向了同一个镜像文件，只是别名不同而已。标签在这里起到了引用或者快捷方式的作用。
我们使用`docker inpsect NAME|ID`即可查看该镜像的详细信息，为json格式。
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_18%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125205158576.png)
我们还可以使用`docker history`命令查看指定镜像的创建历史，命令格式如下：
`docker history [options] IMAGE`
options说明：

* -q :仅列出提交记录ID。

### 搜寻镜像

使用`docker search`命令可以搜索云端仓库中共享的镜像，默认搜寻Docker Hub官方仓库的镜像。命令格式如下：
`docker search [options] TERM`
options说明：

* --automated :只列出 automated build类型的镜像；
* --no-trunc :显示完整的镜像描述；
* -f &lt;过滤条件&gt;:例如列出收藏数不小于指定值的镜像。

例如搜寻带mysql关键字的镜像如下：
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_18%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125205254646.png)

### 删除镜像

使用`docker rmi`命令可以删除镜像，命令格式如下：
`docker rmi IMAGE [IMAGE...]`
其中IMAGE可以为镜像标签或者镜像ID，这ID可以为能进行区分的部分前缀串。
需要注意的是，当有该镜像创建的容器存在时，镜像文件时默认无法删除的，若想要强行删除文件则需要加入`-f`参数来强制删除一个存在容器依赖的镜像，但这样往往会造成一些遗留问题。正确的做法应该是先删除依赖该镜像的所有容器，再来删除镜像。
若要删除所有镜像，可用`docker images -q`列出所有的ID，正确命令为：`docker rmi $(docker images -q)`。

### 创建镜像

创建镜像的方法有三种：基于已有镜像的容器创建、基于本地模板导入、基于Dockerfile创建。

#### 基于已有镜像的容器创建

使用`docker commit`命令即可从容器创建一个镜像，命令格式如下：
`docker commit [options] CONTAINER [REPOSITORY[:TAG]]`
options说明：

* -a :提交的镜像作者；
* -c :使用Dockerfile指令来创建镜像；
* -m :提交时的说明文字；
* -p :在commit时，将容器暂停。

CONTAINER为容器ID。
顺利的话，命令会返回新创建的镜像的ID信息。
实例如下：
` docker commit -a &#34;pursuit&#34; -m &#34;my ubuntu&#34; 3c61e963210c myubuntu:v1`

#### 基于本地模板导入

可以从一个操作系统模板文件导入一个镜像，也可以从网上下载一个模板。一般使用OpenVZ提供的模板来创建。[OpenVZ下载地址](https://wiki.openvz.org/Download/template/precreated)。
首先下载一个，命令如下：
`wget https://download.openvz.org/template/precreated/contrib/arch-20161108-x86_64.tar.gz`
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_17%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125205158683.png)

然后将导入该镜像：
`cat arch-20161108-x86_64.tar.gz | docker import - arch:x86_64`
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/16984ce2669947c780175dfd0f3761a7-20231125205313788.png)
查看新导入的镜像，已经本地存在了：
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/105f0e690da642019b2359c6264fe4f3-20231125205158769-20231125205317924.png)

#### 基于Dockerfile创建

其中Dockerfile是一个文本格式的配置文件，用户可以使用 Dockerfile 快速创建自定义的镜像。
Dockerfile由一行行命令语句组成，并且支持以`#`开头的注释。Dockerfile分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。例如下面的文件模板：

```bash
# This dockerfile user the ubuntu image
# VERSION 2 - EDITON 1
# Author: docker_user
# Command format: Instruction [arguments / command] ..

# 第一行必须指定基于的基础镜像
FROM ubuntu

# 维护者信息
# 例如MAINTAINER  docker_user docker_user@email.com
MAINTAINER pursuit unique.hzf@gmail.com

# 镜像的操作指令
# RUN &lt;command&gt;，当命令较长，可以用\来换行。
RUN echo &#34;Hello, World!&#34;

# 容器启动时执行指令
CMD /bin/bash
```

创建好Dockerfile后，我们利用`docker build`命令，命令格式如下：
`docker build -t PATH|URL repository:tag`
如，我们利用Dockerfile生成镜像
`sudo docker build . -t arch:x86_64_01`
查看本地镜像如下：
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/ee91502cb9374e1782f92fd75597df43-20231125205323651.png)

### 存出和载入镜像

我们可以使用`docker save`和`docker load`命令来存出和载入镜像。 &lt;font color=&#34;red&#34;&gt;存出即将镜像保存成tar归档文件，载入即将使用`docker save`命令导出的tar归档文件载入称为镜像文件。&lt;/font&gt; 命令格式如下：

```bash
docker save -o filename IMAGE
或者
docker save IMAGE&gt;filename

docker load --input filename
或者
docker load &lt;filename

```

实例：
` docker save -o files/ubuntu_20_04.tar ubuntu:20.04`
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/21e37193e1194d8aa1d15d6faa5b6f90-20231125205158807-20231125205328230.png)
然后我们需要删除ubuntu:20.04镜像，再导入。

```bash
docker rmi ubuntu:20.04
docker load &lt;ubuntu_20_04.tar
```

![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/4d05d52c678f40bca2061c4c0f4f21f5-20231125205453434.png)

### 修改镜像

使用`docker tag`命令可以标记本地镜像，将其归入某一仓库。命令格式如下：
`docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]`
实例：
`docker tag ubuntu:20.04 myubuntu/ubuntu:20.04`
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/6a80b40a10a24e198099bbdeadf210e6-20231125205333299.png)

### 上传镜像

使用`docker push`命令将本地的镜像上传到镜像仓库，默认上传到DockerHub官方仓库。要先登陆到镜像仓库。命令格式如下：
`docker push NAME[:TAG]`
例如用户user上传自己的本地镜像ubuntu:20.04，需要注意的是我们需要添加新的标签`user/ubuntu:20.04`（其中user一定要和我们的dockerhub中用户名同名，否则会报错）然后再上传。
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/d1f0f11bb5ae42a5ac1f85bc3903644f-20231125205340531.png)
之后，我们即可在DockerHub官网看到自己的仓库和上传的镜像了。
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16.png)

## 容器具体操作

### 查看容器

使用`docker ps`命令即可列出容器，命令格式如下：
`docker ps [OPTIONS]`
其中options说明：

* -a :显示所有的容器，包括未运行的。
* -f :根据条件过滤显示的内容。
* -q :静默模式，只显示容器编号。

我们运行`docker ps -a`可以得到如图：
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/af07e9a999dc4161954ff95877f47e00.png)
其中输出字段说明如下：

* CONTAINER ID: 容器 ID。
* IMAGE: 使用的镜像。
* COMMAND: 启动容器时运行的命令。
* CREATED: 容器的创建时间。
* STATUS: 容器状态。
* PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。
* NAMES: 自动分配的容器名称。

我们也可以使用`docker inspect`命令列出容器的元信息，输出格式为json格式，这个命令在查看镜像章节已经说明了，用法相同，这里不再叙述。

使用`docker stats`可以查看所有容器的统计信息，包括CPU、内存、存储、网络等信息。
使用`docker top CONTAINER`可查看某个容器内的所有进程。

### 创建容器

Docker的容器十分轻量级，用户可以随时创建或删除容器。

* **新建容器**
	可以使用`docker create`命令新建一个容器，例如：
	`docker create -it ubuntu:20.04`
	即利用镜像ubuntu:20.04创建容器。
* **新建并启动一个容器**
	使用`docker run`命令创建并启动容器。其命令格式如下：
	`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`
	其中options说明：
	1.`--name`：为容器指定一个名称；
	2.`-t`: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
	3.`-d`:  后台运行容器，并返回容器ID。
	4.`-i`: 以交互模式运行容器，通常与 -t 同时使用。
	当使用`docker`创建并启动容器时，Docker在后台运行的标准操作如下：
	1.检查本地是否有指定的镜像，不存在就从公有仓库下载;
	2.利用镜像创建并启动一个容器；
	3.分配一个文件系统，并在只读的镜像层外面挂载一层可读写层；
	4.从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去；
	5.从地址池配置一个IP地址给容器；
	6.执行用户指定的应用程序；
	7.执行完毕后容器被终止。
	下面这个实例将启动一个bash终端，允许用户进行交互，可以使用Linux命令。
	`docker run -it ubuntu:20.04 /bin/bash`
	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/f28c825c687f4545b2f3a5966466051f.png)
	其中root为用户名，@后面的为容器ID。我们用`ps`命令查看进程发现只运行了bash应用，并没有运行其他不需要的进程。可以使用Ctrl&#43;d或输入`exit`命令来退出容器。当退出容器之后，该容器就自动处于终止状态了。 &lt;font color=&#34;red&#34;&gt;这是因为对于Docker容器来说，当运行的应用（此例子中为bash）退出后，容器也就没有再运行的必要了。&lt;/font&gt;

### 启动终止容器

* **启动容器**
	使用`docker start`可以启动一个已存在的容器。命令格式如下：
	`docker start CONTAINER`
* **终止容器**
	使用`docker stop`可以启动一个已存在的容器。命令格式如下：
	`docker stop CONTAINER`
* **重启容器**
	使用`docker restart`可以启动一个已存在的容器。命令格式如下：
	`docker restart CONTAINER`

### 获取容器的日志

使用`docker logs`即可获取容器的日志。命令格式如下：
`docker logs [OPTIONS] CONTAINER`
其中OPTIONS说明：

* -f : 跟踪日志输出
* --since :显示某个开始时间的所有日志
* -t : 显示时间戳
* --tail :仅列出最新N条容器日志

实例：
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/467a19f61691409aaab21237f232399e.png)

### 进入容器

在使用`-d`参数后，容器启动后会自动进入后台，用户无法看到容器中的信息。我们如果需要进入容器进行操作，有多种方法，这里介绍两种。

* **使用`docker attach`**
	命令格式为：`docker attach CONTAINER`
	在容器中，我们可以先按`Ctrl-p`，再按`Ctrl-q`可以挂起容器。
	但是使用attach命令有时候并不方便。当多个窗口同时attach到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞后，其他窗口也无法执行操作了。
* **使用`docker exec`命令**
	Docker自1.3版本起，提供了一个更方便的工具exec，该命令可以在运行的容器中执行命令，命令格式如下：
	`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`
	其中OPTIONS说明：
	1.`-d` :分离模式: 在后台运行
	 2.`-i` :即使没有附加也保持STDIN 打开
	3.`-t` :分配一个伪终端

### 删除容器

可以使用`docker rm`命令来删除&lt;font color=&#34;red&#34;&gt;处于终止状态的容器&lt;/font&gt;，命令格式如下：
`docker rm [OPTIONS] CONTAINER [CONTAINER...]`
其中OPTIONS说明：

* `-f` :通过 SIGKILL 信号强制删除一个运行中的容器。
* `-l`:移除容器间的网络连接，而非容器本身。即删除容器的连接，但保留容器。
* `-v` :删除与容器关联的卷。

如果需要删除所有容器，可使用`docker rm $(docker ps -aq)`。

### 在本地和容器之间复制文件

使用`docker cp`命令用于主机和容器之间的数据拷贝，命令格式如下：

```bash
docker cp xxx CONTAINER:xxx 
docker cp CONTAINER:xxx xxx
```

实例：
`docker cp data.txt 3c61e963210c:data.txt`
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125205401570.png)

### 修改容器

使用`docker rename`可以重命名容器，命令格式如下：
`docker rename CONTAINER1 CONTAINER2`
使用`docker update`可以修改容器配置，命令格式如下：
`docker update CONTAINER [options]`
其中OPTIONS参数过多，使用的时候可以自行百度，例如我们修改容器的内存限制：
`docker update 3c61e963210 --memory 500MB`

### 导入和导出容器

#### 导出容器

导出容器是指导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态，可以使用`docker export`命令导出容器。该命令格式如下：

```bash
docker export [options] xxx.tar CONTAINER 
或者
docker export [options] CONTAINER&gt;xxx.tar
```

其中options说明：

* -o :将输入内容写到文件。

例如将id为3c61e963210的容器按日期保存为tar文件：
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/262372ab59414028b53091c724139c29-20231125205408708.png)
可以将这些文件传输到其他的机器上，在其他机器上通过导入命令实现容器的迁移。

#### 导入容器

导出的文件又可以使用`docker import`命令导入，成为镜像，命令格式如下：
`docker import  [OPTIONS] xxx.tar image_name:tag`
其中OPTIONS说明：

* `-c` :应用docker 指令创建镜像；
* `-m` :提交时的说明文字。

实例：
`docker  import ubuntu-20220212.tar myubunt:latest`
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/fd4457a141224da88a6c87e130f17253-20231125205414592.png)
我们知道，前面我们学习过`docker load`命令来导入一个镜像文件。实际上，既可以使用`docker load`命令来导入镜像存储文件到本地的镜像库，也可以使用`docker import`命令来导入一个容器快照到本地镜像库。 &lt;font color=&#34;red&#34;&gt;这两者的区别在于容器快照文件将丢弃所有的历史记录和元信息（仅保存容器当时的快照状态)，，而镜像存储文件将保存完整记录，体积也要大。&lt;/font&gt;

## 仓库

### Docker Hub

#### Linux登录登出DockerHub

首先在[dockerhub官网](https://hub.docker.com/)注册一个账号，然后使用`docker login`即可登录。登出则直接输入`docker logout`即可。
实例：
![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/63fac7bb71f2486dbccbc625fb1377ae-20231125205420218.png)

#### 基本操作

用户无需登录即可通过`docker search`命令来查找官网仓库的镜像，并利用`docker pull`命令来将它下载到本地。
根据是否为官方提供， 可将这些镜像资源分为两类。一种类似ubuntu这样的基础镜像，称为基础或根镜像。这些镜像是由Docker公司创建、验证、支持、提供的。这样的镜像往往使用单个单词作为名字。还有一种类型，比如pursuit/ubuntu镜像，它是由DockerHub用户pursuit创建并维护的，带有用户名称为前缀，表明是某用户的某仓库。可通过用户名称前缀user_name/ 来指定使用某个用户提供的镜像，比如pursuit用户的镜像前缀为pursuit/。

#### 自动创建

自动创建（Automated Builds）功能对于需要经常升级镜像内程序来说十分方便。有时候，用户创建了镜像，安装了某个软件，如果软件发布新版本则需要手动更新镜像。
而自动创建功能使得用户通过DockerHub指定追踪一个目标网站（目前支持Github或BitBucket）上的项目，一但发现项目新的提交，则自动执行创建。
要配置自动创建，有如下步骤：
1）创建并登录Docker Hub，以及目标网站； * 在目标网站中连接账户到Docker Hub。
2）在Docker Hub中配置一个自动创建。
3）选取一个目标网站中的项目（需要含Dockerfile）和分支。
4）指定Dockerfile的位置，并提交创建。
之后，就可以在DockerHub的”自动创建“页面跟踪每次创建的状态。

## 网络基础配置

### 端口映射实现访问容器

大量的互联网应用服务包括多个服务组件，这往往需要多个容器之间通过网络通信进行相互配合。Docker目前提供了映射容器端口到宿主主机和容器互联机制来为容器提供互联网服务。

在启动容器时，如果不指定对应参数，在容器外部是无法通过网络来访问容器内的网络应用和服务的。当容器中运行一些应用，要让外部访问这些应用时，可以通过`-p`参数来指定端口映射，并且，在一个指定的端口上只可以绑定一个容器。支持的格式有`ip:hostPort:containerPort`、`ip::containerPort`、`hostPort:containerPort`。

* **映射所有接口地址**
	使用`hostPort:containerPort`格式将本地的4000端口映射到容器的4000端口，可以执行如下命令：
	`docker run -itd -p 4000:4000 --name my_docker_ubuntu ubuntu:20.04`
	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/7bd5077670794fd096686c078bbc40b8.png)
	我们可以看到，已经实现了端口映射。多次使用`-p`标记可以绑定多个端口。例如：
	`docker run -itd -p 4000:4000 -p 3000:3000 --name my_docker_ubuntu ubuntu:20.04`
* **映射到指定地址的指定端口**
	可以使用`ip:hostPort:containerPort`格式指定映射使用一个特定地址，比如localhost地址`127.0.0.1`：
	`docker run -itd -p 127.0.0.1:3000:3000 --name my_docker_ubuntu ubuntu:20.04`
* **映射到指定地址的任意端口**
	使用`ip::containerPort`绑定localhost的任意端口到容器的5000端口，本地主机会自动分配一个端口：
	`docker run -itd -p 127.0.0.1::5000 --name my_docker_ubuntu ubuntu:20.04`
* **查看映射端口配置**
	可以使用`docker port container`命令来查看当前映射的端口配置，也可以查看到绑定的地址。
	![](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/e8d0a51dd5a0440387a300ba4f84d0e3-20231125205502860-20231125205513041.png)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.docker%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B/  

