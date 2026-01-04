### osi 分几层
*  OSI（开放系统互联）模型将网络通信分为七层，从下到上依次为：
*  物理层（Physical Layer）功能：负责比特流的传输，处理物理连接、信号传输、硬件接口等。例子：网线、光纤、电压、网卡。
*  数据链路层（Data Link Layer）功能：提供节点到节点的可靠数据传输，处理错误检测、帧同步、MAC地址寻址等。 例子：交换机、网桥。
*  网络层（Network Layer）功能：负责数据包的路由和转发，处理逻辑寻址（如IP地址）。 例子：路由器。
*  传输层（Transport Layer）功能：提供端到端的可靠数据传输，处理分段、重组、流量控制、错误纠正。 例子：端口号、TCP三次握手。
*  会话层（Session Layer）功能：管理通信会话，建立、维护和终止会话。 例子：RPC、NetBIOS。
*  表示层（Presentation Layer）功能：处理数据的表示、编码、加密、压缩，转换数据格式以适应不同系统。 例子：JPEG、SSL/TLS、字符编码（UTF-8）。
*  应用层（Application Layer）功能：为用户提供网络服务接口，直接支持应用程序。 协议：HTTP、FTP、SMTP、DNS。

### 网络进阶与内核面试题

#### 1. 详细描述 TCP 三次握手与四次挥手（附状态流转）
* **三次握手 (Connection Establish)**：
    1. **SYN_SENT**：客户端发送 SYN，Seq=x。
    2. **SYN_RCVD**：服务端收到 SYN，回复 SYN+ACK，Seq=y, Ack=x+1。
    3. **ESTABLISHED**：客户端收到 SYN+ACK，回复 ACK，Ack=y+1。服务端收到后也进入 ESTABLISHED。
* **四次挥手 (Connection Terminate)**：
    1. **FIN_WAIT_1**：主动方发送 FIN。
    2. **CLOSE_WAIT**：被动方收到 FIN，回复 ACK。此时被动方进入半关闭状态（能发不能收）。主动方收到 ACK 进入 **FIN_WAIT_2**。
    3. **LAST_ACK**：被动方处理完数据，发送 FIN。
    4. **TIME_WAIT**：主动方收到 FIN，回复 ACK。等待 2MSL 后进入 CLOSED。被动方收到 ACK 进入 CLOSED。

#### 2. 为什么要 TIME_WAIT 状态？为什么是 2MSL？
* **可靠终止**：确保最后一个 ACK 能到达被动方。如果 ACK 丢了，被动方会重传 FIN，主动方必须还在 TIME_WAIT 才能重发 ACK。
* **防止旧包混淆**：2MSL（Maximum Segment Lifetime，报文最大生存时间）足以让网络中残留的旧报文段全部消失，避免新连接混入旧连接的数据。

#### 3. 什么是 TCP 的 Backlog？（SYN Queue 与 Accept Queue）
* **半连接队列 (SYN Queue)**：服务端收到 SYN 但未收到 ACK 时，连接存放在此。
    * 攻击点：SYN Flood 攻击会打满此队列。
* **全连接队列 (Accept Queue)**：三次握手完成，等待 Application (如 Nginx/PHP-FPM) 调用 `accept()` 取走的连接。
    * 监控：`ss -lnt` 查看 `Send-Q`（当前全连接队列长度）。
    * 问题：如果 Application 处理太慢，全连接队列满，新连接会被拒绝或忽略。

#### 4. TCP 粘包/拆包的原因是什么？如何解决？
* **原因**：TCP 是**面向流**的协议，没有消息边界。
    * **粘包**：发送方 Nagle 算法将小包合并，或接收方读取过慢积压。
    * **拆包**：数据包超过 MSS（最大报文段长度），被拆分发送。
* **解决**：应用层协议定义边界。
    * **定长**：每条消息固定 N 字节。
    * **分隔符**：如 HTTP 的 `\r\n\r\n`。
    * **Length + Body**：消息头包含长度字段（最常用）。

#### 5. 什么是 Nagle 算法与 Delayed ACK？它们有什么冲突？
* **Nagle**：发送端攒够数据或收到 ACK 才发包，为了减少小包（减少网络拥塞）。
* **Delayed ACK**：接收端不立即回 ACK，而是等一会（40ms）看有没有数据要捎带发送（Piggyback），为了减少 ACK 包。
* **冲突**：写-写-读模式下，Nagle 等 ACK，Delayed ACK 等数据，导致 40ms 延迟。
* **优化**：通常开启 `TCP_NODELAY` 禁用 Nagle 算法（如 HTTP Server）。

#### 6. 为什么 HTTP/3 基于 UDP（QUIC）？解决了什么问题？
* **解决 TCP 队头阻塞 (Head-of-Line Blocking)**：TCP 中一个包丢失，后续所有包都要等待重传；QUIC 基于 UDP，流与流之间独立，一个流丢包不影响其他流。
* **连接迁移**：TCP 依赖四元组（IP:Port），切换网络（Wi-Fi 切 4G）IP 变了连接断；QUIC 使用 Connection ID，IP 变了连接依然保持，无需重连。
* **更快的握手**：TLS 1.3 融合在 QUIC 握手中，支持 0-RTT（首次连接 1-RTT，恢复连接 0-RTT）。


### 网络与协议面试题库大全 (补充精选)

#### 1. 基础模型 (OSI & TCP/IP)
1.  **OSI 七层模型 vs TCP/IP 四层模型？**
    *   OSI：物理层、数据链路层、网络层、传输层、会话层、表示层、应用层。
    *   TCP/IP：网络接口层、网络层 (IP)、传输层 (TCP/UDP)、应用层。
2.  **每层对应的常见设备？**
    *   物理层：集线器 (Hub)、中继器。
    *   数据链路层：交换机 (Switch)、网桥。
    *   网络层：路由器 (Router)。
    *   传输层/应用层：网关、防火墙。
3.  **每层对应的数据单元名称？**
    *   物理层：比特 (Bit)。
    *   链路层：帧 (Frame)。
    *   网络层：包 (Packet)。
    *   传输层：段 (Segment) / 报文 (Datagram)。
    *   应用层：消息 (Message)。
4.  **MAC 地址和 IP 地址的区别？**
    *   MAC：物理地址，全球唯一，不可变（理论上），局域网通信。
    *   IP：逻辑地址，可变，广域网寻址。
5.  **ARP 协议的作用？** 地址解析协议，将 IP 地址解析为 MAC 地址。
6.  **RARP 协议？** 反向 ARP，MAC 转 IP（现少用，DHCP 替代）。
7.  **什么是 MTU？** 最大传输单元 (Maximum Transmission Unit)，以太网默认 1500 字节。
8.  **什么是 MSS？** 最大报文段长度 (Maximum Segment Size)，TCP 为了避免 IP 分片设定的，通常是 MTU - IP头 - TCP头 = 1460。
9.  **为什么需要 IP 分片？** 数据包超过 MTU，路由器会将其切分，接收端重组。
10. **网关 (Gateway) 的作用？** 连接两个不同网络协议的设备，通常指默认路由出口。

#### 2. IP 协议与子网
11. **IPv4 和 IPv6 的区别？**
    *   地址长度：32位 vs 128位。
    *   配置：IPv6 支持无状态自动配置 (SLAAC)。
    *   头部：IPv6 头部更简单固定。
12. **A/B/C/D/E 类 IP 地址范围？**
    *   A: 1.0.0.0 - 126.x.x.x (大型网络)。
    *   B: 128.0.0.0 - 191.x.x.x (中型)。
    *   C: 192.0.0.0 - 223.x.x.x (小型)。
    *   D: 组播。E: 保留。
13. **私有 IP 地址范围？**
    *   10.0.0.0/8
    *   172.16.0.0/12
    *   192.168.0.0/16
14. **子网掩码 (Subnet Mask) 的作用？** 区分 IP 地址中的网络号和主机号。
15. **什么是 CIDR (无类别域间路由)？** 使用 `/24` 这种前缀长度表示法代替分类地址，灵活划分子网。
16. **广播 (Broadcast) vs 组播 (Multicast) vs 单播 (Unicast)？**
    *   单播：1对1。
    *   广播：1对全（局域网内）。
    *   组播：1对多（特定组）。
17. **ICMP 协议的作用？** 互联网控制报文协议，用于 Ping (Echo Request/Reply) 和 Traceroute (TTL 超时)。
18. **TTL (Time To Live) 的作用？** 防止数据包在网络中无限循环，每经过一个路由器减 1，为 0 时丢弃并回发 ICMP 超时。
19. **Loopback 地址 (127.0.0.1) 和 0.0.0.0 的区别？**
    *   127.0.0.1：回环接口，仅本机访问。
    *   0.0.0.0：监听所有网卡接口（绑定时）；或者表示“本网络中的本主机”（源地址）。
20. **NAT (网络地址转换) 的作用？** 解决 IPv4 不足，内网多台主机共用一个公网 IP 上网。

#### 3. TCP/UDP 传输层
21. **TCP 和 UDP 的核心区别？**
    *   TCP：面向连接、可靠、有序、流式、慢（重传/握手）。应用：Web, Email, File.
    *   UDP：无连接、不可靠、乱序、报文式、快。应用：DNS, 直播, 游戏, QUIC.
22. **TCP 报头包含哪些信息？** 源/目端口、序号(Seq)、确认号(Ack)、标识位(SYN/ACK/FIN/RST/PSH/URG)、窗口大小、校验和。
23. **TCP 如何保证可靠性？** 校验和、序列号/确认应答、超时重传、连接管理、流量控制、拥塞控制。
24. **什么是滑动窗口 (Sliding Window)？** 允许发送方在未收到 ACK 前连续发送多个包，提高吞吐量。
25. **流量控制 (Flow Control)？** 接收方通过通告窗口大小 (Window Size) 告诉发送方自己还能接收多少数据，防止发太快淹没接收方。
26. **拥塞控制 (Congestion Control)？** 防止网络拥堵。算法：慢启动、拥塞避免、快重传、快恢复。
27. **慢启动 (Slow Start)？** 连接刚建立时，拥塞窗口 (cwnd) 指数增长，直到阈值 (ssthresh)。
28. **快重传 (Fast Retransmit)？** 收到 3 个重复 ACK，立即重传丢失的包，不必等超时。
29. **RST (Reset) 标志位的作用？** 异常强制断开连接（如端口未监听、请求超时）。
30. **URG (Urgent) 标志位？** 紧急指针有效，高优先级数据（如 Telnet 中断指令）。

#### 4. 高级 TCP 特性
31. **TCP 半连接队列满会发生什么？** 丢弃 SYN 包（客户端重发）或 发送 SYN Cookies（若开启）。
32. **SYN Flood 攻击原理？** 伪造 IP 发送 SYN 不回 ACK，耗尽服务端半连接队列。
33. **TCP Fast Open (TFO)？** 在第一次握手的 SYN 包中就携带数据（需之前建立过连接存 Cookie），减少 RTT。
34. **TCP KeepAlive 机制？** 长时间无数据传输，内核发送探测包。
35. **Nagle 算法？** 减少小包发送，攒够 MSS 或收到 ACK 再发。影响实时性 (`TCP_NODELAY` 关闭)。
36. **Delayed ACK？** 延迟发送 ACK，等待捎带数据。
37. **TCP 粘包/拆包本质？** TCP 是流协议，无边界。业务层需定义边界（长度头/分隔符）。
38. **TIME_WAIT 状态过多的危害？** 占用端口，导致无法新建连接。
39. **如何减少 TIME_WAIT？** 客户端长连接、服务端允许重用 (`tcp_tw_reuse` timestamp 必须开启)。
40. **CLOSE_WAIT 状态过多的原因？** 被动关闭方收到 FIN 后，代码没有调用 `close()` 关闭 Socket（通常是代码 Bug）。

#### 5. DNS (域名系统)
41. **DNS 解析流程？** 浏览器缓存 -> 操作系统缓存(hosts) -> 本地 DNS (LDNS) -> 根域名 -> 顶级域名 -> 权威域名。
42. **DNS 查询类型？** 递归查询 (LDNS 帮查到底) vs 迭代查询 (由上级指引去哪查)。
43. **DNS 记录类型？** A (IPv4), AAAA (IPv6), CNAME (别名), MX (邮件), NS (名字服务器), TXT (文本)。
44. **DNS 使用 TCP 还是 UDP？**
    *   查询/响应：默认 UDP (512字节限制，EDNS 可扩)。
    *   区域传输 (Zone Transfer)：TCP (主从同步)。
45. **DNS 劫持 vs DNS 污染？** 劫持是修改你的 DNS Server 指向；污染是路径上抢答伪造包。
46. **DoH (DNS over HTTPS) 和 DoT (DNS over TLS)？** 加密 DNS 流量，防止监听和篡改。
47. **CNAME 和显性 URL 转发的区别？** CNAME 是 DNS 层面别名；URL 转发是 HTTP 301 跳转。
48. **什么是 hosts 文件？** 本地静态 DNS 映射表，优先级高于 DNS 服务器。
49. **TTL 在 DNS 中的含义？** 域名解析记录在 DNS 服务器缓存的时间。
50. **泛域名解析？** `*.example.com` 解析到同一 IP。

#### 6. Socket 编程
51. **Socket 是什么？** 应用层与 TCP/IP 协议族通信的中间软件抽象层 (Handle)。
52. **Socket 五元组？** 源 IP、源端口、目的 IP、目的端口、协议。
53. **Socket 系统调用流程 (TCP Server)？** socket() -> bind() -> listen() -> accept() -> read/write() -> close()。
54. **Socket 系统调用流程 (TCP Client)？** socket() -> connect() -> write/read() -> close()。
55. **UDP Socket 流程？** socket() -> bind() (可选) -> sendto() / recvfrom()。
56. **阻塞 IO vs 非阻塞 IO？**
    *   阻塞：数据未就绪，进程挂起等待。
    *   非阻塞：立即返回错误 (EAGAIN)，需轮询。
57. **Select/Poll/Epoll 区别？**
    *   Select: 轮询所有 fd，有最大限制 (1024)，O(N)。
    *   Poll: 链表存储，无限制，O(N)。
    *   Epoll: 事件驱动，回调通知，O(1)。
58. **Epoll 的 LT (水平触发) 和 ET (边缘触发)？**
    *   LT: 只要有数据就一直通知 (默认，安全)。
    *   ET: 状态变化时只通知一次 (高效，需一次读完)。
59. **什么是 Unix Domain Socket？** 同一台主机进程间通信，不走网络协议栈，效率高。
60. **端口号范围？** 0-65535。0-1023 保留端口 (Root 权限)，1024-49151 注册端口，49152-65535 动态端口。

#### 7. 路由与交换
61. **路由表包含什么？** 目的网络、掩码、下一跳 (Next Hop)、接口、Metric。
62. **静态路由 vs 动态路由？** 手动配置 vs 协议自动学习 (RIP, OSPF, BGP)。
63. **RIP (路由信息协议)？** 基于距离矢量，跳数限制 15。
64. **OSPF (开放最短路径优先)？** 基于链路状态，Dijkstra 算法，适合大型网络。
65. **BGP (边界网关协议)？** 互联网核心协议，自治系统 (AS) 之间的路由。
66. **VLAN (虚拟局域网)？** 在交换机上逻辑隔离广播域。
67. **Trunk 口和 Access 口？** Trunk 允许多个 VLAN 通过 (打标签)，Access 连接终端。
68. **三层交换机？** 具备路由功能的交换机。
69. **VPN (虚拟专用网) 原理？** 隧道技术 (Tunneling) + 加密。常见协议：IPSec, L2TP, OpenVPN, WireGuard。
70. **SD-WAN？** 软件定义广域网，智能调度线路 (MPLS, 4G, Internet)。

#### 8. 工具与排查
71. **`ping` 原理？** 发送 ICMP Echo Request，接收 Echo Reply。排查连通性。
72. **`traceroute` (tracert) 原理？** 发送 TTL=1, 2, 3... 的 UDP/ICMP 包，路径路由器返回 ICMP Time Exceeded。
73. **`netstat` 常用参数？** `-t` (TCP), `-u` (UDP), `-l` (Listen), `-n` (Numeric), `-p` (Process)。`netstat -tlnp`。
74. **`ss` 命令？** 比 netstat 更快，直接读内核信息。`ss -lnt`。
75. **`tcpdump`？** 命令行抓包工具。`tcpdump -i eth0 port 80 -w out.cap`。
76. **`lsof`？** 列出打开的文件 (包括 Socket)。`lsof -i :80`。
77. **`curl` 命令？** 发送 HTTP 请求。`-v` (详细), `-I` (Header), `-L` (跟随跳转)。
78. **`telnet`？** 测试端口连通性。`telnet IP Port`。
79. **`nslookup` / `dig`？** 查询 DNS 解析。
80. **`route` / `ip route`？** 查看和配置路由表。

#### 9. 新技术与趋势
81. **IPv6 普及度？** 移动端、IoT 普及度高。
82. **QUIC 协议？** Google 提出，基于 UDP，解决 TCP 队头阻塞，HTTP/3 基础。
83. **MPTCP (多路径 TCP)？** 同时使用 WiFi 和 4G 传输数据，增加带宽和可靠性 (Siri 使用)。
84. **WebSocket？** 全双工，单一 TCP 连接，适合即时通讯。
85. **CDN 原理？** 智能 DNS 解析到最近边缘节点，缓存静态资源。
86. **Anycast (任播)？** 多个地理位置不同的服务器使用同一个 IP，路由到最近节点 (DNS 根服务器常用)。
87. **Overlay Network (覆盖网络)？** 建立在现有网络之上的虚拟网络 (如 VXLAN, P2P)。
88. **Zero Trust (零信任) 网络？** 不信任内网，所有访问需认证授权。
89. **HTTP/2 的帧 (Frame)？** DATA 帧、HEADERS 帧、SETTINGS 帧等。
90. **Wi-Fi 6 (802.11ax)？** 高并发，OFDMA 技术。

#### 10. 综合场景
91. **输入 URL 到页面展示的全过程？** DNS -> TCP 握手 -> TLS 握手 -> HTTP 请求 -> 后端处理 -> HTTP 响应 -> 浏览器渲染 -> 资源加载 -> 四次挥手。
92. **服务器无法被 Ping 通可能原因？** 防火墙 (iptables/Security Group) 禁 Ping，路由不可达，服务器宕机。
93. **127.0.0.1 通但本机局域网 IP 不通？** 防火墙拦截，服务只监听了 127.0.0.1 (未 bind 0.0.0.0)。
94. **网络抖动怎么查？** `ping` 持续观察丢包率，`mtr` (My Traceroute) 查看路径哪一跳丢包。
95. **如何实现内网穿透？** FRP, Ngrok, SSH Tunnel。
96. **UDP 如何实现可靠传输？** 应用层实现 ACK、重传、序号 (如 KCP, QUIC)。
97. **什么是网络风暴？** 广播帧充斥网络，消耗带宽和 CPU (环路导致)。解决：STP (生成树协议)。
98. **DHCP 过程？** Discover -> Offer -> Request -> Acknowledge (DORA)。
99. **代理 (Proxy) vs 反向代理 (Reverse Proxy)？**
    *   正向代理：代表客户端上网 (VPN, 梯子)。
    *   反向代理：代表服务端接收请求 (Nginx, LB)。
100. **MTU 不匹配会导致什么？** 某些网站打不开 (大包被丢弃)，PMTU 发现机制失效。