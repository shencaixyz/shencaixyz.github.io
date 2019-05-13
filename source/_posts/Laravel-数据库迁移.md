---
title: Laravel 数据库迁移
date: 2019-05-13 16:11:21
tags: [laravel,数据库迁移]
toc: true
---
### 目的

我们将通过给数据库的用户表新增两个字段来熟悉 laravel 的数据库迁移

### 新增字段

查看用户表相关的迁移文件：

database/migrations/2014_10_12_000000_create_users_table.php

```php
.
.
.
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }
.
.
.
```
以上是 users 表里的所有字段，我们将增加『头像』和『个人简介』字段：

接下来我们使用 Laravel 自带的命令来新建迁移文件。由于我们进行的是字段添加操作，因此在命名迁移文件时需要加上前缀，遵照如 add_column_to_table 这样的命名规范，并在生成迁移文件的命令中设置 --table 选项，用于指定对应的数据库表。最终的生成命令如下：

```php
$ php artisan make:migration add_avatar_and_introduction_to_users_table --table=users
```

接下来我们来设计字段的类型和属性。

我们会将头像的图片以文件形式放置于服务器上，然后将路径子串存储于数据库中，所以我们需要用到 string 类型，用户注册并未提供头像上传功能，因此我们还需要将字段设置为 nullable，意为允许空子串。

个人简介字段存储的是短文本内容，此处我们也选择使用 string 类型，同样的我们也允许用户设置空的简介。现在让我们来为新增的迁移文件加上这两个字段：

database/migrations/[timestamp]_add_avatar_and_introduction_to_users_table.php

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class AddAvatarAndIntroductionToUsersTable extends Migration
{
    /**
     * 执行迁移
     *
     * @return void
     */
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('avatar')->nullable();
            $table->string('introduction')->nullable();
        });
    }

    /**
     * 回滚迁移
     *
     * @return void
     */
    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('avatar');
            $table->dropColumn('introduction');
        });
    }
}
```

### 执行迁移

接着我们还需要运行迁移，将字段加入到用户表中：

```php
$ php artisan migrate
```

打开数据库查看工具，即可看到我们新添加的两个字段：