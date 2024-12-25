# NAT\&P2P协议

## NAT介绍

**NAT（Network Address Translation，网络地址转换）**，也叫做网络掩蔽或者IP掩蔽。NAT是一种网络地址翻译技术，主要是**将内部的私有IP地址（private IP）转换成可以在公网使用的公网IP（public IP）**，主要是为了**解决 IP 短缺**的问题。

### NAT实现方式及主要类型

#### 1. 静态NAT

一个公网 IP 对应一个私有 IP，一对一的 IP 转换，无端口转换。

#### 2. 完全锥型NAT（Full Cone NAT，FC）

表现形式：

*   将来自内部同一个 IP 地址同一个端口号（IP\_IN\_A : PORT\_IN\_A）的主机监听/请求，映射到公网 IP 某个端口（IP\_OUT\_B : PORT\_OUT\_B）的监听。任意外部 IP 地址与端口对其自己公网的 IP 这个映射后的端口访问（IP\_OUT\_B : PORT\_OUT\_B），都将重新定位到内部这个主机（IP\_IN\_A : PORT\_IN\_A）。该技术中，基于 C/S 架构的应用可以在任何一端发起连接。

*   只要客户端，由内到外建立一个映射（NatIP:NatPort -> A:P1）之后，其他 IP 的主机 B 或端口 A:P2 都可以使用这个洞给客户端发送数据。

特点：IP 和端口都不受限。

![](NAT&P2P协议/FC-NAT.png)

#### 3. 受限锥型NAT（Restricted Cone NAT）

表现形式：

*   与完全锥形 NAT 不同的是，在公网映射端口后，并不允许所有 IP 进行对于该端口的访问，要想通信**必需内部主机对某个外部 IP 主机发起过连接**，然后这个外部 IP 主机就可以与该内部主机通信了，但**端口不做限制**。
*   举个栗子：当客户端由内到外建立映射(NatIP:NatPort –> A:P1)，A机器可以使用他的其他端口（P2）主动连接客户端，但 B 机器则不被允许。因为 IP 受限啦，但是端口随便。见下图（绿色是允许通信，红色是禁止通信）。

特点：IP受限，端口不受限。![](NAT&P2P协议/RC-NAT.png)

#### 4.端口受限型NAT（Port Restricted Cone NAT）

表现形式：

*   该技术与受限锥形 NAT 相比更为严格。除具有受限锥形 NAT 特性，对于回复主机的端口也有要求。也就是说：只有当内部主机曾经发送过报文给外部主机（假设其 IP 地址为 A 且端口为 P1）之后，外部主机才能以公网 IP:PORT 中的信息作为目标地址和目标端口，向内部主机发送 UDP报文，同时，其请求报文的 IP 必须是 A，端口必须为 P1（使用 IP 地址为 A，端口为 P2，或者 IP 地址为 B，端口为 P1 都将通信失败）。例子见下图。这一要求进一步强化了对外部报文请求来源的限制，从而较 Restrictd Cone 更具安全性。

![](NAT&P2P协议/PRC-NAT.png)

#### 5.对称型NAT（Symmetric NAT）

表现形式：

*   只有来自同一内部 IP:PORT、且针对同一目标 IP:PORT 的请求才被 NAT 转换至同一个公网（外部）IP:PORT，否则的话，NAT 将为之分配一个新的外部（公网）IP:PORT。并且，只有曾经收到过内部主机请求的外部主机才能向内部主机发送数据包。
*   内部主机用同一 IP 与同一端口与外部多 IP 通信。客户端想和服务器 A（IP_A:PORT_A）建立连接，是通过 NAT 映射为 NatIP:NatPortA 来进行的。而客户端和服务器 B（IP_B:PORT_B）建立连接，是通过 NAT 映射为 NatIP:NatPortB 来进行的。
*   **即同一个客户端和不同的目标 IP:PORT 通信，经过 NAT 映射后的公网 IP:PORT 是不同的**。此时，如果 B 想要和客户端通信，也只能通过 NatIP:NatPortB（也就是紫色的洞洞）来进行，而不能通过 NatIP:NatPortA（也就是黄色的洞洞）。

特点：对每个外部主机或端口的会话都会映射为不同的端口（洞）

![](NAT&P2P协议/Symmetric-NAT.png)

### STUN协议

上面讲的几种模型都是基于知道目标主机的公网地址，但实际的网络情况中，设备所处的网络环境是十分复杂的，每个设备都处在不同的 NAT 环境下，对位于不同 NAT 内部的主机通信十分复杂。

为了让处于内网的主机之间能够穿越它们之间的 NAT 建立直接通信，有许多方法被逐渐提出，其中 **STUN（Session Traversal Utilities for NAT，NAT会话穿越应用程序**）技术就是其中比较重要的一种解决方法，并得到了广泛的应用。

**STUN 是一种网络协议，它允许位于 NAT（或多重NAT）后的客户端找出自己的公网地址，查出自己位于哪种类型的 NAT 之后以及 NAT 为某一个本地端口所绑定的 Internet 端端口。**

*   这些信息被用来在两个同时处于 NAT 路由器之后的主机之间建立 UDP 通信。
*   该协议由 RFC 5389 定义，STUN 由三部分组成：STUN 客户端、STUN 服务器端、NAT 路由器。
*   STUN 服务端部署在一台有着两个公网 IP 的服务器上。
*   STUN 客户端通过向服务器端发送不同的消息类型，根据服务器端不同的响应来做出相应的判断，一旦客户端得知了 Internet 端的 UDP 端口，通信就可以开始了。

#### STUN协议基本原理

STUN 协议定义了三类测试过程来检测 NAT 类型：

*   **Test1**：STUN Client 通过端口 {IP-C1\:Port-C1} 向 STUN Server {IP-S1\:Port-S1} 发送一个 Binding Request（没有设置任何属性）。STUN Server 收到该请求后，通过端口 {IP-S1\:Port-S1} 把它所看到的 STUN Client 的 IP 和端口 {IP-M1,Port-M1} 作为 Binding Response 的内容回送给 STUN Client。
*   **Test1#2**：STUN Client 通过端口 {IP-C1\:Port-C1} 向 STUN Server {IP-S2\:Port-S2} 发送一个 Binding Request（没有设置任何属性）。STUN Server收到该请求后，通过端口 {IP-S2\:Port-S2} 把它所看到的 STUN Client 的 IP 和端口 {IP-M1#2,Port-M1#2} 作为 Binding Response 的内容回送给 STUN Client。
*   **Test2**：STUN Client 通过端口 {IP-C1\:Port-C1} 向 STUN Server {IP-S1\:Port-S1} 发送一个 Binding Request（设置了 Change IP 和 Change Port 属性）。STUN Server 收到该请求后，通过端口 {IP-S2\:Port-S2} 把它所看到的 STUN Client 的 IP 和端口 {IP-M2,Port-M2} 作为 Binding Response 的内容回送给 STUN Client。
*   **Test3**：STUN Client 通过端口 {IP-C1\:Port-C1} 向 STUN Server {IP-S1\:Port-S1} 发送一个 Binding Request（设置了Change Port属性）。STUN Server收到该请求后，通过端口 {IP-S1\:Port-S2} 把它所看到的 STUN Client 的 IP 和端口 {IP-M3,Port-M3} 作为 Binding Response 的内容回送给 STUN Client。

通过STUN协议，我们可以检测的类型一共有以下七种：

*   A：公开的互联网IP。主机拥有公网IP，并且没有防火墙，可自由与外部通信。
*   B：完全锥形NAT。&#x20;
*   C：受限制锥形NAT。&#x20;
*   D：端口受限制形NAT。&#x20;
*   E：对称型UDP防火墙。主机出口处没有NAT设备,但有防火墙,且防火墙规则如下：从主机UDP端口A发出的数据包保持源地址，但只有从之前该主机发出包的目的IP/PORT发出到该主机端口A的包才能通过防火墙。&#x20;
*   F：对称型NAT&#x20;
*   G：防火墙限制UDP通信。

STUN 使用下列的算法（取自RFC 3489）来发现 NAT 中间件以及防火墙（firewalls）:

![](NAT&P2P协议/STUN.png)

*   **STEP1：检测客户端是否有能力进行 UDP 通信以及客户端是否位于 NAT 后（Test1）**：客户端建立 UDP socket，然后用这个 socket 向服务器的（IP-1，Port-1）发送数据包要求服务器返回客户端的 IP 和 Port，客户端发送请求后立即开始接受数据包，重复几次。

    *   如果每次都**超时收不到**服务器的响应，则说明客户端无法进行 UDP 通信（**类型为 G**）；
    *   如果**能收到**回应，则把**服务器返回的客户端的（IP\:PORT）同（Local IP: Local Port）比较**：如果**完全相同**，则**说明客户端不处于 NAT 环境中，具有公网 IP 可以直接监听 UDP 端口接收数据进行通信（类型为 A 或 E）**；如果**比较不相同**，说明客户端处于 NAT 环境中要做**进一步的 NAT 类型检测**。
*   **STEP2：检测客户端防火墙类型（Test2）**：STUN 客户端向 STUN 服务器发送请求，要求服务器从其他 IP 和 PORT 向客户端回复包：

    *   收不到服务器从其他 IP 地址的回复，认为包前被前置防火墙阻断（**类型为 E**）。
    *   收到则认为客户端处在一个开放的网络上（**类型为 A**）。
*   **STEP3：检测客户端 NAT 是否是 FULL CONE NAT（Test2**）：客户端建立 UDP socket 然后用这个 socket 向服务器的 (IP-1,Port-1) 发送数据包要求服务器**用另一对 (IP-2,Port-2) 响应**客户端的请求往回发一个数据包，客户端发送请求后立即开始接受数据包，重复这个过程若干次。

    *   如果每次都**超时**，无法接受到服务器的回应，则说明客户端的 NAT 不是一个 Full Cone NAT，具体类型有待下一步检测（继续）。
    *   如果能够**接受**到服务器从 (IP-2,Port-2) 返回的应答 UDP 包，则说明客户端是一个 Full Cone NAT（**类型为 B**），这样的客户端**能够进行 UDP-P2P 通信**。
*   **STEP4：检测客户端 NAT 是否是 SYMMETRIC NAT（Test1#2）**：客户端建立 UDP socket 然后用这个 socket 向服务器的 (IP-1,Port-1) 发送数据包要求服务器返回客户端的 IP 和 Port, 客户端发送请求后立即开始接受数据包，重复这个过程直到收到回应（一定能够收到，因为第一步保证了这个客户端可以进行 UDP 通信）；用同样的方法用一个 socket 向服务器的 (IP-2,Port-2) 发送数据包要求服务器返回客户端的 IP 和 Port。

    *   **比较上面两个过程从服务器返回的客户端 (IP,Port)。**

    *   如果两个过程返回的 (IP,Port) 有一对不同则说明客户端为 Symmetric NAT（**类型为 F**），这样的客户端**无法进行 UDP-P2P通信**（检测停止），因为对称型 NAT，每次连接端口都不一样，所以无法知道对称 NAT 的客户端，下一次会用什么端口。

    *   否则是 Restricted Cone NAT（**类型为 C**），是否为 Port Restricted Cone NAT有待检测（继续）。
*   **STEP5：检测客户端 NAT 是 Restricted Cone 还是 Port Restricted Cone（Test3）**：客户端建立 UDP socket 然后用这个 socket 向服务器的 (IP-1,Port-1) 发送数据包要求服务器**用 IP-1 和一个不同于Port-1 的端口**发送一个 UDP 数据包响应客户端, 客户端发送请求后立即开始接受数据包，重复这个过程若干次。

    *   如果每次都**超时**，无法接受到服务器的回应，则说明客户端是一个 Port Restricted Cone NAT（**类型为 D**）。
    *   如果能够收到服务器的响应则说明客户端是一个 Restricted Cone NAT（**类型为 C**）。
    *   **以上两种 NAT 都可以进行 UDP-P2P 通信。**

> [NAT 穿透原理浅浅说（一）-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1005974)

### NAT ALG

#### 概念

**普通 NAT 实现了对 UDP 或 TCP 报文头中的的 IP 地址及端口转换**功能，但**对应用层数据载荷中的字段无能为力**，在许多应用层协议中，比如多媒体协议（H.323、SIP等）、FTP、SQLNET 等，TCP/UDP 载荷中带有地址或者端口信息，这些内容不能被 NAT 进行有效的转换，就可能导致问题。

**NAT ALG（Application Level Gateway，应用层网关**）技术能对多通道协议进行应用层报文信息的解析和地址转换，将载荷中需要进行地址转换的 IP 地址和端口或者需特殊处理的字段进行相应的转换和处理，从而保证应用层通信的正确性。

例如，FTP 应用就由数据连接和控制连接共同完成，而且数据连接的建立动态地由控制连接中的载荷字段信息决定，这就需要 ALG 来完成载荷字段信息的转换，以保证后续数据连接的正确建立。

NAT ALG 为内部网络和外部网络之间的通信提供了基于应用的访问控制，具有以下优点：

*   ALG 统一对各应用层协议报文进行解析处理，避免其它模块对同一类报文应用层协议的重复解析，可以有效提高报文转发效率。
*   可支持多种应用层协议：FTP、H.323（包括RAS、H.225、H.245）、SIP、DNS、ILS、MSN/QQ、NBT、RTSP、SQLNET、TFTP 等。

#### 实现过程

> [NAT ALG\_五山口老法师-云原生 (csdn.net)](https://devpress.csdn.net/cloudnative/66d558a50bfad230b8b36ba0.html?dp_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6MTI1ODMyMSwiZXhwIjoxNzMzMTM5NDg2LCJpYXQiOjE3MzI1MzQ2ODYsInVzZXJuYW1lIjoiT2JpdG9fbyJ9.5mAdUZui_7kZ8EFyOYvs_UtLHGxjZ4vDHcvCPLEPQGs\&spm=1001.2101.3001.6650.1\&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Eactivity-1-80645576-blog-115111276.235%5Ev43%5Epc_blog_bottom_relevance_base2\&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Eactivity-1-80645576-blog-115111276.235%5Ev43%5Epc_blog_bottom_relevance_base2\&utm_relevant_index=2)

### **DNS Mapping**

在某些应用中，**私网用户希望通过域名访问位于同一私网的内部服务器，而 DNS 服务器却位于公网**。由于通常 **DNS 响应报文中携带的是内部服务器的公网 IP 地址**，因此若 NAT 设备**未将** DNS Server 解析的公网 IP **替换**成内部服务器对应的私网 IP，私网用户将无法通过域名访问到内部服务器。这个问题可以使用 DNS Mapping 方式来解决，通过**配置“域名—公网 IP 地址—公网端口—协议类型”映射表**，建立内部服务器的域名与其公网信息间的对应关系。

![](NAT&P2P协议/DNS-Mapping.jpg)

如图所示，私网用户 Host 希望通过域名方式访问 Web Server， Router 作为 NAT 服务器。当 Router 设备收到 DNS 响应报文后，先根据其中携带的域名查找 DNS Mapping 映射表，再根据“公网 IP 地址—公网端口—协议类型”查找 Web Server，然后将 DNS 响应报文中的公网 IP 地址替换成 Web Server 的私网 IP 地址。这样，Host 收到的 DNS 响应报文中就携带了 Web Server 的私网 IP 地址，从而可以通过域名来访问 Web Server。

> [【网络干货】网络安全之NAT(地址转换)技术详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/136506391)

## P2P协议

### 概念

只有锥型NAT才能进行P2P，原因是由P2P连接的建立过程决定的。

### 1. 中继（Relaying）

这是最可靠但也是最低效的一种 P2P 通信实现。其原理是通过一个有公网 IP 的服务器中间人对两个内网客户端的通信数据进行中继和转发。如下图所示：

![](NAT&P2P协议/P2P-Relaying.png)

客户端 A 和客户端 B 不直接通信，而是先都与服务端 S 建立链接，然后再通过 S 和对方建立的通路来中继传递的数据。这钟方法的缺陷很明显，当链接的客户端变多之后，会显著增加服务器的负担，完全没体现出 P2P 的优势。

### 2. 逆向链接（Connection reversal）

第二种方法在当两个端点中有一个不存在中间件的时候有效。例如，客户端 A 在 NAT 之后而客户端 B 拥有公网 IP 地址，如下图：

![](NAT&P2P协议/P2P-2.png)

客户端 A 内网地址为 10.0.0.1，且应用程序正在使用 TCP 端口 1234。A 和服务器 S 建立了一个链接，服务器的 IP 地址为 18.181.0.31，监听 1235 端口。NAT A 给客户端 A 分配了 TCP 端口 62000，地址为 NAT 的公网 IP 地址 155.99.25.11，**作为客户端 A 对外当前会话的临时 IP 和端口**。因此 S 认为客户端 A 就是 155.99.25.11:62000。**而 B 由于有公网地址，所以对 S 来说 B 就是138.76.29.7:1234**。

当客户端 B 想要发起一个对客户端 A 的 P2P 链接时，要么链接 A 的外网地址 155.99.25.11:62000，要么链接 A 的内网地址 10.0.0.1:1234，然而两种方式链接都会失败。链接 10.0.0.1:1234 失败自不用说，为什么链接 155.99.25.11:62000 也会失败呢？来自 B 的 TCP SYN 握手请求到达 NAT A 的时候会被拒绝，因为对 NAT A 来说只有外出的链接才是允许的。

在直接链接 A 失败之后，B 可以通过 S 向 A 中继一个链接请求，从而从 A 方向“逆向“地建立起 A-B 之间的点对点链接。

很多当前的 P2P 系统都实现了这种技术，但其局限性也是很明显的，只有当其中一方有公网 IP 时链接才能建立。越来越多的情况下，通信的双方都在 NAT 之后，因此就要用到我们下面介绍的第三种技术了。

### 3. UDP打洞（UDP hole punching）

第三种 P2P 通信技术，被广泛采用的，名为“P2P打洞“。P2P 打洞技术依赖于通常防火墙和 cone NAT 允许正当的 P2P 应用程序在中间件中打洞且与对方建立直接链接的特性。

以下主要考虑两种常见的场景，以及应用程序如何设计去完美地处理这些情况。

*   第一种场景代表了大多数情况，即两个需要直接链接的客户端处在两个不同的NAT之后；
*   第二种场景是两个客户端在同一个NAT之后，但客户端自己并不需要知道。

#### 3.1. 端点在不同的NAT之下

假设客户端 A 和客户端 B 的地址都是内网地址，且在不同的 NAT 后面。A、B 上运行的 P2P 应用程序和服务器 S 都使用了 UDP 端口 1234，A 和 B 分别初始化了与 Server 的 UDP 通信，地址映射如图所示:

![](NAT&P2P协议/P2P-3.png)

现在假设客户端 A 打算与客户端 B 直接建立一个 UDP 通信会话。如果 A 直接给 B 的公网地址 138.76.29.7:31000 发送 UDP 数据，NAT B 将很可能会无视进入的数据（**除非是 Full Cone NAT**），因为源地址和端口与 S 不匹配，而最初只与 S 建立过会话。B 往 A 直接发信息也类似。

假设 A 开始给 B 的公网地址发送 UDP 数据的**同时，给服务器 S 发送一个中继请求**，**要求 B 开始给 A的公网地址发送UDP信息**。A 往 B 的输出信息会导致 NAT A 打开一个 A 的内网地址与与 B 的外网地址之间的新通讯会话，B 往 A 亦然。一旦**新的 UDP 会话在两个方向都打开之后，客户端 A 和客户端 B 就能直接通讯**，而无须再通过引导服务器 S 了。

UDP 打洞技术有许多有用的性质。一旦一个的 P2P 链接建立，链接的双方都能反过来作为“引导服务器”来帮助其他中间件后的客户端进行打洞，极大减少了服务器的负载。应用程序不需要知道中间件具体是什么（如果有的话），因为以上的过程在没有中间件或者有多个中间件的情况下也一样能建立通信链路。

#### 3.2. 端点在相同的NAT之下

现在考虑这样一种情景，**两个客户端 A 和 B 正好在同一个NAT之后**（而且可能他们自己并不知道），因此在同一个内网网段之内。客户端 A 和服务器 S 建立了一个 UDP 会话，NAT 为此分配了公网端口 62000，B 同样和 S 建立会话，分配到了端口 62001，如下图：

![](NAT&P2P协议/P2P-4.png)

假设 A 和 B 使用了上节介绍的 UDP 打洞技术来建立 P2P 通路，那么会发生什么呢？首先 A 和 B 会得到由 S 观测到的对方的公网 IP 和端口号，然后给对方的地址发送信息。\*\*两个客户端只有在 NAT 允许内网主机对内网其他主机发起 UDP 会话的时候才能正常通信，我们把这种情况称之为"回环传输“(lookback translation)，因为从内部到达NAT 的数据会被“回送”到内网中而不是转发到外网。\*\*例如，当 A 发送一个 UDP 数据包给B的公网地址时，数据包最初有源 IP 地址和端口地址 10.0.0.1:1234 和目的地址 155.99.25.11:62001，NAT 收到包后，将其转换为源 155.99.25.11:62000（A的公网地址）和目的 10.1.1.3:1234，然后再转发给 B。**即便 NAT 支持回环传输，这种转换和转发在此情况下也是没必要的**，且有可能会增加 A 与 B 的对话延时和加重NAT的负担。

对于这个问题，解决方案是很直观的。\*\*当 A 和 B 最初通过 S 交换地址信息时，他们应该包含自身的 IP 地址和端口号（从自己看），同时也包含从服务器看的自己的地址和端口号。\*\*然后客户端同时开始从对方已知的两个的地址中同时开始互相发送数据，并使用第一个成功通信的地址作为对方地址。如果两个客户端在同一个 NAT 后，发送到对方内网地址的数据最有可能先到达，从而可以建立一条不经过 NAT 的通信链路；如果两个客户端在不同的 NAT 之后，发送给对方内网地址的数据包根本就到达不了对方，但仍然可以通过公网地址来建立通路。值得一提的是，虽然这些数据包通过某种方式验证，但是在不同 NAT 的情况下完全有可能会导致 A 往 B 发送的信息发送到其他A 内网网段中无关的结点上去的。

#### 3.3. 固定端口绑定

UDP打洞技术有一个主要的条件：**只有当**两个 NAT 都是 Cone NAT（或者非NAT的防火墙）时才能工作。**因为其维持了一个给定的（内网 IP，内网 UDP）二元组和（公网 IP， 公网 UDP）二元组固定的端口绑定**，只要该 UDP 端口还在使用中，就不会变化。如果像对称 NAT一样，给每个新会话分配一个新的公网端口，就会导致 UDP 应用程序无法使用跟外部端点已经打通了的通信链路。由于 Cone NAT 是当今最广泛使用的，尽管有一小部分的对称 NAT 是不支持打洞的，UDP 打洞技术也还是被广泛采纳应用。

> <https://blog.csdn.net/qq_33240946/article/details/104717234>
>
> [P2P通信原理与实现(C++) - 有价值炮灰 - 博客园 (cnblogs.com)](https://www.cnblogs.com/pannengzhi/p/4800526.html)
>
> [P2P通信原理与实现 - 有价值炮灰 (evilpan.com)](https://evilpan.com/2015/10/31/p2p-over-middle-box/)

