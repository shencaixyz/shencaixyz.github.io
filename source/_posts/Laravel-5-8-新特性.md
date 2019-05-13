---
title: Laravel 5.8 新特性
date: 2019-05-13 11:04:02
tags: [laravel]
toc: true
---
Laravel 5.8 现在面向所有人正式发布了。这个版本包括了几个新特性以及最新的错误修复和对框架核心的改进。

一些新特性如下：

### PHP dotenv

Laravel 5.8 集成了 PHP 的 dotenv 3.0 ，下面是 PHP dotenv 3.0 的新特性：

- 在阅读和更改环境变量部分具有更大的灵活性
- 对多行变量的一流支持
- 不再格式化值，你获取到的值就是它们现在的样子
- 支持按顺序多行查找 dotenv 文件，以前只支持一行
- 更强的变量名称验证，避免静态变量或模糊变量造成的错误

### 支持 Carbon 2.0

Laravel 5.8 上可以使用 Carbon 1.0 或 Carbon 2.0， 包括可以使用  CarbonImmutable， 甚至可以默认使用  CarbonImmutable 。本地化 Carbon 2.0 做了很大改变，2.0 版本相比较 1.0 版本提供了更友好的国际化支持。了解更多资讯。 Carbon 类在 Laravel 5.8 上的升级.

### Cache TTL 的改变

可能产生中到高影响的重大改变是 来自 Laravel 5.8 的 Cache TTL 的改变 。现在将整型传到缓存的方法由分改为秒。如果你想要在迁移过程中将整型改为 Carbon 或 \DateInterval 实例，请查看我的文章。

### 已弃用的字符串和数组辅助函数

不用太担心这个修改，在使用上虽然变更为类的方式，但是具体的使用方法与之前一致。并且 Laravel 有计划将 Helper 作为可选扩展包发布，你仍然可以在项目中使用它们。

### 自动解析策略
从 Laravel 5.8 开始，只要解析策略和模型位于传统位置，您就不需要在 AuthServiceProvider 类中注册它们。

如果您更喜欢将非常规路径用于模型和解析策略，则可以注册回调以注册策略或继续手动配置它们：

```php
Gate::guessPolicyNamesUsing(function ($class) {
    // Do stuff
    return $policyClass;
});
```

### 更多新功能
- Nexmo 和 Slack 信息通知通道
- Blade 模板文件路径
- Markdown 文件目录的改变
