---
layout: post
title: "Zend Framework 2 KpGrab 模块"
date: 2015-02-09
categories: php
---

KpGrab是一个基于Zend Framweork 2模块，主要功能是抓取整站静态页面

## 2.安装

[github下载](https://github.com/h112367/KpGrab.git) 或者 `composer require "kittencup/kp-grab": "dev-master"`

```php
#application.config.php
return [
    'modules' => [
        // ...
        'KpGrab',
    ],
];

```

## 3.使用

```php
php public/index.php grab site <url> [--save-dir=] [--save-name=]

```

*   <url>要抓取的网站地址,比如http://www.kittencup.com/index.html.</url>
*   --save-dir=DIR, 抓取的内容保存的目录,不填写默认根据配置提供,目录要可写.
*   --save-name=NAME, 抓取的内容保存的文件夹名，不填写随机生成.

例子

```php
php public/index.php grab site http://admindesigns.com/framework/dashboard.html  --save-dir=/Users/Kittencup/WebServer/zf2/data --save-name=admindesigns

```

## 4.配置

具体的配置内容在KpGrab/config/module.config.php内，使用kp_grab键值

*   http_adapter => (String) 使用http连接方式
*   http_adapter_options => (Array) http连接方式的选项
*   console_error_message_color => (Int) 控制台报错信息的颜色,使用Zend\Console\ColorInterface内常量
*   show_message => (bool) 显示抓取具体信息
*   max_reconnection_count => (int) 连接失败重新连接次数
*   xdebug_max_nesting_level => (int) xdebug下函数递归太多层可能会报错，尽量提高该配置,
*   default_save_dir => (String) 默认的保存文件夹
*   grab_allow_page_suffix => (Array) 允许抓取的页面后缀
*   grab_allow_static_suffix => (Array) 允许抓取的静态文件后缀
*   output_error => (bool) 是否将错误信息输出到文件中
*   output_error_filename => (string) 保存错误信息的文件名

例子

```php
'kp_grab' => [
        'http_adapter' => 'Zend\Http\Client\Adapter\Curl',
        'http_adapter_options' => [
            'curloptions' => [
                CURLOPT_ENCODING => 'gzip',
                CURLOPT_FOLLOWLOCATION => false,
                CURLOPT_TIMEOUT => 20,
                CURLOPT_NOSIGNAL => 1
            ]
        ],
        'console_error_message_color' => \Zend\Console\ColorInterface::RED,
        'show_message' => true,
        'max_reconnection_count' => 5,
        'xdebug_max_nesting_level' => 600,
        'default_save_dir' => realpath(__DIR__ . '/../data'),
        'grab_allow_page_suffix' => ['html'],
        'grab_allow_static_suffix' => ['png', 'jpeg', 'jpg', 'gif', 'css', 'js', 'woff', 'ttf', 'eot', 'svg'],
        'output_error' => true,
        'output_error_filename' => 'error.md'
    ]

```