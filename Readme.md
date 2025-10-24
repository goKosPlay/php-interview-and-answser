### laravel 应用哪些设计模式
* 单例模式 (Singleton Pattern)
* 应用场景：Laravel 的服务容器（Service Container）使用单例模式来管理类的实例。例如，当你通过容器解析一个类并标记为单例时，同一个实例会在整个应用生命周期中复用。
```php
$app->singleton('example', function () {
    return new ExampleClass();
});
```
* 工厂模式 (Factory Pattern)
* 应用场景：Laravel 的模型工厂（Model Factories）用于生成测试数据或模拟数据。Eloquent 模型通过工厂模式创建对象，方便测试和数据填充。
```php
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

$user = User::factory()->create();
```
* 观察者模式 (Observer Pattern)
* 应用场景：Eloquent 模型的事件系统使用观察者模式。当模型发生创建、更新、删除等事件时，可以触发相应的观察者逻辑。
```php
class UserObserver {
    public function created(User $user) {
        // 用户创建后的逻辑
    }
}
```
```php
User::observe(UserObserver::class);
```
* 装饰者模式 (Decorator Pattern)
* 应用场景：Laravel 的中间件（Middleware）机制使用装饰者模式。中间件可以在请求处理前后动态添加逻辑，包装核心逻辑。
```php
public function handle($request, Closure $next) {
    // 前置逻辑
    $response = $next($request);
    // 后置逻辑
    return $response;
}
```
* 策略模式 (Strategy Pattern)
* 应用场景：Laravel 的认证系统（Authentication）允许通过配置切换不同的认证策略（如基于 Session 或 Token 的认证）。
```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'api' => [
        'driver' => 'token',
        'provider' => 'users',
    ],
],
```
* 门面模式 (Facade Pattern)
* 应用场景：Laravel 的门面（Facade）提供了一个静态接口来访问服务容器中的对象，简化了复杂子系统的调用。
```php
use Illuminate\Support\Facades\Cache;

Cache::put('key', 'value', 60);
```
* 建造者模式 (Builder Pattern)
* 应用场景：Eloquent 查询构建器（Query Builder）使用建造者模式，允许链式调用来构建复杂的数据库查询。
```php
$users = DB::table('users')
            ->where('age', '>', 18)
            ->orderBy('name')
            ->get();
```

* 依赖注入模式 (Dependency Injection Pattern)
* 应用场景：Laravel 的服务容器通过依赖注入（Dependency Injection）自动解析类依赖，降低耦合性。
```php
class UserController extends Controller {

    public function __construct(protected UserService $userService) {

    }
}
```

* 代理模式 (Proxy Pattern)
* 应用场景：Laravel 的 Eloquent 模型通过代理模式实现了延迟加载（Lazy Loading）关联关系。例如，访问模型的关联属性时，会动态加载相关数据。
```php
$user = User::find(1);
$posts = $user->posts; // 延迟加载 posts 关联
```

* 命令模式 (Command Pattern)
* 应用场景：Laravel 的 Artisan 命令行工具使用命令模式。每个 Artisan 命令封装了一个具体的操作，方便扩展和执行。
```php
php artisan make:command SendEmails
```
```php
class SendEmails extends Command {
    protected $signature = 'email:send';
    public function handle() {
        // 命令逻辑
    }
}
```

* 适配器模式 (Adapter Pattern)
* 应用场景：Laravel 的缓存系统通过适配器模式支持多种缓存驱动（如 Redis、Memcached、File）。适配器将不同驱动的接口统一为一致的 API。
```php
Cache::store('redis')->put('key', 'value', 60);
```

* 模板方法模式 (Template Method Pattern)
* 应用场景：Laravel 的控制器基类（Controller）定义了一些通用方法，子类可以继承并重写特定行为。
```php
class UserController extends Controller {
    // 继承并自定义逻辑
}
```