---
layout: post
title: "Linux Apache2 安装 Ghost"
date: 2013-10-29
categories: ghost
---

[GHOST官网](http://www.ghost.org) 服务器:ubuntu

http服务器:apache2

### 安装nodejs

#### 1.先安装依赖包

```bash
sudo apt-get install g++ curl libssl-dev apache2-utils
sudo apt-get install git-core
```

#### 2.安装nodejs

```bash

git clone git://github.com/ry/node.git
cd node
# 请别安装最新的nodejs,GHOST默认的sqllite3只支持nodejs版本小于 0.11.0
git checkout v0.10.0
./configure
make
sudo make install

```

#### 3.下载GHOST上传到服务器，我放在/var/www/blog下

```bash

cd /var/www/blog
npm install --production
npm start
```

#### 4.通过apache2反向代理绑定域名

```bash
#安装apache2 模块
sudo apt-get install libapache2-mod-proxy-html

#在/etc/apache2/apache2.conf配置里添加
LoadModule  proxy_module         /usr/lib/apache2/modules/mod_proxy.so
LoadModule  proxy_http_module    /usr/lib/apache2/modules/mod_proxy_http.so
LoadModule  headers_module       /usr/lib/apache2/modules/mod_headers.so
LoadModule  deflate_module       /usr/lib/apache2/modules/mod_deflate.so

#添加反向代理的配置
#在etc/apache2/sites-available里建个blog.conf

     ServerName blog.maozhua.pw
     ProxyPreserveHost on
     #ProxyPass 这里的内容是你在GHOST里config.js里配置的host和port
     ProxyPass / http://127.0.0.1:2368/

```

#### 重启apache2

```bash
apachectl restart
```

### 通过Forever让GHOST永久运行

```bash
npm install forever -g
cd /var/www/blog
#启动GHOST
NODE_ENV=production forever start index.js
```

### 停止GHOST

```bash
forever stop index.js[
```