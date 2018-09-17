---
title: PHP自动加载类实现
date: 2018-09-17 22:48:53
tags: [PHP]
categories: [PHP]
---
有关类的自动加载，在[官方文档](http://php.net/manual/zh/index.php)中有详细说明:

> 类的自动加载
> ---
> 在编写面向对象（OOP） 程序时，很多开发者为每个类新建一个 PHP 文件。 这会带来一个烦恼：每个脚本的开头，都需要包含（include）一个长长的列表（每个类都有个文件）。
>
> 在 PHP 5 中，已经不再需要这样了。 
> 
> spl_autoload_register() 函数可以注册任意数量的自动加载器，当使用尚未被定义的类（class）和接口（interface）时自动去加载。通过注册自动加载器，脚本引擎在 PHP 出错失败前有了最后一个机会加载所需的类。

<!--more-->

详情点击[这里](http://php.net/manual/zh/language.oop5.autoload.php)，其中，涉及到方法`spl_autoload_register()`，以下是官方文档说明：

> spl_autoload_register
> ---
> (PHP 5 >= 5.1.0, PHP 7)
> 
> spl_autoload_register — 注册给定的函数作为 __autoload 的实现
> 
> 说明
> ---
> bool spl_autoload_register ([ callable $autoload_function [, bool $throw = true [, bool $prepend = false ]]] )
> 
> 将函数注册到SPL __autoload函数队列中。如果该队列中的函数尚未激活，则激活它们。
>
> 如果在你的程序中已经实现了_\_autoload()函数，它必须显式注册到_\_autoload()队列中。因为 spl_autoload_register()函数会将Zend Engine中的_\_autoload()函数取代为spl_autoload()或spl_autoload_call()。
>
> 如果需要多条 autoload 函数，spl_autoload_register() 满足了此类需求。 它实际上创建了 autoload 函数的队列，按定义时的顺序逐个执行。相比之下， __autoload() 只可以定义一次。
> 
>参数
>---
>autoload_function
>
>欲注册的自动装载函数。如果没有提供任何参数，则自动注册 autoload 的默认实现函数spl_autoload()。
>
>throw
>
>此参数设置了 autoload_function 无法成功注册时， spl_autoload_register()是否抛出异常。
>
>prepend
>
>如果是 true，spl_autoload_register() 会添加函数到队列之首，而不是队列尾部。
>
>返回值
>---
>成功时返回 TRUE， 或者在失败时返回 FALSE。

通过官方说明，可以知道，`spl_autoload_register()`方法可以将自定义方法注册为自动加载方法，当脚本程序中，出现未定义的类的时候，便会调用注册过的自定义方法，如以下程序中:
```php
<?php
spl_autoload_register(function(){
    echo 'Foo is an unknown method, and I don\'t want to load it.';
    exit;
});`

$foo = new Foo();
```
运行结果为:
```bash
Foo is an unknown method, and I don't want load it.
```
即当脚本程序中出现未知的类的时候，PHP会调用在`spl_autoload_register()`中自定义的方法，来做一些处理，当然，这个自定义程序不可能如上面的程序这么定义，一般是做一些包含文件的预处理，或者直接包含文件。
```php
# ./BaseYii.php
class BaseYii
{
    // 类文件路径映射表
    public static $classMap = [];

    // 自动加载方法
    public static function autoload($className)
    {
        if(isset(static::$classMap[$className])){
            $classFile = static::$classMap[$className];
        }else{
            throw new \Exception("Unknown Class.");
        }

        include $classFile;
    }
}

spl_autoload_register(['BaseYii', 'autoload']);
Yii::$classMap = require __DIR__ . '/classes.php';

# ./classes.php
<?php
// 表示路径映射表
return [
    'yii\base\BaseObject' => __DIR__ . '/base/BaseObject.php'
];

# ./base/BaseObject.php
<?php
// 简单的一个类
namespace yii\base;

class BaseObject
{
    public function __construct()
    {
        echo 'New BaseObject instance.';
    }   
}

# ./test.php
<?php
// 测试脚本文件
require __DIR__ . '/BaseYii.php';

use yii\base\BaseObject;

$baseObject = new BaseObject();
```
在`test.php`同级目录下运行`php ./test.php`，将得到结果:
```
New BaseObject instance.
```

如此便实现了一个简单的自动加载功能。