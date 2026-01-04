### PHP 基础与类型
* 弱类型 vs 强类型：PHP 是弱类型，隐式转换多，需掌握 `==` 与 `===` 差异。
* 常见陷阱：`0 == '0'` 为真，`0 == 'abc'` 也为真（字符串转 0）。
* 类型声明：`declare(strict_types=1)` 开启严格模式，函数参数/返回值必须匹配。

### PHP 面向对象常见面试点
* 访问控制：`public/protected/private` 作用域差异。
* 接口 vs 抽象类：接口只能定义方法签名，抽象类可包含实现。
* Trait 解决多继承问题，同名方法冲突用 `insteadof` 和 `as` 解决。
* 魔术方法：`__construct/__destruct/__get/__set/__call/__invoke` 等触发场景。
* 后期静态绑定：`static::` vs `self::`，`static` 指向运行时类。

### PHP 垃圾回收机制
* 引用计数为主：变量引用数为 0 即释放。
* 循环引用：通过 `gc_collect_cycles()` 手动触发或使用根缓冲区算法清理。
* 内存泄漏排查：Xdebug、memory_get_usage()、循环引用检测。

### PHP 协程（Swoole/Fiber）常见面试题
* Fiber（PHP 8.1+）：用户态轻量级线程，遇到 IO 主动让出，避免回调地狱。
* 与生成器区别：Fiber 可挂起任意位置，生成器只能 `yield` 返回。
* Swoole 协程：Hook 原生函数，实现同步写法异步执行；注意 CPU 密集会阻塞事件循环。

### PHP 性能优化实战
* OpCache：开启并调优 `opcache.memory_consumption/opcache.max_accelerated_files`。
* 预加载（PHP 7.4+）：`opcache.preload` 提前加载框架与常用类。
* 避免在循环中创建大量临时对象。
* 使用生成器处理大数据集，节省内存。

### Composer 常见面试点
* 自动加载：遵循 PSR-4，映射命名空间到目录。
* 版本约束：`^`、`~`、`*` 语义化差异；`composer.lock` 锁定版本。
* 生产部署：`composer install --no-dev --optimize-autoloader`。
* 私有包：使用 `repositories` 配置 Git/Satis 源。

### Laravel 框架面试高频
* 服务容器（IoC）：绑定与解析、单例 vs 原型、自动依赖注入。
* 中间件：请求/响应过滤链，定义顺序影响执行顺序。
* Eloquent ORM：延迟加载（N+1 问题）、预加载（with）、模型事件。
* 队列：驱动（Redis/Database）、任务失败重试、延迟任务。
* 事件系统：观察者模式，同步 vs 队列事件。

### Symfony 框架面试高频
* Bundle 体系：可复用模块，配置覆盖优先级。
* 依赖注入组件：YAML/XML/注解配置，自动装配（autowire）。
* 表单组件：数据转换、验证、CSRF 保护。
* 缓存：HTTP Cache、ESI、Tag-aware Cache。

### PHP 安全常见面试题
* SQL 注入：使用 PDO 预处理语句，禁止拼接。
* XSS：输出转义 `htmlspecialchars`，CSP 策略。
* CSRF：Token 校验、SameSite Cookie。
* 文件上传：MIME 检测、文件头校验、限制目录执行权限。
* 反序列化漏洞：禁止 `unserialize` 用户输入，使用 JSON 代替。

### PHP 8 新特性面试点
* JIT：Opcache JIT 提升 CPU 密集场景性能，默认关闭。
* 命名参数：`foo(name: 'bar')` 提高可读性。
* 属性提升：`class User { public function __construct(public string $name) {} }`。
* Match 表达式：更简洁的 switch 替代，支持返回值。
* Nullsafe 运算符：`$obj?->foo()?->bar()`。

### PHP 单元测试与 CI/CD
* PHPUnit：断言、数据提供器、Mock 对象。
* 覆盖率：Xdebug + PHPUnit --coverage-html。
* GitHub Actions：自动运行测试、静态分析（PHPStan/Psalm）、代码风格检查（PHP CS Fixer）。

### PHP 静态分析工具
* PHPStan：逐级严格模式，发现潜在类型错误。
* Psalm：支持模板、泛型、更高级类型推断。
* Phan：基于 AST 分析，发现未定义变量、死代码。

### PHP 部署与容器化
* Dockerfile：多阶段构建、使用官方 PHP 镜像、安装扩展 `docker-php-ext-install`。
* 环境变量：`.env` 加载、敏感信息不提交代码。
* 健康检查：HTTP 探针、进程探针确保服务可用。

### PHP-FPM 深入与调优
* 进程管理模式：
    * `static`：固定进程数，适合高并发、内存充足场景，减少 fork 开销。
    * `dynamic`：动态调整，适合波动流量，节省内存。
    * `ondemand`：按需启动，适合开发环境或极低流量。
* 关键参数：`pm.max_children`（最大子进程数）、`pm.max_requests`（防止内存泄漏，处理多少请求后重启）。
* 慢日志：开启 `request_slowlog_timeout` 捕获执行超时的脚本，定位性能瓶颈。
* 502 Bad Gateway 原因：PHP-FPM 进程繁忙/卡死、脚本执行超时、进程被杀。

### PHP-FPM 核心面试题
* **CGI、FastCGI 与 PHP-FPM 的区别？**
    * **CGI**：旧协议，每次请求都需要 fork 新进程加载解释器，处理完销毁，性能较差。
    * **FastCGI**：CGI 的升级版（协议），进程常驻内存（Long-Lived），复用进程处理请求，无需重复加载配置。
    * **PHP-FPM**：FastCGI 的 PHP 具体实现与进程管理器，提供了进程池管理、平滑重启、慢日志等功能。
* **Nginx 与 PHP-FPM 的通信方式（Unix Socket vs TCP）？**
    * **Unix Socket**：通过文件（如 `/tmp/php-cgi.sock`）通信，不走网络协议栈，内核层交互，单机性能略好；高并发下可能因内核缓冲区满而不稳定。
    * **TCP Socket**：通过 IP:Port（如 `127.0.0.1:9000`）通信，走网络协议栈，稳定，支持 Nginx 与 PHP 分离部署。
* **PHP-FPM 如何实现平滑重启？**
    * 向 Master 进程发送 `USR2` 信号。
    * Master 重新加载配置，启动新 Worker 进程。
    * 旧 Worker 进程处理完当前请求后关闭，实现无缝切换。

### 线上故障排查实战
* CPU 100% 排查：
    * `top` 确认是 php-fpm 进程。
    * `strace -p <pid>` 跟踪系统调用（慎用，会拖慢进程）。
    * 开启慢日志查看卡在哪个函数。
    * 检查是否有死循环或密集计算。
* 内存泄漏排查：
    * 观察内存增长曲线。
    * 检查长期运行的脚本（如消费者）。
    * 检查循环引用、大对象未释放、静态变量无限增长。
* 接口响应慢：
    * 链路追踪（SkyWalking/Jaeger）。
    * 检查 MySQL 慢查询、Redis 延迟、外部 API 调用耗时。

### 架构与工程化
* 分布式 Session：存储在 Redis/Memcached，而非本地文件。
* 幂等性设计：防止表单重复提交，使用 Token 机制或数据库唯一索引。
* 限流策略：漏桶算法、令牌桶算法（Redis Lua 实现）。
* 灰度发布：基于用户 ID 或 IP 路由到不同版本的服务。

### PHP 设计模式实战
* 单例模式：数据库连接、配置类（注意 PHP 生命周期，请求结束即销毁）。
* 工厂模式：解耦对象创建逻辑，便于扩展。
* 策略模式：多种支付渠道（支付宝/微信/银联）切换。
* 观察者模式：用户注册后发送邮件、短信、记录日志（解耦业务逻辑）。
* 装饰器模式：不改变原有类，动态增加功能（如中间件、日志记录）。

### Laravel 源码与原理
* 请求生命周期：
    * `public/index.php` 入口 -> 加载 Composer Autoload -> 创建 Application 实例。
    * Http Kernel 处理请求 -> 加载 Service Providers（Register/Boot 阶段）。
    * 路由分发 -> 中间件过滤 -> 控制器执行 -> 返回 Response。
* 服务容器（Container）底层：
    * 利用 **PHP 反射（Reflection）** 机制分析构造函数依赖。
    * 递归解析依赖树，自动实例化对象。
* Facade 原理：
    * `__callStatic` 魔术方法转发调用到容器解析出的具体实例。

### PHP 微服务与高性能架构
* RPC vs RESTful：
    * RPC（gRPC/Thrift）：基于 TCP/HTTP2，二进制传输，体积小效率高，强类型，适合内部服务调用。
    * RESTful：基于 HTTP1.1，JSON 文本传输，通用性强，适合对外 API。
* 服务治理组件：
    * 注册中心（Consul/Etcd/Nacos）：服务自动发现与健康检查。
    * 熔断与降级（Sentinel/Hystrix）：防止下游故障拖垮整个链路。
    * 配置中心：动态下发配置，无需重启服务。
* 消息队列（Kafka/RabbitMQ）场景：
    * 削峰填谷：应对突发流量。
    * 异步解耦：注册后发邮件、生成报表。
    * 顺序消费：保证消息处理顺序（如订单状态流转）。

### MySQL 与 PHP 配合实战
* 长连接 vs 短连接：
    * 短连接：每次请求新建，消耗握手资源，但无连接池管理负担。
    * 长连接（Persistent Connection）：复用连接，需防范连接超时失效（MySQL `wait_timeout`）和内存泄漏。
* 读写分离：
    * 框架层：Laravel 配置 `read/write` 数组，自动路由 SQL。
    * 中间件层：MySQL Router / MyCat / ProxySQL 透明代理。
* 分库分表策略：
    * 垂直拆分：按业务模块拆分库，按字段冷热拆分表。
    * 水平拆分：按 UserID 取模或 Range 拆分数据行，解决单表数据量过大（>2000w）。

### PHP 内核与进阶面试题

#### 1. PHP 7 的 Zval 结构与 PHP 5 有什么本质区别？
* **PHP 5**：Zval 是在堆（Heap）上分配的，包含 `refcount` 和 `is_ref`，不仅占用内存大，而且不利于 CPU 缓存（指针跳转多）。
* **PHP 7**：Zval 变为栈（Stack）上分配（或直接嵌入结构体），去除了 `refcount`（引用计数移到了 Value 指向的具体结构体中，如 `zend_string`、`zend_array`）。
* **优势**：大幅减少内存分配次数，提升 CPU 缓存命中率，整数/浮点数直接存 Zval，不再需要指针。

#### 2. PHP 数组（Array）底层是如何实现的？
* **数据结构**：**HashTable（哈希表）**。
* **有序性**：PHP 数组是有序的，因为底层维护了一个 `arData` 数组存储 Bucket，Bucket 按插入顺序排列；同时用 `nTableMask` 和 `nIndex` 数组维护哈希映射（Hash -> Index）。
* **Packed Array**：对于连续整数索引的数组（如 `[1, 2, 3]`），PHP 7 会优化为 **Packed Array**，不使用哈希查找，直接用索引作为偏移量，省内存且快。

#### 3. 详细描述 PHP 的垃圾回收（GC）过程（引用计数 + 循环引用解决）
* **基础**：引用计数（RefCount）。变量离开作用域或 unset 时，refcount 减 1；为 0 则释放。
* **问题**：循环引用（A 引用 B，B 引用 A）导致 refcount 永远不为 0，造成内存泄漏。
* **解决（GC 收集器）**：
    1. 当可能存在循环引用（如数组/对象）的 refcount 减小但未归零时，将其放入 **Root Buffer（根缓冲区）**。
    2. 当缓冲区满（默认 10000）时，触发回收算法。
    3. **标记-清除**：
        * **DFS 标记**：深度遍历引用链，模拟将 refcount 减 1。
        * **判断**：如果模拟减后 refcount 为 0，说明它是垃圾；如果不为 0，说明还有外部引用，恢复 refcount。
    4. **清除**：释放确认为垃圾的变量。

#### 4. OpCache 的 Shared Memory（共享内存）里存了什么？
* **Opcode**：编译后的脚本指令。
* **Interned Strings**：驻留字符串（相同的字符串在内存中只存一份，如函数名、类名、字面量）。
* **Function/Class Table**：函数和类的定义。
* **优势**：避免了每个 PHP-FPM 进程重复编译和存储相同的字符串/类定义，极大节省内存。

#### 5. 什么是 Copy On Write（写时复制）？
* **机制**：当多个变量指向同一个值（如 `$a = $b`）时，PHP 不会立即复制内存，而是让它们共享同一个 Zval（refcount++）。
* **触发**：只有当其中一个变量发生修改（如 `$a = 1`）时，才会真正分配新的内存空间并复制数据。
* **意义**：在读多写少的场景下极大节省内存。

#### 6. 弱类型比较中的 Hash 冲突攻击（Hash DoS）是什么？
* **原理**：攻击者构造大量 Hash 值相同的 Key（如 `0` 和某些特定字符串），存入数组。
* **后果**：数组退化为链表，插入查找复杂度从 O(1) 变为 O(N)，导致 CPU 100%。
* **防御**：PHP 限制了 `max_input_vars`（默认 1000），并且在 Hash 算法中引入了随机种子（Random Seed）防止预测。

### PHP 面试题库大全 (补充精选)

#### 1. 数组与数据结构
1.  **`array_merge` 和 `+` 号合并数组的区别？**
    *   `array_merge`：字符串键名相同会覆盖（后覆盖前），数字键名会重排（重建索引）。
    *   `+`：键名相同只取第一个（前覆盖后），无论字符串还是数字键。
2.  **`isset` 和 `empty` 的区别？**
    *   `isset`：变量存在且不为 NULL 返回 true。
    *   `empty`：变量不存在、或为 false/0/""/NULL/[] 时返回 true。
3.  **如何从数组中随机取出一个值？** `array_rand()`。
4.  **`array_map` 和 `array_walk` 的区别？**
    *   `array_map`：返回新数组，回调函数参数是 value，支持多数组。
    *   `array_walk`：返回 bool，直接修改原数组（需引用传参），参数是 value, key。
5.  **如何实现多维数组排序？** `array_multisort()` 或 `usort()`。
6.  **PHP 数组也是哈希表，时间复杂度是多少？** 查找/插入/删除平均 O(1)。
7.  **`count()` 函数在 PHP 7.2+ 对 NULL 的行为？** 抛出 Warning（之前返回 0）。
8.  **如何判断一个变量是数组？** `is_array()`。
9.  **`implode` 和 `explode` 的作用？** 数组转字符串 / 字符串转数组。
10. **`compact` 和 `extract` 的作用？** 变量转数组 / 数组转变量。

#### 2. 字符串与正则
11. **单引号和双引号的区别？** 双引号解析变量和转义字符，单引号不解析（效率稍高）。
12. **`strlen` 和 `mb_strlen` 的区别？** `strlen` 字节数，`mb_strlen` 字符数（支持多字节编码）。
13. **`str_replace` 和 `preg_replace` 的区别？** 前者普通替换，后者正则替换（功能强但慢）。
14. **常用的字符串截取函数？** `substr`, `mb_substr`。
15. **如何反转字符串？** `strrev` (不支持中文)，`mb_strrev` (需自定义)。
16. **`trim` 的作用？** 去除首尾空白或指定字符。
17. **`strpos` 查找失败返回什么？** `false`（需用 `=== false` 判断，防止位置 0 被误判）。
18. **`sprintf` 的作用？** 格式化字符串。
19. **如何将字符串转为数组？** `str_split` (按长度)，`explode` (按分隔符)。
20. **PHP 正则表达式的定界符是什么？** 通常是 `/`，也可以是 `#`, `~` 等。

#### 3. 函数与作用域
21. **`include` 和 `require` 的区别？** `include` 失败报 Warning 继续执行；`require` 失败报 Fatal Error 停止执行。
22. **`include_once` 和 `require_once` 的作用？** 防止重复加载。
23. **什么是静态变量（static variable）？** 函数内 `static $count`，在函数调用结束后不销毁，保留值到下一次调用。
24. **global 关键字的作用？** 在函数内访问全局变量。
25. **传值和传引用的区别？** 传值拷贝副本；传引用操作同一内存地址（`&`）。
26. **匿名函数（Closure）与 `use` 关键字？** `use` 用于引入父作用域变量（值传递，除非加 `&`）。
27. **可变函数是什么？** `$func = 'foo'; $func();`。
28. **`func_get_args` 的作用？** 获取函数所有参数数组。
29. **PHP 7 的标量类型声明有哪些？** int, float, string, bool, array, iterable, object, void。
30. **`declare(ticks=1)` 是什么？** 每执行一条低级语句触发一次 register_tick_function。

#### 4. 面向对象 (OOP)
31. **`$this`, `self`, `static` 的区别？**
    *   `$this`：当前对象实例。
    *   `self`：当前类（定义时的类）。
    *   `static`：调用类（运行时的类，支持后期静态绑定）。
32. **构造函数和析构函数？** `__construct`, `__destruct`。
33. **抽象类和接口的区别？**
    *   接口：多继承，只能定义公有方法签名，无属性。
    *   抽象类：单继承，可有属性和具体方法。
34. **`final` 关键字的作用？** 修饰类不可被继承，修饰方法不可被重写。
35. **`__toString` 何时触发？** 对象被当做字符串输出时。
36. **`__call` 和 `__callStatic`？** 调用不存在的对象方法 / 静态方法时触发。
37. **`__get` 和 `__set`？** 访问/设置不存在的属性时触发。
38. **`__invoke`？** 对象被当做函数调用时触发。
39. **`clone` 关键字与 `__clone` 魔术方法？** 浅拷贝对象，`__clone` 中可处理深拷贝逻辑。
40. **什么是 Trait？优先级？** 当前类 > Trait > 父类。
41. **PSR-4 自动加载规范是怎样的？** 命名空间与文件路径一一映射。
42. **如何实现单例模式？** 私有构造、私有克隆、静态实例属性、静态获取方法。
43. **什么是依赖注入（DI）？** 将依赖对象通过构造参数传入，而非内部 new。

#### 5. 错误与异常
44. **PHP 7 的 `Throwable` 接口？** `Exception` 和 `Error` 都实现了它。
45. **如何捕获 Fatal Error？** `register_shutdown_function` + `error_get_last`。
46. **`set_error_handler` 能捕获哪些错误？** Warning, Notice, User Error；不能捕获 Fatal Error, Parse Error。
47. **`try...catch...finally` 执行顺序？** catch 捕获异常，finally 无论是否异常都执行（常用于清理资源）。
48. **`error_reporting` 的作用？** 设置报错级别（如 E_ALL & ~E_NOTICE）。
49. **显示错误配置？** `display_errors = On`。
50. **自定义异常类？** 继承 `Exception` 类。

#### 6. 文件与目录
51. **`fopen` 的模式有哪些？** r, w, a, x, r+, w+, a+ 等。
52. **如何读取整个文件到字符串？** `file_get_contents`。
53. **如何按行读取文件？** `fgets` 或 `file` (读入数组)。
54. **`file_put_contents` 的 flags？** `FILE_APPEND` (追加), `LOCK_EX` (独占锁)。
55. **如何遍历目录？** `scandir`, `glob`, `DirectoryIterator`。
56. **`chmod` 的作用？** 修改文件权限。
57. **`is_file` 和 `is_dir`？** 判断是否文件/目录。
58. **如何删除文件？** `unlink`。
59. **如何创建目录？** `mkdir($path, 0777, true)` (递归)。
60. **`pathinfo` 返回什么？** dirname, basename, extension, filename。

#### 7. Web 特性与 Session
61. **`$_GET`, `$_POST`, `$_REQUEST` 的区别？** `$_REQUEST` 包含 GET, POST, COOKIE。
62. **`session_start()` 之前能有输出吗？** 不能（除非开启 buffer），因为要发 Header。
63. **Session 默认存哪里？** 文件（`session.save_path`）。
64. **如何销毁 Session？** `session_destroy()` 并清空 `$_SESSION`。
65. **Cookie 的属性有哪些？** name, value, expire, path, domain, secure, httponly。
66. **如何禁止 JS 读取 Cookie？** 设置 `HttpOnly` 为 true。
67. **`header()` 函数的作用？** 发送原生 HTTP 头（如跳转、状态码、缓存）。
68. **`http_response_code()`？** 获取或设置 HTTP 响应码。
69. **`$_SERVER['REMOTE_ADDR']` 是什么？** 客户端 IP（可能被代理伪造）。
70. **如何获取上传文件信息？** `$_FILES`。

#### 8. 数据库 (PDO)
71. **PDO 是什么？** PHP Data Objects，统一的数据库访问层。
72. **PDO 预处理语句的优势？** 防 SQL 注入，多次执行效率高。
73. **`bindParam` 和 `bindValue` 的区别？**
    *   `bindParam`：绑定变量引用（执行时才取值）。
    *   `bindValue`：绑定具体值（立即绑定）。
74. **PDO 事务处理？** `beginTransaction`, `commit`, `rollBack`。
75. **PDO::FETCH_ASSOC 是什么？** 只返回关联数组。
76. **如何获取最后插入 ID？** `lastInsertId()`。
77. **持久化连接？** `PDO::ATTR_PERSISTENT => true`。

#### 9. 高级与底层
78. **CGI, FastCGI, PHP-FPM 的关系？**
    *   CGI：协议，单进程单请求。
    *   FastCGI：协议，常驻进程。
    *   PHP-FPM：FastCGI 的实现管理程序。
79. **PHP 内存管理机制？** 预分配内存块、引用计数、写时复制、垃圾回收。
80. **什么是 ZTS？** Zend Thread Safety（线程安全），多线程环境（如 Apache Worker）需开启。
81. **PHP 的生命周期？** 模块初始化(MINIT) -> 请求初始化(RINIT) -> 执行脚本 -> 请求关闭(RSHUTDOWN) -> 模块关闭(MSHUTDOWN)。
82. **OpCache 原理？** 缓存 opcode，避免重复编译。
83. **JIT (Just In Time) 编译器？** PHP 8 引入，将 opcode 编译为机器码运行。
84. **生成器 (Generator) 与 yield？** 迭代过程中生成值，节省内存（不生成整个数组）。
85. **PHP 弱类型比较陷阱？** md5('240610708') == md5('QNKCDZO') 都是 "0e..." 开头会被当做科学计数法 0。
86. **`serialize` 和 `unserialize`？** 序列化/反序列化对象或数组。
87. **反射 (Reflection) API？** 运行时获取类、方法、属性的信息。
88. **SPL (Standard PHP Library) 有哪些常用类？** `SplStack`, `SplQueue`, `SplFixedArray`, `ArrayIterator`。
89. **`pcntl_fork` 作用？** 创建子进程（仅 CLI）。
90. **Composer 的 `vendor` 目录和 `composer.lock`？** 依赖包存放目录；锁定具体版本号。

#### 10. 安全与其他
91. **SQL 注入防御？** 预处理语句。
92. **XSS 防御？** `htmlspecialchars` 转义输出。
93. **CSRF 防御？** Token 验证。
94. **文件上传漏洞防御？** 检查后缀白名单、MIME类型、内容头、重命名、非 Web 目录存储。
95. **`eval` 的危害？** 执行任意 PHP 代码，极易被利用。
96. **`password_hash` 使用什么算法？** 默认 bcrypt。
97. **`disable_functions` 的作用？** 禁用危险函数（如 `exec`, `system`）。
98. **如何获取 POST 原始数据？** `file_get_contents("php://input")`。
99. **`json_encode` 失败原因？** 编码非 UTF-8。
100. **PHP 是解释型语言吗？** 是，但先编译成 opcode 再由 VM 执行。