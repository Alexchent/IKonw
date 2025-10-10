- [laravel](#laravel)
  - [Repository模式](#repository模式)
      - [目的](#目的)
  - [怎么写一个facades](#怎么写一个facades)
  - [安装 telescope](#安装telescope)
  - [用户认证](#用户认证)

# laravel
- 初始化一个laravel项目
```
composer create-project laravel/laravel Laravel8 --prefer-dist "8.*"
```

## Repository模式
一个table对应一个model，一个model对应一个repositoryInterface，一个repositoryInterface对应一个repository，repository需要注册到容器内，controller依赖注入interface。


> 一个经典的laravel开源项目 [laracom](https://github.com/jsdecena/laracom)

## 安装 telescope
```
composer require laravel/telescope v3.5.1 --dev
```
选择 `版本` 可以去 [packagist.org](https://packagist.org/packages/laravel/telescope) 查找，我的 Laravel 版本是7，因此选择的 telescope 版本号是 `v3.5.1`


安装 Telescope 后，可以在 Artisan 使用 `telescope:install` 命令来配置扩展实例。安装 Telescope 后，还应运行 migrate 命令：
```
php artisan telescope:install

php artisan migrate
```

更新 Telescope 时，您应该重新配置加载 Telescope 实例：
```
php artisan telescope:publish
```

> [Telescope监控工具](https://learnku.com/docs/laravel/7.x/telescope/7518)

## 用户认证 
登录&注册
```php
public function login() {
    $credentials = $request->only('email', 'password');
    if (Auth::attempt($credentials)) {
//  切换看守器api
//  if (Auth::guard('api')->attempt($credentials)) {
        // 通过认证..
        $user = Auth::user();
        return response()->json(['success' => $user], 200);
    }
}

public function register(Request $request)
{
    $this->validate($request, [
        'name' => 'required|unique:users|max:50',
        'email' => 'required|email|unique:users|max:255',
        'password' => 'required|confirmed|min:6'
    ]);

    $user = User::where("email", $request->email)->first();
    if ($user) return response()->json(['success'=> false, 'data' => "用户已存在"]);

    $user = User::create([
        'name' => $request->name,
        'email' => $request->email,
        'password' => bcrypt($request->password),
    ]);

    Auth::login($user);
    return response()->json(['success'=> true, 'data' => $user]);
}
```


## schedule 任务调度

laravel任务调度扩展 [laravel-totem](https://github.com/codestudiohq/laravel-totem)