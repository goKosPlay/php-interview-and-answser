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

### Laravel 内核与扩展面试题

#### 1. 中间件（Middleware）的 `terminate` 方法有什么用？
* **场景**：需要在 HTTP 响应发送给浏览器**之后**执行某些操作（如记录日志、统计耗时），避免影响用户感知的响应速度。
* **实现**：在中间件类中定义 `terminate($request, $response)` 方法。
* **原理**：Laravel 的入口 `index.php` 中，在 `$kernel->terminate()` 时会被调用。

#### 2. 什么是 Eloquent Global Scope（全局作用域）？如何移除？
* **作用**：为模型的所有查询自动添加默认约束（如软删除 `SoftDeletes` 就是一个全局作用域，自动加 `deleted_at is null`）。
* **自定义**：实现 `Scope` 接口的 `apply` 方法，并在模型 `booted` 方法中注册 `static::addGlobalScope`。
* **移除**：使用 `withoutGlobalScope(MyScope::class)` 或 `withoutGlobalScopes()`。

#### 3. Laravel 容器的 `make` 和 `build` 有什么区别？
* **make($abstract)**：是容器的标准入口。它会检查单例缓存、处理上下文绑定、应用扩展（extend），最后如果需要创建新实例，会调用 `build`。
* **build($concrete)**：是底层的实例化逻辑。它利用反射（Reflection）解析构造函数依赖，递归解决依赖关系。一般不直接调用。

#### 4. 如何扩展 Laravel 的驱动（如自定义 Cache 或 Session 驱动）？
* **步骤**：
    1. 实现对应的接口（如 `Illuminate\Contracts\Cache\Store`）。
    2. 在 ServiceProvider 的 `boot` 方法中使用 `Cache::extend('my_driver', closure)` 注册驱动。
    3. 在 `config/cache.php` 中配置使用新驱动。

#### 5. 什么是 Repository Pattern（仓库模式）？在 Laravel 中有必要吗？
* **定义**：在 Controller 和 Model 之间增加一层，用于封装数据访问逻辑。
* **优点**：解耦（方便切换 ORM 或数据源）、易于测试（Mock Repository）。
* **争议**：Eloquent 本身已经是 Active Record 模式（自带数据访问），简单的 CRUD 加 Repository 会显得过度设计。
* **适用**：业务逻辑极复杂、可能更换底层存储、或需要遵循 DDD（领域驱动设计）的项目。

#### 6. Laravel 的事件监听器（Listener）中的 `shouldQueue` 是做什么的？
* **作用**：标记该监听器为**异步执行**。
* **原理**：实现了 `ShouldQueue` 接口的监听器，Laravel 不会立即执行 `handle`，而是将任务序列化并推送到默认队列中，由 Worker 进程后台处理。

#### 7. 什么是 N+1 问题？在 Laravel 中怎么发现和解决？
* **场景**：遍历模型列表时，循环内访问关联属性导致产生 N 条 SQL。
* **发现**：使用 `Laravel Debugbar` 或 `DB::listen` 监控 SQL 条数。
* **解决**：使用 Eager Loading（预加载），即 `with('relation')`。
* **强制防范**：在开发环境调用 `Model::preventLazyLoading(!app()->isProduction())`，触发懒加载时直接抛出异常。


### Laravel 面试题库大全 (补充精选)

#### 1. 基础与生命周期
1.  **Laravel 的入口文件是哪个？** `public/index.php`。
2.  **`public/index.php` 主要做了哪几件事？** 加载 Composer autoload，创建 Application 实例，处理请求。
3.  **什么是 Kernel（内核）？** 处理 HTTP 请求 (`App\Http\Kernel`) 或 Console 命令 (`App\Console\Kernel`) 的核心类，负责加载中间件和引导程序。
4.  **Application 实例是什么？** 整个应用的核心容器，继承自 `Illuminate\Container\Container`。
5.  **Service Provider 的作用？** 引导和注册服务（绑定到容器、注册路由、事件等）。
6.  **`config` 目录下的文件如何访问？** `config('app.name')`。
7.  **`.env` 文件的作用？** 存储环境变量，不同环境（开发/生产）配置不同。
8.  **如何获取环境变量？** `env('APP_ENV')`。
9.  **Laravel 的目录结构中 `storage` 是做什么的？** 存放日志、编译后的模板、文件上传、缓存文件。
10. **`bootstrap/cache` 目录的作用？** 存放框架生成的优化缓存文件（如 `config.php`, `routes.php`）。

#### 2. 路由 (Routing)
11. **Laravel 支持哪些 HTTP 请求方法？** GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD。
12. **如何定义一个资源路由？** `Route::resource('photos', PhotoController::class)`。
13. **路由参数如何约束？** `where('id', '[0-9]+')`。
14. **命名路由有什么好处？** 生成 URL 时解耦，修改路径不影响代码。`route('profile')`。
15. **路由组（Route Group）有哪些属性？** middleware, prefix, domain, name, namespace。
16. **API 路由和 Web 路由的区别？** API 默认无 Session/CSRF 中间件，前缀 `/api`，限流 `api`。
17. **如何获取当前路由名称？** `Route::currentRouteName()`。
18. **路由缓存的命令？** `php artisan route:cache`。
19. **Fallback 路由（兜底路由）？** `Route::fallback(function () { ... })` 处理 404。
20. **什么是路由模型绑定？** 自动将路由参数（ID）解析为模型实例。`Route::get('users/{user}', ...)`。

#### 3. 中间件 (Middleware)
21. **中间件的作用？** 过滤 HTTP 请求（如 Auth, CSRF, TrimStrings）。
22. **如何注册全局中间件？** 在 `Kernel.php` 的 `$middleware` 数组中。
23. **如何注册路由中间件？** 在 `Kernel.php` 的 `$routeMiddleware` (或 `$middlewareAliases`) 中。
24. **中间件组（Middleware Groups）？** 如 `web` 和 `api`，包含一组中间件。
25. **前置中间件和后置中间件？** 前置：`$next($request)` 之前执行；后置：之后执行。
26. **如何创建一个中间件？** `php artisan make:middleware EnsureTokenIsValid`。
27. **中间件参数如何传递？** `middleware('role:editor')`。
28. **如何排除某些 URI 的 CSRF 保护？** 在 `VerifyCsrfToken` 中间件的 `$except` 数组配置。
29. **`TrimStrings` 中间件的作用？** 自动去除请求参数首尾空格。
30. **`ConvertEmptyStringsToNull`？** 将空字符串参数转为 `null`。

#### 4. 控制器 (Controllers)
31. **控制器的作用？** 处理路由分发的请求，返回响应。
32. **如何创建一个资源控制器？** `php artisan make:controller PhotoController --resource`。
33. **单行为控制器（Single Action Controller）？** 只有一个 `__invoke` 方法。
34. **依赖注入在控制器中如何使用？** 构造函数或方法参数中类型提示。
35. **如何验证请求数据？** `$request->validate([...])` 或 FormRequest。
36. **控制器方法可以直接返回数组吗？** 可以，Laravel 会自动转为 JSON 响应。
37. **`authorize` 方法的作用？** 在控制器中快速检查权限（基于 Policy/Gate）。
38. **API 资源控制器有什么不同？** `--api` 参数，不生成 create/edit 方法（不需要视图）。
39. **如何重定向？** `redirect('/home')` 或 `return to_route('login')`。
40. **重定向并携带 Flash Data？** `return redirect('form')->with('status', 'Success');`。

#### 5. 请求与响应 (Request & Response)
41. **如何获取请求输入？** `$request->input('name')`。
42. **`$request->all()` 和 `$request->collect()`？** 返回数组 / 返回 Collection。
43. **如何判断请求是否包含某参数？** `$request->has('name')`。
44. **如何获取上传文件？** `$request->file('photo')`。
45. **如何判断请求是 AJAX？** `$request->ajax()` 或 `$request->wantsJson()`。
46. **如何获取 Cookie？** `$request->cookie('name')`。
47. **如何返回 JSON 响应？** `response()->json([...])`。
48. **如何返回文件下载？** `response()->download($path)`。
49. **如何设置响应头？** `response($content)->header('Content-Type', $type)`。
50. **如何附加 Cookie 到响应？** `response(...)->cookie(...)`。

#### 6. 视图 (Blade Templates)
51. **Blade 模板文件的后缀？** `.blade.php`。
52. **如何在 Blade 中输出变量？** `{{ $name }}` (自动转义)。
53. **如何输出未转义的 HTML？** `{!! $html !!}`。
54. **Blade 的控制结构？** `@if`, `@foreach`, `@for`, `@while`。
55. **模板继承的指令？** `@extends`, `@section`, `@yield`。
56. **如何包含子视图？** `@include('view.name')`。
57. **组件（Components）是什么？** 类似 Vue 组件，`<x-alert type="error" />`。
58. **`@csrf` 的作用？** 生成 CSRF Token 的隐藏 Input。
59. **`@method('PUT')` 的作用？** 生成模拟 PUT 请求的隐藏 Input。
60. **`@auth` 和 `@guest`？** 判断用户是否登录 / 未登录。
61. **如何自定义 Blade 指令？** `Blade::directive`。
62. **View Composer 的作用？** 在视图渲染前绑定数据（如所有页面都需要菜单数据）。
63. **`asset()` 辅助函数？** 生成 public 目录下的资源 URL。
64. **`url()` 和 `route()` 的区别？** `url` 指定路径，`route` 指定路由名称。
65. **Blade 注释语法？** `{{-- Comment --}}` (不会输出到 HTML)。

#### 7. 数据库与 Eloquent ORM
66. **Laravel 支持哪些数据库？** MySQL, PostgreSQL, SQLite, SQL Server。
67. **如何配置数据库？** `.env` 文件或 `config/database.php`。
68. **Query Builder（查询构造器）是什么？** `DB::table('users')->get()`，流畅的 SQL 接口。
69. **Eloquent Model 的默认主键？** `id`。
70. **如何修改默认表名？** `protected $table = 'my_users';`。
71. **如何禁用时间戳（created_at/updated_at）？** `public $timestamps = false;`。
72. **Eloquent 的 `find($id)` 和 `findOrFail($id)`？** 后者找不到抛出 ModelNotFoundException (404)。
73. **Soft Deletes（软删除）是什么？** 不真删数据，只标记 `deleted_at`。需引入 `SoftDeletes` trait。
74. **如何查询被软删除的数据？** `withTrashed()`。
75. **Scopes（查询作用域）？** 本地作用域 `scopeActive($query)`，全局作用域 `GlobalScope`。
76. **Eloquent 的 `create` 方法需要注意什么？** 需要配置 `$fillable` 或 `$guarded` (批量赋值保护)。
77. **Accessors（访问器）？** `getFirstNameAttribute`，获取属性时自动处理。
78. **Mutators（修改器）？** `setFirstNameAttribute`，设置属性时自动处理。
79. **Attribute Casting（属性转换）？** `$casts = ['is_admin' => 'boolean']`。
80. **Eloquent 事件有哪些？** creating, created, updating, updated, saving, saved, deleting, deleted...

#### 8. 模型关联 (Relationships)
81. **一对一？** `hasOne` / `belongsTo`。
82. **一对多？** `hasMany` / `belongsTo`。
83. **多对多？** `belongsToMany` (需要中间表)。
84. **远层一对多（Has Many Through）？** Country -> User -> Post，Country 获取 Posts。
85. **多态关联（Polymorphic）？** Comment 可以属于 Post 或 Video。`morphTo`, `morphMany`。
86. **如何定义多对多中间表名称？** 第二个参数指定，`belongsToMany(Role::class, 'role_user')`。
87. **如何预加载关联（解决 N+1）？** `with('posts')`。
88. **延迟预加载？** `load('posts')`。
89. **关联查询？** `whereHas('posts', function($q){ ... })`。
90. **更新关联？** `$user->posts()->save($post)` 或 `$user->roles()->attach($roleId)`。
91. **`sync` 方法？** 多对多同步，删除不在数组中的，添加新的。
92. **`toggle` 方法？** 多对多切换，有则删，无则加。
93. **`withCount`？** 统计关联数量而不加载关联模型。
94. **关联模型默认外键格式？** `ModelName_id` (snake_case)。
95. **`touch` 方法？** 更新父模型的时间戳 (`protected $touches = ['post']`)。

#### 9. 数据库迁移与填充
96. **Migration（迁移）的作用？** 数据库的版本控制，团队共享结构变更。
97. **如何创建迁移？** `php artisan make:migration create_users_table`。
98. **`up` 和 `down` 方法？** `up` 执行变更，`down` 回滚变更。
99. **如何运行迁移？** `php artisan migrate`。
100. **如何回滚上一次迁移？** `php artisan migrate:rollback`。
101. **Seeder（填充器）的作用？** 填充测试数据或初始数据。
102. **Factory（工厂）的作用？** 快速生成大量伪造数据模型。
103. **Faker 库？** 用于生成假名、地址、文本等。
104. **如何调用 Seeder？** `php artisan db:seed` 或在 `DatabaseSeeder` 中调用。
105. **Schema Builder？** `Schema::create`, `Schema::table`。

#### 10. 验证 (Validation)
106. **如何在控制器验证？** `$request->validate(['title' => 'required|max:255'])`。
107. **验证失败会发生什么？** 自动重定向回上页，并带回错误信息和输入数据；AJAX 则返回 422 JSON。
108. **Form Request（表单请求）？** 独立的验证类，将验证逻辑从控制器分离。`php artisan make:request StorePostRequest`。
109. **常用验证规则？** required, email, unique, confirmed, min, max, numeric, date...
110. **如何自定义错误信息？** 在 FormRequest 的 `messages` 方法中返回数组。
111. **`bail` 规则？** 第一个规则失败后停止后续规则检查。
112. **`sometimes` 规则？** 只有当字段存在时才验证。
113. **自定义验证规则？** `php artisan make:rule Uppercase`。
114. **`unique` 规则如何排除当前 ID（更新时）？** `Rule::unique('users')->ignore($user->id)`。
115. **数组验证？** `'items.*.id' => 'required'`。

#### 11. 安全 (Security)
116. **Laravel 如何处理用户认证（Authentication）？** 内置 Auth 门面，支持 Session/Token 驱动。Starter Kits: Breeze, Jetstream。
117. **`Auth::user()`？** 获取当前登录用户。
118. **`Auth::check()`？** 检查是否登录。
119. **`Auth::attempt($credentials)`？** 尝试登录。
120. **Laravel 如何处理授权（Authorization）？** Gates (闭包) 和 Policies (策略类)。
121. **如何创建 Policy？** `php artisan make:policy PostPolicy --model=Post`。
122. **CSRF 保护原理？** `VerifyCsrfToken` 中间件验证 `_token`。
123. **XSS 保护？** Blade `{{ }}` 自动转义。
124. **SQL 注入保护？** Eloquent/Query Builder 使用 PDO 参数绑定。
125. **密码哈希？** `Hash::make()` (Bcrypt/Argon2)。
126. **加密/解密？** `Crypt::encryptString()` / `decryptString()` (AES-256-CBC)。
127. **签名路由（Signed Routes）？** 防止 URL 被篡改（如邮箱验证链接）。

#### 12. 缓存 (Cache)
128. **支持哪些缓存驱动？** file, database, redis, memcached, array, dynamodb。
129. **如何获取缓存？** `Cache::get('key', 'default')`。
130. **如何设置缓存？** `Cache::put('key', 'value', $seconds)`。
131. **`Cache::remember`？** 缓存不存在时执行闭包并缓存结果，存在则直接返回。
132. **`Cache::forget`？** 删除缓存。
133. **`Cache::tags`？** 缓存标签（File/Database 驱动不支持），支持按标签清除。
134. **原子锁（Atomic Lock）？** `Cache::lock('foo', 10)->get()`。

#### 13. 会话 (Session)
135. **支持哪些 Session 驱动？** file, cookie, database, redis, memcached...
136. **如何获取 Session 数据？** `session('key')` 或 `session()->get('key')`。
137. **`session()->put()` 和 `session()->push()`？** 设置值 / 压入数组。
138. **Flash Data（闪存数据）？** `session()->flash('status', 'Task was successful!')`，只在下一次请求有效。
139. **重新生成 Session ID？** `session()->regenerate()` (登录时防止固定会话攻击)。

#### 14. 队列 (Queues)
140. **队列的作用？** 异步处理耗时任务（发送邮件、视频转码），提升响应速度。
141. **支持驱动？** sync (同步), database, redis, beanstalkd, sqs.
142. **如何创建 Job？** `php artisan make:job ProcessPodcast`。
143. **如何分发 Job？** `ProcessPodcast::dispatch($podcast)`。
144. **延迟分发？** `dispatch($podcast)->delay(now()->addMinutes(10))`。
145. **队列连接与队列名称？** `onConnection('redis')->onQueue('high')`。
146. **如何运行队列 Worker？** `php artisan queue:work`。
147. **`queue:listen` 和 `queue:work`？** `listen` 每次请求重载代码（开发用），`work` 长驻内存（生产用，需重启生效）。
148. **最大尝试次数与超时？** `--tries=3 --timeout=90`。
149. **失败任务处理？** 记录到 `failed_jobs` 表。`queue:retry` 重试。
150. **Job Middleware？** 在 Job 处理前后执行逻辑（如限流）。

#### 15. 事件 (Events)
151. **事件系统的作用？** 解耦代码，观察者模式。
152. **Event 和 Listener？** Event 是发生的事情，Listener 是对该事情的响应。
153. **如何生成？** 在 `EventServiceProvider` 注册后运行 `php artisan event:generate`。
154. **触发事件？** `event(new OrderShipped($order))`。
155. **队列化监听器？** Listener 实现 `ShouldQueue` 接口，自动异步执行。

#### 16. 邮件与通知
156. **Mailable 类？** `php artisan make:mail OrderShipped`。
157. **发送邮件？** `Mail::to($user)->send(new OrderShipped($order))`。
158. **Markdown 邮件？** 支持构建美观的响应式邮件。
159. **Notifications（通知）系统？** 支持多种渠道（Mail, SMS, Slack, Database）发送通知。
160. **通知类？** `php artisan make:notification InvoicePaid`。
161. **发送通知？** `$user->notify(new InvoicePaid($invoice))`。

#### 17. 任务调度 (Scheduling)
162. **Scheduler 的作用？** 用 PHP 代码定义 Cron 任务，只需在服务器配置一个 Crontab。
163. **在哪里定义？** `App\Console\Kernel` 的 `schedule` 方法。
164. **定义频率？** `->daily()`, `->hourly()`, `->everyMinute()`.
165. **避免任务重叠？** `->withoutOverlapping()`。
166. **在多服务器环境运行？** `->onOneServer()` (需 Redis/Memcached 缓存驱动支持)。
167. **后台运行？** `->runInBackground()`。

#### 18. Artisan 命令行
168. **如何列出所有命令？** `php artisan list`。
169. **自定义命令？** `php artisan make:command SendEmails`。
170. **命令的 `signature`？** 定义参数和选项，如 `email:send {user} {--queue}`。
171. **`Tinker`？** `php artisan tinker`，交互式 REPL 环境，调试利器。
172. **维护模式？** `php artisan down` (显示 503), `php artisan up`。

#### 19. 测试 (Testing)
173. **Laravel 使用什么测试框架？** PHPUnit (默认) 或 Pest。
174. **`tests` 目录结构？** `Feature` (功能测试, 请求 API/页面), `Unit` (单元测试, 类/方法)。
175. **HTTP 测试？** `$response = $this->get('/'); $response->assertStatus(200);`。
176. **数据库测试？** `RefreshDatabase` trait (测试后自动回滚/迁移)。
177. **模拟（Mocking）？** `$this->mock(Service::class, ...)`，基于 Mockery。
178. **Faking？** `Mail::fake()`, `Event::fake()`, `Queue::fake()`，断言是否触发而不真执行。

#### 20. 生态系统与扩展
179. **Laravel Breeze / Jetstream？** 身份验证脚手架。
180. **Laravel Passport / Sanctum？** API 认证（Passport OAuth2 全功能，Sanctum 轻量级 SPA/Token）。
181. **Laravel Scout？** 全文搜索（Driver: Algolia, Meilisearch）。
182. **Laravel Echo？** WebSocket 客户端（配合 Pusher/Reverb）。
183. **Laravel Horizon？** Redis 队列监控 UI。
184. **Laravel Telescope？** 本地调试助手（监控请求、异常、数据库、日志等）。
185. **Laravel Nova / Filament？** 管理后台构建工具。
186. **Laravel Socialite？** 社交登录（Facebook, GitHub, Google）。
187. **Laravel Octane？** 高性能应用服务（Swoole/RoadRunner）。
188. **Laravel Sail？** Docker 开发环境。
189. **Laravel Pint？** 代码风格修复工具（基于 PHP-CS-Fixer）。
190. **Livewire？** 在 Laravel 中构建动态前端（无需写复杂 JS）。

#### 21. 其他
191. **什么是 Facade 的实时的（Real-time）？** 在类名前加 `Facades\` 命名空间自动将类转换为 Facade。
192. **如何手动记录日志？** `Log::info('Message', ['context' => '...'])`。
193. **本地化（Localization）？** `resources/lang`，使用 `__('messages.welcome')`。
194. **HTTP Client（Guzzle 封装）？** `Http::get('http://example.com')`。
195. **Collection（集合）的强大之处？** 提供类似 Lodash/Underscore 的链式操作数组的方法（map, filter, reduce, pluck...）。
196. **LazyCollection？** 懒集合，利用生成器处理大数据集，低内存占用。
197. **Storage::disk('s3')？** 文件存储抽象，轻松切换本地/云存储。
198. **Helpers（辅助函数）？** `dd()`, `dump()`, `now()`, `abort()`, `info()`...
199. **`macro` 方法？** 给类动态添加方法（Collection, Request, Response 等支持）。
200. **Laravel 的核心优势？** 开发速度快、文档完善、生态丰富、语法优雅、现代 PHP 特性支持好。


