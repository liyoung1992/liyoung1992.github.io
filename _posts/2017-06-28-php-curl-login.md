---
layout: post
title:  "使用curl模拟登陆并抓取数据"
categories: php
tags:  php curl
---


* content
{:toc}

在我们做网页爬虫分析数据时，经常要用到模拟登陆，具体的实现逻辑很简单，主要流程如下：

<!--excerpt-->

## 分析网页的一些必要信息

1. 登陆页面的地址
2. 验证码的地址（[简单验证码的识别](https://liyoung1992.github.io/2017/06/30/2017-06-30-php-identification-captcha/)）
3. 提交登陆表达的字段名称和提交方式（get或者post）
4. 表单提交地址
5. 登陆成功，了解所要分析的网页地址


## 获取cookie并存储

```php

  $login_url = 'http://www.xxxxx';   //登录页面地址

  $cookie_file = dirname(__FILE__)."/pic.cookie";    //cookie文件存放位置（自定义）

  $ch = curl_init();
  curl_setopt($ch, CURLOPT_URL, $login_url);
  curl_setopt($ch, CURLOPT_HEADER, 0);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER,1);
  curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);
  curl_setopt($ch, CURLOPT_MAXREDIRS, 20);
  curl_setopt($ch, CURLOPT_COOKIEJAR, $cookie_file);
  curl_exec($ch);
  curl_close($ch);

```


## 获取验证码地址，并保存识别

如果网站登陆界面有验证码，就需要识别验证码才能登陆，下一篇文章将对验证码的识别做分析；



## 模拟提交登录表单

```php

  $ post_url = 'http://www.xxxx';     //登录表单提交地址
  $post = "username=$account&password=$password&seccodeverify=$verifyCode";

                              //表单提交的数据（根据表单字段名和用户输入决定）
  $ch = curl_init();
  curl_setopt($ch, CURLOPT_URL, $ post_url);
  curl_setopt($ch, CURLOPT_HEADER, false);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER,1);
  curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);
  curl_setopt($ch, CURLOPT_MAXREDIRS, 20);
  curl_setopt($ch, CURLOPT_POSTFIELDS, $post);         //提交方式为post
  curl_setopt($ch, CURLOPT_COOKIEFILE, $cookie_file);
  curl_exec($ch);
  curl_close($ch);

```

## 抓取数据

```php

  $data_url = "http://www.xxxx";     //数据所在地址

  $ch = curl_init();
  curl_setopt($ch, CURLOPT_URL, $data_url);
  curl_setopt($ch, CURLOPT_HEADER, false);
  curl_setopt($ch, CURLOPT_HEADER, 0);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER,0);
  curl_setopt($ch, CURLOPT_COOKIEFILE, $cookie_file);
  $data = curl_exec($ch);
  curl_close($ch);

```

## 对https的特殊处理

为了跳过https验证，在curl中加入如下参数：

```php

  curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
  curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false);

```




