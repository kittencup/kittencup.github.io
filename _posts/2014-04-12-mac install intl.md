---
layout: post
title: "mac install intl"
date: 2014-04-12
categories: mac
---

获取icu版本

```bash
brew search icu # returns 'icu4c'
```

安装icu

```bash
brew install icu4c
```

记录icu安装的目录地址

```bash
$ /usr/local/Cellar/icu4c/xxxxx
```

安装autoconf

```bash
brew install autoconf
```

pecl安装intl

```bash
$ sudo pecl update-channels
$ sudo pecl install intl
```
安装过程中会提示输入icu目录地址，输入前面记录的icu目录地址

配置php.ini

```bash
extension=intl.so
```

检查是否安装成功

```bash
$ php -m | grep intl # should return 'intl'
```