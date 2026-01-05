# PHP Workerman 面试题库大全 (精选 200 题)

### 1. 基础概念与原理 (Basics)
1.  **Workerman 是什么？**
    *   一款纯 PHP 开发的开源高性能异步 PHP socket 框架。支持 TCP/UDP/WebSocket/HTTP 等协议。
2.  **Workerman 与传统 PHP-FPM 的主要区别？**
    *   **运行模式**：Workerman 是常驻内存 (CLI 模式)；PHP-FPM 是请求结束释放资源。
    *   **协议**：Workerman 支持多种协议；PHP-FPM 主要处理 HTTP (FastCGI)。
    *   **性能**：Workerman 避免了重复加载文件和初始化，性能更高。
3.  **Workerman 需要安装扩展吗？**
    *   核心功能不依赖扩展 (使用 PHP 原生 Stream Socket)。建议安装 `event` 或 `libevent` 扩展以支持海量并发。
4.  **Workerman 的运行流程是怎样的？**
    *   启动 Master 进程 -> 解析配置 -> Fork 出 Worker 进程 -> Worker 进程监听端口并处理业务。
    *   **流程图解**：
    ```mermaid
    graph TD
        Start[CLI 启动 php start.php start] --> Init[初始化环境 / 解析配置]
        Init --> Daemon{是否守护进程 -d?}
        Daemon -- Yes --> ForkMaster[Fork & Detach]
        ForkMaster --> MasterRunning[Master 进程启动]
        Daemon -- No --> MasterRunning
        
        subgraph MasterProcess [Master 进程]
            MasterRunning --> CreateSocket[创建监听 Socket]
            CreateSocket --> ForkWorkers[Fork Worker 进程]
            ForkWorkers --> MasterLoop[Master 监控循环]
            MasterLoop -- 收到 SIGCHLD --> ReloadWorker[回收 & 重启 Worker]
            ReloadWorker --> ForkWorkers
            MasterLoop -- 收到 Stop 信号 --> StopWorkers[停止 Worker & 退出]
        end
        
        ForkWorkers -- Fork --> WorkerStart
        
        subgraph WorkerProcess [Worker 进程]
            WorkerStart[Worker 启动] --> UserSetup[降权 / 设置用户]
            UserSetup --> CallWorkerStart[回调 onWorkerStart]
            CallWorkerStart --> AddEvent[添加 Socket 到 EventLoop]
            AddEvent --> Loop{Event Loop 监听事件}
            
            Loop -- 监听端口可读 --> Accept[Accept 建立连接]
            Accept --> OnConnect[回调 onConnect]
            OnConnect --> Loop
            
            Loop -- 客户端可读 --> Read[SysRead 读取 TCP 数据]
            Read --> Protocol{协议层解包}
            Protocol -- 包不完整 --> WaitMore[等待更多数据]
            WaitMore --> Loop
            Protocol -- 包完整 --> Decode[Decode & 回调 onMessage]
            Decode --> Logic[业务逻辑处理]
            Logic -- Send --> Encode[Encode & SysWrite]
            Encode --> Loop
            
            Loop -- 客户端关闭 --> Close[Close 连接]
            Close --> OnClose[回调 onClose]
        end
    ```
5.  **什么是 Master 进程？**
    *   主进程，负责监控 Worker 进程的状态。当 Worker 进程退出时，Master 会自动 Fork 新的 Worker。Master 不处理业务。
6.  **什么是 Worker 进程？**
    *   子进程，负责监听端口、接收连接、读取数据、处理业务逻辑。
7.  **Workerman 支持 Windows 吗？**
    *   支持，但在 Windows 下不支持多进程 (只能开一个 Worker)，主要用于开发调试。生产环境建议 Linux。
8.  **Workerman 如何实现异步非阻塞？**
    *   利用 PHP 的 `stream_select` (默认) 或 `event` 扩展 (Epoll) 实现 IO 多路复用。
9.  **Workerman 的 `count` 参数有什么用？**
    *   设置当前 Worker 实例启动多少个进程。通常设置为 CPU 核数的 1-4 倍。
10. **如何平滑重启 Workerman？**
    *   运行 `php start.php reload`。Master 会逐个向 Worker 发送信号，Worker 处理完当前请求后退出，Master 启动新 Worker。

### 2. 核心组件与配置 (Components)
11. **`Worker` 类是做什么的？**
    *   核心容器类，用于创建服务监听端口。如 `$worker = new Worker('tcp://0.0.0.0:1234');`。
12. **`Connection` 对象代表什么？**
    *   代表一个客户端连接。通过它可以向客户端发送数据 (`$connection->send()`) 或关闭连接 (`$connection->close()`)。
13. **Workerman 支持哪些事件回调？**
    *   `onWorkerStart`: 进程启动时。
    *   `onConnect`: 客户端连接时。
    *   `onMessage`: 收到数据时。
    *   `onClose`: 连接关闭时。
    *   `onWorkerStop`: 进程退出时。
14. **`onWorkerStart` 常用于做什么？**
    *   初始化数据库连接、Redis 连接、定时器等。因为只在进程启动时执行一次，避免重复连接。
15. **如何在 Workerman 中使用定时器？**
    *   使用 `Workerman\Lib\Timer` 类。`Timer::add($interval, callback)`。
16. **定时器是基于什么实现的？**
    *   基于 `pcntl_alarm` (低精度) 或 Event Loop (高精度，如 Epoll 超时机制)。
17. **Workerman 的日志文件在哪里配置？**
    *   `Worker::$logFile = '/tmp/workerman.log';`
18. **如何设置 Worker 进程的运行用户？**
    *   `$worker->user = 'www-data';` (需要 Root 启动)。
19. **`stdoutFile` 的作用？**
    *   将 `echo`, `var_dump` 等标准输出重定向到指定文件，用于守护进程模式下的调试。
20. **如何设置 PID 文件路径？**
    *   `Worker::$pidFile = '/var/run/workerman.pid';`

### 3. 协议与通信 (Protocols)
21. **Workerman 默认支持哪些协议？**
    *   HTTP, WebSocket, TCP, UDP, Text (以换行符结尾), Frame (二进制流)。
22. **如何自定义协议？**
    *   创建一个类实现 `ProtocolInterface`，实现 `input` (分包), `encode` (打包), `decode` (解包) 方法。
23. **Text 协议是什么？**
    *   简单的文本协议，每个数据包以 `\n` 结尾。适合 Telnet 调试。
24. **Frame 协议是什么？**
    *   简单的二进制协议，头部 4 字节表示包长。解决 TCP 粘包问题。
25. **WebSocket 协议在 Workerman 中如何使用？**
    *   `new Worker('websocket://0.0.0.0:8080');`
26. **如何在 WebSocket 握手时验证用户？**
    *   设置 `onWebSocketConnect` 回调，检查 `$_GET` 或 `$_COOKIE`，不合法则 `$connection->close()`。
27. **TCP 粘包 (Stick Package) 是什么？**
    *   发送方将多个小包合并发送，或接收方未及时读取导致缓冲区堆积，导致一次读到多个包。
28. **Workerman 如何解决 TCP 粘包？**
    *   使用固定包头协议 (如 Frame) 或 分隔符协议 (如 Text)。协议类的 `input` 方法返回包长，框架自动截取。
29. **Heartbeat (心跳) 的作用？**
    *   检测死连接 (如断网未发 FIN 包)，释放服务端资源。
    *   防止防火墙/路由器的 NAT 超时 (连接老化)。
30. **如何在 Workerman 中实现心跳？**
    *   在 `onMessage` 更新该连接的最后通信时间。
    *   Master 或 Worker 定时检查所有连接，超过规定时间未通信则关闭。

### 4. 高级特性 (Advanced)
31. **GatewayWorker 是什么？**
    *   基于 Workerman 开发的一套 TCP 长连接应用框架。将业务逻辑 (BusinessWorker) 和 连接维持 (Gateway) 分离。
32. **GatewayWorker 的架构优势？**
    *   **Gateway**：只负责网络 IO，维持连接，无状态，可水平扩展。
    *   **BusinessWorker**：只负责业务逻辑，挂了不掉线。
    *   **Register**：服务注册中心。
    *   **架构图解**：
    ```mermaid
    graph LR
        Client[客户端 Client] -- TCP 长连接 --> Gateway[Gateway 进程组]
        
        subgraph Cluster [GatewayWorker 集群]
            Register[Register 服务注册中心]
            
            subgraph Gateways [Gateway 层]
                G1[Gateway Node 1]
                G2[Gateway Node 2]
            end
            
            subgraph Workers [BusinessWorker 层]
                BW1[BusinessWorker Node 1]
                BW2[BusinessWorker Node 2]
            end
            
            G1 -- 注册/拉取 IP --> Register
            G2 -- 注册/拉取 IP --> Register
            BW1 -- 注册 --> Register
            BW2 -- 注册 --> Register
            
            G1 <== 内部通讯 ==> BW1
            G1 <== 内部通讯 ==> BW2
            G2 <== 内部通讯 ==> BW1
            G2 <== 内部通讯 ==> BW2
        end
        
        BW1 -.-> DB[(MySQL/Redis)]
    ```
33. **Workerman 如何支持 SSL/HTTPS？**
    *   在构造 Worker 时传入 `$context` 选项，配置 `ssl` 证书路径，并开启 `transport => 'ssl'`。
34. **Workerman 支持协程 (Coroutine) 吗？**
    *   原生不支持 (是多进程同步阻塞/异步回调模式)。但可以配合 `Swoole` 或 `Revolt` (Fiber) 使用。Workerman v5 尝试引入协程。
35. **GlobalData 组件是什么？**
    *   Workerman 提供的一个简单的分布式变量共享组件 (类似精简版 Redis)。
36. **Channel 组件是什么？**
    *   基于订阅/发布的分布式通讯组件，用于不同进程/服务器间通讯。
37. **AsyncTcpConnection 是什么？**
    *   作为客户端异步连接其他 Server (如连接 WebSocket 服务端、MySQL、Redis)。
38. **如何在 Workerman 中发送异步 HTTP 请求？**
    *   使用 `AsyncTcpConnection` 包装 HTTP 协议，或者使用 `workerman/http-client` 组件。
39. **Workerman 如何处理文件上传？**
    *   HTTP 协议下，Workerman 会解析 `multipart/form-data`，文件保存在临时目录，通过 `$_FILES` 访问。
40. **如何实现分布式部署？**
    *   使用 GatewayWorker。Gateway 部署在多台机器，BusinessWorker 也可以部署在多台机器，通过 Register 组网。

### 5. 数据库与存储 (Database)
41. **Workerman 中可以直接用 `mysqli` 或 `PDO` 吗？**
    *   可以，但要注意连接断开问题 (MySQL wait_timeout)。需捕获 `2006` 错误并重连。
42. **为什么在 `onWorkerStart` 中初始化数据库？**
    *   避免每个请求都建立连接 (短连接)。每个进程只建立一个复用连接 (长连接)。
43. **Workerman 中操作 Redis？**
    *   推荐使用 `Redis` 扩展或 `Predis`。同样建议长连接。
44. **Workerman 中数据库查询阻塞会怎样？**
    *   会阻塞当前进程，导致无法处理其他连接的请求。
45. **如何解决数据库阻塞问题？**
    *   多开 Worker 进程 (增加并发度)。
    *   使用异步数据库客户端 (如 `react/mysql`，但在 PHP 中生态不如同步成熟)。
    *   将耗时任务投递到 Task 进程或消息队列。
46. **MySQL 连接超时断开怎么处理？**
    *   设置定时器定时发送 `select 1` 心跳。
    *   或者封装 DB 类，在 query 失败时检测 error code，自动重连。
47. **Workerman 支持连接池吗？**
    *   每个进程持有一个连接，本身就是一种进程级连接池。多进程 = 连接池。
48. **`workerman/mysql` 组件？**
    *   官方提供的简单 MySQL 封装，具有自动重连功能。
49. **GatewayWorker 中的 `$_SESSION`？**
    *   存储在 Gateway 进程内存中。BusinessWorker 想要修改 Session，会通过内部通讯同步给 Gateway。
50. **GatewayWorker 重启 Gateway 会丢失 Session 吗？**
    *   会。如果是分布式部署，可以自行将会话存储到 Redis。

### 6. 框架集成 (Integration)
51. **Workerman 可以集成到 Laravel 吗？**
    *   可以。如 `laravel-s` 或自行编写 Command 启动 Workerman。
52. **ThinkPHP 5/6 集成 Workerman？**
    *   TP 官方提供了 `think-worker` 扩展，一键启动 Workerman http server。
53. **Workerman 如何运行 Laravel 项目？**
    *   监听 HTTP 端口，在 `onMessage` 中加载 `public/index.php` (需改造)，模拟 Request/Response。
54. **集成框架时的静态资源处理？**
    *   Workerman 对静态文件支持较弱，建议 Nginx 反向代理处理静态资源，PHP 处理动态请求。
55. **Workerman 与 Nginx 的配合？**
    *   Nginx (80) -> Proxy Pass -> Workerman (8080)。
    *   Nginx 负责 SSL、Gzip、负载均衡、静态文件。
56. **如何获取客户端真实 IP (经过 Nginx)？**
    *   Nginx 设置 `X-Real-IP` 或 `X-Forwarded-For`。
    *   Workerman 中读取 `$_SERVER['HTTP_X_REAL_IP']`。
57. **在 Laravel 中使用 GatewayWorker？**
    *   可以将 GatewayWorker 作为独立服务运行，Laravel 通过 `GatewayClient` 向客户端推送消息。
58. **`GatewayClient` 是什么？**
    *   一个 PHP 客户端库，不需要 Workerman 环境，可以在 FPM (普通 Web 环境) 中调用 Gateway 服务推送消息。

### 7. 性能与调优 (Performance)
59. **Workerman 性能瓶颈主要在哪里？**
    *   业务逻辑中的同步阻塞 IO (DB, Curl)。
    *   CPU 密集型计算。
60. **如何设置合理的进程数？**
    *   IO 密集型 (DB 操作多)：CPU 核数 * 3 ~ 8。
    *   CPU 密集型 (计算多)：CPU 核数。
61. **Event 扩展和 Libevent 扩展的区别？**
    *   `Event` 是较新的扩展，支持更多特性。两者都能提供 Epoll 支持，显著提高并发能力。
62. **如何处理 100 万并发连接？**
    *   安装 Event 扩展。
    *   优化 Linux 内核参数 (文件句柄数 `ulimit -n`，TCP 缓冲区等)。
    *   增加内存 (每个连接占用一定内存)。
    *   GatewayWorker 分布式部署 Gateway。
63. **内存泄漏 (Memory Leak) 怎么排查？**
    *   观察内存占用是否持续增长。
    *   检查全局数组 (`global`, `static`) 是否只增不减。
    *   设置 `Worker::$max_request`，处理一定请求后自动重启释放内存。
64. **`max_request` 的作用？**
    *   进程处理多少个请求后自动退出。Master 会拉起新进程。防止内存泄漏。
65. **僵尸进程 (Zombie Process) 是怎么产生的？**
    *   子进程退出，父进程没有 wait 回收。Workerman Master 会自动处理 SIGCHLD 信号回收子进程。
66. **如何查看当前 Workerman 的状态？**
    *   `php start.php status`。
67. **`status` 命令输出的 `connections` 是什么？**
    *   当前保持的 TCP 连接数。
68. **`status` 命令输出的 `failures` 是什么？**
    *   失败次数 (如 accept 失败)，通常意味着系统负载过高或文件句柄不够。
69. **Workerman 能利用多核吗？**
    *   能。通过多进程模型利用多核。
70. **惊群效应 (Thundering Herd)？**
    *   多个进程同时监听同一个端口，新连接到来唤醒所有进程。Workerman 通过设置 `reusePort` (需 OS 支持) 或 Master 负责 Accept 转发 (旧版) 解决。

### 8. 故障排查 (Troubleshooting)
71. **启动报错 `Address already in use`？**
    *   端口被占用。`lsof -i:端口` 查看占用进程并 kill，或换端口。
72. **客户端连接不上？**
    *   检查防火墙 (iptables/firewalld/安全组)。
    *   检查监听 IP 是否是 `0.0.0.0` (如果是 `127.0.0.1` 只能本机连)。
73. **连接建立后立即断开？**
    *   可能协议不匹配 (如客户端用 WebSocket 连 TCP Server)。
    *   业务逻辑主动 `$connection->close()`。
74. **出现 `send buffer full` 错误？**
    *   发送数据太快，客户端接收太慢 (网络差)，导致发送缓冲区满。
    *   `onBufferFull` 回调可以处理 (暂停发送)。
75. **Worker 进程频繁退出 (exit with status 65280)？**
    *   业务代码有 Fatal Error。查看 `workerman.log` 或 `php_error.log`。
76. **Reload 无效？**
    *   代码有语法错误，导致新进程启动失败。
    *   PID 文件丢失，无法向旧进程发信号。
77. **GatewayWorker 报错 `Time wait`？**
    *   端口耗尽或 TCP 连接处于 TIME_WAIT 状态过多。
78. **如何调试 Workerman 代码？**
    *   `echo`/`var_dump` (如果是 `-d` 守护模式需看日志)。
    *   Xdebug (较难配置 remote debug)。
    *   日志打点。
79. **浏览器访问 WebSocket 报 404？**
    *   WebSocket 握手也是 HTTP 请求，如果 Server 端判断路径不对可能会关闭连接。
80. **Curl 请求 Workerman HTTP 接口超时？**
    *   Workerman 进程被阻塞。或者防火墙拦截。

### 9. 场景与实战 (Scenarios)
81. **Workerman 适合做什么？**
    *   即时通讯 (IM)、游戏服务器、物联网 (IoT)、消息推送、API 接口。
82. **Workerman 不适合做什么？**
    *   极其复杂的网页逻辑 (虽然能做，但不如 Laravel/TP + Nginx 方便)。
    *   强 CPU 密集型计算 (会阻塞 Loop)。
83. **如何实现群聊功能？**
    *   维护一个群组 ID 到 连接对象列表的映射数组。
    *   收到消息循环 `$group_connections` 发送。
    *   GatewayWorker 中使用 `Gateway::sendToGroup($group_id, $msg)`。
84. **如何实现单对单私聊？**
    *   维护 `User_ID => Connection` 映射。
    *   GatewayWorker 使用 `Gateway::bindUid($client_id, $uid)` 和 `Gateway::sendToUid($uid, $msg)`。
85. **直播间弹幕系统怎么设计？**
    *   WebSocket 协议。
    *   GatewayWorker 加入 Group (房间号)。
    *   前端定时心跳。
    *   消息积压处理 (丢弃非重要弹幕)。
86. **IoT 设备数据采集 (TCP/Hex)？**
    *   使用自定义协议 (Protocol)，解析二进制流。
    *   处理粘包。
    *   心跳维持。
87. **Workerman 做 HTTP 接口性能如何？**
    *   通常比 PHP-FPM 高几倍 (Hello World 压测)，因为无启动开销。但业务逻辑复杂后差距缩小。
88. **如何在 Workerman 中发送邮件？**
    *   不要同步发。投递到 Redis 队列，另起 Task 进程消费队列异步发送。
89. **游戏服务器的状态同步？**
    *   帧同步或状态同步。需要高频广播 (如每秒 20 次)。
90. **Workerman 支持 UDP 广播吗？**
    *   支持。`new Worker('udp://0.0.0.0:1234')`。

### 10. 对比 Swoole
91. **Workerman 和 Swoole 的核心区别？**
    *   **Workerman**：纯 PHP 代码实现，依赖 Event 扩展 (可选)，代码可读性强，稳健，门槛低。
    *   **Swoole**：C/C++ 编写的 PHP 扩展，性能更极致，功能更强大 (协程)，门槛较高，安装复杂。
92. **性能对比？**
    *   Swoole 通常更高，但 Workerman 配合 Event 扩展后，在 IO 密集型场景下差异不大。
93. **开发难度？**
    *   Workerman 更像写原生 PHP。Swoole 需要理解更多底层概念。
94. **协程支持？**
    *   Swoole 原生支持协程。Workerman 目前主要靠 Fiber (v5) 或 Swow。
95. **生态圈？**
    *   Swoole 生态更丰富 (Hyperf, Swoft)。Workerman 也有 Webman (性能很强)。
96. **部署难度？**
    *   Workerman 只要有 PHP 就能跑。Swoole 需编译扩展，还要解决扩展冲突。
97. **Windows 支持？**
    *   Workerman 支持 (受限)。Swoole 不支持 Windows (需 WSL)。
98. **Webman 是什么？**
    *   基于 Workerman 的高性能 HTTP 框架，性能对标 Go Gin，远超 Laravel。
99. **Workerman 可以用 Swoole 作为底层驱动吗？**
    *   可以 (Swoole 作为 Event Loop 实现)。
100. **选择 Workerman 还是 Swoole？**
    *   追求极致性能、微服务、协程 -> Swoole (Hyperf)。
    *   追求稳健、简单、快速开发、IoT/IM -> Workerman (Webman)。

### 11. 深入原理与源码 (Internals)
101. **Workerman 的 `reusePort` (端口复用) 是如何工作的？**
    *   利用 Linux Kernel 3.9+ 的 `SO_REUSEPORT` 选项。允许多个进程监听同一个 IP:Port，由内核负责负载均衡 (唤醒哪个进程)。
102. **如果没有 `reusePort`，Workerman 如何实现多进程监听？**
    *   Master 进程创建 Socket，Fork 子进程时 Socket 句柄被继承，所有子进程共享同一个 Socket 监听。但会有惊群问题。
103. **Workerman 如何处理信号 (Signal)？**
    *   使用 `pcntl_signal` 注册信号处理函数 (如 `SIGINT`, `SIGTERM`, `SIGUSR1`)。
    *   Master 收到 `SIGINT` 会通知所有 Worker 退出。
104. **Workerman 的 Event Loop 是如何抽象的？**
    *   定义了 `Events\EventInterface` 接口 (add, del, loop)。
    *   适配了 Select, Libevent, Event, Swow 等驱动。
105. **定时器 ID 是什么？**
    *   一个整数。`Timer::add` 返回该 ID，`Timer::del($id)` 用于取消。
106. **Worker 进程如何与 Master 进程通信？**
    *   主要通过**信号**。Master 发信号控制 Worker。
    *   Worker 退出时通过管道或退出状态码告知 Master。
107. **Workerman 的文件监控 (FileMonitor) 原理？**
    *   在开发模式下，额外启动一个 FileMonitor 进程，定时扫描文件 `mtime`，变化则发送 Reload 信号。
108. **`onBufferDrain` 回调什么时候触发？**
    *   当应用层发送缓冲区数据被写完 (变空) 时触发。通常用于在大流量发送后恢复发送。
109. **`onBufferFull` 默认阈值是多少？**
    *   `TcpConnection::$maxSendBufferSize`，默认 1MB。
110. **Workerman 的 `fork` 是全量复制吗？**
    *   Linux 下是写时复制 (Copy On Write, COW)。Fork 瞬间内存开销很小，只有修改内存时才真正复制。

### 12. GatewayWorker 进阶
111. **Gateway 和 BusinessWorker 之间是如何通信的？**
    *   也是通过 TCP 长连接。内部自定义了通讯协议。
112. **Register 服务的作用是什么？**
    *   简单的服务注册发现。Gateway 和 BusinessWorker 启动时都连 Register，交换 IP 地址，然后建立直连。
113. **GatewayWorker 如何实现全员广播？**
    *   `Gateway::sendToAll($msg)`。BusinessWorker 发给所有 Gateway，Gateway 再发给所有 Client。
114. **`Gateway::isOnline($client_id)` 准确吗？**
    *   准确。基于 Gateway 内存中的连接状态。
115. **BusinessWorker 进程数设置多少合适？**
    *   看业务复杂度。CPU 密集型设 CPU 核数，IO 密集型设 CPU * 2~4。
116. **Gateway 进程数设置多少合适？**
    *   Gateway 主要做转发，IO 密集但 CPU 消耗低。通常 CPU 核数即可，或者更少。
117. **如何自定义 GatewayWorker 的内部通讯端口？**
    *   在 `start_gateway.php` 和 `start_businessworker.php` 中配置 `lanIp` 和端口范围。
118. **GatewayWorker 支持 WebSocket 子协议吗？**
    *   支持。在构造 Gateway 时设置。
119. **如何踢掉某个用户？**
    *   `Gateway::closeClient($client_id)`。
120. **如何判断用户是否在某个群组？**
    *   `Gateway::getClientSessionsByGroup($group_id)` 获取群组所有 Session，自行判断 (效率较低)。通常建议业务层自己维护群成员列表 (Redis)。

### 13. 安全 (Security)
121. **如何开启 SSL/TLS (HTTPS/WSS)？**
    *   `$context = ['ssl' => ['local_cert' => '...', 'local_pk' => '...']];`
    *   `$worker->transport = 'ssl';`
122. **WebSocket 如何防止 CSRF (跨站请求伪造)？**
    *   在 `onWebSocketConnect` 中检查 `$_SERVER['HTTP_ORIGIN']`。
123. **如何限制单个 IP 的连接数？**
    *   在 `onConnect` 中记录 IP 连接数 (存 Redis/GlobalData)，超限则关闭。
124. **如何防止恶意刷包 (DDoS)？**
    *   统计单位时间内的包量。超频则拉黑 IP 或断开连接。
125. **Workerman 运行权限问题？**
    *   不建议以 root 运行。启动后降权 (`$worker->user`)。
126. **敏感数据加密传输？**
    *   使用 SSL (WSS) 保证链路安全。
    *   业务数据内部再做 AES 加密。
127. **SQL 注入防范？**
    *   Workerman 只是容器，防注入靠业务代码 (使用 PDO 预处理)。
128. **XSS 攻击防范？**
    *   聊天室场景，对用户发送的消息进行 `htmlspecialchars` 转义。
129. **如何隐藏 Workerman 的报错信息？**
    *   `ini_set('display_errors', 'off');`
    *   守护进程模式下默认不输出到屏幕。
130. **Webman 的安全中间件？**
    *   Webman 提供了 RateLimiter (限流) 等中间件。

### 14. 协议与编码细节
131. **HTTP 协议中 `Keep-Alive` 的作用？**
    *   复用 TCP 连接。Workerman 的 HTTP 默认开启 Keep-Alive。
132. **如何发送 Chunked 数据？**
    *   Workerman HTTP 支持 Chunked Transfer Encoding。
    *   `$connection->send(new Response(200, ['Transfer-Encoding' => 'chunked'], $content));`
133. **WebSocket 的 Ping/Pong 帧？**
    *   协议层面的心跳。Workerman 会自动处理 Ping 帧 (回复 Pong)。
134. **如何解析二进制协议中的大端/小端 (Endian)？**
    *   使用 PHP 的 `pack`/`unpack` 函数，指定 `N` (大端) 或 `V` (小端)。
135. **TLV 格式协议 (Tag-Length-Value)？**
    *   常见的二进制协议格式。自定义 Protocol 类解析。
136. **Workerman 支持 MQTT 吗？**
    *   支持。有 `workerman/mqtt` 组件。
137. **Workerman 支持 Hex (16进制) 通讯吗？**
    *   支持。在 `onMessage` 中接收到的 `$data` 是字符串，用 `bin2hex($data)` 转为 16 进制查看。
138. **如何处理 Protobuf 数据？**
    *   使用 Google Protobuf PHP 扩展或库生成 PHP 类，在 `onMessage` 中 `deserialize`。
139. **最大包长限制 (`max_package_size`)？**
    *   默认 10MB。防止恶意大包耗尽内存。可以在 Protocol 类中修改。
140. **如何获取 WebSocket 的 Header？**
    *   在 `onWebSocketConnect` 中访问 `$_SERVER`。

### 15. 高级部署与运维
141. **Docker 部署 Workerman 的注意点？**
    *   CMD 必须是 `php start.php start` (非 daemon 模式)。
    *   端口映射。
142. **Kubernetes (K8s) 部署？**
    *   Deployment 部署无状态 BusinessWorker。
    *   StatefulSet 部署 Gateway (需要固定 IP 或 DNS 供客户端连接)。
143. **Supervisor 管理 Workerman？**
    *   配置 `command=php start.php start`。
    *   `autorestart=true`。
144. **如何监控 Worker 进程内存泄漏？**
    *   Prometheus + Grafana。定期采集 `memory_get_usage()`。
145. **Workerman 能够做分布式任务调度吗？**
    *   可以。结合 Channel 组件或 Redis 队列，分发任务给不同 Worker。
    *   有现成项目 `workerman/crontab`。
146. **如何实现平滑升级代码 (不掉线)？**
    *   GatewayWorker 中，BusinessWorker 可以 reload 更新逻辑，Gateway 保持连接不掉线。
    *   如果 Gateway 代码变更，则必须断线重启。
147. **日志切割 (Log Rotation)？**
    *   Workerman 本身不支持自动切割。需配合 `logrotate` 工具，发送信号让 Workerman 重开日志文件。
148. **如何查看每个 Worker 的连接数？**
    *   `status` 命令会列出每个 PID 的 connection 数量。
149. **TCP 连接数上限受什么限制？**
    *   内存。
    *   文件句柄 (`ulimit -n`)。
    *   端口范围 (仅对 Client 端有限制，Server 端主要受内存和文件句柄限制)。
150. **Workerman 的 `globalEvent`？**
    *   全局的 Event Loop 对象。可以手动 `add` 自定义事件。

### 16. Webman 专题 (Webman 是 Workerman 的进阶)
151. **Webman 的路由机制？**
    *   基于 `FastRoute`。支持注解路由。
152. **Webman 的中间件？**
    *   PSR-15 风格。洋葱模型。
153. **Webman 的依赖注入？**
    *   支持 `php-di` 或 `laravel container`。
154. **Webman 如何处理静态资源？**
    *   内置了静态文件中间件。生产环境建议 Nginx。
155. **Webman 的控制器复用？**
    *   控制器常驻内存。**不能在控制器属性中存储请求相关的状态** (会污染下一个请求)。
    *   请求数据应从 `$request` 对象获取。
156. **Webman 支持视图吗？**
    *   支持 Raw PHP, Twig, Blade, Think-Template。
157. **Webman 的性能为什么强？**
    *   基于 Workerman，无 FPM 开销。
    *   路由/配置等只加载一次。
    *   极致优化 (JIT 等)。
158. **Webman 中的 Session 存储？**
    *   支持 File, Redis, RedisCluster。
    *   注意：File Session 在多进程下可能不一致 (如果没做 Sticky Session)，推荐 Redis。
159. **Webman 支持 SSE (Server-Sent Events) 吗？**
    *   支持。直接输出特定 Header 和流式数据。
160. **Webman 的插件系统？**
    *   Composer 包形式，自动配置。丰富插件 (Admin, Monitor, etc.)。

### 17. 场景题：设计与架构
161. **设计一个百万级长连接推送系统？**
    *   GatewayWorker 架构。
    *   Gateway 集群化，DNS 轮询或 LVS 负载均衡。
    *   Redis 存储 `uid -> gateway_ip` 映射。
    *   BusinessWorker 负责业务推送。
    *   系统参数调优 (内核)。
162. **Workerman 如何处理高并发短连接 (如 HTTP API)？**
    *   开启 `reusePort`。
    *   增加 Worker 数量。
    *   减少代码阻塞。
163. **如何实现一个简单的 RPC 框架？**
    *   Protocol: JSON-RPC 或 自定义二进制。
    *   Client: `AsyncTcpConnection` + 异步回调/Promise。
    *   Server: Workerman 监听 + 反射调用类方法。
164. **Workerman 做游戏服务器，数据如何持久化？**
    *   内存中高频读写 (Cache)。
    *   定时器 (Timer) 异步回写 DB (Write Behind)。
    *   关服时全量回写。
165. **如何实现分布式锁？**
    *   Workerman 本身不提供。使用 Redis (`setnx`) 或 Etcd。
166. **两个 Worker 进程间如何共享数据？**
    *   **不能**直接读写对方变量 (内存隔离)。
    *   方法：Redis, GlobalData, Channel, IPC (管道/共享内存 - 较复杂)。
167. **如何监控 Workerman 进程僵死 (Deadlock)？**
    *   外部监控脚本，定期请求 Worker 的心跳接口。
    *   如果超时未响应，报警或 Kill。
168. **Workerman 中使用单例模式要注意什么？**
    *   单例在进程生命周期内一直存在。
    *   注意资源 (如 DB 连接) 是否断开，需重连机制。
    *   静态数组会一直增长，需注意内存。
169. **如何限制上传文件大小？**
    *   HTTP 协议下修改 `max_package_size` (影响 POST Body 总大小)。
    *   流式解析时统计大小。
170. **Workerman 能做视频流媒体服务器吗？**
    *   可以 (如 RTMP/HLS)。但 PHP 处理音视频编解码效率低，通常只做信令服务器或转发，编解码交给 FFmpeg/C++。

### 18. 常见错误分析
171. **报错 `Call to undefined function pcntl_fork()`？**
    *   PHP 没有安装/开启 `pcntl` 扩展。Windows 下不可用。
172. **报错 `StreamSelect::add()` 失败？**
    *   文件描述符超限。检查 `ulimit -n`。
173. **客户端收到数据乱码？**
    *   协议不一致。如 Server 用 Frame，Client 用 Text。
    *   加密解密算法不匹配。
174. **业务运行一段时间后变慢？**
    *   内存泄漏导致频繁 GC。
    *   数据库连接池耗尽。
    *   死循环或长耗时任务阻塞。
175. **Worker 进程 ID 乱跳？**
    *   进程频繁 Crash 重启。检查 Fatal Error。

### 19. 版本与生态
176. **Workerman v3 vs v4 vs v5？**
    *   v3: 稳定版。
    *   v4: 增加 Composer 支持，代码重构，支持 PHP7+。
    *   v5: 支持 Fiber (协程)，基于 Revolt (未来趋势)。
177. **Channel 组件的底层原理？**
    *   一个独立的 Server 进程维护订阅关系。
    *   Client 进程通过 TCP 连接该 Server 发布/接收消息。
178. **GlobalData 组件的底层原理？**
    *   类似 Channel。一个 Server 维护 PHP 数组变量。
    *   Client 通过 TCP 请求读写。CAS (Compare And Swap) 保证原子性。
179. **PHPSocket.io 是什么？**
    *   Workerman 版的 Socket.io 服务端实现。兼容 JS Socket.io 客户端。
180. **Workerman 的 `Autoloader`？**
    *   遵循 PSR-4。

### 20. 综合考察
181. **什么是 IO 多路复用？**
    *   单线程同时监控多个 FD (Socket) 的可读/可写状态。Epoll 是 Linux 下最高效的实现。
182. **Workerman 为什么比 Apache/PHP-FPM 快？**
    *   常驻内存 (无磁盘 IO/编译开销)。
    *   IO 多路复用 (高并发)。
183. **如何理解 Workerman 的 "异步"？**
    *   框架层面的网络 IO 是异步非阻塞的 (基于 Event Loop)。
    *   但在回调函数内的业务代码，默认是同步阻塞的 (除非用协程)。
184. **Workerman 是否支持 QUIC / HTTP3？**
    *   目前原生支持尚在实验阶段 (v5+)。
185. **Workerman 的 `STDIN`, `STDOUT`, `STDERR`？**
    *   CLI 模式下的标准输入输出。守护进程模式下会被重定向到 `/dev/null` 或指定文件。
186. **如何实现 WebSocket 广播？**
    *   遍历 `$worker->connections` 数组，对每个 `$connection` 调用 `send`。
187. **Workerman 如何处理 `emfile` 错误？**
    *   `Too many open files`。提高 `ulimit`，修改 `/etc/security/limits.conf`。
188. **GatewayWorker 如何获取客户端 IP？**
    *   `$_SESSION['REMOTE_IP']` (Gateway 自动设置)。
189. **Workerman 可以在 Worker 中再 Fork 进程吗？**
    *   可以 (使用 `pcntl_fork`)，但要注意资源释放和僵尸进程处理。不推荐，建议用 Task Worker。
190. **Workerman 中的 `declare(ticks=1)`？**
    *   旧版本用于信号处理。新版本基于 `pcntl_signal_dispatch` 或 Event Loop，不再强依赖 ticks。
191. **如何优雅地关闭 Workerman？**
    *   发送 `SIGTERM` 信号。Worker 会捕获信号，停止接收新连接，处理完剩余请求后退出。
192. **Workerman 支持 IPv6 吗？**
    *   支持。`new Worker('tcp://[::]:8080')`。
193. **如何让 Workerman 在后台运行 (Daemon)？**
    *   `php start.php start -d`。
194. **Workerman 依赖的 `posix` 扩展作用？**
    *   获取进程组 ID，用户 ID，发送信号等。
195. **如何处理 WebSocket 分片传输 (Fragmentation)？**
    *   Workerman 底层已处理 RFC6455 协议分片，应用层收到的是完整的 Message。
196. **Workerman 的 `Timer` 精度？**
    *   依赖底层驱动。`pcntl_alarm` 秒级。`Event` 扩展 毫秒/微秒级。
197. **GatewayWorker 的 `Gateway::isUidOnline($uid)` 原理？**
    *   Gateway 内存维护了 `uid -> client_id` 的索引。
198. **Workerman 支持 `SO_KEEPALIVE` 吗？**
    *   支持。`$worker->transport = 'tcp';` 设置 socket context 选项。
199. **Workerman 中使用 `sleep()` 会怎样？**
    *   致命错误！会阻塞整个进程，暂停所有服务，导致心跳超时。严禁使用。
200. **Workerman 的未来？**
    *   向协程化 (Fiber) 发展，与 Webman 生态融合，挑战 Go/Java 在微服务领域的地位。

