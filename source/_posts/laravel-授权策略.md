---
title: laravel 授权策略
date: 2019-05-09 23:27:50
tags: [laravel,授权策略]
toc: true
---
### 创建授权策略
我们可以使用以下命令来生成一个名为 UserPolicy 的授权策略类文件，用于管理用户模型的授权。
```
$ php artisan make:policy UserPolicy
```

所有生成的授权策略文件都会被放置在 app/Policies 文件夹下。

让我们为默认生成的用户授权策略添加 update 方法，用于用户更新时的权限验证。
```
app/Policies/UserPolicy.php

<?php

namespace App\Policies;

use Illuminate\Auth\Access\HandlesAuthorization;
use App\Models\User;

class UserPolicy
{
    use HandlesAuthorization;

    public function update(User $currentUser, User $user)
    {
        return $currentUser->id === $user->id;
    }
}
```
update 方法接收两个参数，第一个参数默认为当前登录用户实例，第二个参数则为要进行授权的用户实例。当两个 id 相同时，则代表两个用户是相同用户，用户通过授权，可以接着进行下一个操作。如果 id 不相同的话，将抛出 403 异常信息来拒绝访问。

使用授权策略需要注意以下两点：

我们并不需要检查 $currentUser 是不是 NULL。未登录用户，框架会自动为其 所有权限 返回 false；
调用时，默认情况下，我们 不需要 传递当前登录用户至该方法内，因为框架会自动加载当前登录用户（接着看下去，后面有例子）。
### 注册授权策略
Laravel 提供两种注册授权策略的方式，第一种是手动指定，第二种是 Laravel 5.8 新增功能 —— 自动授权注册。
#### 第一种方法
我们需要在 AuthServiceProvider 类中对授权策略进行设置。AuthServiceProvider 包含了一个 policies 属性，该属性用于将各种模型对应到管理它们的授权策略上。我们需要为用户模型 User 指定授权策略 UserPolicy。

app/Providers/AuthServiceProvider.php
```
<?php

namespace App\Providers;
.
.
.
class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\Model' => 'App\Policies\ModelPolicy',
        \App\Models\User::class  => \App\Policies\UserPolicy::class,
    ];
    .
    .
    .
}
```

#### 第二种方法

自动授权默认会假设 Model 模型文件直接存放在 app 目录下，鉴于我们已将模型存放目录修改为 app/Models，接下来还需自定义自动授权注册的规则，修改 boot() 方法：

app/Providers/AuthServiceProvider.php
```
<?php

namespace App\Providers;
.
.
.
class AuthServiceProvider extends ServiceProvider
{
    .
    .
    .
    public function boot()
    {
        $this->registerPolicies();
        // 修改策略自动发现的逻辑
        Gate::guessPolicyNamesUsing(function ($modelClass) {
            // 动态返回模型对应的策略名称，如：// 'App\Model\User' => 'App\Policies\UserPolicy',
            return 'App\Policies\\'.class_basename($modelClass).'Policy';
        });
    }
}
```
### 使用授权策略
授权策略定义完成之后，我们便可以通过在用户控制器中使用 authorize 方法来验证用户授权策略。默认的 App\Http\Controllers\Controller 类包含了 Laravel 的 AuthorizesRequests trait。此 trait 提供了 authorize 方法，它可以被用于快速授权一个指定的行为，当无权限运行该行为时会抛出 HttpException。authorize 方法接收两个参数，第一个为授权策略的名称，第二个为进行授权验证的数据。

我们需要为 edit 和 update 方法加上这行：
```
$this->authorize('update', $user);
```
这里 update 是指授权类里的 update 授权方法，$user 对应传参 update 授权方法的第二个参数。正如上面定义 update 授权方法时候提起的，调用时，默认情况下，我们 不需要 传递第一个参数，也就是当前登录用户至该方法内，因为框架会自动加载当前登录用户。

书写的位置如下：

app/Http/Controllers/UsersController.php
```
<?php

namespace App\Http\Controllers;
.
.
.
class UsersController extends Controller
{
    .
    .
    .
    public function edit(User $user)
    {
        $this->authorize('update', $user);
        return view('users.edit', compact('user'));
    }

    public function update(User $user, Request $request)
    {
        $this->authorize('update', $user);
        $this->validate($request, [
            'name' => 'required|max:50',
            'password' => 'nullable|confirmed|min:6'
        ]);

        $data = [];
        $data['name'] = $request->name;
        if ($request->password) {
            $data['password'] = bcrypt($request->password);
        }
        $user->update($data);

        session()->flash('success', '个人资料更新成功！');

        return redirect()->route('users.show', $user->id);
    }
}
```