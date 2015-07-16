---
layout: post
title: "使用roots快速构建一个基础的wordpress主题"
date: 2015-02-07
categories: php
---

[roots](http://roots.io/)是一个基于[HTML5 Boilerplate](http://html5boilerplate.com/) & [Bootstrap](http://getbootstrap.com/)构建WoredPress的基础主题


2.特性
--------
* 使用[Grunt](http://gruntjs.com/)来将LESS或SASS编译成CSS,检查JS错误，压缩JS,CSS,优化图片等
* [Bower](http://bower.io/) 前端包管理工具
* [HTML5 Boilerplate](http://html5boilerplate.com/)
* [Bootstrap](http://getbootstrap.com/)12

3.安装
--------
通过git或者[github](https://github.com/roots/roots)下载安装

```bash
cd wp-content/themes
sudo git clone git://github.com/roots/roots.git
```

安装nodejs的npm

```bash
brew install npm
```
安装grunt-cli及bower

```bash
npm install -g grunt-cli bower
```

根据package.json的配置安装依赖模块

```bash
npm install
```

根据bower.json的配置安装第三方模块

```bash
bower install
```

如果你是开发模式 在config.php文件设置PHP常量

```php
# roots/lib/config.php 
define('WP_ENV','development');
```

然后执行grunt任务生成js,css文件

```bash
grunt dev
```

开发完毕以后去掉config.php的常量，执行grunt

```bash
grunt build
```
进行js,css压缩简单等

这样一个简单的WordPress基础主题样式就搭建完成。