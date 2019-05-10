---
title: laravel 利用模型工厂和数据填充生成假数据
date: 2019-05-10 23:16:59
tags: [laravel,模型工厂,数据填充]
toc: true
---
### 需求描述
在实际的项目开发过程中，我们经常会用到一些假数据来对数据库进行填充以方便调试程序，原始的做法是手工一个个在数据库中创建，或者从队友的机器那导出数据填充到开发机器中。Laravel 提供了一套更加现代化、非常简单易用的数据填充方案。接下来让我们使用 Laravel 提供的数据填充来批量生成假用户。

假数据的生成分为两个阶段：

1. 对要生成假数据的模型指定字段进行赋值 - 『模型工厂』；
2. 批量生成假数据模型 -『数据填充』；

### 模型工厂

Laravel 默认为我们集成了 Faker 扩展包，使用该扩展包可以让我们很方便的生成一些假数据。

示例如下：
```php
// 使用 factory 来创建一个 Faker\Generator 实例
$faker = Faker\Factory::create();

// 生成用户名
$faker->name; // "Janie Roob"

// 生成安全邮箱
$faker->safeEmail; // "claire.wuckert@example.net"

// 生成随机日期
$faker->date // "2011-02-10"

// 生成随机时间
$faker->time // "13:03:55"

```

我们可以借助 Faker 和 Eloquent 模型工厂来为指定模型的每个字段设置随机值。

本项目中生成的模型工厂如下：

database/factories/UserFactory.php

```php
<?php

use App\Models\User;
use Illuminate\Support\Str;
use Faker\Generator as Faker;

$factory->define(User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
        'remember_token' => Str::random(10),
    ];
});
```

define 定义了一个指定数据模型（如此例子 User）的模型工厂。define 方法接收两个参数，第一个参数为指定的 Eloquent 模型类，第二个参数为一个闭包函数，该闭包函数接收一个 Faker PHP 函数库的实例，让我们可以在函数内部使用 Faker 方法来生成假数据并为模型的指定字段赋值。

让我们对生成的模型工厂文件进行修改，修改后如下。

database/factories/UserFactory.php

```php
<?php

use App\Models\User;
use Illuminate\Support\Str;
use Faker\Generator as Faker;

$factory->define(User::class, function (Faker $faker) {
    $date_time = $faker->date . ' ' . $faker->time;
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => '$2y$10$TKh8H1.PfQx37YgCzwiKb.KjNyWgaHb9cbcoQgdIVFlYg7B77UdFm', // secret
        'remember_token' => Str::random(10),
        'created_at' => $date_time,
        'updated_at' => $date_time,
    ];
});
```

我们使用生成的假日期对用户的创建时间和更新时间进行赋值。

### 数据填充

在 Laravel 中我们使用 Seeder 类来给数据库填充测试数据。所有的 Seeder 类文件都放在 database/seeds 目录下，文件名需要按照『驼峰式』来命名，且严格遵守大小写规范。Laravel 默认为我们定义了一个 DatabaseSeeder 类，我们可以在该类中使用 call 方法来运行其它的 Seeder 类，以此控制数据填充的顺序。我们可以使用下面命令来生成一个 UsersTableSeeder 文件，用于填充用户相关的假数据。

```php
$ php artisan make:seeder sersTableSeeder
```

在我们定义好了用户模型工厂之后，便可以在生成的用户数据填充文件中使用 factory 这个辅助函数来生成一个使用假数据的用户对象。

现在让我们使用该方法来创建 50 个假用户。

database/seeds/UsersTableSeeder.php

```php
<?php

use Illuminate\Database\Seeder;
use App\Models\User;

class UsersTableSeeder extends Seeder
{
    public function run()
    {
        $users = factory(User::class)->times(50)->make();
        User::insert($users->makeVisible(['password', 'remember_token'])->toArray());

        $user = User::find(1);
        $user->name = 'Summer';
        $user->email = 'summer@example.com';
        $user->save();
    }
}
```

times 和 make 方法是由 FactoryBuilder 类 提供的 API。times 接受一个参数用于指定要创建的模型数量，make 方法调用后将为模型创建一个 集合。makeVisible 方法临时显示 User 模型里指定的隐藏属性 $hidden，接着我们使用了 insert 方法来将生成假用户列表数据批量插入到数据库中。最后我们还对第一位用户的信息进行了更新，方便后面我们使用此账号登录。

接着我们还需要在 DatabaseSeeder 中调用 call 方法来指定我们要运行假数据填充的文件。

database/seeds/DatabaseSeeder.php

```php
<?php

use Illuminate\Database\Seeder;
use Illuminate\Database\Eloquent\Model;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        Model::unguard();

        $this->call(UsersTableSeeder::class);

        Model::reguard();
    }
}
```

完成上面操作之后，我们便可以开始为用户生成批量假数据了，在运行生成假数据的命令之前，我们需要使用 migrate:refresh 命令来重置数据库，之后再使用 db:seed 执行数据填充。

```php
$ php artisan migrate:refresh
$ php artisan db:seed
```

如果我们要单独指定执行 UserTableSeeder 数据库填充文件，则可以这么做：

```php
$ php artisan migrate:refresh
$ php artisan db:seed --class=UsersTableSeeder
```

你也可以使用下面一条命令来同时完成数据库的重置和填充操作：

```php
$ php artisan migrate:refresh --seed
```
