# laravel facade
门面的作用就是让我们可以像使用静态方法一样使用类的非静态方法。


## 怎么用
### 第一步，准备好这么一个类
```php
<?php namespace App\Services;

use Illuminate\Support\Facades\Log;

class MyLog
{
    public function error($message, array $info)
    {
        Log::error($message, $info);
    }

    public function info($message, array $info)
    {
        Log::info($message, $info);
    }
}
```

### 第二步，创建一个对应的facade
```php
<?php namespace App\Facades;

use Illuminate\Support\Facades\Facade;

/**
 * Class MyLog
 * @package App\Facades
 *
 * @method static info($message, array $info)
 * @method static error($message, array $info)
 * @see \App\Services\MyLog
 */
class MyLog extends Facade {

    /**
     * Get the registered name of the component.
     *
     * @return string
     */
    protected static function getFacadeAccessor() { return 'mylog'; }

}
```

### 第三步，注册到容器中
在`AppServiceProvider.php`的`register`方法中添加
```
$this->app->singleton('mylog', function ($app) {
            return new MyLog();
        });
```
### 第四步，给这个facade起一个别名
在`config/app.php`的`aliases`数组中添加
```
 'MyLog' => 'App\Facades\MyLog',
```

## 现在就可以使用这个新的门面了
```
Mylog::info(...)
```