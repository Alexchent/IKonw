# laravel 

## 生命周期

### 第一步
1. 入口文件在 `public/index.php`
2. index文件首先加载`composer`生成的自动类加载器 `autoload.php`，然后从 `bootstrap/app.php` 获得应用程序的实例。这个app实例包含了 http/console内核

### http/console内核
接下来根据请求的类型，传入的请求将被发给给http内核或console内核。这两个内核充当所有请求流经的中心位置。

关于http内核，会进行配置异常处理，配置日志，检查应用程序环境，http内核定义了一个一个http中间件列表，所有的请求都会经过这些中间件的处理

### 服务提供者
http内核最重要的引导操作之一是为应用程序加载服务提供者。laravel将遍历服务提供者列表，并实例化他们。

### 路由
应用程序中最重要的服务提供者之一是 `App\Providers\RouteServiceProvider`。此服务提供者加载应用程序的 `routes`目录中包含的路由文件


> [laravel 请求周期](https://learnku.com/docs/laravel/10.x/lifecycle/14841#863a85)