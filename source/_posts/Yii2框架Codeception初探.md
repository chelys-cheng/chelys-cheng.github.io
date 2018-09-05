---
title: Yii2框架Codeception初探
date: 2018-09-05 14:47:02
categories: [PHP, Yii2]
tags: [PHP, Yii2]
---
## 说明
[配置初始化Yii2框架](https://chelys-cheng.github.io/2018/09/04/Yii2框架初始化前，添加api应用/)后，进行Codeception的使用。

由于笔者没有系统学习过Codeception这个框架，一些原理尚未清晰，此文仅作为一次探索的记录。

文末有一些参考资料，均是在笔者做实验时所参考使用，具有较高的质量，可供大家参考。

后续若有时间专门研究Codeception，将进行更详尽的解析。

以下所用代码模板，为[《Yii2框架初始化前，添加api应用》](https://chelys-cheng.github.io/2018/09/04/Yii2框架初始化前，添加api应用/)中初始化后的代码。
<!--more-->
**注:**
`/web-root`：项目根目录
`api`:  本项目中专门的API应用
本文为后续整理，在一些文件权限，以及账户标识上，可能会有误差，路径我会做到准确，请读者自行根据自己项目具体情况进行参考。

## 详细记录
### 进入应用目录
```bash
[root@localhost web-root]# cd api
```
### 生成api套件
执行指令，注意此时所在的路径为`/web-root/api/`
```bash
[root@localhost api]# ../vendor/bin/codecept g:suite api
Helper \api\tests\Helper\Api was created in /web-root/api/tests/_support/Helper/Api.php
Actor ApiTester was created in /web-root/api/tests/_support/ApiTester.php
Suite config api.suite.yml was created.

Next steps:
1. Edit api.suite.yml to enable modules for this suite
2. Create first test with generate:cest testName ( or test|cept) command
3. Run tests of this suite with codecept run api command
Suite api generated
```
此时`/web-root/api/tests/`文件夹下，新增四个文件：
```bash
	./_support/Helper/Api.php
	./_support/ApiTester.php
	./api/_bootstrap.php
	./api.suite.yml
```
### 编辑配置文件
**注：**此处内容仅为经过多次试验得出可用，暂时不明原理。

```yaml
# /web-root/api/tests/api.suite.yml
actor: ApiTester
modules:
    enabled:
        - REST:
            depends: Yii2
    config:
        - Yii2
```
### 生成测试文件
**注：**此处的`display`测试，对应的在`api`应用中，拥有一个`\api\controllers\DisplayController`控制器。
```
[root@localhost api]# ..\vendor\bin\codecept g:cept api display
```
此时在`/web-root/api/tests/api/`文件夹下，新增：
```
	./displayCept.php
```
先运行一次脚本
```
[root@localhost api]# ..\vendor\bin\codecept run api displayCept
```
此时会生成文件
```
	/web-root/api/tests/_support/_generated/ApiTesterActions.php
```
### 编辑测试脚本
```php
	<?php
	use api\tests\ApiTester;
	use Codeception\Util\HttpCode;
	
	$I = new ApiTester($scenario);
	$I->wantTo('Get the display');
	$I->sendGET('/displays/1');
	$I->seeResponseCodeIs(HttpCode::OK);
	$I->seeResponseIsJson();
```
### 运行测试脚本
```bash
[root@localhost api]# ..\vendor\bin\codecept run api displayCept
    Codeception PHP Testing Framework v2.4.0
    Powered by PHPUnit 6.5.7 by Sebastian Bergmann and contributors.

    Api\tests.api Tests (1) -------------------------------------------
    + displayCept: Get the display (0.19s)
    -------------------------------------------------------------------
    
    
    Time: 351 ms, Memory: 10.00MB

    OK (1 test, 3 assertions)
```
成功！

## 参考资料
[Codeception](https://codeception.com/)是一款PHP全栈测试框架，具备了包括**单元测试**、**功能测试**、**验收测试**等功能，是一款相当全面的测试框架。

[官方文档](https://codeception.com/docs)提供了相当完善的教程，具体可以搜索参照。

如果英文看起来费劲，[这里](https://www.cloudxns.net/Support/lists.html?keyword=【Codeception中文文档】)可以找到一些中文文档，可作部分参考。

同时，Codeception作为Yii2框架唯一自带整合的测试框架，官方有专题介绍[Codeception for yii2](https://codeception.com/for/yii)，里面提供了一些实例可作参考。
