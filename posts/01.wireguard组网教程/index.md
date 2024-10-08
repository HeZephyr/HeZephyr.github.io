# WireGuard组网教程


## 引言

### 什么是WireGuard

![WireGuard: fast, modern, secure VPN tunnel](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/wireguard.svg)

官方介绍如下：

&gt;WireGuard ® 是一款极其简单但快速且现代的 VPN，采用最先进的加密技术。它的目标是比 IPsec 更快、更简单、更精简、更有用，同时避免令人头疼的问题。它的性能远高于 OpenVPN。 
&gt;
&gt;WireGuard 被设计为通用 VPN，可在嵌入式接口和超级计算机上运行，适合许多不同的情况。它最初针对 Linux 内核发布，现在已跨平台（Windows、macOS、BSD、iOS、Android）且可广泛部署。它目前正在大力开发中，但它可能已被视为业内最安全、最易于使用且最简单的 VPN 解决方案。

我们可以用一句话概括它：

&lt;font color=&#34;red&#34;&gt;WGuard是一款可以组建虚拟私人局域网（VPN）的软件，允许用户通过公共网络（如互联网）安全地传输数据，同时保持数据的机密性和完整性。&lt;/font&gt;

**WireGuard有如下优势**：

- 更轻便：以Linux内核模块的形式运行，资源占用小。
- 更高效：相比目前主流的IPSec、OpenVPN等协议，WireGuard的效率要更高。
- 更快速：比目前主流的VPN协议，连接速度要更快。
- 更安全：使用了更先进的加密技术。
- 更易搭建：部署难度相对更低。
- 更隐蔽：以UDP协议进行数据传输，比TCP协议更低调。
- 不易被封锁：TCP阻断对WireGuard无效，IP被墙的情况下仍然可用。
- 更省电：不使用时不进行数据传输，移动端更省电。

### WireGuard可以用来做什么

1. 建立VPN（不限设备类型）

   WireGuard支持多种平台，包括电脑、智能手机和路由器。这一特性使其成为构建虚拟私有网络（VPN）的理想选择，能在这些设备上实现安全连接。无论是用于远程工作、保护数据隐私，还是绕过地理限制，WireGuard都能提供稳定且安全的网络连接。

2. 实现内网穿透

   &gt; 内网穿透，即NAT（Network Address Translator）穿透，是**指计算机在内网（局域网）内使用私有IP地址，在连接外网（互联网）时使用全局IP地址的技术**。该技术被普遍使用在有多台主机但只通过一个公有IP地址访问的私有网络中。
   &gt;
   &gt; 举个例子：比如我在实验室配置了一个服务器 Server A，当我在实验室的时候，就可以通过自己的笔记本使用SSH连接【**因为我和服务器处于一个局域网**】，当我回宿舍以后，就没有办法直接使用SSH连接了【**因为我和服务器不在一个局域网**】，这个时候就需要进行NAT穿透，让我在宿舍也可以使用SSH连接Server A。

   &gt; 通过Wireguard可以将广域网上的主机连接起来，形成一个局域网。只需要有一台具有固定公网IP的服务器，就可以将其作为我们搭建的局域网的中心节点，让其他的主机（不论是否有公网IP，不论是否在NAT内），都通过这个中心节点和彼此相连。由此就构建了一个中心辐射型的局域网，实现了内网穿透等功能。

3. Docker容器通信

   WireGuard还可用于Docker容器之间的通信。在Docker环境中，容器之间的网络通信是一个重要的问题。WireGuard通过提供一种安全的通信方式，能够在不同容器之间建立一个加密的网络连接，从而保障数据的安全传输。这对于需要在不同容器间安全共享数据的应用尤为重要。

### WireGuard原理

[WireGuard源码地址](https://git.zx2c4.com/wireguard-linux/tree/drivers/net/wireguard)

WireGuard 是一种在第 3 层（网络层）运行的安全网络隧道，与传统的 VPN 解决方案（如 IPsec 和 OpenVPN）相比，它的设计更安全、性能更高且更易于使用。它是作为 Linux 内核虚拟网络接口实现的，基于安全隧道的基本原理：将peer的公钥与隧道源 IP 地址关联。

相关术语：

&gt; * Peer/Node/Device
&gt;
&gt;   连接到VPN 并为自己注册一个VPN子网地址（如 192.0.2.3）的主机。还可以通过使用逗号分隔的 CIDR 指定子网范围，为其自身地址以外的 IP 地址选择路由。
&gt;
&gt; * 中继服务器（Bounce Server）
&gt;
&gt;   一个公网可达的peer，可以将流量中继到 `NAT` 后面的其他peer。`Bounce Server` 并不是特殊的节点，它和其他peer一样，唯一的区别是它有公网 IP，并且开启了内核级别的 IP 转发，可以将 威屁恩 的流量转发到其他客户端。
&gt;
&gt; * 子网（Subnet）
&gt;
&gt;   一组私有 IP，例如 `192.0.2.1-255` 或 `192.168.1.1/24`，一般在 NAT 后面，例如办公室局域网或家庭网络。
&gt;
&gt; * CIDR 表示法
&gt;
&gt;   CIDR，即无类域间路由（Classless Inter-Domain Routing），是一种用于对IP地址进行灵活表示和分配的标准。
&gt;
&gt; * NAT
&gt;
&gt;   子网的私有 IP 地址由路由器提供，通过公网无法直接访问私有子网设备，需要通过 NAT 做网络地址转换。路由器会跟踪发出的连接，并将响应转发到正确的内部 IP。
&gt;
&gt; * 公开端点（Public Endpoint）
&gt;
&gt;   节点的公网 IP 地址:端口，例如 `123.124.125.126:1234`，或者直接使用域名 `some.domain.tld:1234`。如果peer节点不在同一子网中，那么节点的公开端点必须使用公网 IP 地址。
&gt;
&gt; * 私钥（Private key）
&gt;
&gt;   单个节点的 WireGuard 私钥，生成方法是：`wg genkey &gt; example.key`。
&gt;
&gt; * 公钥（Public key）
&gt;
&gt;   单个节点的 WireGuard 公钥，生成方式为：`wg pubkey &lt; example.key &gt; example.key.pub`。
&gt;
&gt; * DNS
&gt;
&gt;   域名服务器，用于将域名解析为 VPN 客户端的 IP，不让 DNS请求泄漏到 VPN 之外。

主要功能和原理如下

&gt; WireGuard 通过添加一个（或多个）网络接口来工作，例如 `eth0` 或 `wlan0` ，称为 `wg0` （或 `wg1` 、 `wg2` 、 `wg3` 等）。然后可以使用 `ifconfig(8)` 或 `ip-address(8)` 正常配置该网络接口，并使用 `route(8)` 或 `ip-route(8)` 添加和删除其路由，以及所有普通网络实用程序都是如此。接口的特定 WireGuard 方面使用 `wg(8)` 工具进行配置。该接口充当隧道接口。
&gt;
&gt; WireGuard 将隧道 IP 地址与公钥和远程端点相关联。当接口向peer发送数据包时，它会执行以下操作：
&gt;
&gt; 1. 该数据包适用于 192.168.30.8。那是哪位peer啊？让我看看...好吧，这是给peer `ABCDEFGH` 的。 （或者，如果它不适合任何已配置的peer，则丢弃该数据包。）
&gt; 2. 使用peer `ABCDEFGH` 的公钥加密整个 IP 数据包。
&gt; 3. Peer `ABCDEFGH` 的远程端点是什么？让我看看...好的，端点是主机 216.58.211.110 上的 UDP 端口 53133。
&gt; 4. 使用 UDP 通过 Internet 将步骤 2 中的加密字节发送到 216.58.211.110:53133。
&gt;
&gt; 当接口收到数据包时，会发生以下情况：
&gt;
&gt; 1. 我刚刚从主机 98.139.183.24 上的 UDP 端口 7361 收到一个数据包。让我们来解密吧！
&gt; 2. 它为peer `LMNOPQRS` 正确解密和验证。好的，让我们记住，peer `LMNOPQRS` 的最新 Internet 端点是使用 UDP 的 98.139.183.24:7361。
&gt; 3. 解密后，明文数据包来自 192.168.43.89。是否允许peer `LMNOPQRS` 以 192.168.43.89 向我们发送数据包？
&gt; 4. 如果是，则在接口上接受数据包。如果没有，就放弃它。
&gt;
&gt; WireGuard 的核心是一个称为加密密钥路由的概念，它的工作原理是将公钥与隧道内允许的隧道 IP 地址列表相关联。每个网络接口都有一个私钥和一个peer点列表。每个peer都有一个公钥。公钥短小且简单，由peer用来相互验证。它们可以通过任何带外方法传递以在配置文件中使用，类似于将 SSH 公钥发送给朋友以访问 shell 服务器的方式。

### WireGuard安装

[wireGuard官方安装教程](https://www.wireguard.com/install/)

## WireGuard组网实现内网穿透

### 前提条件

1. **公网服务器：** 必须拥有一台具有公网IP地址的服务器，这是内网穿透的关键。该服务器充当中转站，负责将外部请求传递到内部网络。
2. **网络设备配置权限：** 需要对内部网络的路由器或防火墙有一定的配置权限，以便进行端口映射或其他必要的网络设置。这确保了从公网服务器到内网的连接是有效的。
3. **安装WireGuard：** 在公网服务器和内网设备上都需要安装和配置WireGuard软件。确保两端的WireGuard配置一致，包括公私钥的生成和网络接口的配置。
4. **开启相应端口：** 在公网服务器的防火墙配置中，需要打开WireGuard所使用的端口（默认是51820/UDP），以确保能够接收来自内网设备的连接请求。
5. **合适的网络拓扑：** 确保了解内部网络的拓扑结构，以便正确设置WireGuard配置，包括允许流量通过的子网、路由等。

### 网络拓扑结构

![wireGuard](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/wireGuard.png)

### 具体步骤

#### 中继服务器配置

1. 创建密钥对

   &gt; `wg genkey | tee server_privatekey | wg pubkey &gt; server_publickey`
   &gt;
   &gt; 执行以上两条命令后，会在执行命令的当前文件夹自动生成2个文件：
   &gt;
   &gt; ![image-20231115205242990](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20231115205242990.png)

* 开启IP地址转发

  &gt; `sudo sysctl net.ipv4.ip_forward`
  &gt;
  &gt; 如果显示1则说明已开启，否则则未开启。
  &gt;
  &gt; ```bash
  &gt; echo &#34;net.ipv4.ip_forward = 1&#34; &gt;&gt; /etc/sysctl.conf
  &gt; echo &#34;net.ipv4.conf.all.proxy_arp = 1&#34; &gt;&gt; /etc/sysctl.conf
  &gt; sysctl -p /etc/sysctl.conf
  &gt; ```

* 设置IP地址伪装

  ```bash
  # 允许防火墙伪装IP
  firewall-cmd --add-masquerade
  # 检查是否允许伪装IP
  firewall-cmd --query-masquerade
  # 禁止防火墙伪装IP
  firewall-cmd --remove-masquerade
  ```

* 配置wireguard虚拟网卡（不推荐，只是让读者直观了解过程）

  ```bash
  sudo ip link add wg0 type wireguard # 添加一块叫 wg0 的虚拟 wireguard 网卡
  sudo ip addr add 192.168.71.1/24 dev wg0 # 给 wg0 网卡添加 ip 地址 192.168.71.1，子网掩码 255.255.255.0
  sudo wg set wg0 private-key ./server-privatekey # wireguard 设置密钥
  sudo ip link set wg0 up # 启用刚刚添加的网卡
  ```

  &gt; 我们可以通过`ip addr`命令查看到wg0网卡的状态
  &gt;
  &gt; ![image-20231115211730531](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20231115211730531.png)
  &gt;
  &gt; 可以看到网卡`wg0` 接口是已启用的，具有 IPv4 地址 `192.168.71.1`
  &gt;
  &gt; 输入`wg`命令则可以看到配置信息，配置文件通常在
  &gt;
  &gt; ![image-20231115212013931](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20231115212013931.png)

  有想继续尝试这种方式的可以看一下[官方教程](https://www.wireguard.com/quickstart/)

* 编写配置文件配置网卡（推荐，应该`wg set`命令需要提供很多参数，很容易出错）

  &gt; 我们在`/etc/wireguard`目录中创建`wg0.conf`并编写配置，配置项请看2.4 配置项说明
  &gt;
  &gt; ```ini
  &gt; [Interface]
  &gt; # 本机密钥
  &gt; PrivateKey = KIDTljv66CgVYBNlrSD13Au6qfUdIcFJkTBkuErhTEk=
  &gt; # 本机地址
  &gt; Address = 192.168.71.1/24
  &gt; # 监听端口
  &gt; ListenPort = 51820
  &gt; 
  &gt; [Peer]
  &gt; # 对端的publickey
  &gt; PublicKey = iWy57DmR6wVXcVzMDOa2WyywO0WT5JRAGYIlh0v/nW8=
  &gt; # 对端地址
  &gt; AllowedIPs = 192.168.71.2/24
  &gt; ```

* 重新启动网卡

  ```bash
  sudo wg-quick down wg0
  sudo wg-quick up wg0
  ```

#### 其他peer

我这里只列举MacOS的操作方式（其他都同理，就是要配置私钥和公钥）

![image-20231115220359604](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20231115220359604.png)

操作完之后，它会给出密钥对，我们只需要添加好其他信息即可。

![image-20231115220409785](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20231115220409785.png)

配置文件如下：

```ini
[Interface]
# 本机密钥
PrivateKey = kDUqWzkbaB1EU5C2ADoId1TXtZF89xxn0VV45EcjFHs=
# 本机地址
Address = 192.168.71.2/24

[Peer]
# 对端公钥，即公网服务器公钥
PublicKey = bEm1p736FQySfKlTTUCeHmiwTmna5umZWOWLGWqioSk=
# 允许此对等方的传入流量并指定传出流量的目标。
AllowedIPs = 192.168.71.0/24
# 公网IP&#43;监听端口号
Endpoint = 1.1.1.1:51820
PersistentKeepalive = 25
```

#### 测试

MacOS端：

![image-20231115221940900](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20231115221940900.png)

服务器Ping 主机：

![image-20231115222015045](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20231115222015045.png)

### WireGuard配置文件说明

* **interface部分**：
  - `PrivateKey`: 由 `wg genkey` 生成的 Base64 编码的私钥。必须配置。
  - `ListenPort`: 用于监听的 16 位端口。可选，如果未指定，则随机选择端口。
  - `DNS`: 指定 DNS 服务器的 IP 地址。
  - `FwMark`: 用于传出数据包的 32 位 fwmark。如果设置为 0 或 &#34;off&#34;，则禁用此选项。可选。可以以十六进制形式指定，例如，以 &#34;0x&#34; 开头。可选。
* **Peer 部分：**
  - `PublicKey`: 由 `wg pubkey` 根据私钥计算的 Base64 编码的公钥。必须配置。
  - `PresharedKey`: 由 `wg genpsk` 生成的 Base64 编码的预共享密钥。可选，可以省略。此选项为现有的公钥加密提供了额外的对称密钥加密层，以增强对抗后量子计算的能力。
  - `AllowedIPs`: 逗号分隔的 IP 地址（IPv4 或 IPv6）列表，带有 CIDR 掩码，用于允许此对等方的传入流量并指定传出流量的目标。可以多次指定。可用 0.0.0.0/0 匹配所有 IPv4 地址，使用 ::/0 匹配所有 IPv6 地址。
  - `Endpoint`: 一个 IP 地址或主机名，后跟冒号，然后是一个端口号。此端点将自动更新为来自对等方的正确经过身份验证的数据包的最新源 IP 地址和端口。可选。
  - `PersistentKeepalive`: 保持活跃的时间间隔，介于 1 和 65535 之间，表示多久发送一次对等方的身份验证空数据包，以保持有状态的防火墙或 NAT 映射的有效性。如果设置为 0 或 &#34;off&#34;，则禁用此选项。可选，默认情况下此选项被禁用。

下面是一个简单的配置文件示例：

```ini
[Interface]
PrivateKey = yAnz5TF&#43;lXXJte14tji3zlMNq&#43;hd2rYUIgJBgB3fBmk=
ListenPort = 51820

[Peer]
PublicKey = xTIBA5rboUvnH4htodjb6e697QjLERt1NAB4mZqp8Dg=
Endpoint = 192.95.5.67:1234
AllowedIPs = 10.192.122.3/32, 10.192.124.1/24

[Peer]
PublicKey = TrMvSoP4jYQlY6RIzBgbssQqY3vxI2Pi&#43;y71lOWWXX0=
Endpoint = [2607:5300:60:6b0::c05f:543]:2468
AllowedIPs = 10.192.122.4/32, 192.168.0.0/16

[Peer]
PublicKey = gN65BkIKy1eCE9pP1wdc8ROUtkHLF2PfAqYdyYBz6EA=
Endpoint = test.wireguard.com:18981
AllowedIPs = 10.10.10.230/32
```

## WireGuard工具

### wg-easy

[github地址](https://github.com/wg-easy/wg-easy)

这是一个用于管理 WireGuard 设置的 Web 用户界面。使用它之前我们得先安装docker和docker-compose。这里我给出`docker-compose.yml`配置文件示例。还有很多配置项可在仓库中找到，灵活配置VPN

```yml
version: &#39;3&#39;
services:
  wg-easy:
    image: weejewel/wg-easy
    container_name: wg-easy
    environment:
      - WG_HOST=YOUR_SERVER_IP # 公网IP
      - PASSWORD=YOUR_ADMIN_PASSWORD # Web UI登录密码
      - WG_PORT=51820 # 监听端口
      - WG_PERSISTENT_KEEPALIVE=25 # 保持“连接”打开的值（以秒为单位）
      - WG_DEFAULT_ADDRESS=192.168.71.0 # 客户端 IP 地址范围
      - WG_ALLOWED_IPS=192.168.71.0/24 # 客户端将使用的允许 IP
    volumes:
      - ~/.wg-easy:/etc/wireguard
    ports:
      - 51820:51820/udp
      - 51821:51821/tcp
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv4.ip_forward=1
    restart: unless-stopped
```

通过`docker compose up -d`部署好后，我们进入Web界面即可添加Client。![image-20231116085710244](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20231116085710244.png)

这里，我们只需要将这三个配置文件分给对应的Client的即可完成网络搭建，特别方便！

### wg-gen-web

[wg-gen-web](https://github.com/vx3r/wg-gen-web)

跟wg-easy类似，不过功能更强大。

![Screenshot](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/wg-gen-web_screenshot.png)

### dsnet

[github地址](https://github.com/naggie/dsnet)

一款用于管理集中式wireguard VPN 的FAST 命令。

![dsnet add](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/init&#43;add.png)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.wireguard%E7%BB%84%E7%BD%91%E6%95%99%E7%A8%8B/  

