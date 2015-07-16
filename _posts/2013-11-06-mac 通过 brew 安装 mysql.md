---
layout: post
title: "mac 通过 brew 安装 mysql"
date: 2013-11-06
categories: php
---

通过 brew 安装 mysql

```bash
brew install mysql
```

查看mysql版本


```bash
mysql -v
```

在 /usr/local/etc/ 下创建或修改 my.cnf

```bash
[client]
port = 3306
socket = /tmp/mysql.sock
default-character-set = utf8

[mysqld]
collation-server = utf8_unicode_ci
character-set-server = utf8
init-connect ='SET NAMES utf8'
max_allowed_packet = 64M
bind-address = 127.0.0.1
port = 3306
socket = /tmp/mysql.sock
innodb_file_per_table=1

[mysqld_safe]
timezone = '+0:00'
</code>
</pre>
```

执行

```bash
mysql_install_db --verbose --user=`MAC名` --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp
```

修改php.ini

```bash
pdo_mysql.default_socket= /tmp/mysql.sock
```

启动

```bash
mysql.server start
```

