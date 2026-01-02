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