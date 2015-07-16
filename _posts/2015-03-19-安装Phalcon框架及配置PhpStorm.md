---
layout: post
title: "安装Phalcon框架及配置PhpStorm"
date: 2015-03-19
categories: php
featured_image: /images/cover.jpg
---


1.概述
-------
新公司用的是phalcon框架，一个基于c语言的框架，虽然我的职位是前端开发，但是也阻止不了我对PHP框架的研究

2.brew 安装 phalcon
-------

搜索phalcon版本，根据php版本选择合适的phalcon

```bash
brew search phalcon
```

```bash
brew install php55-phalcon
```


3.设置php.ini
--------
在最后行添加

```bash
extension=/usr/local/Cellar/php55-phalcon/1.3.4/phalcon.so
```

4.安装Phalcon 开发工具
--------

```bash
wget -q --no-check-certificate -O phalcon-tools.zip http://github.com/phalcon/phalcon-devtools/zipball/master
unzip -q phalcon-tools.zip
mv phalcon-phalcon-devtools-* phalcon-tools
```

使用pwd查看当前目录

```bash
pwd
```

/Users/kittencup

```bash
vi ~/.profile
```

添加

```bash
export PATH=$PATH:/Users/kittencup/phalcon-tools
export PTOOLSPATH=/Users/kittencup/phalcon-tools
```

连接

```bash
ln -s ~/phalcon-tools/phalcon.sh ~/phalcon-tools/phalcon
chmod +x ~/phalcon-tools/phalcon
```

重开下命令行工具，执行phalcon命令确定安装成功

```bash
phalcon
```

5.使用Phalcon DevTools创建项目
--------

```bash
cd /Users/kittencup/WebServer 

phalcon create-project phalcon-test
```

6.设置apache,hosts并重启apache
-------

```bash
#/private/etc/apache2/extra/httpd-vhosts.conf 最后添加
..........
<VirtualHost *:80>
    DocumentRoot "/Users/kittencup/WebServer/phalcon-test/public"
    ServerName phalcon-test.kittencup.com
</VirtualHost>
```

```bash
#etc/hosts 最后添加一行

127.0.0.1 phalcon-test.kittencup.com
```

```bash
#重启apache
sudo apachectl restart
```


```bash
#给phalcon-test权限
chmod -R 777 phalcon-test
```

7.设置phpstrom
-------
Phalcon是基于c的框架，所以你在开发时，phpStorm等开发ide不会对其代码进行提示，通过配置可以解决这个问题

phpstorm=>Preferences

搜索include => Languages & Framework => php

include path => 添加 => /Users/kittencup/phalcon-tools/ide/1.3.4 (phalcon-tools安装目录下版本根据自己下载的phalcon版本)













