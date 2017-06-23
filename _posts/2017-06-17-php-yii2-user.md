---
layout: post
title:  "基于Yii2的用户模块Yii2-user详解"
categories: php
tags:  php Yii2
---


* content
{:toc}

	在我们开发网站的过程，用户的登录，注册，密码忘记和修改等想通的操作基本都会涉及，在Yii框架中，Yii2-user提供了这方法，下面详细介绍Yii2-user的安装和使用。

<!--excerpt-->

## Yii2-user安装使用教程
#### 1. 准备工作
 yii2-user使用composer安装，国内的网络很难安装上composer，具体查看 [Composer下载](https://getcomposer.org/download/) 和 [Composer中国镜像安装方法](https://pkg.phpcomposer.com/#how-to-use-packagist-mirror)

#### 2. 下载安装Yii2-user
命令行下执行： `composer require dektrium/yii2-user`

#### 3. 配置Yii2-user
##### 基础版
在在main.php配置文件里面加入：

```php

'modules' => [
    'user' => [
        'class' => 'dektrium\user\Module',
    ],
],
```

##### 高级版

1.修改公共配置main.php文件@common/config/main.php:

```php

'modules' => [
    'user' => [
        'class' => 'dektrium\user\Module',
        // you will configure your module inside this file
        // or if need different configuration for frontend and backend you may
        // configure in needed configs
    ],
],

```

2.修改frontend的main.php文件@frontend/config/main.php

```php

'modules' => [
    'user' => [
        // following line will restrict access to admin controller from frontend application
        'as frontend' => 'dektrium\user\filters\FrontendFilter',
    ],
],

```

3.修改backend的mian.php文件@backend/config/main.php

```php

'modules' => [
    'user' => [
        // following line will restrict access to profile, recovery, registration and settings controllers from backend
        'as backend' => 'dektrium\user\filters\BackendFilter',
    ],
],

```

4.不要忘记注释前台和后台的user组件，如下：

```php

'components' => [
        ...
        /*'user' => [
            'identityClass' => 'common\models\User',
            'enableAutoLogin' => true,
        ],*/
        ...
    ],

```

5.前后端使用不同的sessions

当我们想前后端的session分开时，可以使用如下配置

修改前端main.php

```php

'components' => [
    'user' => [
        'identityCookie' => [
            'name'     => '_frontendIdentity',
            'path'     => '/',
            'httpOnly' => true,
        ],
    ],
    'session' => [
        'name' => 'FRONTENDSESSID',
        'cookieParams' => [
            'httpOnly' => true,
            'path'     => '/',
        ],
    ],
],

```

修改后端main.php

```php

'components' => [
    'user' => [
        'identityCookie' => [
            'name'     => '_backendIdentity',
            'path'     => '/admin',
            'httpOnly' => true,
        ],
    ],
    'session' => [
        'name' => 'BACKENDSESSID',
        'cookieParams' => [
            'httpOnly' => true,
            'path'     => '/admin',
        ],
    ],
],

```
6.配置邮箱
在main.php里面添加配置：

```php

'components' => [
        'mailer' => [
            'class' => 'yii\swiftmailer\Mailer',
            'viewPath' => '@app/mailer',
            'useFileTransport' => false,
            'transport' => [
                'class' => 'Swift_SmtpTransport',
                'host' => 'smtp.163.com',
                'username' => '1212121@163.com',
                'password' => '1212121',
                'port' => '25',
                'encryption' => 'tls',
            ],
        ],
],

```


#### 4.更新数据库
在确认好数据库可以连接的情况下，执行如下命令：

`$ php yii migrate/up --migrationPath=@vendor/dektrium/yii2-user/migrations`

#### 5.开始使用Yii2-user
以上配置好后，我们就可以使用Yii2-user的丰富功能了，yii2-user主要提供如下功能：

- /user/registration/register 用户注册
- /user/registration/resend 重新发送邮件
- /user/registration/confirm 注册用户确认
- /user/security/login 用户登录
- /user/security/logout 用户登出
- /user/recovery/request 找回密码请求
- /user/recovery/reset 重置密码
- /user/settings/profile 用户信息编辑
- /user/settings/account 账户设置（email，用户名，密码）

这些只是常用的几个功能，yii2-user远不止这些。具体参考[Yii2-user文档](https://github.com/liyoung1992/yii2-user/blob/master/docs/README.md)






