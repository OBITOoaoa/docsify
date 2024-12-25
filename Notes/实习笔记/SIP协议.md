# SIP协议

## VOIP技术

VOIP 是一种以 IP 电话为主的技术，基本原理是通过语音的压缩算法对语音数据编码进行压缩处理，然后把这些语音数据按 TCP/IP 标准进行打包，经过 IP 网络把数据包送至接收地，再把这些语音数据包串起来，经过解压处理后，恢复成原来的语音信号，从而达到由互联网传送语音的目的。

## SIP协议

### SIP概念

**SIP(Session Initiation Protocol)是一个**应用层的信令控制协议。用于创建、修改和释放一个或多个参与者的会话。这些会话可以是 Internet 多媒体会议、IP 电话或多媒体分发。会话的参与者可以通过组播（multicast）、网状单播（unicast）或两者的混合体进行通信。

**SIP 会话使用四个主要组件：SIP 用户代理、SIP 注册服务器、SIP 代理服务器和 SIP 重定向服务器**。这些系统通过传输包括了 SDP 协议（用于定义消息的内容和特点）的消息来完成 SIP 会话。

*   用户代理：SIP 用户代理(UA) 是终端用户设备，如用于创建和管理 SIP 会话的移动电话、多媒体手持设备、PC、PDA 等。用户代理客户机发出消息，用户代理服务器对消息进行响应。
*   注册服务器：SIP 注册服务器是包含域中所有用户代理的位置的数据库。在 SIP 通信中，这些服务器会检索出对方的 IP 地址和其他相关信息，并将其发送到 SIP 代理服务器。
*   代理服务器：SIP 代理服务器接受 SIP UA 的会话请求并查询 SIP 注册服务器，获取收件方 UA 的地址信息。然后，它将会话邀请信息直接转发给收件方 UA（如果它位于同一域中）或代理服务器（如果 UA 位于另一域中）。
*   重定向服务器：SIP 重定向服务器允许 SIP 代理服务器将 SIP 会话邀请信息定向到外部域。SIP 重定向服务器可以与 SIP 注册服务器和 SIP 代理服务器同在一个硬件上。

### SIP报文解析

SIP 消息是由一个起始行(start-line)，一个或多个字段组成的消息头(field)，一个标志消息头结束的空行(CRLF)，作为可选项的消息体(messge - body)。

*   描述消息体(message-body)的头称为实体头(entity-header)
*   起始行分为：

    *   请求行(Request - Line)：请求消息的起始行
    *   状态行(Statue - Line) ：响应消息的起始行

SIP规定了六种方法：INVITE ACK CANCEL OPTIONS BYE REGISTER

*   INVITE：用于邀请用户或服务参加一个会话
*   ACK：请求用于客户端向服务器证实收到对INVITE请求的最终应答
*   CANCEL：用于取消一个Call-ID From To Cseq 字段相同正在进行的请求,但取消不了已完成的请求
*   OPTIONS：用于向服务器查询其能力
*   BYE：用于结束会话
*   REGISTER：用于客户向注册服务器注册用户位置等信息

SIP 消息头字段:

*   via 字段给出请求消息迄今为止经过的路径
*   Request-URI 注册请求的目的地址
*   Max-Forwords 请求消息允许被转发的次数
*   From 请求的发起者
*   To 请求的目的接收方
*   Call-ID 唯一标识特定邀请或某个客户机的注册请求消息
*   Cseq 标识服务器发出的不同请求,若Call-ID相同Cseq值必须各不相同
*   Contact 给出一个URL,用户可以根据此URL进一步的通讯
*   Content-Length 消息体的大小
*   Content-Type 消息体的媒体类型
*   Expires 消息内容截止的日期和时间
*   User-Agent 发起请求的用户代理客户及相关的信息

SIP消息体：

*   v 协议的版本
*   o 与会话所有者的相关参数
*   s 会话标题或会话名称
*   c 真正流媒体使用的IP地址
*   t 会话的开始时间与结束时间
*   m 会话所支持的媒体类型
*   a 媒体的属性行

SIP状态码定义如下：

*   1XX：请求已经收到继续处理请求
*   2XX：行动已成功的接收到
*   3XX：为完成呼叫请求还需采取进一步动作
*   4XX：请求有语法错误不能被服务器端执行,客户端需修改请求,再次重发
*   5XX：服务器出错不能执行合法请求
*   6XX：任何服务器都不能执行请求

> [sip协议呼叫流程详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/504262401)

[SIP(会话发起协议) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/691687606)

[Wireshark网络抓包工具抓包 || 实例分析一个Sip Call流程 - 习久性成 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hls-code/p/16054209.html)
