---
layout: post
title:  "基于Yii2的后台权限管理（RABC）Yii2-admin使用详解"
categories: php
tags:  php Yii2
---


* content
{:toc}

上一篇文章讲解了Yii2-user的使用，主要是对用户登录注册等功能，下面要说的就是对用户权限的管理（RABC）。

yii2的访问权限默认是由自带的rbac组件在管理，需要自己编写相应的规则去实现权限管理，无图形界面。
yii2-admin是将rbac的管理可视化，只需要点几下鼠标就能设置好简单的规则。

<!--excerpt-->

## 下载安装

同样的，Yii2-admin也是用composer安装，国内安装过程出现问题的话，请参考上一篇文章；

#### linux下安装

```php

php composer.phar require mdmsoft/yii2-admin "~2.0"
php composer.phar update

```

#### windos下安装

```php

composer require mdmsoft/yii2-admin "~2.0"
composer update

```

## 配置

修改配置文件main.php：（因为我们的权限控制大多作用于前台，所以我们修改前台的配置文件）

```php

return [
    'modules' => [
        'admin' => [
            'class' => 'mdm\admin\Module',
             'layout' => 'left-menu',//yii2-admin的导航菜单
        ]
        ...
    ],
    ...
    'components' => [
        ...
        'authManager' => [
            'class' => 'yii\rbac\DbManager', // 使用数据库管理配置文件
        ]
    ],
    'as access' => [
        'class' => 'mdm\admin\components\AccessControl',
        'allowActions' => [
            'site/*',//允许访问的节点，可自行添加
            'admin/*',//允许所有人访问admin节点及其子节点
        ]
    ],
];

```

## 使用Yii2-admin

经过上面几个配置，就能实现RABC的基本功能，是不是so easy！！！Yii RBAC主要提供如下功能：

```php

http://localhost/path/to/index.php?r=admin
http://localhost/path/to/index.php?r=admin/route
http://localhost/path/to/index.php?r=admin/permission
http://localhost/path/to/index.php?r=admin/menu
http://localhost/path/to/index.php?r=admin/role
http://localhost/path/to/index.php?r=admin/assignment
http://localhost/path/to/index.php?r=admin/user

```

## 效果截图

![](https://mdmunir.files.wordpress.com/2016/03/image01.png?w=529&h=304)
![](https://mdmunir.files.wordpress.com/2016/03/image02.png?w=529&h=293)
![](https://mdmunir.files.wordpress.com/2016/03/image03.png?w=529&h=315)
![](https://mdmunir.files.wordpress.com/2016/03/image04.png?w=529&h=292)
![](https://mdmunir.files.wordpress.com/2016/03/image05.png?w=529&h=292)
![](https://mdmunir.files.wordpress.com/2016/03/image06.png?w=529&h=315)




