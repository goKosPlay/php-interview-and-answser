### Laravel 的生命周期是怎样的？
* **入口文件**：所有请求都通过 `public/index.php` 进入。
* **加载自动加载器**：引入 `composer` 生成的自动加载文件。
* **创建应用实例**：从 `bootstrap/app.php` 获取 Laravel 应用实例。
* **内核处理（Kernel）**：
    * 请求发送到 HTTP 内核 (`App\Http\Kernel`) 或 Console 内核。
    * 加载环境变量 (`.env`) 和配置 (`config/`)。
    * 启动异常处理、日志记录等引导程序。
* **注册服务提供者（Service Providers）**：
    * 调用所有 Provider 的 `register` 方法（仅绑定服务到容器）。
    * 调用所有 Provider 的 `boot` 方法（此时所有服务已注册，可进行交互）。
* **路由分发**：请求通过路由中间件（Middleware），最终到达控制器或闭包。
* **响应返回**：控制器返回响应，经过中间件后处理，最终发送给浏览器。
* **终止**：调用 `terminate` 方法进行收尾工作（如 Session 写入、日志关闭）。

### 什么是服务容器（Service Container）与依赖注入？
* **服务容器**：是 Laravel 管理类依赖和执行依赖注入的核心工具。它本质上是一个高级的 IOC（控制反转）容器。
* **依赖注入**：通过构造函数或方法参数类型提示，由容器自动解析并注入所需的对象，而非在类内部 `new` 对象。
* **绑定方式**：
    * `bind()`：每次解析都创建新实例。
    * `singleton()`：单例模式，整个生命周期只创建一个实例。
    * `instance()`：绑定一个已存在的对象实例。

### Service Provider 的 register 和 boot 方法有什么区别？
* **register()**：
    * **只用于绑定服务到容器**。
    * 不能在其中调用其他服务（因为其他服务可能还没注册）。
* **boot()**：
    * **在所有服务提供者注册完毕后调用**。
    * 可以安全地调用其他已注册的服务（如注册路由、视图合成器、事件监听）。

### Facade（门面）的原理是什么？
* Facade 提供了一个静态接口来访问服务容器中的类。
* **原理**：
    * 每个 Facade 继承 `Illuminate\Support\Facades\Facade`。
    * 实现 `getFacadeAccessor()` 方法，返回容器中绑定的服务别名（字符串）。
    * 利用 PHP 的魔术方法 `__callStatic()`，将静态调用转发给容器解析出的具体对象实例。

### Eloquent ORM 中 N+1 问题及解决方案
* **问题**：遍历模型列表并访问关联关系时，如果每个模型都执行一次 SQL 查询关联数据，会导致大量 SQL（1 次查列表 + N 次查关联）。
* **解决**：使用 **预加载（Eager Loading）**，即 `with()` 方法。
    * 示例：`Post::with('comments')->get();`
    * 这将执行两次 SQL（一次查 Post，一次用 `IN` 查 Comment），然后在内存中匹配。

### Laravel 用到了哪些设计模式？
* **单例模式 (Singleton Pattern)**：
    * **应用**：服务容器 (`app()`)、数据库连接、配置加载器。
    * **体现**：`singleton()` 绑定确保整个请求周期内只存在一个实例。
* **工厂模式 (Factory Pattern)**：
    * **应用**：`view()` 辅助函数、`response()`、Database Factories。
    * **体现**：通过简单的接口创建复杂的对象实例。
* **门面模式 (Facade Pattern)**：
    * **应用**：`Cache::get()`, `Route::get()`, `DB::table()`。
    * **体现**：为复杂的子系统（服务容器中的对象）提供一个简单的静态访问接口（虽然严格来说 Laravel Facade 是代理模式的一种变体）。
* **观察者模式 (Observer Pattern)**：
    * **应用**：Eloquent 模型事件 (`creating`, `saved`, `deleted`)、事件系统 (`Event::listen`)。
    * **体现**：对象状态改变时通知所有观察者（监听器）。
* **策略模式 (Strategy Pattern)**：
    * **应用**：Cache 驱动 (Redis/Memcached/File)、Session 驱动、Queue 驱动。
    * **体现**：定义一系列算法（驱动），将它们封装起来，并使它们可以互换。
* **代理模式 (Proxy Pattern)**：
    * **应用**：Laravel Facade 本质上是服务容器中对象的静态代理。
* **装饰器模式 (Decorator Pattern) / 管道模式 (Pipeline)**：
    * **应用**：**中间件 (Middleware)**。
    * **体现**：请求像流一样穿过一系列中间件，每个中间件可以在请求处理前后添加逻辑（装饰）。
* **适配器模式 (Adapter Pattern)**：
    * **应用**：Filesystem (Flysystem) 适配不同的存储后端（本地、S3）。
    * **体现**：将不同接口转换成统一接口。
* **注册表模式 (Registry Pattern)**：
    * **应用**：服务容器本身就是一个注册表，存储了所有的服务绑定。
* **迭代器模式 (Iterator Pattern)**：
    * **应用**：集合 (`Collection`)、Eloquent 生成器 (`cursor()`)。

### Laravel 进阶与架构面试题

#### 1. 什么是 Laravel Contracts（契约）？它与 Facades 有什么区别？
* **Contracts**：是一组定义了核心服务（如缓存、队列、日志）的 Interface（接口）。
    * **优点**：松耦合，依赖注入接口而非具体实现，利于单元测试（Mocking）和替换实现。
* **Facades**：提供静态访问接口。
    * **对比**：Facades 使用简单但隐式依赖，测试时需用 `Cache::shouldReceive`；Contracts 显式依赖，代码结构更清晰，适合大型项目。

#### 2. Laravel 队列系统（Queue）的原理与使用场景
* **原理**：
    * **Producer**：将任务序列化（serialize）后推送到驱动（Redis/Database/SQS）。
    * **Consumer**：`php artisan queue:work` 进程轮询队列，反序列化任务并执行 `handle()` 方法。
* **重试机制**：如果任务抛出异常，Worker 会捕获并根据 `tries` 配置重试，超过次数后存入 `failed_jobs` 表。
* **Horizon**：Redis 队列的仪表盘，用于监控吞吐量、堆积情况和失败任务。

#### 3. 什么是 Laravel Octane？它是如何提升性能的？
* **原理**：基于 Swoole 或 RoadRunner，启动一次应用程序后常驻内存（Application 实例不销毁），避免了每次请求都重新加载 Composer、Boot Kernel 和 Provider 的开销。
* **注意点**：
    * **内存泄漏**：因为常驻内存，静态变量不会自动重置，需特别小心 `static` 属性和单例状态污染。
    * **依赖注入**：在构造函数中注入的单例服务在整个生命周期中只有一份，需注意请求隔离。

#### 4. Laravel 性能优化有哪些手段？
* **配置缓存**：`php artisan config:cache`（将所有配置合并为一个文件，大幅提升加载速度）。
* **路由缓存**：`php artisan route:cache`（序列化路由表，避免每次解析路由文件）。
* **类映射优化**：`composer dump-autoload -o`。
* **查询优化**：使用 `debugbar` 分析 N+1 问题，使用索引，避免 `SELECT *`。
* **异步处理**：耗时操作（发邮件、生成报表）放入队列。

#### 5. 什么是上下文绑定（Contextual Binding）？
* **场景**：当两个类依赖同一个接口，但需要注入不同的实现时。
* **代码**：
  ```php
  $this->app->when(PhotoController::class)
            ->needs(Filesystem::class)
            ->give(function () { return Storage::disk('local'); });

  $this->app->when(VideoController::class)
            ->needs(Filesystem::class)
            ->give(function () { return Storage::disk('s3'); });
  ```

#### 6. Laravel 的 Pipeline（管道）模式是如何实现的？
* **核心**：`Illuminate\Pipeline\Pipeline` 类。
* **机制**：
    * `send($request)`：设置初始数据。
    * `through($pipes)`：设置中间处理层（数组）。
    * `then($destination)`：执行闭包，利用 `array_reduce` 将所有管道层包裹成洋葱模型。
* **应用**：中间件（Middleware）就是最典型的 Pipeline 实现。

#### 7. 如何防范 CSRF 攻击？Laravel 是怎么做的？
* **原理**：跨站请求伪造，攻击者诱导用户在已登录状态下发起恶意请求。
* **Laravel 防御**：
    * 生成 `_token` 存入 Session 和 HTML 表单（或 Meta 标签）。
    * `VerifyCsrfToken` 中间件拦截 `POST/PUT/DELETE` 请求，比对请求中的 Token 与 Session 中的 Token 是否一致。

#### 8. Eloquent 的 `fillable` vs `guarded` 有什么区别？为什么重要？
* **批量赋值（Mass Assignment）漏洞**：用户可能通过 HTTP 请求传入隐藏字段（如 `is_admin=1`）修改数据库敏感列。
* **fillable**：白名单，仅允许列出的字段被批量赋值（`create($data)` 或 `update($data)`）。
* **guarded**：黑名单，列出的字段禁止批量赋值（`*` 表示全部禁止）。
* **最佳实践**：优先使用 `fillable`，确保安全明确。

#### 9. 什么是延迟加载服务提供者（Deferred Service Providers）？
* **作用**：如果一个 Provider 仅用于注册绑定到容器的服务，且该服务不是每次请求都必须的，可以设为延迟加载。
* **实现**：实现 `DeferrableProvider` 接口并定义 `provides()` 方法。
* **优势**：只有当尝试解析该服务时才加载 Provider，提升启动性能。

#### 10. Laravel 的 Macro（宏）机制是什么？
* **原理**：基于 `Macroable` Trait，允许在运行时向类添加方法。
* **应用**：`Collection`, `Request`, `Response`, `Str` 等均支持。
* **示例**：
  ```php
  Response::macro('caps', function ($value) {
      return Response::make(strtoupper($value));
  });
  ```

#### 11. Gate 和 Policy（策略）有什么区别？
* **Gate**：基于闭包的简单权限判断，适合全局性或非模型绑定的权限（如 `Gate::allows('update-settings')`）。
* **Policy**：围绕特定模型（Model）组织的授权逻辑类（如 `PostPolicy` 对应 `Post` 模型），适合复杂的 CRUD 权限控制。

#### 12. Eloquent 的 Accessor（访问器）/ Mutator（修改器）与 Casts（类型转换）的区别？
* **Accessor/Mutator**：自定义处理逻辑（如 `getNameAttribute`），功能最强但写法繁琐。
* **Casts**：内置的常用类型转换（如 `json`, `date`, `boolean`, `encrypted`），写法简洁（`protected $casts = [...]`）。
* **推荐**：优先用 Casts，无法满足时用 Accessor/Mutator。

#### 13. Laravel 任务调度（Task Scheduling）是如何工作的？
* **传统方式**：为每个任务设置一个 Cron 条目，难以维护。
* **Laravel 方式**：服务器只需配置一个 Cron 条目 `* * * * * php /path/to/artisan schedule:run`。
* **原理**：每分钟触发一次 `schedule:run`，Laravel 检查 `Console/Kernel.php` 中定义的所有任务，判断是否到期执行。

#### 14. Feature Test（功能测试）与 Unit Test（单元测试）侧重点有何不同？
* **Unit Test**：测试单个类或方法，Mock 所有依赖，速度快，隔离性好（`tests/Unit`）。
* **Feature Test**：测试完整的 HTTP 请求流程（路由->控制器->数据库->响应），模拟真实用户行为，覆盖面广但速度较慢（`tests/Feature`）。
* **Laravel 现状**：倾向于写更多的 Feature Test 以保证业务流程正确。

#### 15. Laravel Broadcasting（广播）是做什么的？
* **场景**：构建实时应用（聊天室、通知）。
* **流程**：服务器端事件（Event）触发 -> 推送到驱动（Redis/Pusher） -> WebSocket 服务转发 -> 前端 JS（Laravel Echo）接收并更新 UI。
